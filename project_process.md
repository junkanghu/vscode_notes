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
- [x] 用paper中的blur pool替换当时为了简便使用的maxpool。
- [x] paper中使用的Discriminator似乎不是patchGAN，而当时为了简便直接用的patchGAN，因此需要替换。
   *paper使用的是lsgan，即取image patch的MSEloss作为Discriminator的输出，这与patchGAN一样，因此不需要修改。*
- [ ] 几个network的activation function需要思考一下到底怎么用什么，Relu或Leacky Relu或Sigmoid。

3. loss部分
- [ ] 加上GAN loss，查看效果是否有提升。这里的GAN loss不是对整张masked image进行计算，而是对脸部区域进行crop然后对patch进行计算。
- [x] 加上VGG loss，查看效果是否有提升。这里的VGG loss并不是直接将image输入VGG然后使用其后面某一层的结果作为feature，而是取第1-5个activation function的输出算residual，然后算一个weight map去赋予不同的pixel不同的loss贡献权重，具体参考*LookinGood: Enhancing Performance Capture with Real-time Neural Re-Rendering*。
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
   2. 原理：要实现对一个积分进行Monte Carlo近似时，尽可能地以较少的sample rate实现较高地近似程度，我们需要选择sample的pdf以承接上面的说法。一种intuitive的做法就是以整个积分作为pdf（必须对积分进行normalize使得其cdf=1），这样就满足了上述的积分值越大的sample点pdf值越大，但是直接取整个积分作为pdf时，难以以这种pdf进行采样，因此通常的做法是取积分中的某一个能够代表不同sample点大小的term作为pdf（需要normalize或称为scale使得cdf=1，亦即pdf正比于这个term），或者取多个这样的term来进行multiple importance sampling。若这样的term作为pdf易于采样（能够求解cdf的显式表达，然后以inverse sampling进行采样），则可以直接使用，若这样的pdf难以采样，则还需要另外的易于采样的分布进行辅助求解。综上，通过这样的方法，我们就可以进行importance sampling。
   3. 以辅助分布进行采样：
   ![weight](./images/importance_weight.jpg)
   4. multiple importance sampling：适用于积分中不同的term含有不同的分布，那么以不同分布进行采样会更准确
   ![multiple](./images/multiple_importance.jpg)
   5. 以PBR为例，简要阐述其与importance sampling和期望的联系：
   ![expectation](./images/expectation.jpg)
   6. GGX importance sampling推导（通常用D作为pdf）：

   D的表达式为：

   $$
   D(h)=\frac{\alpha^2}{\pi\left(\left(\alpha^2-1\right) \cos ^2 \theta+1\right)^2}
   $$

   由于D不是完全的pdf，需要乘上$cos\theta$（详细见下面的microfacet剖析），因此pdf为：

   $$
   p_h(\omega)=\frac{\alpha^2 \cos \theta}{\pi\left(\left(\alpha^2-1\right) \cos ^2 \theta+1\right)^2}
   $$

   然后再根据$p(\omega)$和$\theta$ $\phi$的推导方式即可得到：

   $$
   \begin{aligned}
   P_h(\theta) & =\int_0^\theta \frac{2 \alpha^2 \cos (t) \sin (t)}{\left(\cos ^2 t\left(\alpha^2-1\right)+1\right)^2} d t \\
   & =\int_\theta^0 \frac{\alpha^2}{\left(\cos ^2 t\left(\alpha^2-1\right)+1\right)^2} d\left(\cos ^2 t\right) \\
   & =\frac{\alpha^2}{\alpha^2-1} \int_0^\theta d \frac{1}{\cos ^2 t\left(\alpha^2-1\right)+1} \\
   & =\frac{\alpha^2}{\alpha^2-1}\left(\frac{1}{\cos ^2 \theta\left(\alpha^2-1\right)+1}-\frac{1}{\alpha^2}\right) \\
   & =\frac{\alpha^2}{\cos ^2 \theta\left(\alpha^2-1\right)^2+\left(\alpha^2-1\right)}-\frac{1}{\alpha^2-1}
   \end{aligned}
   $$

   根据CDF反采样方式最终可以得到（uniform采样$\xi_1$ $\xi_2$）：

   $$
   \theta=\arccos \sqrt{\frac{1-\xi_1}{\xi_1 \left(\alpha^2-1\right)+1}}=\arctan \left(\alpha \sqrt{\frac{\xi_1}{1-\xi_1}}\right) \\
   \phi = 2\pi \xi_2
   $$

   7. phong lobe importance sampling推导：
   ![phong](./images/phong_importance.jpg)
5. 对于image来说，如果直接用int去query第几个pixel，从理解上来说比较容易。但是如果需要用到float类型的index，则需要搞明白一点，若image space建一个二维坐标系，则img[0][0]这个pixel的中心为(0.5, 0.5)，而img[0][1]这个pixel的中心在(0.5, 1.5)。因此若需要用到float类型的inex（sample；这个project中利用hdr envmap生成direction），若要取到某个pixel，必须取其中心，这样才能更加准确。
6. Microfacet BRDF：用macrofacet来等效其中的无数个小的microfacet的综合效果。
   1. Normal Distribution Function D(m):
      1. 作用：其中的m代表的是Macrofacet中的microfacet的normal方向。理解为某一个macrofacet中normal方向为m的microfacet（完美镜面）的"面积"（非投影"面积"）总和。之所以加引号是因为D(m)的面积只是一个没有单位的ratio，其为其$\frac{真实面积}{macrofacet面积}$。(本来就是想以D(m)来表示normal=m的microfacet的比例，只不过这个比例用面积来表示而已)
      2. 特性
      - 在某个macrofacet中，在某个单位立体角$d\omega_m$内的normal方向为m的microfacet总面积为：
      $$
      {\rm d}A_m = D(m){\rm d}\omega_m {\rm d}A \\
      \frac{{\rm d}A_m}{{\rm d}A} = D(m){\rm d}\omega_m （即为1中所述的ratio，只不过加了个微元）
      $$
      - D(m)必须大于0: $0\leq D(m) \leq \infty$
      - 由于D(m)是非投影面积，相对与macrofacet来说，其中的所有microfacet的面积总和大于macrofacet的面积总和: 
      $$
      1 \leq \int D(m)d\omega_m \Rightarrow dA \leq \int D(m)d\omega_m dA
      $$
      - 把所有macrofacet中的microfacet的面积投影到任意方向$\vec{v}$上的投影总面积等于将macrofacet的面积投影到那个方向上得到的投影面积:
      $$
      \vec{v} \cdot \vec{n} = \int D(m)(\vec{v} \cdot \vec{m})d\omega_m \\
      \Rightarrow \vec{v} \cdot \vec{n} dA = \int D(m)(\vec{v} \cdot \vec{m})dAd\omega_m \\
      $$
      - 上一条的特殊情况为$\vec{v}=\vec{n}$，此时所有microfacet投影到macrofacet上的投影面积等于macrofacet的面积$dA$，反应在ratio上就是投影signed面积总和为1，代表为1个单位的macrofacet面积：
      $$
      1 = \int D(m)(\vec{n} \cdot \vec{m})d\omega_m \\
      \Rightarrow dA = \int D(m)(\vec{n} \cdot \vec{m})dAd\omega_m
      $$

      3. 常用表达式（GGX Normal Distribution）

      $$
      D_{G G X}(h, \alpha)=\frac{\alpha^2}{\pi\left((n \cdot h)^2\left(\alpha^2-1\right)+1\right)^2} \\
      \alpha = roughness ^ 2
      $$

   2. Shadowing-Masking Function $G(\vec{i}, \vec{o}, \vec{m})$
   ![G](./images/G.png)
      1. 作用：masking代表视线方向上有物体遮挡导致看不见，shadowing代表光线方向上有物体遮挡导致光线照射不到。因此G代表着visibility，综合表示物体到光源的visibility和人眼到物体的visibility，总体表示从入射方向射入的光有多少能够被人眼看到。其跟roughness有关，roughness值越大代表macrofacet表面越粗糙，越有可能遮挡。
      2. 特性
      - 由于其代表着visibility，因此其值位于0-1之间，0代表完全看不到，1代表完全无遮挡：$0 \leq G(\vec{i}, \vec{o}, \vec{m}) \leq 1$。
      - 由于G代表的是综合特性，因此根据光路可逆性，将入射和出射方向调换，其值不变：$G(\vec{i}, \vec{o}, \vec{m}) = G(\vec{o}, \vec{i}, \vec{m})$
      - 由于macrofacet和microfacet都是有方向的面积元，因此当光线或者视线方向不在normal所在的半球内，G必然为0：
      $$
      G(\vec{i}, \vec{o}, \vec{m}) = 0 \\
      if (\vec{i} \cdot \vec{m})(\vec{i} \cdot \vec{n}) \leq 0 \\
      or for \vec{o}
      $$
      3. 结合D：上述所说D(m)是macrofacet中单位立体角内的normal方向为m的microfacet面积总和，算面积是为了进一步算irradiance进而算radiance。如果该面积元的方向在某个视线方向上不可见，也就不可能朝着视线方向辐射光线，因此我们要的是可见的面积元总面积，即visible area。D(m)只能够表示面积大小无法表示visibility，因此将G乘上D(GD)即可获得visible area。
      4. 常用表达式（Smith-GGX formulation）

      $$
      G(\mathbf{v}, \mathbf{l}, \mathbf{\alpha}) = G_1(\mathbf{l}, \mathbf{\alpha}) G_1(\mathbf{v}, \mathbf{\alpha}) \\
      G_1(\mathbf{v}, \mathbf{\alpha}) = G_{GGX}(\mathbf{v}, \mathbf{\alpha}) = \frac{2(\mathbf{n} \cdot \mathbf{v})}{\mathbf{n} \cdot \mathbf{v} + \sqrt{\alpha^2 + (1-\alpha^2)(\mathbf{n} \cdot \mathbf{v})^2}} \\
      G(v, l, \alpha)=\frac{2(n \cdot l)}{n \cdot l+\sqrt{\alpha^2+\left(1-\alpha^2\right)(n \cdot l)^2}} \frac{2(n \cdot v)}{n \cdot v+\sqrt{\alpha^2+\left(1-\alpha^2\right)(n \cdot v)^2}}
      $$


   3. Fresnel Function $F(\mathbf{i}, \mathbf{o}, \mathbf{m})$
      1. 解释：光线入射到一个法向为normal=**n**的表面上时，由于IOR的影响，当入射方向与normal方向不同时，被反射（完美角度）的光线的能量和被折射的光线的能量比例不同（但它们的能量总和等于入射光线），一般取air射入glass作为参照。在入射角为grazing angle时，反射的比例最大为100%，折射的比例为0%；在normal方向入射时，反射的比例最小为4%，折射的比例为96%。因此在计算反射的radiance时，需要考虑反射部分的总能量为多少，即入射能量乘以Fresnel。（Fresnel指的是反射的比例，折射的比例为1-F）
      2. Fresnel与物体颜色的关系：dielectric material（metallic=0）含有diffuse part和specular part，但是其specular reflectance是achromatic的，因此无法代表其basecolor，但是其diffuse albedo可以代表其basecolor（由于wavelength的原因，对不同通道含有不同的反射率，因此albedo是三通道量）；而对于Conductors（metallic=1），其几乎不含有或含有可忽略的diffuse分量（$f_0$三个通道的反射率接近1，那么非$f_0$几乎都是等于1），因此其只有specular reflectance，一般用其在normal方向的specular reflectance($f_0$)作为其basecolor。
      3. 常用的表达式（Schlick-fresnel）:

      $$
      F_{Schlick}(\mathbf{v}, \mathbf{h}, f_0, f_{90}) = f_0 + (f_{90}-f_0)(1 - \mathbf{v} \cdot \mathbf{h}) ^ 5
      $$

      其中$f_0$代表在normal入射方向的反射率，其为4%，$f_{90}$代表在grazing anle入射方向的反射率，其为100%。
   4. Macrosurface BSDF Integral Derivation
      1. 总原则：macrofacet朝着某个立体角辐射的power总和等于所有microfacet朝着同一个立体角辐射的power总和。
      2. 推导
      首先计算从立体角$d\omega_i$射入到macrofacet的irradiance：
      $$
      d E=L_i d \omega_i|(\mathbf{n} \cdot \mathbf{i})|
      $$
      从收到的这部分irradiance向单位立体角$d\omega_o$辐射的radiance为：
      $$
      d L_o=f_s(\mathbf{i}, \mathbf{o}, \mathbf{n}) d E
      $$
      为了获得macrofacet向单位立体角$d\omega_o$内辐射能量的irradiance（一般称为radiance exitence或exit radiance，其代表这块面积往$d\omega_o$方向辐射radiance的能力，因为radiance同磁感线一样都是往normal方向辐射的，要算其他方向的分量，必须乘上$cos\theta$；而当面积元吸收能量时，其最佳吸收方向为normal，若光线方向不为normal，必须乘上$cos\theta$），我们首先需要将radiance乘上立体角和$cos\theta$（radiance定义）：

      $$
      dL_o = \frac{d \Phi^2}{d \omega_o dA cos\theta} = \frac{d M}{d \omega_o cos\theta} \\
      \Rightarrow dM = d L_o d\omega_o |\vec{n} \cdot \vec{o}|
      $$

      上面的$cos\theta= |\vec{n} \cdot \vec{o}|$是因为同吸收能量的irradiance一样，需要算往某个方向辐射能量的irradiance。
      获得了exit radiance后再计算power：

      $$
      d\Phi = dMdA=f_s(\mathbf{i}, \mathbf{o}, \mathbf{n}) L_i |(\mathbf{n} \cdot \mathbf{i})||\vec{n} \cdot \vec{o}| d\omega_i d\omega_o dA
      $$

      为了计算所有microfacet向同样方向内辐射的power，我们首先从normal方向为m的所有microfacet开始算其辐射的power，然后对所有m方向进行积分获得总的辐射power。对某一个单位面积的microfacet（normal=m）来说，其irradiance为：

      $$
      dE_m = L_i d\omega_i |(m \cot i)|
      $$

      其往$\omega_o$方向辐射的radiance为：

      $$
      dL_{mo} = f^m_s (\mathbf{i}, \mathbf{o}, \mathbf{m}) dE_m
      $$

      其往$\omega_o$方向所在的单位立体角内辐射能量的irradiance为:

      $$
      dM_m = d L_{mo} d\omega_o |(\mathbf{m} \cdot \mathbf{o})|
      $$

      所有单位立体角$d\omega_m$内normal=m的microfacet的visible area为：

      $$
      d A_m = D(m)G(\mathbf{i}, \mathbf{o}, \mathbf{m}) d\omega_m dA
      $$

      因此该单位立体角$d\omega_m$内向$d\omega_o$辐射的power为：

      $$
      d \Phi_m = d M_m d A_m
      $$

      故所有单位立体角内（macrofacet的normal方向所在的hemisphere）的microfacet向$d\omega_o$方向辐射的总power为：

      $$
      \begin{aligned}
      d \Phi &= \int d \Phi_m d \omega_m \\
      &= \int f^m_s(\mathbf{i}, \mathbf{o}, \mathbf{m}) L_i |(\mathbf{m} \cdot \mathbf{i})| |(\mathbf{m} \cdot \mathbf{o})| D(m) G(\mathbf{i}, \mathbf{o}, \mathbf{m}) d\omega_i d\omega_o dA d\omega_m
      \end{aligned}
      $$

      根据能量相等总原则：
      $$
      f_s(\mathbf{i}, \mathbf{o}, \mathbf{n}) L_i |(\mathbf{n} \cdot \mathbf{i})||\vec{n} \cdot \vec{o}| d\omega_i d\omega_o dA = \int f^m_s(\mathbf{i}, \mathbf{o}, \mathbf{m}) L_i |(\mathbf{m} \cdot \mathbf{i})| |(\mathbf{m} \cdot \mathbf{o})| D(m) G(\mathbf{i}, \mathbf{o}, \mathbf{m}) d\omega_i d\omega_o dA d\omega_m \\ 
      \Rightarrow f_s(\mathbf{i}, \mathbf{o}, \mathbf{n}) = \int f^m_s(\mathbf{i}, \mathbf{o}, \mathbf{m}) \frac{|(\mathbf{i} \cdot \mathbf{m})|}{|(\mathbf{i} \cdot \mathbf{n})|} \frac{|(\mathbf{o} \cdot \mathbf{m})|}{|(\mathbf{o} \cdot \mathbf{n})|} D(m) G(\mathbf{i}, \mathbf{o}, \mathbf{m}) d\omega_m
      $$
      3. 
   5. Microsurface Specular BSDFs（推导方式1）
      1. 前提：每个microfacet都是一个完美的镜面。
      2. 推导
      4中的integral中每个microfacet镜面的BSDF$f_s^m$并没有显式地表达出来，在这里，我们将其显式表达，然后带入4中的式子，最终得到BRDF表达式。

      $f_s^m(\mathbf{i}, \mathbf{o}, \mathbf{m})$通常的表达式为：   
      $$
      f_s^m(\mathbf{i}, \mathbf{o}, \mathbf{m})=\rho \frac{\delta_{\omega_o}(\mathbf{s}, \mathbf{o})}{|\mathbf{o} \cdot \mathbf{m}|}
      $$

      其推导过程如下：
      从$\omega_i$方向入射的radiance使得镜面面积元的irradiance为：
      $$
      d E=L_i|\vec{i} \cdot \vec{n}| \cdot d \vec{w}_i
      $$

      往$d \omega_o$立体角内辐射的radiance为：

      $$
      d L_o=d E \cdot f_s(\vec{i}, \overrightarrow{0}, \vec{m})
      $$

      射入面积元的总power为：

      $$
      \Phi_{\text {in }}=L_i|\vec{i} \cdot \vec{n}| \cdot d \vec{\omega}_i \cdot d A
      $$
      
      从单位面积元往外辐射的irradiance为（一般的irradiance都以$d \omega_o$来算，但是这里由于为完美的镜面反射，因此所有能量集中到一个单一的方向上，因此需要以$\delta$函数来表示）：
      $$
      d M=\int d L_o \cdot|\vec{n} \cdot \vec{o}| \cdot \delta(\vec{s}, \vec{o}) \cdot d \vec{w}_o
      $$

      往出射方向辐射的总power为：
      $$
      \Phi_{\text {out }}=d M d A=\int d L_o \cdot|\vec{n} \cdot \vec{o}| \cdot \delta(\vec{s}, \vec{o}) \cdot d \vec{\omega}_o \cdot d A
      $$

      根据出射power=入射power即可得到上面的表达式。消去一些term后，delta函数会消失，但是由于只往一个单一方向辐射，因此也需要在BRDF中加入delta项表示这种单一方向的辐射。

      现在要将$f_s^m(\mathbf{i}, \mathbf{o}, \mathbf{m})$从关于$\vec{o}$的函数变换为关于half vector $\vec{h}$的函数。$h(\vec{o})$是关于$\vec{o}$的函数，对于$f_s^m(\mathbf{i}, \mathbf{o}, \mathbf{m})$来说，从物理意义上来理解，除了delta function需要改成关于$\mathbf{h}$的函数，其它项都是constant不需要改变。虽然我们可以将delta function理解为若方向相同则为1，但是其是根据积分严格定义的，后续的推导中需要对其进行积分，因此在这里我们需要考虑将$\mathbf{o}$改为$\mathbf{h}(o)$之后的微元变化。那么我们首先考虑对$d \omega_h$进行积分（$\mathbf{s}$代表镜面的完美反射角度）：

      $$
      f_s^m(\mathbf{i}, \mathbf{s}, \mathbf{m})=\int_{\omega_h} \rho(\mathbf{i}, \mathbf{m}) \frac{\delta_{\omega_m}(\mathbf{h}(\mathbf{i}, \mathbf{o}), \mathbf{m})}{|\mathbf{o} \cdot \mathbf{m}|} d \omega_h \\
      此时\mathbf{o} = \mathbf{s}，\  故\mathbf{h}=\mathbf{h}(\mathbf{i}, \mathbf{s}) \\
      结果为 \\
      f_s^m(\mathbf{i}, \mathbf{s}, \mathbf{m}) = \rho (\mathbf{i}, \mathbf{m}) \frac{1}{|\mathbf{s} \cdot \mathbf{m}|}
      $$
      
      由于在上面的积分式子中，$\mathbf{i}$和$\mathbf{m}$都可以视为constant，因此上式可以简化为：

      $$
      f_s^m(\mathbf{i}, \mathbf{s}, \mathbf{m})=\int_{\omega_h} \rho(\mathbf{i}, \mathbf{m}) \frac{\delta_{\omega_m}(\mathbf{h}(\mathbf{o}))}{|\mathbf{o} \cdot \mathbf{m}|} d \omega_h 
      $$ 

      要将其转化为关于$d \omega_o$的积分需要算Jacobian：

      $$
      f_s^m(\mathbf{i}, \mathbf{s}, \mathbf{m})= \int_{\omega_o} \rho(\mathbf{i}, \mathbf{m}) \frac{\delta_{\omega_m}(\mathbf{h}(\mathbf{i}, \mathbf{o}), \mathbf{m})}{|\mathbf{o} \cdot \mathbf{m}|}\left\|\frac{\partial \omega_{\mathbf{h}}}{\partial \omega_{\mathbf{o}}}\right\| d \omega_o
      $$

      这是直接求了$\mathbf{s}$方向的BRDF值，BRDF function仍为：

      $$
      \rho(\mathbf{i}, \mathbf{m}) \frac{\delta_{\omega_m}(\mathbf{h}(\mathbf{i}, \mathbf{o}), \mathbf{m})}{|\mathbf{o} \cdot \mathbf{m}|}\left\|\frac{\partial \omega_{\mathbf{h}}}{\partial \omega_{\mathbf{o}}}\right\|
      $$

      可以算出Jacobian（见6中第二种推导方式）然后代入其中得到：

      $$
      f_r^m(\mathbf{i}, \mathbf{o}, \mathbf{m})=F(\mathbf{i}, \mathbf{m}) \frac{\delta_{\omega_m}\left(\mathbf{h}_{\mathrm{r}}, \mathbf{m}\right)}{4\left(\mathbf{i} \cdot \mathbf{h}_{\mathrm{r}}\right)^2}
      $$

      delta function是多元函数，在之前的积分中对变量$d \omega_o$进行积分时可将$\mathbf{m}$视为constant，后续的推导中需要对$d \mathbf{m}$进行积分。将该式代入4中的表达式，可以最终得到标准的Microfacet BRDF公式：
      
      $$
      \begin{aligned}
      f_s(\mathbf{i}, \mathbf{o}, \mathbf{n}) &= \int f^m_s(\mathbf{i}, \mathbf{o}, \mathbf{m}) \frac{|(\mathbf{i} \cdot \mathbf{m})|}{|(\mathbf{i} \cdot \mathbf{n})|} \frac{|(\mathbf{o} \cdot \mathbf{m})|}{|(\mathbf{o} \cdot \mathbf{n})|} D(m) G(\mathbf{i}, \mathbf{o}, \mathbf{m}) d\omega_m \\
      &= \int F(\mathbf{i}, \mathbf{m}) \frac{\delta_{\omega_m}\left(\mathbf{h}_{\mathrm{r}}, \mathbf{m}\right)}{4\left(\mathbf{i} \cdot \mathbf{h}_{\mathrm{r}}\right)^2} \frac{|(\mathbf{i} \cdot \mathbf{m})|}{|(\mathbf{i} \cdot \mathbf{n})|} \frac{|(\mathbf{o} \cdot \mathbf{m})|}{|(\mathbf{o} \cdot \mathbf{n})|} D(m) G(\mathbf{i}, \mathbf{o}, \mathbf{m}) d\omega_m
      \end{aligned}
      $$

      当$\omega_m=h_r$时，即可得到标准的microfacet specular BRDF公式。

   6. Microsurface Specular BSDFs（推导方式2）

   给定入射方向$\vec{i}$和出射方向$\vec{o}$，只有满足normal方向为half vector的microfacet能够把光线反射，这些microfacet的总面积为：

   $$
   d A\left(\omega_h\right)=D\left(\omega_h\right) d \omega_h d A
   $$

   若考虑遮挡，即得到visible area，其面积为：


   $$
   d A\left(\omega_h\right)=D\left(\omega_h\right) G d \omega_h d A
   $$

   射入这些microfacet的总power为（$d A^{\perp}$指的是这部分microfacet的面积在入射方向上的投影面积）：

   $$
   \begin{aligned}
   d \Phi_i & =L_i\left(\omega_i\right) d \omega_i d A^{\perp}\left(\omega_h\right) \\
   & =L_i\left(\omega_i\right) d \omega_i \cos \theta_h d A\left(\omega_h\right) \\
   & =L_i\left(\omega_i\right) d \omega_i \cos \theta_h D\left(\omega_h\right) G d \omega_h d A
   \end{aligned}
   $$

   从这些microfacet输出的总power为（乘上反射能量的比例）：

   $$
   d \Phi_o = \rho d \Phi_i \\
   \because take \ Fresnel \ F \  as \ \rho \\
   \therefore d \Phi_o = F d \Phi_i \\
   $$

   根据出射的radiance定义得：
   $$
   d L_o\left(\omega_o\right)=\frac{d \Phi_o}{d \omega_o \cos \theta_o d A}=\frac{L_i\left(\omega_i\right) d \omega_i \cos \theta_h D\left(\omega_h\right) GF d \omega_h d A}{d \omega_o \cos \theta_o d A}
   $$

   从macrofacet的BRDF定义来说：

   $$
   f_{\text {cook-torrance }}\left(\omega_i, \omega_o\right)=\frac{d L_o\left(\omega_o\right)}{d E_i\left(\omega_i\right)}=\frac{d L_o\left(\omega_o\right)}{L_i\left(\omega_i\right) \cos \theta_i d \omega_i}=\frac{\cos \theta_h D\left(\omega_h\right) GF d \omega_h}{\cos \theta_o \cos \theta_i d \omega_o}
   $$

   接下来需要算$\frac{d \omega_h}{d \omega_o}$，其数学意义是，当$\omega_o$变化$d \omega_o$时算出$\omega_h$变化的量$d \omega_h$，然后求比值即可。见下图。

   ![domega](./images/domega.png)

   根据solid angle的定义可以算出$d \omega_o$对应的面积$d A$为：

   $$
   d A = \frac{d \omega_o}{r^2} = d \omega_o
   $$

   其对应在$\vec{h_r}$所在的圆上的solid angle为：

   $$
   d \omega_h = \frac{d A'}{||\vec{h_r}|| ^2} = \frac{d A |\vec{o} \cdot h_r|}{||\vec{h_r}|| ^2} = \frac{ |\vec{o} \cdot h_r|}{||\vec{h_r}|| ^2} d \omega_o
   $$

   因为$\vec{h_r}$为单位向量$h_r$的带长度版本，因此：

   $$
   \vec{h_r} = sign(\vec{i} \cdot \vec{n})(\vec{i} + \vec{o}) = sign(\vec{i} \cdot \vec{n}) \cdot 2|\vec{o} \cdot h_r| h_r \\
   d \omega_h = \frac{d A'}{||\vec{h_r}|| ^2} = \frac{d A |\vec{o} \cdot h_r|}{||\vec{h_r}|| ^2} = \frac{ |\vec{o} \cdot h_r|}{||\vec{h_r}|| ^2} d \omega_o \\
   代入可得：
   \frac{d \omega_h}{d \omega_o}=\frac{1}{4 \cos \theta_h}
   $$

   将$\frac{d \omega_h}{d \omega_o}$代入BRDF式子可得：

   $$
   f_{\text {cook-torrance }}\left(\omega_i, \omega_o\right)=\frac{F_r\left(\omega_o, \omega_h\right) D\left(\omega_h\right) G\left(\omega_i, \omega_o\right)}{4 \cos \theta_o \cos \theta_i}
   $$

7. Phong BRDF
   1. 理论：基于lobe，即入射光线在反射时，若反射表面不是完美镜面，则所有出射方向都围绕在完美反射方向周围，并含有不同的能量分布，这种能量分布就是phong lobe对应的BRDF。由于光路可逆性可知，某一个出射方向的能量是由入射的一个lobe所决定，它们围绕在完美入射方向周围，并贡献不同的能量。
   2. 表达式

   $$
   \begin{aligned}
   f_r\left(\mathbf{x}, \omega_i, \omega_r\right) & =f_{r, d}\left(\mathbf{x}, \omega_i, \omega_r\right)+f_{r, s}\left(\mathbf{x}, \omega_i, \omega_r\right) \\
   & =\rho_d \frac{1}{\pi}+\rho_s \frac{n+2}{2 \pi} \cos ^n \alpha
   \end{aligned}
   $$

   其中$\rho_d$代表入射的能量有多少以diffuse的形式被反射出去，$\rho_s$代表入射的能量有多少以specular的形式被反射出去。当光线入射到表面时，一部分被reflected成为reflectance，一部分被transmitted成为transmittance，被transmitted的这部分光线进入物体表面，一部分被物体所吸收，一部分被再次反射出物体表面成为diffuse分量。因此$\rho_d + \rho_s \leq 1$而不是等于1，因为可能有一部分能量被吸收。

8. BlurPool：CNN不具有shift-invariant，使得微小的shift造成输出的剧烈变化，这是由于downsample时忽略了信号处理中的采样原则（需要利用低通滤波去除掉混叠的高频部分）。Blurpool能够缓解这种shift-variant，保证CNN对相同的图像内容输出的feature更加地相似，提高对图像的识别能力，也能做到anti-aliasing。
9. split sum approximation
   1. 总原则：
   对于radiance计算公式：

   $$
   L_o=\int_{\Omega} L_i(l) f(\alpha, v, n, l)\langle n, l\rangle d l
   $$

   如果想要将$L_i$单独从积分中提取出来，可以将积分分解为：

   $$
   L_o=L_p(v, n) \int_{\Omega} f(\alpha, v, n, l)\langle n, l\rangle d l
   $$

   那么我们可以获得只含有入射光的term：

   $$
   L_p(\alpha, v, n) \approx \frac{\int_{\Omega} L_i(l) f(\alpha, n, n, l)\langle n, l\rangle d l}{\int_{\Omega} f(\alpha, n, n, l)\langle n, l\rangle d l}
   $$

   这就是prefiltered lightmap最原始的推导公式，我们需要结合Monte Carlo一步步进行简化

   2. 对于lambertion的diffuse分量计算irradiance map
   对于lambertion来说：

   $$
   L_o = \frac{\rho}{\pi} \int_{\Omega} L_i(l) \langle n, l\rangle d l
   $$

   积分中已经不含有BRDF项，只含有light相关的term，因此可以认为1中的总原则公式就等于此式中的积分：

   $$
   L_p(v, n) = \int_{\Omega} L_i(l) \langle n, l\rangle d l 
   $$

   利用Monte Carlo进行importance sampling：

   $$
   L_p(v, n) = \frac{\frac{1}{N} \sum_{i=1}^{N} L_i cos \theta}{p(\omega_i)} = \frac{\frac{1}{N} \sum_{i=1}^{N} L_i cos \theta}{\frac{cos\theta}{\pi}} = \frac{\pi}{N} L_i
   $$

   对于lambertion来说，这些$L_i$的方向分布在surface normal为中心的hemisphere周围，因此对于每个surface来说，总的入射光应该取prefiltered map的normal方向的value。而prefiltered map可以对每个方向进行hemisphere上的weighted sum。

   3. 对于phong lobe对应的BRDF来说：
   其BRDF为：

   $$
   f_s = \frac{n + 2}{2 \pi} cos ^n \alpha
   $$

   其中$\alpha$为light direction与view direction关于normal的完美反射角之间的关系。
   
   <br />

   从phong BRDF的公式来看，不管view direction对应的完美反射角为多少，对于同一个n，围绕在其周围的lobe形状都是完全一样的。

   <br />

   因此不管view direction是怎么样的，我们可以认为空间中每个方向都可能成为一个关于view direction和normal的完美反射角。因此我们可以以其为z轴建立局部坐标系，根据固定lobe的形状，算其weighted sum。在取得实际的view direction和normal后，可以根据它们算出完美的反射方向，然后以这个方向去query算好的prefiltered envmap来获得总的关于light的term。

   <br />

   根据1中的总原则公式：

   $$
   L_p(\alpha, v, n) \approx \frac{\int_{\Omega} L_i(l) \frac{n + 2}{2 \pi} cos ^n \alpha \langle n, l\rangle d l}{\int_{\Omega} \frac{n + 2}{2 \pi} cos ^n \alpha \langle n, l\rangle d l}
   $$

   以phong BRDF作为PDF进行Monte Carlo计算（由于我们是在local frame中计算这个式子，所以$\alpha=\theta$，其为光线与z轴的夹角）：

   $$
   L_p(\alpha, v, n) \approx \frac{\frac{\frac{1}{N} \sum_{1}^N L_i(l) \frac{n + 2}{2 \pi} cos ^n \theta cos \theta'}{\frac{n + 2}{2 \pi} cos ^n \theta}}{\frac{\frac{1}{N} \sum_1^N \frac{n + 2}{2 \pi} cos ^n \alpha cos \theta'}{\frac{n + 2}{2 \pi} cos ^n \theta}} = \frac{\sum_1^N L_i cos \theta'}{\sum_1^N cos \theta'}
   $$

   <br />

   需要注意的是这里的$cos \theta'$指的是入射光线与surface normal的夹角，但是由于我们实现不知道surface normal，且对于不同的normal其$\theta'$不同，因此在这里做了一个近似，即$\theta'=\theta$，因为本来就是对$cos \theta$的一个weighted sum。在这样等效后，应该有人验证了与直接渲染得到的结果误差很小，人眼几乎看不出，因此采用了这种方法去做估计。（证明了在graphic中只要看起来合理就是合理的）

   根据这个公式可以把原来的envmap的每一个方向上的radiance存储为根据lobe prefiltered后的结果，那么就可以根据normal和view direction算出的完美light方向去query prefiltered envmap来获得其$L(\mathbf{v}, \mathbf{n})$。

   4. 对于microfacet对应的BRDF来说：
   在phong lobe中，由于BRDF本身就规定了对于同一个n，因此所有的lobe形状完全一样，但是在microfacet中，对于某一个normal的macrosurface来说，不同的view direction对应的围绕在完美incident light的lobe形状是不同的，所以没办法像phong lobe一样自然地去直接计算。

   <br />

   为了像phong lobe一样能够类似地计算，unreal engine也假设lobe形状都是相同的，而这个形状是根据microfacet BRDF去规定的。其假设这个lobe的形状为，当normal和view direction相同时，入射光线所形成的lobe形状（此时完美的镜面反射方向也等于normal方向，即$\mathbf{n}=\mathbf{v}=\mathbf{i}$，而能够产生贡献的light direction围绕在这个完美方向周围），当物体表面的roughness确定时，这个lobe的形状就是确定的，其用于整个map的prefilter。可以通过改变roughness来改变这个lobe的形状。

   <br />

   那么我们就可以像phong lobe一样的方式去计算（此时也放在一个local frame中计算，也像phong lobe中一样将原来光线与surface normal的$cos \theta$近似为光线与lobe中心的夹角）：
   ![lobe](./images/lobe.png)

   $$
   f_s(\mathbf{\omega_o}, \mathbf{\omega_i}, \mathbf{N}) = \frac{F D G}{4\left(N \cdot \omega_o\right)\left(N \cdot \omega_i\right)} \\
   近似为上述lobe:\\  
   R=N=V（将原来的V和N算出来的完美入射角方向R作为像phong \ lobe一样的z轴，后续的计算都围绕这个z轴，那么这个lobe的形状对于同一个roughness都是确定的了）\\
   \begin{aligned}
   L_p(\alpha, v, n) &= \frac{\int_{\Omega} L_i(l) f_s(\mathbf{R}, \mathbf{\omega_i(R)}, \mathbf{R}) \langle R, l \rangle dl}{\int_{\Omega} f_s(\mathbf{R}, \mathbf{\omega_i(R)}, \mathbf{R}) \langle R, l \rangle dl} \\
   &= \frac{\frac{1}{N} \sum_k^N \frac{D\left(\omega_h^{(k)}\right) F\left(\omega_o \cdot \omega_h^{(k)}\right) G\left(\omega_o, \omega_i^{(k)}\right) L_i\left(\omega_i^{(k)}\right)\left(n \cdot \omega_i^{(k)}\right)}{4\left(\omega_o \cdot n\right)\left(\omega_i^{(k)} \cdot n\right) p\left(\omega_i^{(k)}\right)}}{\frac{1}{N} \sum_k^N \frac{D\left(\omega_h^{(k)}\right) F\left(\omega_o \cdot \omega_h^{(k)}\right) G\left(\omega_o, \omega_i^{(k)}\right)\left(n \cdot \omega_i^{(k)}\right)}{4\left(\omega_o \cdot n\right)\left(\omega_i^{(k)} \cdot n\right) p\left(\omega_i^{(k)}\right)}} \\
   &= \frac{\sum_k^N \frac{D\left(\omega_h^{(k)}\right) F\left(\omega_o \cdot \omega_h^{(k)}\right) G\left(\omega_o, \omega_i^{(k)}\right) L_i\left(\omega_i^{(k)}\right)}{4\left(\omega_0 \cdot n\right) \frac{D\left(\omega_h^{(k)}\right)\left(\omega_h^{(k)} \cdot n\right)}{4\left(\omega_0 \cdot \omega_h^{(k)}\right)}}}{\sum_k^N \frac{D\left(\omega_h^{(k)}\right) F\left(\omega_o \cdot \omega_h^{(k)}\right) G\left(\omega_o, \omega_i^{(k)}\right)}{4\left(\omega_0 \cdot n\right) \frac{D\left(\omega_h^{(k)}\right)\left(\omega_h^{(k)} \cdot n\right)}{4\left(\omega_0 \cdot \omega_h^{(k)}\right)}}} \\
   &= \frac{\sum_k^N \frac{F\left(\omega_o \cdot \omega_h^{(k)}\right) G\left(\omega_o, \omega_i^{(k)}\right) L_i\left(\omega_i^{(k)}\right)\left(\omega_o \cdot \omega_h^{(k)}\right)}{\omega_h^{(k)} \cdot n}}{\sum_k^N \frac{F\left(\omega_o \cdot \omega_h^{(k)}\right) G\left(\omega_o, \omega_i^{(k)}\right)\left(\omega_o \cdot \omega_h^{(k)}\right)}{\omega_h^{(k)} \cdot n}} \\
   &= \frac{\sum_k^N \frac{F\left(\omega_o \cdot \omega_h^{(k)}\right) G_1\left(\omega_i^{(k)}\right) G_1\left(v\right) L_i\left(\omega_i^{(k)}\right)\left(\omega_o \cdot \omega_h^{(k)}\right)}{\omega_h^{(k)} \cdot n}}{\sum_k^N \frac{F\left(\omega_o \cdot \omega_h^{(k)}\right) G_1\left(\omega_i^{(k)}\right) G_1\left(v\right) \left(\omega_o \cdot \omega_h^{(k)}\right)}{\omega_h^{(k)} \cdot n}} \\
   &= \frac{\sum_k^N \frac{F\left(\omega_o \cdot \omega_h^{(k)}\right) G_1\left(\omega_i^{(k)}\right) L_i\left(\omega_i^{(k)}\right)\left(\omega_o \cdot \omega_h^{(k)}\right)}{\omega_h^{(k)} \cdot n}}{\sum_k^N \frac{F\left(\omega_o \cdot \omega_h^{(k)}\right) G_1\left(\omega_i^{(k)}\right)\left(\omega_o \cdot \omega_h^{(k)}\right)}{\omega_h^{(k)} \cdot n}} \\
   由于R=N=V \\
   &= \frac{\sum_k^N F\left(\omega_o \cdot \omega_h^{(k)}\right) G_1\left(\omega_i^{(k)}\right) L_i\left(\omega_i^{(k)}\right)}{\sum_k^N F\left(\omega_o \cdot \omega_h^{(k)}\right) G_1\left(\omega_i^{(k)}\right)} \\
   \end{aligned} \\
   在这样的lobe下，\ \omega_o 与 \omega_h的夹角必然小于等于\frac{\pi}{2}（当lobe为hemisphere时为\frac{\pi}{2}），根据Fresnel的函数形状可知，小于\frac{\pi}{2}时其几乎为constant，因此可以消去。 \\
   \begin{aligned}
   L_p(\alpha, v, n) &= \frac{\sum_k^N F\left(\omega_o \cdot \omega_h^{(k)}\right) G_1\left(\omega_i^{(k)}\right) L_i\left(\omega_i^{(k)}\right)}{\sum_k^N F\left(\omega_o \cdot \omega_h^{(k)}\right) G_1\left(\omega_i^{(k)}\right)} \\
   &= \frac{\sum_k^N L\left(\omega_i^{(k)}\right) G_1\left(\omega_i^{(k)}\right)}{\sum_k^N G_1\left(\omega_i^{(k)}\right)}
   \end{aligned}
   $$

   <br />

   关于上式中以D(h)作为PDF推导的$p(\omega_i)$的表达式：

   由于D是关于half vector的函数，而我们的PDF需要的是关于$\omega_i$的函数，而D是PDF需要参与积分，所以我们要算correction factor，即$\frac{d \omega_h}{d \omega_i}$。在推导microfacet时已经介绍过了推导方法，不再赘述。可以得知：

   $$
   d \omega_h=\frac{d \omega_i}{4 \cos \theta_h} \\
   \int_{\mathcal{H}^2} |\omega_h \cdot n| D\left(\omega_h(\omega_i)\right) \frac{d \omega_i}{4 |\omega_h \cdot \omega_o|}=1 \\
   \therefore p(\omega_i) = \frac{|\omega_h \cdot n| D(\omega_h)}{4 |\omega_h \cdot \omega_o|}
   $$

   <br />

   Fresnel的function形状: https://www.desmos.com/calculator/u5unsfrcbe?lang=zh-CN

   <br />

   在unreal engine中，将$G_1$近似为$\omega_i \cdot R$，这种近似带来了很大的速度提升，且根据实际效果对比，发现与直接渲染得到的效果差不多，故沿用这种近似（看起来合理的就是合理的）。

10. 

## 可调用的api
1. image space上的bilinear sample（numpy版本，torch可用grid_sample）。
2. 对hdr image在w维度上进行rotation。
3. cv2读取hdr。
4. 使用torchvision VGG的方式。
5. 获得image bounding box范围的方式。
6. 利用VGG的1-5层activation function结果算loss的方式。


## 犯过的错误
1. 不管是利用np还是torch，最好一直保持数据的dtype为float32。本次出错的地方在于，没有限制每个数据都为float32，导致获得的filtered_envmap为float64，将其存成hdr图像时，由于hdr默认格式为float32，所以不匹配，最终导致存取下来的hdr图像所有pixel值都为0，没有内容。
2. 当需要用到某个term作为pdf时，必须检查其是否大于0，检查其cdf是否为1。在这个project中，犯过的错误为直接用$cos(\theta)$作为pdf，但是其积分为$\pi$。

## 好用的工具
1. desmos：非常简易地画函数图像的工具，可以用这个工具画出函数，然后对其值有一个直观的了解，在这个project中，这个工具用于画出Fresnel-Schlick，发现其位于$\frac{\pi}{4}$内时的值非常小，可以用常数来表示。
   - 网址：https://www.desmos.com/calculator/u5unsfrcbe?lang=zh-CN
2. 