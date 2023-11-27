# 能让你“隐身”的对抗补丁

## 对抗补丁攻击现状
随着深度学习的迅速发展，深度神经网络已然成为当前计算机领域研究和应用最广泛的技术之一，成功应用于计算机视觉、自然语言处理、语音识别等多个领域。尽管深度神经网络已广泛应用于各种现实场景，但研究表明，其容易受到对抗样本的攻击，导致模型产生错误的输出，进而影响到实际应用系统的可靠性和安全性。目前大多数对抗攻击的研究主要集中于数字场景下的对抗样本，例如FGSM、BIM、PGD和C&W等方法。而在物理场景下的人工智能系统往往通过摄像头和传感器等设备获取输入，无法直接接受数字输入，因此在数字图像空间对输入样本进行细粒度逐维度修改的攻击方式就变得不切实际了。现实场景中的人工智能系统往往通过摄像头和传感器等设备获取输入，而无法直接接受数字输入。比如，在人脸识别场景中，用户的面部图像是通过摄像头采集的，而不是可以直接上传一张图片。在这些场景下，对输入样本进行细粒度逐维度修改的攻击方式就变得不切实际了，需要特殊的物理世界攻击(physical-world attack)方法来增强它们在真实环境中的对抗性。

## 方法
与基于L1、L2或L∞范数的全局扰动和难以察觉的数字场景下的对抗攻击不同，物理对抗攻击允许攻击者扭曲一个有界区域，牺牲了其隐匿性。物理场景下的对抗攻击最早被提出用于攻击图像分类，随后许多工作被提出来用来攻击目标检测、语义分割和图像检索等任务。相较于数字场景下的对抗攻击，物理场景下的对抗攻击具有其独特的挑战：（1）物理对抗样本需要考虑摄像头和传感器等设备成像的影响；（2）物理对抗样本需要考虑空间相对位置变化和环境变化的影响；（3）物理对抗样本还需考虑其隐匿性的问题。

（1）ap
Brown等人提出了一种对抗补丁（Adversarial Patch）方法，对图片的局部区域进行较大幅度的对抗扰动，生成具有强对抗性的补丁。由于扰动幅度很大，对抗补丁可以被打印出来，在物理场景中攻击深度学习模型，例如使物体检测模型忽略特定的物体并预测错误的类别。不同于传统梯度优化的算法，Brown等人用生成的补丁替换图像中的一部分来实现攻击。给定一张图片$x \in \mathbb{R}^{w \times h \times c}$、 一个补丁$r$、补丁位置$l$和补丁变换$t$（如旋转和缩放等），定义补丁应用操作$A(r, x, l, t)$，先将变换$t$应用于补丁$r$，之后将变换后的补丁$r$作用于图片$x$的位置$l$上。Brown等人使用了变换期望（Expectation Over Transformation，EOT）框架\cite{athalye2018synthesizing}来获得训练后的补丁$\widehat{r}$，这种框架通过模拟和求期望来拟合现实世界中的各种变换。

<video src="Adversarial Patch.mp4" controls="controls" width="500" height="300"></video>

（2）物理世界中的对抗样本

Kurakin等人是最早开始研究物理攻击的学者。他们将对抗图像打印出来，然后通过手机拍摄后再次输入分类模型中，发现很大一部分对抗样本在重新拍摄后依然能够使模型犯错，也就是这些样本在物理环境中也依然具有对抗性。如下图所示，他们先使用传统的攻击算法(可以统称为“数字攻击”)生成一些对抗图片。之后，他们将原始图片和对抗图片打印出来，并重新拍摄成照片(使用谷歌手机 Nexus 5X)。最后，将这些照片中所包含的图像分割出来，输入模型进行分类测试。 实验结果表明，一部分对抗样本在经过拍照后依然可以成功攻击。进一步 实验发现，改变照片亮度和对比度对对抗样本影响不大，但是模糊、噪点和 JPEG 编码会在很大程度上破坏对抗样本的有效性。
![图片](Kurakin.jpg "物理世界攻击")

（3）鲁棒的物理扰动RP2

Eykholt等人提出鲁棒物理扰动(robust physical perturbations，RP2 )攻击算法，该算法可以产生一个可见但并不显眼的、只作用于目标物体而非环境的扰动，并且这个扰动对不同距离和角度的摄像头具有较高的鲁棒性。



 
实践 - 自己生成一个物理对抗补丁
本实践算法基于"DPatch: An Adversarial Patch Attack on Object Detectors."。

相关实验代码已发布在 GitHub,

https://github.com/veralauee/DPatch
