[toc]
# Lecture Notes
## games101 
### lecture 15
1. irradiance是一块面积dA上获得的power($\Phi$)，其目的在于解释某一块面积接收到的能量；radiance是一块面积上往某一个方向角辐射的能量（既可以是朝向这块面积，代表接受能量；也可以是远离这块面积，代表辐射能量），在radiance的基础上加了方向角，又因为方向角与该面积的法线方向有夹角$\theta$，所以有一个cos$\theta$的乘积。（irradiance理解为某一块面积接收到的光的能量，radiance理解为光传播时的某个方向的光能对一块有方向的面积辐射的能量）
   ![irradiance](./images/irradiance-radiance.png)
2. BRDF(bidirectional reflectance distribution function)：代表了某个入射方向对某个出射方向radiance的贡献。
   ![brdf](./images/brdf.png)
   ![equation](./images/reflectance%20equation.png)
   
3. 一块面积上接收到的能量power（也即irradiance）为，每个方向角辐射过来的radiance的积分。
4. 光反射的理解
   1. 光被物体表面弹走，改变方向
   2. 光被物体表面吸收，再以一定的规律将其辐射出去，这就可以用radiometry来解释
5. 直接光照：由光源直接照射到物体反射得到的光照
6. 间接光照：由光源经过一次反射到物体，且反射的光照
7. 全局光照：直接和所有间接光照的总和
8. rendering equation（渲染方程）解释：p代表着某块面积，$\omega$代表方向，$\Omega^+$代表正表面
   ![rendering](./images/rendering_equation.png)
9.  

### lecture 16
1. 蒙特卡罗积分（Monte Carlo Integral）
   ![monte_carlo](./images/monte_carlo.png)
   ![monte_uni](./images/monte_carlo_uni.png)
   ![monte_exam](./images/monte_example.png)
   ![monte_notes](./images/monte_notes.png)
   **notes: Sample on x, integrate on x**
2. 渲染方程中的$\omega$可以对应为一个随机变量，我们要做的就是随机取一个方向，作为蒙特卡罗积分中的随机变量进行积分。
3. Whitted-Style Ray Tracing Problems
    ![assumptions](./images/assumptions.png)
   1. problem 1
        ![problem_1](./images/whitted_problem1.png)
        ways:
            1. introducing Monte Carlo
            ![using_monte_carlo](./images/using_monte_carlo.png)
            ![pseudo_code](./images/pseudo.png)
            2. introducing global illuminance
            ![global_illu](./images/global_illum.png)
            ![code](./images/code.png)
            new problems: ray explosion
            ![explosion](./images/explosion.png)
            new ways: everytime one ray, random sample n times, take the average
            ![mind](./images/mind.png)
            ![new_way](./images/new_way.png)
            ![path_tracing](./images/pace_tracing.png)
            ![average](./images/average.png)
            ![code](./images/code1.png)
        
   2. problem2
        ![problem_2](./images/problem2.png)
        ![no_stop](./images/nostop.png)
        ways(俄罗斯轮盘赌):
        ![Russian](./images/Russian.png)
        ![sample](./images/sample.png)
        ![why_not_efficient](./images/why_not_efficient.png)
        project the light source area to the unit sphere:
        ![projection](./images/projection.png)
        light origin:
        ![light_origin](./images/light_origin.png)
        ![code](./images/code2.png)
        ![improve](./images/improve.png)
        this version is almost right:
        ![right](./images/right.png)
4. ray-tracing concept
   ![concept](./images/concept.png)
5. 俄罗斯轮盘赌：每次弹射都以一定的概率存活，这个规则适用于光线传播过程中的所有弹射。其是为了解决现实中光无限弹射的问题，通过轮盘赌的方式计算出来的光线的期望等于真实世界无限次弹射得到的光线。
6. extensions
   ![extensions](./images/extensions.png)

### lecture 17
1. BRDF=Material(BRDF决定了物体表面的材质，因为它解释了光的反射规则)
2. 漫反射表面不吸收能量，入射的光和出射的光能量相同，即irradiance守恒（L_in=L_out）
   ![diffuse](./images/diffuse_equation.png)
3. albedo理解
   1. reflectance表示一种波长的反射能量与入射能量之比；albedo表示全波段的反射能量与入射能量之比
   2. 非金属表面前提下是漫反射颜色，金属表面前提下是反射率。
   3. albedo：反射率（既可以是单通道的数，也可以是rgb三通道的数），0-1之间，由它可以定义颜色
4. Diffuse/ Lambertian Material：漫反射表面，朝任何方向的反射都是均与的，即能量分布均匀，因此不会有高光、反光特别强的部分，最能反映物体颜色。
   ![diffuse](./images/diffuse_effect.png)
5. Glossy material：不完全是镜子，但是接近镜子（抛光的金属） 
   ![glossy](./images/glossy_material.png)
6. Ideal reflective/refractive material (BSDF*)：水和玻璃(反射BRDF+折射BTDF=散射BSDF)
   ![ideal](./images/ideal.png)
7. Perfect specular reflection
   ![specular](./images/specular_reflection.png)
8. Specular refractive
9.  Fresnel Reflection/Term（菲涅尔项）：用来反映光线射到表面后发生折射和反射的能量的分布

   下面这张图片的意思是：以地板作为反射表面，角度与地板平行时看到的光更多，即反射更多
   ![Fresnel](./images/Fresnel.png)
   ![map](./images/fresnel_map.png)
   ![precise](./images/precise.png)
11. Microfacet Theory（微表面模型）：
    ![microfacet](./images/microfacet.png)
    1. 假设其为粗糙的表面
    2. PBR（physically-based rendering一定会用到microfacet theory）
    3. Macroscale（从远处看）：flat & rough（看不到凹凸不平的表面，但是是粗糙的）(从远处看到的是材质、外观)
    4. Microscale（从近处看）：bumpy & specular（可以看到凹凸不平的表面，每一个表面都是一个镜面）（从近处看看到的是几何）
    5. Microfacet BRDF
       ![brdf](./images/microfacet_brdf.png)
       1. 若所有表面的法线基本上围绕在宏观表面法线周围，比较集中，则认为是glossy表面
       2. 若所有表面的法线比较分散，则认为是diffuse表面
    6. Formula
         ![formula](./images/microfacet_brdf_formula.png))
         1. h：考虑某一个微表面能否把某一个入射方向反射到我们想要的出射方向上去，就是看固定的入射和出射方向的half vector（即h）是否与这个微表面的法线方向相同，若相同，则这个微表面能够实现我们想要的入射和出射，否则不能
         2. F(i,h)：考虑了根据不同的入射方向（与法线夹角不同）会反射不同能量的光线（**修正入射、出射方向**）
         3. D(h)（NDF: Normal Distribution Function）：表示法线分布，反映了有多少表面的法线分布是与half vector一致的（half vector是入射方向和出射方向的中间方向）(可以这么理解：一块微表面由无数块镜面组成，给定入射方向和想要的出射方向（view direction），只有那些法线方向和half vector相同的镜面才可以将入射方向的光线反射到我们想要的出射方向上去，而D（h）正是反映在这块微表面上有多少的小镜面的法线方向与给定的出入射方向的half vector方向相同。当这样的小镜面越多的时候，代表微表面可以往view direction反射的能量越大，否则越小)(**修正法线分布**)
         4. G(i,o,h)：不同的微表面之间可能会互相遮挡光，导致有些微表面反射的光被阻挡，到不了我们想要的出射方向上去。当光线几乎平着打到表面上（grazing angle）时，容易发生shadowing-masking（**修正grazing angle**）
    7. Isotropic & Anisotropic Materials（另一种区分材质的方式）
         ![isotropic](./images/isotropic.png)
         1. Isotropic：各向同性，所有表面的法线分布在不同方向上比较均匀，没有哪个方向的法线较多或较少
            反映在BRDF上就是，如果同时旋转入射方向和出射方向，看到的是相同的BRDF那就是各向同性
         2. Anisotropic：各向异性，几乎所有表面的法线分布在一个方向上
            反映在BRDF上就是，如果同时旋转入射和出射方向，看到的是不同的BRDF
         $\theta$表示入射方向与表面的夹角，$\phi$用于表示当入射方向与表面的夹角固定时（相交点在一个圆上），入射光线为哪一根（即从top-down view上看到的角度）
         ![brdf](./images/isotropic_brdf.png)
    8. properties of BRDF
         1. 非负
            ![property1](./images/property1.png)
         2. 线性（可以用不同方向的BRDF计算Radiance然后相加来计算）
         3. 可逆性
            ![property2](./images/property2.png)
         4. 能量守恒（出射的能量必定小于等于入射）
         5. 各向同性可以降维（只需要考虑方位角的差值）
            ![dimension](./images/dimension.png)
    9. Measuring BRDFs
         1. theoretically: 拿光源以不同的方向照射到一个点，同时拿相机在每个光源方向一定的前提下，拍摄不同方向的图像
            ![measure](./images/measure_brdf.png)
         2. actually：基于理论，由仪器实现
         3. improving efficiency：
            1. 若各向同性，只需要测量某个$\theta$即可知道所有相同$\theta$不同$\phi$的BRDF
            2. 若考虑可逆性，则可以省略一半的角度，因为某块区域都有成对的角度其含有相同的BRDF
            3. 
    10. 

### lecture 18
1. Biased & Unbiased Monte Carlo Estimators
   1. Unbiased即为无偏的，即估计方法的期望就是正确值
   2. biased即为有偏的，其期望可以通过无限采样逼近正确值
2. 双向路径追踪（Bidirectional Path Tracing（BDPT））
   ![BDPF](./images/BDPT.png)
3. Metropolis（人名） Light Transport（MLT）
   ![Metroplis](./images/Metropolis.png)
   ![comparison](./images/Metroplis_comparison.png)
   1. 采用 Markov Chain Monte Carlo（MCMC），即马尔可夫链
      1. 之前的Monte Carlo使用的是均匀采样，马尔可夫Monte Carlo使用的是非均匀采样（可以使用任意形状的PDF）   
   2. 给定一个路径的前提下，可以在周围产生更多的相似路径，即新样本（局部性）
   3. 越难的场景产生的效果越好
   4. 缺点
      1. 难以衡量收敛的程度
      2. 难以保证每个像素收敛到相同的比率
      3. 由于局部性，每个像素自己进行收敛，像素之间的相关性不高，会产生噪声
      4. 因此，通常不用于渲染动画（因为不同帧之间相同像素的收敛程度不一样）
      ![disadvantage](./images/disadvantage.png)
4. Photon Mapping
   ![photon_mapping](./images/photon_mapping.png)
   1. caustics：因为光线聚焦产生的非常强的图案
   2. step
      1. stage 1
      2. stage 2
      3. calculation
   3. An easier understanding bias in rendering
      biased == blurry
      consistent == not bluuy with infinite samples
   4. 
5. 