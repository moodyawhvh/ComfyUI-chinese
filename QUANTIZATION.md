> 🌐 本文档由 [Comfy-Org/ComfyUI](https://github.com/Comfy-Org/ComfyUI) 翻译,英文原版见原项目。

# Comfy 量化指南


## 量化是如何工作的?

量化旨在把高精度数值 x_f 以最小的精度损失映射到低精度格式。这些更小的格式可以降低模型的内存占用,并借助专用硬件提升吞吐量。

单纯用四舍五入方法把数值从 FP16 转到 FP8 时,可能遇到两个问题:
- FP16 的动态范围 (-65,504, 65,504) 远超 E4M3 (-448, 448) 或 E5M2 (-57,344, 57,344) 等 FP8 格式,可能导致数值被截断
- 原始数值集中在一个很小的范围(如 -1,1)内,使许多 FP8 位"闲置"

通过引入缩放因子,我们希望把这些数值映射进量化 dtype 的取值范围,充分利用整个值域。最简单也最常见的做法之一是按张量(per-tensor)绝对值最大值缩放。

```
absmax = max(abs(tensor))
scale = amax / max_dynamic_range_low_precision

# Quantization
tensor_q = (tensor / scale).to(low_precision_dtype)

# De-Quantization
tensor_dq = tensor_q.to(fp16) * scale

tensor_dq ~ tensor
```

由于需要额外的信息(缩放因子)才能"解读"量化后的数值,我们将其称为派生数据类型。


## Comfy 中的量化

```
QuantizedTensor (torch.Tensor subclass)
  ↓ __torch_dispatch__
Two-Level Registry (generic + layout handlers)
  ↓
MixedPrecisionOps + Metadata Detection
```

### 表示方式

为了表达这些派生数据类型,ComfyUI 使用 torch.Tensor 的子类,即 `comfy/quant_ops.py` 中的 `QuantizedTensor` 类来实现。

`Layout` 类定义了某种特定量化格式的行为方式:
- 所需参数
- 量化方法
- 反量化方法

```python
from comfy.quant_ops import QuantizedLayout

class MyLayout(QuantizedLayout):
    @classmethod
    def quantize(cls, tensor, **kwargs):
        # Convert to quantized format
        qdata = ...
        params = {'scale': ..., 'orig_dtype': tensor.dtype}
        return qdata, params
    
    @staticmethod
    def dequantize(qdata, scale, orig_dtype, **kwargs):
        return qdata.to(orig_dtype) * scale
```

要用这些 QuantizedTensor 执行运算,我们通过两级注册表来定义受支持的操作。
第一级是**通用注册表**,处理所有量化格式通用的操作(如 `.to()`、`.clone()`、`.reshape()`)。

第二级注册表与布局(layout)相关,允许实现像 nn.Linear 这样的快速路径。
```python
from comfy.quant_ops import register_layout_op

@register_layout_op(torch.ops.aten.linear.default, MyLayout)
def my_linear(func, args, kwargs):
    # Extract tensors, call optimized kernel
    ...
```
当 `torch.nn.functional.linear()` 以 QuantizedTensor 作为参数被调用时,`__torch_dispatch__` 会自动路由到已注册的实现。
对于任何未支持的操作,QuantizedTensor 会回退为调用 `dequantize`,并改用高精度实现来执行。


### 混合精度

`MixedPrecisionOps` 类(`comfy/ops.py` 第 542-648 行)支持按层做量化决策,允许模型中不同的层使用不同的精度。当模型配置包含 `layer_quant_config` 字典(指明哪些层需要量化以及如何量化)时,该机制被激活。

**架构:**

```python
class MixedPrecisionOps(disable_weight_init):
    _layer_quant_config = {}  # Maps layer names to quantization configs
    _compute_dtype = torch.bfloat16  # Default compute / dequantize precision
```

**关键机制:**

自定义的 `Linear._load_from_state_dict()` 方法在模型加载时逐层检查:
- 如果层名**不在** `_layer_quant_config` 中:以 `_compute_dtype` 作为常规张量加载权重
- 如果层名**在** `_layer_quant_config` 中:
  - 以指定的布局(如 `TensorCoreFP8Layout`)把权重加载为 `QuantizedTensor`
  - 同时加载关联的量化参数(scale、block_size 等)

**为什么需要它:**

并非所有层对量化的耐受度都相同。像最终投影(final projection)这类敏感操作可以保持较高精度,而计算密集的矩阵乘法则被量化。这样能在保持质量的同时拿到大部分性能收益。

当 `model_config.layer_quant_config` 存在时,`pick_operations()` 会选用该系统,使其成为优先级最高的运算模式。


## Checkpoint 格式

量化 checkpoint 以标准 safetensors 文件存储,包含量化后的权重张量及其关联的缩放参数,外加一个描述量化方案的 `_quantization_metadata` JSON 条目。

量化 checkpoint 会包含与原 checkpoint 相同的层,但:
- 权重以量化值存储,有时使用不同的存储 dtype。例如用 uint8 容器存 fp8。
- 按照具体方案,每个量化权重旁边会额外存储若干缩放参数。
- 我们会在最终 safetensor 的元数据中存放一个 metadata.json,其中 `_quantization_metadata` 描述了哪些层被量化以及使用了什么布局。

### 缩放参数详情
我们定义了 4 种缩放参数,应能覆盖近期绝大多数量化方案:
- **weight_scale**:权重的量化缩放器
- **weight_scale_2**:双重缩放场景下的全局缩放器
- **pre_quant_scale**:用于平滑显著权重(salient weights)的缩放器
- **input_scale**:激活值的量化缩放器

| 格式 | 存储 dtype | weight_scale | weight_scale_2 | pre_quant_scale | input_scale |
|--------|---------------|--------------|----------------|-----------------|-------------|
| float8_e4m3fn | float32 | float32 (scalar) | - | - | float32 (scalar) |

已定义的格式可在 `comfy/quant_ops.py`(QUANT_ALGOS)中找到。

### 量化元数据

随 checkpoint 一起存储的元数据包含:
- **format_version**:定义标准版本的字符串
- **layers**:把层名映射到其量化格式的字典。格式字符串对应 `QUANT_ALGOS` 中的定义。

示例:
```json
{
  "_quantization_metadata": {
    "format_version": "1.0",
    "layers": {
      "model.layers.0.mlp.up_proj": {"format": "float8_e4m3fn"},
      "model.layers.0.mlp.down_proj": {"format": "float8_e4m3fn"},
      "model.layers.1.mlp.up_proj": {"format": "float8_e4m3fn"}
    }
  }
}
```


## 创建量化 Checkpoint

要创建兼容的 checkpoint,可以使用任何量化工具,只要其输出符合上述 checkpoint 格式,并使用 `QUANT_ALGOS` 中定义的布局即可。

### 权重量化

权重量化很简单——直接用前文所述的绝对值最大值法从权重张量计算缩放因子。每一层的权重独立量化,并与对应的 `weight_scale` 参数一起存储。

### 校准(用于激活量化)

激活量化(例如用于 FP8 Tensor Core 运算)需要 `input_scale` 参数,它无法仅凭静态权重确定。由于激活值取决于实际输入,我们采用**训练后校准(PTQ)**:

1. **收集统计信息**:在 N 个有代表性的样本上运行推理
2. **追踪激活值**:记录每个被量化层的输入绝对值最大值(`amax`)
3. **计算缩放**:根据收集到的统计信息推导 `input_scale`
4. **存入 checkpoint**:将 `input_scale` 参数与权重一并保存

校准数据集应当能代表你的目标使用场景。对扩散模型而言,这通常意味着一组多样化的提示词和生成参数。
