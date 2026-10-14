- input: 
	- image -> tensor
	- tensor ->  numpy 
	- numpy  ->  outpainting
	- outpainting -> numpy
	- numpy -> tensor
	- tensor -> image

1. 读取comfyui 上个节点的图片 加载 tensor 
2. 对tensor 进行处理
3. 转换为 numpy 数组 进 opencv里处理
4. 处理完转换回 tensor 格式

https://github.com/comfyanonymous/ComfyUI/discussions/2635

***

* * *

*****

- - -

----------
*斜体文本*
_斜体文本_
**粗体文本**
__粗体文本__
***粗斜体文本***
___粗斜体文本___