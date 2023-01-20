## Lumos复现

## 待思考或解决的问题
1. 数据读取部分
- [x] 弄明白hdr格式的envmap和exr格式的relit image到底怎么用，是直接转换为LDR使用还是?
  - hdr和exr存储方式和jpg、png的区别是，jpg和png存的是uint，而hdr和exr存的是float。当pixel的值位于0-1时可以采取乘以255用uint8的方式存储，但当pixel的值大于1时，无法再用uint8编码，而可以直接用hdr或者exr存取float类型的pixel value。在这个project里面，直接读取hdr格式的envmap和exr格式的portrait image，然后将它们直接以float类型输入网络train，不需要对它们做scale或者clip。
  - 如果只是为了将hdr在display上show出来，可以将读取进来的float hdr image首先进行gamma=1/2.2的compression，然后再clip到0-1并scale到0-255，存成png或者jpg来show。这是因为pixel的值控制电压进而控制屏幕亮度，若不进行gamma compression，pixel对应的值无法线性地表示亮度（pixel value扩大两倍就使得亮度变成两倍）。如果不进行gamma compression直接存成image，会导致暗的区域非常暗，亮的区域非常亮。clip是因为，display只能显示0-255，若有大于1的部分，屏幕也显示不出来。
- [ ] 研究一下怎么生成prefiltered envmap更加高效，更加接近好的效果？
- [ ] 搞明白如何从prefiltered envmap得到diffuse and specular light map。
  1. hdr envmap（image格式）怎么去围绕人，即horizontal方向上怎么去包住人。
  2. hdr envmap的radius对ray direction的方向有什么影响？当spherical envmap的radius越大时，其total area也越大，一个pixel对应的area也越大，那么当normal方向向外延伸时，在不同radius的sphere上query到的pixel可能不是同一个。
  3. diffuse light map是在prefiltered map上直接query normal方向得到的，那么specular呢，如果也是query normal方向则不符合physical model。

2. network部分
- [ ] 用paper中的blur pool替换当时为了简便使用的maxpool。
- [ ] paper中使用的Discriminator似乎不是patchGAN，而当时为了简便直接用的patchGAN，因此需要替换。
- [ ] 几个network的activation function需要思考一下到底怎么用什么，Relu或Leacky Relu或Sigmoid。

3. loss部分
- [ ] 加上GAN loss，查看效果是否有提升。
- [ ] 加上VGG loss，查看效果是否有提升。这里的VGG loss并不是直接将image输入VGG然后使用其后面某一层的结果作为feature，而是取第1-5个activation function的输出算residual，然后算一个weight map去赋予不同的pixel不同的loss贡献权重，具体参考*LookinGood: Enhancing Performance Capture with Real-time Neural Re-Rendering*。
- [ ] 加上specular loss，查看效果是否有提升。

## 新的知识扩充
1. 用index去query（采样）image pixel时，如果coordinate是float类型，不要直接取整，而是利用interpolation去sample，否则可能会产生噪声，导致not smooth。（这适用于所有的sample）
2. inverse sampling
   1. 作用：从uniform分布中生成满足某个pdf分布的sample。
   2. 方法：利用uniform分布随机生成一个0-1的随机数$\xi$，$CDF(x)=\xi$，因此$x=CDF^{-1}(\xi)$，得到的x就是sample点。
   3. 推导
   ![inverse](./images/inverse_sample.png)
3. 从hemisphere上的pdf $p(\omega)$推导出$p(\theta, \phi)$进而利用$\theta$、$\phi$进行半球采样：
![transform](./images/hemisphere_sampling.jpg)
4. importance sampling
   1. 作用：对积分进行Monte Carlo采样计算时，如果单纯进行uniform采样，那么在采样数量较少时，效果会比较差。原因是，对于一个积分来说，在不同的sample point，其值大小有不同，值大的那些point对积分的贡献大，为了使得积分更准确，它们应该被更多地采样到（想象一下f(x)=正态分布pdf，如果均匀采样3个点，如果3个点都在值非常小的地方，那么蒙特卡罗计算后的积分近似值非常小，离正确的积分值误差非常大。但如果三个点都围绕在钟形的顶点周围，则用它们估算的积分值相对会更接近于正确值，误差较小）。因此采用importance sampling的目的就是使得sample时更有可能sample到value大的point，在概率上来说就是pdf大的point，这样才能更加有效地采样，在有限的sample rate内实现教好地逼近积分的近似。
   2. 原理：要实现对一个积分进行Monte Carlo近似时，尽可能地以较少的sample rate实现较高地近似程度，我们需要选择sample的pdf以承接上面的说法。一种intuitive的做法就是以整个积分作为pdf（必须对积分进行normalize使得其cdf=1），这样就满足了上述的积分值越大的sample点其pdf值越大，但是直接取整个积分作为pdf时，难以以这种pdf进行采样，因此通常的做法是取积分中的某一个能够代表不同sample点大小的term作为pdf（需要normalize或称为scale使得cdf=1，亦即pdf正比于这个term），或者取多个这样的term来进行multiple importance sampling。若这样的term作为pdf易于采样（能够求解cdf的显式表达，然后以inverse sampling进行采样），则可以直接使用，若这样的pdf难以采样，则还需要用另外的易于采样的分布进行辅助求解。综上，通过这样的方法，我们就可以进行importance sampling。
   3. 以辅助分布进行采样：
   ![weight](./images/importance_weight.jpg)
   4. multiple importance sampling：适用于积分中不同的term含有不同的分布，那么以不同分布进行采样会更准确
   ![multiple](./images/multiple_importance.jpg)
   5. 以PBR为例，简要阐述其与importance sampling和期望的联系：
   ![expectation](./images/expectation.jpg)
   6. GGX importance sampling推导（通常用N作为pdf）：
   
   7. phong lobe importance sampling推导：
   ![phong](./images/phong_importance.jpg)
5. 对于image来说，如果直接用int去query第几个pixel，从理解上来说比较容易。但是如果需要用到float类型的index，则需要搞明白一点，若image space建一个二维坐标系，则img[0][0]这个pixel的中心为(0.5, 0.5)，而img[0][1]这个pixel的中心在(0.5, 1.5)。因此若需要用到float类型的index（sample；这个project中利用hdr envmap生成direction），若要取到某个pixel，必须取其中心，这样才能更加准确。
6. 

## 可调用的api
1. image space上的bilinear sample（numpy版本，torch可用grid_sample）。
2. 对hdr image在w维度上进行rotation。
3. cv2读取hdr。


## 犯过的错误
1. 不管是利用np还是torch，最好一直保持数据的dtype为float32。本次出错的地方在于，没有限制每个数据都为float32，导致获得的filtered_envmap为float64，将其存成hdr图像时，由于hdr默认格式为float32，所以不匹配，最终导致存取下来的hdr图像所有pixel值都为0，没有内容。
2. 当需要用到某个term作为pdf时，必须检查其是否大于0，检查其cdf是否为1。在这个project中，犯过的错误为直接用$cos(\theta)$作为pdf，但是其积分为$\pi$。

## 好用的工具
1. desmos：非常简易地画函数图像的工具，可以用这个工具画出函数，然后对其值有一个直观的了解，在这个project中，这个工具用于画出Fresnel-Schlick，发现其位于$\frac{\pi}{4}$内时的值非常小，可以用常数来表示。
   - 网址：https://www.desmos.com/calculator/u5unsfrcbe?lang=zh-CN
2. 