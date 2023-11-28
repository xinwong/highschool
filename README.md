# 能让你“隐身”的对抗补丁

## 什么是对抗补丁
随着深度学习的迅速发展，深度神经网络已然成为当前计算机领域研究和应用最广泛的技术之一，成功应用于计算机视觉、自然语言处理、语音识别等多个领域。尽管深度神经网络已广泛应用于各种现实场景，但研究表明，其容易受到对抗攻击，导致模型产生错误的输出，进而影响到实际应用系统的可靠性和安全性。目前大多数对抗攻击的研究主要集中于数字场景下的对抗样本，一般通过向干净图片中添加细微的、人眼无法察觉的对抗噪声来构造对抗样本，例如FGSM、BIM、PGD和C&W等方法。

![图片](adv.png)

现实场景中的人工智能系统往往通过摄像头和传感器等设备获取输入，而无法直接接受数字输入。比如，在人脸识别场景中，用户的面部图像是通过摄像头采集的，而不是可以直接上传一张图片。在这些场景下，对输入样本进行细粒度逐维度修改的攻击方式就变得不切实际了，需要特殊的物理世界攻击(physical-world attack)方法来增强它们在真实环境中的对抗性。

## 对抗补丁攻击方法介绍
与基于L1、L2或L∞范数的全局扰动和难以察觉的数字场景下的对抗攻击不同，物理对抗攻击允许攻击者扭曲一个有界区域，牺牲了其隐匿性。物理场景下的对抗攻击最早被提出用于攻击图像分类，随后许多工作被提出来用来攻击目标检测、语义分割和图像检索等任务。相较于数字场景下的对抗攻击，物理场景下的对抗攻击具有其独特的挑战：（1）物理对抗样本需要考虑摄像头和传感器等设备成像的影响；（2）物理对抗样本需要考虑空间相对位置变化和环境变化的影响；（3）物理对抗样本还需考虑其隐匿性的问题。

（1）对抗补丁攻击（Adversarial Patch Attack）
Brown等人提出了一种对抗补丁（Adversarial Patch）方法，对图片的局部区域进行较大幅度的对抗扰动，生成具有强对抗性的补丁。由于扰动幅度很大，对抗补丁可以被打印出来，在物理场景中攻击深度学习模型，例如使物体检测模型忽略特定的物体并预测错误的类别。不同于传统梯度优化的算法，Brown等人用生成的补丁替换图像中的一部分来实现攻击。给定一张图 $x \in \mathbb{R}^{w \times h \times c}$、 一个补丁 $r$、补丁位置  $l$和补丁变换 $t$（如旋转和缩放等），定义补丁应用操作 $A(r, x, l, t)$，先将变换 $t$应用于补丁 $r$，之后将变换后的补丁 $r$作用于图片 $x$的位置 $l$上。Brown等人使用了变换期望（Expectation Over Transformation，EOT）框架来获得训练后的补丁 $\widehat{r}$，这种框架通过模拟和求期望来拟合现实世界中的各种变换。

<video src="Adversarial Patch.mp4" controls="controls" width="500" height="300"></video>

（2）对抗T恤（Adversarial T-shirt）

上述物理攻击都是针对不易形变的刚性物体，如道路标识牌等。在一般情况下这些物体是静止的，贴在其表面的对抗图案不会产生形变，导致这些算法在容易形变的非刚性物体上效果有限。针对此问题，Xu等人提出对抗T恤（adversarial T-shirts，AdvTShirt），使得对抗样本在T恤这类会根据人类姿态和动作而随时发生形变的非刚性物体上也能发挥作用。当攻击者穿上印有对抗图案的T恤后能躲过物体检测模型，使其无法检测到攻击者的存在。图\ref{fig_6.1.3_10}展示了该算法生成的对抗T恤在数字和物理世界中攻击YOLOv2模型时的有效性。

为了适应物理环境变化，Xu等人将中的期望转换算法（即EOT）推广到对抗性T恤的设计。如前文所述，该方法将可能发生在现实世界中的多种变化，如缩放、旋转、模糊、光线、噪声等，通过模拟和求期望来拟合现实，且在对象为刚性物体时有不错的效果。但该方法无法模拟T恤在人体运动时产生的褶皱，而这种褶皱会使对抗样本失去作用。于是Xu等人开发了一种基于\ma{薄板样条插值}（thin plate spline，TPS）的变换算法来模拟由人体姿态变化引起的T恤变形，TPS算法已被广泛用于图像对齐和形状匹配中的非刚性变换模型。

图\ref{fig_6.1.3_9}展示了该算法的总体流程。
具体地，以视频中的两帧为例，有一个锚点图像 $x_0$和一个目标图像 $x_i$，对于 $x_0$中给定的人物边界框 $M_{p, 0} \in\{0,1\}^{d}$和T恤边界框 $M_{c, 0} \in\{0,1\}^{d}$，使用从 $x_0$到 $x_i$的透视变换来获得图像 $x_i$中的人物边界框 $M_{p, i}$和T恤边界框 $M_{c, i}$。于是，尚未考虑物理变换的关于 $x_i$的扰动图像 $x'_{i}$可表示为：

$$
x_i^\prime  = (1 - M_{p,i}) \cdot x_{i} + M_{p,i} \cdot x_{i} - M_{c,i} \cdot x_{i} + M_{c,i} \cdot x_{\delta}
$$

其中， $(1 - M_{p,i}) \cdot x_{i}$表示人物边框外的背景区域， $M_{p,i} \cdot x_{i}$是人物边界区域， $M_{c,i} \cdot x_{i}$表示删除T恤边界框内的像素值， $M_{c,i} \cdot x_{\delta}$是新引入的加性扰动。该公式可简化为对抗样本的常规表述：  $\left(1-M_{c, i}\right) \circ x_{i}+M_{c, i} \circ \mathbf{\delta}$。

接下来，Xu等人考虑三种主要类型的物理转换：1）对扰动 $\mathbf{\delta}$进行TPS转换 $t_{\text{TPS}} \in T_{\text{TPS}}$，以此来模拟布料变形的影响；2）物理颜色转换 $t_{\text{color}}$，这种转换可将数字颜色转换为在物理世界中可被打印出来的颜色；以及 3) 应用于人物边框内区域的常规物理变换 $t \in T$。这里 $ T_{\text{TPS}}$表示非刚性变换集合， $t_{\text{color}}$由一个可将数字空间色谱映射到对应的印刷品的回归模型给出， $T$ 表示常用的物理变换集合，包括缩放、平移、旋转、亮度、模糊和对比度等。综合考虑以上不同物理转换后的算法公式为：

$$
    x_i^\prime = t_{\text{env}}\left(\text{A}+t\left(\text{B}-\text{C}+t_{\text{color}}\left(M_{c, i} \circ t_{\text{TPS}}(\mathbf{\delta}+\mu v)\right)\right)\right)
$$

其中， $t \in T$, $t_{\text{TPS}} \in T_{\text{TPS}} $， $v \sim N(0,1)$， $t_{\text{env}}$代表对环境亮度条件建模的亮度变换， $\mu v$是允许像素值变化的加性高斯噪声，它可使最终的目标函数更平滑，更有利于优化过程中的梯度计算， $\mu$是给定的平滑参数。

最终，用于欺骗单个检测器的期望转换公式为：

$$
\min_{\delta} \frac{1}{M} \sum_{i=1}^{M} E_{t,t_{TPS},v} \left[ \mathcal{L}_{adv} (x'_i) \right] + \lambda g(\delta),
$$

其中， $\mathcal{L}_{adv}$是导致错误检测的对抗损失， $g$ 是增强扰动平滑度的\ma{变分范数}（total variation norm）， $\lambda>0$ 是正则化参数。
实现显示，通过上述算法生成的对抗T恤在数字和物理世界中对YOLOv2体检测模型的攻击成功率分别可达到74%和 57%，相比之前方法有巨大提升。



（3）Dpatch

实践 - 自己生成一个物理对抗补丁
本实践算法基于"DPatch: An Adversarial Patch Attack on Object Detectors."。

相关实验代码已发布在 GitHub,

https://github.com/veralauee/DPatch
