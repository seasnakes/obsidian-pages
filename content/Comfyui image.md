-  蒙版 图片 latent 在comfy 中 都是 `torch.Tensor `格式, 形状为 `[B,H,W,C]`  batchsize , 高 宽、频道RGB， C=3 
- mask 、  [H,W] or [B,C,H,W]
- 图像节点文档 https://docs.comfy.org/essentials/custom_node_images_and_masks
- 遮罩是一个形状为[B,H,W]的张量。 在许多情况下，遮罩具有二进制值（0 或 1），用于指示哪些像素应进行特定操作。 在某些情况下，介于 0 和 1 之间的值表示遮罩的范围（例如，改变透明度、调整滤镜或合成图层）。
- loadImage 节点通过 rgb 上加a 来代表蒙版。
```python
i = Image.open(image_path)
i = ImageOps.exif_transpose(i)
if i.mode == 'I':
    i = i.point(lambda i: i * (1 / 255))
image = i.convert("RGB")
image = np.array(image).astype(np.float32) / 255.0
image = torch.from_numpy(image)[None,]

```
3. `if i.mode == 'I': i = i.point(lambda i: i * (1 / 255))`: 这里检查如果图像的模式(mode)为'I'，那么对图像的每个像素点进行一个运算，将其值乘以1/255。'I'模式代表32位整型灰度图像，灰度值在0-65535之间。这行代码是将灰度值转换到0-1之间。

5. `image = np.array(image).astype(np.float32) / 255.0`: 这一步将图像数据转换为numpy数组，强制数据类型为浮点型，然后将像素值归一化到0-1之间。归一化的目的是为了将像素值的尺度统一，便于进行计算。

6. `image = torch.from_numpy(image)[None,]`: 这一步是将numpy数组转为PyTorch的Tensor，None表示增加一个新的维度，通常是用于增加批次大小的维度。在训练神经网络的时候，我们通常使用批次进行训练，因此需要添加这个维度。
```python
for (batch_number, image) in enumerate(images):
    i = 255. * image.cpu().numpy()
    img = Image.fromarray(np.clip(i, 0, 255).astype(np.uint8))
    filepath = # some path that takes the batch number into account
    img.save(filepath)

```
这个代码段在图像处理任务中，主要用于将一批处理后的图像保存为文件。每一步的解释如下：

1. `for (batch_number, image) in enumerate(images):`：这是一个for循环，其中`enumerate(images)`函数会返回一个迭代器，产生由每个元素索引值和元素组成的元组。这里`images`应该是一个包含多个图像的列表或类似的集合。在每次循环中，`batch_number`将是图像的索引（从0开始），而`image`将是对应的图像。

2. `i = 255. * image.cpu().numpy()`: `image.cpu()`方法将图像数据从GPU内存移到CPU内存（如果它在GPU内存中）。`numpy()`方法将PyTorch张量转化为NumPy数组。乘以255是因为之前图像数据可能被归一化到0-1之间，乘以255将数据恢复到0-255的范围内，这是图像像素的标准范围。

3. `img = Image.fromarray(np.clip(i, 0, 255).astype(np.uint8))`: np.clip()函数将数组中的元素限制在0和255之间，即小于0的数会变为0，**大于255的数会变为255。astype()将数组的数据类型转换为无符号8位整数类型，**这是一种常用于图像的数据类型。`Image.fromarray()`将NumPy数组转换为PIL图片。

## 蒙版mask

当用于内绘制或分割等任务时，MASK 的值最终会被四舍五入为最接近的整数，因此它们是二进制的--0 表示忽略区域，1 表示目标区域。 不过，在将 MASK 传递给这些节点之前，这不会发生。 这种灵活性使您可以像在数码摄影中使用透明度层那样使用 MASK：
# tensor 
pytorch 多维 向量 矩阵
rank 
vector 1
matrix 2

`torch.Tensor` represents a tensor, which is the mathematical generalization of a vector or matrix to any number of dimensions. A tensor’s _rank_ is the number of dimensions it has (so a vector has _rank_ 1, a matrix _rank_ 2); its _shape_ describes the size of each dimension.

So an RGB image (of height H and width W) might be thought of as three arrays (one for each color channel), each measuring H x W, which could be represented as a tensor with _shape_ `[H,W,3]`. In Comfy images almost always come in a batch (even if the batch only contains a single image). `torch` always places the batch dimension first, so Comfy images have _shape_ `[B,H,W,3]`, generally written as `[B,H,W,C]` where C stands for Channels.

如果一个张量有一个大小为 1 的维度（称为折叠维度），那么它就等同于去掉了该维度的同一个张量（有 1 个图像的批次只是一个图像）。 移除这种折叠维度被称为挤压，而插入一个维度则被称为取消挤压。

### note
- .shape -> torch.Size
- [:,None]
- reshape((1,-1))
- torch.Tensor((1,2))
- tuple


# 总结 custom codes
- comfy 数据结构
	- tensor datatypes， 图像和蒙版都是tensor 格式
		- IMAGE
			- 形状 B, H, W, C - C = 3 -> RGB
			-  可以用 .shape  来查看tensor 的 size 返回的是一个tuple 元组
			- 然而有限 pytorch操作期望的是 通道优先的 [B,C,H,W]格式
		- MASK
			- 在IMAGE 基础上还有可能少两维 只剩 H, W
- Images, Masks 
	-  如果节点只返回一个输出 `return (image,)`
	- Images
		- 如果保存，需要 `PIL`库 
	
	- Mask 
		- 形状 [B,H,W] , 值 0 或 1 划定操作像素范围- 指定 或 不指定、某个操作的幅度 透明度、图层叠加 
		- `LoadImage` 节点使用 alpha通道来创建 蒙版 ，alpha 通道值会被归一化为[0,1] torch.float32 浮点格式 然后反转
		- 有些格式图片不包含 alpha通道。此时会创建一个默认大小的蒙版
		- 蒙版形状
			- 在 numpy、PIL 和许多其他库中，单通道图像（如 MASK）通常表示为形状为 [H,W] 的二维数组。 这意味着 C（通道）维度是隐含的，因此与 IMAGE 类型不同，成批的 MASK 只有三个维度： [B、H、W]。 在使用 MASK 时，通常需要通过取消挤压来匹配形状，以生成 C=1 的 [B,H,W,C] 形状。要取消挤压 C 维，就应该取消挤压（-1），要取消挤压 B 维，就应该取消挤压（0）。 如果您的节点接收一个 MASK 作为输入，您最好总是检查 len(mask.shape) 。

