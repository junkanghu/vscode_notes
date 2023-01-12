## Lumos复现

## 待思考或解决的问题
1. 数据读取部分
- [ ] 弄明白hdr格式的envmap和exr格式的relit image到底怎么用，是直接转换为LDR使用还是?
- [ ] 研究一下怎么生成prefiltered envmap更加高效，更加接近好的效果？
- [ ] 搞明白如何从prefiltered envmap得到diffuse and specular light map。
  1. hdr envmap（image格式）怎么去围绕人，即horizontal方向上怎么去包住人。
  2. diffuse light map是在prefiltered map上直接query normal方向得到的，那么specular呢，如果也是query normal方向则不符合physical model。

2. network部分
- [ ] 用paper中的blur pool替换当时为了简便使用的maxpool。
- [ ] paper中使用的Discriminator似乎不是patchGAN，而当时为了简便直接用的patchGAN，因此需要替换。
- [ ] 几个network的activation function需要思考一下到底怎么用什么，Relu或Leacky Relu或Sigmoid。

3. loss部分
- [ ] 加上GAN loss，查看效果是否有提升。
- [ ] 加上VGG loss，查看效果是否有提升。这里的VGG loss并不是直接将image输入VGG然后使用其后面某一层的结果作为feature，而是取第1-5个activation function的输出算residual，然后算一个weight map去赋予不同的pixel不同的loss贡献权重，具体参考*LookinGood: Enhancing Performance Capture with Real-time Neural Re-Rendering*。
- [ ] 加上specular loss，查看效果是否有提升。

