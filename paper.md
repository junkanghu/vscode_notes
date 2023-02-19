[toc]
# Notes for papers

## IDR：Multiview Neural Surface Reconstruction with Implicit Lighting and Material

### innovation
1. 之前的工作没考虑光照和surface material对appearance的影响，也没有考虑优化camera locations和orientations。此工作可以同时进行。
2. 以一个continuous implicit neural network来建模BRDF、lighting、geometry（造成shadow-secondary lighting）综合作用下产生的surface appearance，即rgb。

### methodology
1. ray-tracing到surface point的depth。
2. 对surface point进行reparameterization。
3. 计算rgb

### details
1. IGR（Implicit Geometry Regularization），即eikon loss，能够使得surface更加smooth和realistic。
2. 这里将某个camera location c、camera view direction v下通过sdf $S_{\Theta}$看到的surface point的depth t，作为上面这三个参数的函数，即$t(c,v,\Theta)$。而c、v、$\Theta$都是continuous的，因此t在整个定义域上是continuous且differentiable的。因此对point x（$x=c+tv$）算它们三个的梯度时，能传到$\Theta$的梯度只能经由t传到。
3. 对c、v、$\Theta$的优化可以这样来理解：随着优化进行，当前pixel产生的ray的出发点（camera center）和方向都会逼近真实的camera pose，在不断的优化下，camera pose逐渐接近真实或者说可以认为已经优化完成，此时需要更注意sdf的优化，通过对x（或者说t）在当前view direction方向上进行移动，可以讲surface point在view上进行移动，以让其更接近真实的shape。
4. 对x进行reparameterization后，我们可以发现对某个固定的c和v（即假设不优化c和v，或c和v的优化已经收敛时），在固定的c和v上的更新后的x的下个参数，只会在view direction上变动。

---

## Physg: Inverse Rendering with Spherical Gaussians for Physics-based Material Editing and Relighting

### problems in previous works 
1. Neural rendering methods work well for the task of interpolating novel views, but do not factorize appearance into lighting and materials, precluding physically-based appearance manipulation like material editing or relighting.
2. 

### innovation


### limitation
1. Don't model self-occlusion or indirect illumination.
2. 

### overview
1. MLP for SDF function.
2. SGs for environment map which represents the environment light.
3. BRDF consists of spatially varying diffuse albedo and a shared monochrome isotropic specular component. The diffuse albedo $a$ is calculated by a MLP $\Phi$.
   That is:
   $$
   f_r(\omega_o, \omega_i;x) = \frac{a}{\pi} + f_s(\omega_o, \omega_i;x)
   $$
   where:
   $$
   f_s(\omega_o, \omega_i;x) = M(\omega_o, \omega_i)D(h)
   $$
   In the BRDF euqtion, $f_s$ is represented by SGs. Besides, $\omega_i\cdot n$ is represented by a SG.
4. 



## Modeling Indirect Illumination for Inverse Rendering


### some conceptions
1. ray-sphere intersection
   ![explainment](./images/sphere_intersection.png)
   Here, the center of the sphere is at the origin of world coordinate. So, $Q=P$.
   ``` python
    cam_loc = cam_loc.unsqueeze(-1)
    # directions are world coordinates
    ray_cam_dot = torch.bmm(ray_directions, cam_loc).squeeze()
    under_sqrt = ray_cam_dot ** 2 - (cam_loc.norm(2,1) ** 2 - r ** 2)

    # Here the center of the sphere is the origin of the world coordinate
    under_sqrt = under_sqrt.reshape(-1)
    mask_intersect = under_sqrt > 0  # These ones greater than 0 can intersect sphere.

    sphere_intersections = torch.zeros(n_imgs * n_pix, 2).cuda().float()
    sphere_intersections[mask_intersect] = \
        torch.sqrt(under_sqrt[mask_intersect]).unsqueeze(-1) * torch.Tensor([-1, 1]).cuda().float()
    sphere_intersections[mask_intersect] -= ray_cam_dot.reshape(-1)[mask_intersect].unsqueeze(-1)
   ```
   In the code, *under_sqrt* is $\frac{b^2}{4} - ac$. To calculate for t, we have
   $$
   t = \frac{-b\pm \sqrt \Delta}{2a}
   $$
   We can know from the code that ```torch.sqrt(under_sqrt[mask_intersect]) * torch.Tensor([-1, 1])``` gets $\pm \Delta$. So by addition, we get the two ts.
2. 

### problems in previous work
1. Previous capture systems, such as light-stages with controlled light directions and cameras [8, 11, 31], using a colocated flashlight and camera in a dark room [2, 3], and rotating objects with a turntable [7, 26], **show limitations in user-friendliness**.
2. Methods capturing objects under natural illumination often show complex effects such as soft shadows and interreflections. Prior methods usually ignore both selfocclusion and interreflection [29] in order to reduce computation, or only model visibility [32] or limit the indirect lighting to a single bounce with known light sources. As a result, the effect of indirect illumination in the captured images is prone to being baked into the estimated diffuse albedo to compensate for this gap.
3. Single-image inverse rendering methods rely heavily on the strong prior of the planar geometry. They can effectively infer plausible materials and normal maps from a single image but usually cannot recover **spatially-varying 3D representations** of these factors.

### limitations
1. Strongly relies on fine geometry as an input. We cannot deal with the case where the geometry fails to be reconstructed.
2. We parameterize BRDF with fixed F0 = 0.02 in the Fresnel term. In other words, we assume that the recovered materials are dielectric.



## IRON: Inverse Rendering by Optimizing Neural SDFs and Materials from Photometric Images

### innovation
1. Compared to IDR、Nerf、Neus etc. that entangle lighting and meterial, this work can disentangle them.
2. Previous works don't consider the derivative of edge points, leading to inability to deform shape along the edge normals.
   
![edge](./images/edge_inalbility.png)

In this figure, under the single viewpoint condition, algorithms without computing edge derivative along normal directions are not able to defrom the shape along normal directions.（也就是在初始化的sdf下，没办法把shape往normal的方向拉，因为reparamterization时只考虑沿着ray direction的方向去优化）
3. Be able to rendering sub-pixels.

### methodology
1. 首先用IDR的方式训练一个sdf net和一个neural radiance field net。
2. 然后同时训练albedo（第一步的neural radiance field作为初始化）、roughness、specular、sdf、point light，并以GGX model进行rendering。

### details
1. 作者使用co-located camera和light（point light）拍摄，并在计算时将point light的位置近似和camera center相同，这样使得view direction和light direction相同（差个符号）。这种近似在object较远的时候误差会较小。
![co-located](./images/co-located.png)
2. 作者使用一个点光源来近似light，并采用inverse-square fall-off对光源的强度进行建模。
3. 当view direction与surface normal垂直时，代表我们看到的是object的edge。
4. 比较重要的一个关于梯度计算的点：我们计算loss时本质上是要从loss back-propagate 到net weights的，但是在这里因为x的获得不是differential的，所以梯度没办法通过x back-propagate到network的weights中，因此需要reparameterize x。本质上来说要reparameter x，需要保证构造的equation在当前参数下的值($S_{\Theta}, n_0$)等于$x_0$，且当前参数下的梯度等于未构建实体equation而通过implicit function计算时的first-order梯度（因为算weight gradient时用的都是first-order），因此后续才有构造一个equation证明这两点的推导。
5. 只对edge-pixels进行walk process来找edge surface points，而找这种edge pixels，是通过ray-tracing得到每一个pixel的depth之后，给每个pixel附上这个depth值，然后算Sobel 梯度，梯度较大的pixel说明其与周围的pixel之间存在depth突变，则其为edge pixels。
6. Figure 2、4想要告诉我们的点时，在一个view point上进行训练时，IDR由于只计算演着view direction方向的t变化，导致shape只会在view direction方向被伸缩，而没办法往edge normal的方向伸缩，因此无法将一个初始化的sdf shape在edge normal方向慢慢通过优化拉伸变大。但事实上在multi-view训练时，由于每个viewpoint都几乎会被扫到，所以整个shape可以朝向各个方向被伸缩，最终达到准确的shape。


### limitation


### some conceptions
1. **edge pixel & interior pixel & subpixel**
   * **edge pixel**: where shading colors is a combination if colors at disconnected surface pieces.

    ![edge_pixels](./images/edge_pixels.jpg)

   * **interior pixel**: pixels whose content comes from smooth and continuous surface

   * **subpixel**: a small area inside a pixel
2. photometric images: camera and light is co-located.
3. 可以讲3D normal投影到2D images形成2D normal map。


### tricks


### mathematical derivation
1. walk step
![walk](./images/IRON_algo.jpg)
2. 

---

## NeuS: Learning Neural Implicit Surfaces by Volume Rendering for Multi-view Reconstruction

### advantages and limitations in previous works
* IDR:
  1. fail to reconstruct objects with complex structures that causes abrupt depth changes: The cause of this limitation is that the surface rendering method used in IDR only considers a single surface intersection point for each ray. Consequently, the gradient only exists at this single point, which is too local for effective back propagation and would get optimization stuck in a poor local minimum when there are abrupt changes of depth on images
  ---
  illustration of surface rendering and volume rendering:
  ![rendering_methods](./images/rendering_methods.jpg)

  (with the radical depth change caused by the hole, the neural network would incorrectly predict the points near the front surface to be blue)

  ---
  examples of idr limitation near the edges with abrupt depth changes: 
  ![idr_abrupt_changes](./images/idr_limitation.jpg)

  ---
  2. object masks are needed as supervision for converging to a valid surface

---

* traditional multi-view 3D reconstruction methods(surface and volumetric methods)
  1. surface methods(point-(sparse) and surface-based(dense) methods)
     * pipeline: sparse reconstruction -> dense reconstruction
     * limitations: heavily relies on the quality of correspondence matching. and the difficulties in
     matching correspondence for objects without rich textures often lead to severe artifacts and missing parts in the reconstruction results
  2. volumetric methods
     * pipeline: estimating occupancy and color in a voxel grid from multi-view images and evaluating the color consistency of each voxel

  limitations for traditional methods: Due to limited achievable voxel resolution, these methods cannot achieve high accuracy

---

* neural implicit representation
  * surface rendering-based methods
    * assume that the color of ray only relies on the color of an intersection of the ray with the scene geometry, which makes the gradient only backpropagated to a local region near the intersection
    * limitations: struggle with reconstructing complex objects with severe self-occlusions and sudden depth changes
  * volume rendering-based methods
    * render an image by α-compositing colors of the sampled points along each ray
    * advantages: can handle abrupt depth changes, because it considers multiple points along the ray and so all the sample points, either near the surface or on the far surface, produce gradient signals for back propagation.
    * limitations: since it is intended for novel view synthesis rather than surface reconstruction, NeRF only learns a volume density field, from which it is difficult to extract a high-quality surface

---
### innovation

---
### limitation

---
### some conceptions
* gradient$\nabla$
  * assuming $f(x,y,z)$ has a first-order continuous partial derivative in area $G$, for every $P_0(x_0,y_0,z_0)\in G$, there exist a vertor
  <br>
  $$
  f_x(x_0,y_0,z_0)\vec{i}+f_y(x_0,y_0,z_0)\vec{j}+f_z(x_0,y_0,z_0)\vec{k}
  $$
  which is the gradient of $f(x,y,z)$ at $P_0(x_0,y_0,z_0)$, it is denoted as **grad** $f(x_0,y_0,z_0)$ or $\nabla f(x_0,y_0,z_0)$
  <br>
  so
  **grad**$f(x_0,y_0,z_0)$ = $\nabla f(x_0,y_0,z_0)$
  &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;=$f_x(x_0,y_0,z_0)\vec{i}+f_y(x_0,y_0,z_0)\vec{j}+f_z(x_0,y_0,z_0)\vec{k}$
  <br>
  therefore:
  $\nabla f$ is such a vector that its direction is the direction that f changes the fastest.
  <br>
  if we introduce a surface $f(x,y,z)=c$ as a level-set, we can know that direction of the gradient $\nabla f(x_0,y_0,z_0)$ at point $(x_0,y_0,z_0)$ of $f(x,y,z)$ is the direction of the normal of $f(x,y,z)=c$ at this point.

* the Eikonal term loss:
  When optimizing the loss using the loss functions, two questions emerge:
  1. Why the parameters$\theta$ of the MLP found by the optimization lead MLP to be a signed distance function. Usually, adding a quadratic penalty with a finite weight is not guaranteed to provide feasible critical solutions, i.e., solutions that satisfy the desired constraint itself.
  2. Even if the critical solution found is a signed distance function, why should it be a plausible one? There is an infinite number if signed distance functions vanishing on arbitrary discrete sets of points $\xi$ with arbitrary normal directions.

  It is proved that this loss function used in gradient descent is effective for getting a smooth and plausible one.

* Why perform upsamping using different s in the code?
![s](./images/s_prin.jpg)
* 

---
### tricks
1. cosine decay schedule--learning rate:
   * definination: 
      * *self.iter_step*: current step
      * *self.warm_up_end*: warmming-up iter
      * *self.end_iter*: the endding iter
      * *alpha*: the final lr
   * usage
      * for **$iter < warm\_up\_end$**, lr increases linearly from 0 to *self.learning_rate*
      * for **$warm\_up\_end < iter < end\_iter $**, lr decreases from *self.learning_rate* to *alpha* following the cosine decay schedule

   ``` python
   if self.iter_step < self.warm_up_end:
            learning_factor = self.iter_step / self.warm_up_end
        else:
            alpha = self.learning_rate_alpha
            progress = (self.iter_step - self.warm_up_end) / (self.end_iter - self.warm_up_end)
            learning_factor = (np.cos(np.pi * progress) + 1.0) * 0.5 * (1 - alpha) + alpha

        for g in self.optimizer.param_groups:
            g['lr'] = self.learning_rate * learning_factor
   ```
2. about number processing
   * number should deviate from its theoretical one:
      ``` python
      z_vals_outside = torch.linspace(1e-3, 1.0 - 1.0 / (self.n_outside + 1.0), self.n_outside)
      ```
      which takes numbers between 1e-3 and 1.0-... rather than 0 and 1
   ---
   * when a number serves as a denominator, it should add a small number(usually **1e-5**) to avoid cases like divided by 0 or numbers almost approximates 0.

---
### problems
1. Note that the standard deviation of $\phi_s(x)$ is given by $\frac{1}{s}$, which is also a trainable parameter, that is, $\frac{1}{s}$ approaches to zero as the network training converges.
   solution: As the training goes, the ideal situation is that the weight function is very steek which means the standard deviation is very small.
2. figure 3: visible and invisible surface
3. why this way can solve the problems caused by abrupt depth change and occlusion and why nerf can handle abrupt depth problems
4. why $\alpha$ takes the *max* operation? According to the feature of $\rho(t)$, $\alpha$ naturally obeys this rule.
5. mask loss
6. renderer.py line 230: dirs are unit vectors, but gradients may be not though the gradient loss forces them to be like unit vectores
7. cos anneal ratio at renderer.py line 230

---

### mathematical derivation
* $$
  \begin{aligned}
  \omega(t) &= \displaystyle\frac{\phi_s(f(p(t)))}{\int_{-\infty}^{+\infty}\phi_s(f(p(u))){\rm d}u}\\
  &= \displaystyle\frac{\phi_s(f(p(t)))}{\int_{-\infty}^{+\infty}\phi_s(-|{\rm cos}\theta|\cdot(u-t^*)){\rm d}u}\\
  &= \displaystyle\frac{\phi_s(f(p(t)))}{-|{\rm cos}\theta|^{-1}\int_{-\infty}^{+\infty}\phi_s(-|{\rm cos}\theta|\cdot(u-t^*)){\rm d}(-|{\rm cos}\theta|\cdot(u-t^*))}\\
  &= \displaystyle\frac{\phi_s(f(p(t)))}{-|{\rm cos}\theta|\cdot\int_{-\infty}^{+\infty}{\rm d}\Phi(-|{\rm cos}\theta|\cdot(u-t^*))}\\
  &= \displaystyle\frac{\phi_s(f(p(t)))}{-|{\rm cos}\theta|\cdot\int_{+\infty}^{-\infty}{\rm d}\Phi(v)}\\
  &= \displaystyle\frac{\phi_s(f(p(t)))}{-|{\rm cos}\theta|\cdot(-1)\cdot1}\\
  &= |{\rm cos}\theta|\phi_s(f(p(t)))
  \end{aligned}
  $$

* consider this naive solution
  $$
  \omega(t) = \displaystyle\frac{\phi_s(f(p(t)))}{\int_{-\infty}^{+\infty}\phi_s(f(p(u))){\rm d}u}
  $$
  the reason why this $\omega (t)$ is not occusion-aware is that when this one is disentangled with $T(t)$ and $\rho (t)$, $\rho (t)$ may be negtive in certain segment of the ray, thus the $T(t) = e^{-\int_0^{t}\rho(u){\rm d}u}$ will increase and at the second surface intersection, $T(t)$ becomes equal to the first intersection. And the $\rho(t)$s at these two intersections are equal, thus the two $\omega(t)$s equal. After constrain the $\rho(t)$ as ${\rm max}(\cdots, 0)$, $T(t)$ won't increase anymore but keep decreasing, so although $\rho(t)$s at these two intersections are equal, their $\omega(t)$s don't equal anymore.



## NeRV: Neural Reflectance and Visibility Fields for Relighting and View Synthesis

### innovation
1. 相对于NeRF，解决了NeRF不能恢复material和进行relighting的缺点。
2. 相对于一些传统方法，又可以利用NeRF重建3D model。
3. 虽然NeRF in the wild采用appearance code来表示每一张image的lighting可以进行某种程度上的relighting，但是它不能利用一些新的lighting而只能用学到的lighting。

### Methodology
前提：global illumination中，只衡量一次弹射。
定义：三个MLP
   1. 一个MLP输出每个point x的volume density $\sigma$，输入为x的坐标。（这里将volume density作为shape的表示，其实也可以理解，因为density越大的地方代表着有物体，空气的density小）
   2. 一个MLP输出每个point x的diffuse albedo（这里默认每个point没有specular分量来简化）和roughness。（遵循microfacet模型），输入为x的坐标。
   3. 一个MLP输出在某个点沿着某个方向的visibility（衡量能不能看到light source，看得到light source时可利用这个MLP输出的visibility来计算radiance，看不到时认为是打到了物体上，被物体遮挡，此时可使用这个MLP的termination depth）和termination depth（从这个点到termination的距离）。这里的visibility其实就是NeRF中的transmittance（即volume density的积分），在NeRF中认为某个点的transmittance越低，其radiance（rgb）传输到camera的比例越少。
方法：
   1. 传到某一个camera pixel的radiance（rgb）由两部分组成，一个是direct illumination，一个是indirect illumination。
   2. 对某个pixel的渲染还是采用NeRF在ray上sample点的方式。不同的是，在NeRF中通过MLP获得每个点的rgb值，即认为每个点自己含有能量，能够emit radiance；但是在NeRV中，认为每个点本身不含能量，它们的能量来源于别的光线，即每个点朝向pixel发射的radiance来源于反射（遵循PBR）。
   3. 对于direct分量来说：先算出ray上每个sample的点的reflectance作为NeRF中的rgb值，然后根据MLP得出每个点的volume density，然后根据NeRF中的方式获得ray的rgb值。每个点的reflectance通过PBR获得，incident light由light source直接产生（在sample点的hemisphere上采样入射光方向），然后将这个incident radiance与visibility（MLP获得）相乘作为最终到达sample点的radiance；diffuse albedo和roughness由MLP获得，然后根据microfacet模型产生BRDF；点的normal通过对density的MLP进行求导获得（方式与SDF net一样。可以这么理解这个normal的计算，在sdf中sdf=0代表着某个surface，而对ƒ求导代表着surface的normal；在这里density=m代表着某个surface，故用其对coordinate求导也代表着normal）；最终对采样的入射光方向积分获得最终的朝向pixel的reflectance。
   4. 对于每个indirect分量来说：不再对ray上的每个sample点计算refletance，然后根据volume rendering获得rgb值。而是认为每条光线最终会打到一个hard surface上产生一个intersection（它有一个depth），然后对这个intersection进行PBR产生一个reflectance（incident light不来源于light source而是来源于被light source照射到的点，因此light到intersection时已经是第二次反射，这才成为indirect），这个reflectance就类似于surface rendering的rgb值。在这里，这个depth遵循NeRF中计算depth的方式（仅将volume rendering中的rgb值换成每个sample点的depth值，rgb权重大的点，其depth的权重也大）。而有了depth后，就可以计算这个intersection的坐标。得到坐标后，在这个intersection的hemisphere上采样m条光线，将intersection坐标和m条光线的方向输入visibility MLP来计算每条光线的termination depth $t''$，根据这个$t''$和m条光线方向可以计算出m个termination的坐标。这m个termination都可能获得light source的radiance（由MLP产生的visibility来衡量是否能接收到light source的radiance），然后再根据PBR产生m个reflectance作为intersection的入射光线，然后在intersection处做PBR，输出的reflectance作为pixel的indirect分量。



## NeRF in the Wild: Neural Radiance Fields for Unconstrained Photo Collections

### innovation
 NeRF短板：NeRF认为不同的image中，从同一个方向观察同一个点会得到相同的rgb值(因为NeRF适用的场景都在一些室内的短时间场景，其间光照条件什么的比较恒定)。但是这不符合outdoor photography呈现出来的效果（1、室外的环境时刻在变，即光照、空气等时刻在变，这会导致射入camera的radiance时刻在变。且由于radiance在变，相机内部也会动态调整exposure、white balance、tone-mapping等，这使得该效应被加剧。 2、现有的一些数据集都是在无人的地方拍摄的，镜头中几乎不会有正在动的object（如transient occluders），但是室外环境中拍摄的image可能包括行人，他们的出现会改变拍摄的一组图像中不同图像对同一方向同一个点的观察）。因此当NeRF用于移动的物体或多样的光照时，其效果大打折扣。这使得NeRF无法用于大规模的野外照片集（光照不同：照片之间的跨度往往以小时甚至年来算；包含行人或车子）。

 改进：
 1. 利用apperance code来建模不同的光照环境，以解决NeRF需要恒定光照的限制。
 2. 利用transient head和uncertainty来建模transient occluders，建模后可以直接移除transient分量以获得static分量。

### methodology
![nerf-w](./images/nerf-w.png)

1. 最终输出的rgb分为两个分量：static head、transient head。
2. static：在NeRF输出$\sigma$和$z(t)$（feature vector）后，加入一个latent apperance code来建模每张图片单独的光照条件（每张照片都有一个appearance code，trainable，意味着只有训练数据才会有code。在test时，可对不同的code进行插值来获得新的code，或者直接随机一个code，实验证明这只会改变光照条件，不会改变geometry）。
3. transient：如上图一样在每个sample点处再生成一个transient的量，其volume rendering同static分量相同。最终的rgb值由static分量和transient分量线性叠加。
4. uncertainty：$\beta$是由MLP产生的。本方法对uncertainty的建模认为，每个pixel的颜色值遵循正态分布，每个pixel在当前训练阶段预测的颜色值是正态分布的均值，然后这个pixel的variance值就是$\beta^2$。variance值越大代表当前pixel的值不太稳定，其是不可靠的pixel，在计算loss时应该被排除在外。一条ray上的每个sample点都有一个$\beta_i(t)$，然后对一条ray进行alpha compositing才能获得这条ray的variance $\beta_i$。
5. loss
![total](./images/total_loss.png)
![loss](./images/loss.png)
   1. 本文采用NeRF的coarse-to-fine策略，total loss一共有两部分loss，一个是coarse，一个是fine。
   2. coarse部分只采用appearance code（static和transient都有），没采用uncertainty。
   3. fine部分的loss中，第一项代表着variance$\beta$越大，这个pixel越不可靠，因此其权重应该越小，对loss的影响越小；第二项是对第一项的惩罚loss，用于减小$\beta$值，防止所有pixel的$\beta$值都很大，使网络认为所有pixel都不可靠而学不到东西。第三项是对transient density的惩罚，防止transient density过大，抹杀了static density的作用（算法本身想获得static scene）。

### thinking
1. 为什么算transmittance时static和transient的density可以相加？
如果只有static，那么空间中的object的位置是固定的，每个location的density也是固定的。而有了transient后，其在原来的static基础上相当于在scene中的某个location放了新的object，因此除了新object所在的位置，其他的density保持与static时一样，而新object位置的density由于object的存在而变大了。因此在算transmittance时，要把static和transient各自的density相加，理想的情况是static density就是代表没有transient object时的density，而transient density除了有新object的地方，其他地方的density几乎为0（没有在这些地方加入新的particle），而有新object地方的density在static density中几乎为0，加上了transient density后总量变大了。这就造成scene中，如果有transient遮挡了static部分，由于新obejct处的density很大，计算transmittance时，新object后面的sample point的transmittance会非常小，即只有transient处的sample point对ray rgb有贡献，而后面的static object没有贡献，这就可以解释新object遮挡了static scene中的内容。
2. 为什么uncertainty能够work？
文中说uncertainty的作用是减少含有transient的pixel对于loss的贡献值，以使其梯度减小，这使得不含有transient的表示static scene的pixel的贡献较大。首先因为transient是image-dependent的，即每张image张除了static scene的内容是不变的（只是改变了view point），transient的量有可能在每张image中都不相同，因此为了model这种diversity，给每张image一个transient code（在static part中也有一个code，其是appearance code）以model当前image中的transient。这个transient code使得transient Nerf能够随着其变化（对于当前image，相当于对当前image中的transient object建了一个Nerf；对另一张image，对其transient object建了另一个Nerf。这两个Nerf并不是通过建立两组MLPs来实现，而是通过1组MLP配合不同的code实现，输入不同的code代表不同组MLP之间的切换）。

<br />

transient ray上的每个sample point的$\beta_t$的大小应该是与transient density一致的，即transient density越大的点其$\beta$越大。因此一条ray上属于static部分的point的uncertainty应该趋向于0（因为其是static的，方差不大，即color几乎不会变，但是transient由于occluder会变，因此方差较大），而属于transient部分的point的uncertainty较大，因此在alpha compositing时，属于transient部分的point的贡献较大。这个pixel的uncertainty应该就是为了explicitly指示出哪些pixel含有transient分量，由于我们的最终目的是获得static部分的density和color，含有transient分量的pixel，其对于static部分的density和color学习几乎起不到什么作用，因为它看不到static部分，因此这些pixel从理想上来说应该不用作算loss，用这个uncertainty可以将这些pixel的相对loss weight调低，从而降低对网络参数的控制能力。这个transient head不仅能够将transient和static分离开，在将它们分离开来后，还能够控制loss的weight，使得static部分优化得更好。

---

## HDR-Plenoxels: Self-Calibrating High Dynamic Range Radiance Fields

### innovation
1. 前人的方法都是通过固定相机并调整曝光度以获得multi-exposure images，然后根据这些multi-exposure LDR images去合成HDR images。这种方法局限于view point，不能改变相机位置。本文提出对HDR radiance进行重建，可以在任何视点获得LDR images。
2. 本文可以通过控制white balance、exposure来获得不同效果的image。


### methodology
前提：不用mlp；不用successive的空间表达，而是用voxel。
方法：
1. 整个bounded space在训练初始阶段被划分成均匀的voxels。每个voxel的八个顶点分别存有28维的参数，其中1维代表顶点的volume density值，27维中的3个9维参数分别对应rgb中每个通道的SH coefficients。在voxel中的任一点的上述28维参数，可以通过对8个顶点的值进行trilinear插值获得。插值完后可以得到该点的28维参数。然后可以根据SH basis function，把27维参数输入其中得到rgb值（SH basis function中，可以输入相机中心到该点的方向即view direction来获得该点不同方向的颜色）。
2. 建HDR radiance时同nerf一样，从每个pixel发出一条光线，在光线上sample点，然后根据每个点所处的voxel进行插值（bounded space被voxel划分，同时也可以有连续的3d坐标，它们只是尺度不同，判断point位于哪个voxel是通过，将3d坐标（例如坐标位于0-1）值scale到voxel的坐标值来判断位于哪个voxel。例如voxel的shape为[256,256,256]，坐标值为(0.1, 0.2, 0.3)，那么将坐标值乘上256即可获得其位于哪一个voxel里），获得每个点的rgb值和density值，然后根据nerf的volume rendering来获得该ray（pixel）的$I_H$值。随着训练的进行，不含内容的voxel（所有顶点的28维参数均为0，且该voxel周围的所有voxel也满足参数都为0）会被删除，含内容的voxel会被进一步划分成更小的voxel来提高精度。实际的数据结构通过存储含有内容的voxel的indices来query voxel。
3. 得到$I_H$后，其会进入某个pose的相机，然后相机进行一系列加工将其转换为$I_L$。首先对rgb值进行白平衡，其次将白平衡后的rgb值去query CRF获得最终的LDR rgb值。

策略：
1. 因为white balance和exposure都是直接manipulate pixel intensity，所以在这里会有歧义，增大曝光时间和改变白平衡系数可能对pixel intensity的改变是一样的。，因此将它们的效果都归到white balance的3-channel系数中。作者发现在这样的策略下，white balance系数可能会被训练得非常大或者非常小。这一块的笔记做在了文中。
2. 除了exposure和white balance之间有歧义，white balance和SH coefficients之间也会有歧义。这一块内容笔记也做在paper中。
3. 在输入数据极其不稳定的情况下，在上述两种策略的加持下，训练的前期可能还是稳定的，但是后期当白平衡系数更新的速度与SH coefficients更新的速度不一致时，可能又会发生上述问题。解决方法的笔记在文中。
4. 过饱和区域、欠饱和区域的pixels的loss权重应该尽可能小，避免网络学习过饱和、欠饱和的模式。文中认为0.15-0.9的dynamic range不属于过饱和、欠饱和。

loss：
1. reconstruction loss中的saturation mask就是控制过饱和、欠饱和区域的loss权重。
2. CRF的smooth loss其实就是对256个控制点中的每个点算二阶离散导数($f(x+1)+f(x-1)-2f(x)$即为x点的二阶导数)，求其MSE并最小化。这使得CRF更加平滑，插值出来的最后的rgb值也会更加平滑（实际上位于两个控制点中间的部分都是连续的，因为是线性函数插值获得的。求二阶导数使其趋向于0使得分段函数的分段点更加平滑，即分段点两侧单侧一阶导数趋向相等，可导->连续）。
3. TV loss也是一种全局平滑，这使得相邻的density和SH coefficients更加连续，以保证空间geometry和颜色的一致性。

---

## NERF++: ANALYZING AND IMPROVINGNEURAL RADIANCE FIELDS

### innovation
1. NeRF在训练时，对synthetic dataset界定bounding depth以在这个depth range中取t值，对real dataset使用NDC将0->无穷的depth变换到0-1来sample当成0-无穷。这样的sample方式可能会不够精确，实际的foregroud内容可能只占了depth range的一小部分。本方法利用一个bounding sphere将foregroud和backgroud分开。分别采用不同的坐标来表示内外sample点的坐标，分别对内外sample点用2个MLP来获取rgb值和density值，进行volume rendering。

### methodology
精华部分都在paper section 4，笔记都做在paper中。

---

## Nerfies: Deformable Neural Radiance Fields

### innovation
1. Free capture system(hand-held monocular cameras: 1. Solve the problem of nonrigidity, i.e., cannot keep still. 2. Solve the problem of priors for previous work, here there is no priors, just regularization)
   1. Use elastic regularization to keep rigidity and allow existing nonrigidity like face.
   2. Use background regulatization to align object location.
2. 

### methodology
1. 利用canonical coordinate去建模一个template volume，其中包含static scene。
2. 然后为每一帧建一个observation coordinate和一个trainable deformation code，渲染方式与Nerf相同，唯一的区别就是sample的点需要同deformation code一起query deformation MLP来获得其在template volume中的坐标（即其相对于起初的static scene有了哪些变化）。利用mapping后的point去query Nerf MLP（含appearance code）获得rgb and density。

### detail
1. 通过hand-held的方式采集数据，但是因为人是动的，所以无法用colmap对人上的feature进行reconstruction。这里采用foreground mask，截掉foreground部分，仅利用background（static）部分的feature进行colmap reconstruction。
2. Elastic部分：
   1. deformation mlp将observation frame下的一个point X mapping到canonical frame下的一个point Y，这种映射由mlp隐式实现，我们无从得知具体的transformation matrix，但是可以利用Jacobian来近似得到这个transformation（Jacobian可以获得这个映射的最精确的first-order approximation）。由于一个n*n的matrix J代表着一个transformation，而这个transformation由scale和rotation组成，且这个J可以svd分解为$J=U\sum_{}V^T$，因为U和V都是orthogonal matrix，因此它们都是旋转矩阵，而$\sum_{}$是一个diagonal matrix代表着scale，因此svd就可以把这两种transformation decompose。由于我们在transformation中不想要scale，而只想要rigid transformation（rotation and translation），所以要把scale尽量变为1.
   2. 将scale变为1有多种方式。以前的paper是通过算J和$UV^T$的loss来达到目的，因为当它们俩严格相等时，也就是意味着$\sum_{}$为identity matrix。而本文直接通过singular value来优化。
   3. 在优化singular value时，取log的作用：当你考虑一个值为0.1和值为1.9的singular value的时候，不用log算出来的loss是一样大的，但这显然不合理，因为0.1意味着缩小的倍数是10倍左右，1.9意味着放大的倍数是2倍左右，在这样的情况下，缩小0.1倍这个样本应该有更大的梯度去优化网络，但是loss相同时网络并不能捕捉到这一点，我们需要让网络知道这一点。结果就是在这种不平衡的loss下，网络在训练多个样本之后，倾向于将，这样就会导致elastic loss更偏向于缩小。因此nerfies提出用log，这样0.1的loss和10的loss是一样的（正负号相反，但因为取abs，所以一样），就能控制网络不会偏向于放大或者缩小。
   4. 文章又同时考虑到，人脸等区域确实有nonrigidity部分，因此并不把这种transformation完全限制为rigid。于是作者加了一个robust loss，利用这个了loss对singular value求导后可以发现，对于那些value比较大的情况，loss只传很小的梯度去使得mlp 做rigid transformation，意思就是value比较大的情况就是代表着类似人脸变动这样的nonrigidity，我们不应该去强制限制其为rigid，所以对这样的样本不传递或几乎不传递gradient。而value比较小的情况，可能就确实是rigid部分，只不过离rigid有一些bias，因此我们对其施加regularization。
   5. 对rigid transformation的整体理解：通常我们说transformation都是一个matrix，但是因为这里用mlp implicitly encode the mapping，因此我们无法获得具体的transformation matrix，对于这个scene来说，可能对不同区域存在多个transformation matrix，但是因为我们是implicity mapping，我们可以整体理解为mlp是对整个scene的一个transformation，为了让这个transformation 是rigid的，我们就需要让sample到的点都满足scale的约束。由于mlp是连续的，因此对sample的点进行约束后，整体的mlp上的每个点都会倾向于scale=1，这就实现了整体的transformation的rigidity。
3. Background regularization部分：
   1. 这里通过colmap重建出来的sparse points的coordinates来约束在canonical和observation frame中，bg部分的point的坐标不应该改变。
   2. 原因：我们可以intuitively认为bg部分不会改变位置，因此让他们坐标保持不变很正常。此外，这种保持不变还可以有一个效果，就是如果我们不加这种约束，我们无法得到一个合理的canonical volume，会存在ambiguity，因为可以从不同的canonical volume（bg和fg整体都在空间中变动，这样就会有multi solution）通过不同的deformation来获得observation volume，我们无法得知每个observation volume源于对哪个canonical volume做deformation获得。但是施加这个regularization后，我们可以知道整体的scene就是固定在那不动的。
4. coarse to fine部分：
   1. 做法：在起初的iteration中，高频的positional encoding部分不加入网络训练，使得网络起初只学习smooth的部分；随着iteration增加，逐步加入高频的positional encoding，使得网络学习high frequency内容。
   2. 原因：应该是想到了这个story然后去做了实验，发现在当前task有效，然后强行解释以增加一个novelty。实际上在其他task上可能无效。
5. deformation code interpolation：这种linear interpolation根本没有理论依据，作者只是想到了可能会有用，就去做了尝试，发现有效果，但是其实没有理论依据，就是一种实验结果。


## NRF：Neural Reflectance Fields for Appearance Acquisition

### innovation

#### problems in previous work
1. Nerf之类的工作只能用于view synthesis，并没有将scene appearance model为reflectance和lighting的相互作用。因此也就没办法用于relighting、scene editing。
2. voxel-based 方法由于memory（resolution）的限制，没办法model high-frequency information。

#### improvement
1. Based on Nerf， this work can be used for relighting。

### methodology

#### training
1. 利用一个MLP，输入point coordinate，输出该point的density、normal和计算BRDF的参数（基于不同的BRDF计算模型输出不同的参数）。
2. 利用ray marching，在pixel对应的ray上进行sampling（同nerf），然后将sample points输入MLP得到上述参数。
3. 利用这些参数算出任何一个sample point的transmittance，先用这个transmittance算出light到该point的intensity（$L_i(x,\omega_{i})=\tau_{l}(x)L_l(x)$），然后利用该点的BRDF参数算出BRDF，然后用BRDF、normal、incident light、light direction（同view direction）根据PBR算出该点沿着camera ray方向射向camera的radiance（nerf中不经过如此复杂的建模过程，而是直接用MLP获得这个radiance）。
4. 得到ray上每个点的radiance后，再利用nerf的方法进行rendering（根据sampling进行coarse-to-fine）。

#### testing
由于在training中co-located的设定，算camera和light transmittance时，只需要算一组就行，因为camera ray和light ray的direction是相同的。但是在test时由于我们希望获得放在任意位置的light point照射下的relighting结果，因此为了节省memory，我们需要pre-compute transmittance volume，然后在rendering的时候直接通过query和interpolation方式直接获得某个点的transmittance。本质上来说，导致traning和testing需要进行不同operation的原因就是，保证light transmittance的计算。

1. 将test setting下的point light作为一个virtual camera的center，然后在其面前放一个image plane（plane离light的距离，即virtual camera的focal length需要保证virtual camera能够看到整个object）作为virtual camera，利用这个virtual camera做ray marching。
2. 利用training阶段得到的coarse和fine network（用于sample point）对virtual camera的每个pixel进行ray marching，在sample point处算出它的transmittance然后存储。
3. 在rendering阶段，对camera ray进行ray marching时，每个sample point的light transmmitance可以这样获得：根据step 2存储的transmittance volume，寻找离当前sample point最近的n个已在step 2算出的point的transmittance进行linear interpolation。
4. 得到light transmittance之后就可以算出light到某个sample point的intensity，进而进行rendering。


### details
1. 由于camera和light是co-located的，因此针对某一个point，camera ray和light ray的direction是一致的，因此算camera transmittance和light transmittance时，它们是相同的，不需要重复算。


## Learning Neural Transmittance for Efficient Rendering of Reflectance Fields

### innovation


### methodology

### details

---

## Lumos: Learning to Relight Portrait Images via a Virtual Light Stage and Synthetic-to-Real Adaptation

### innovation

### brief summary
Based on TR(total rendering), Lumos modifys the rendering network, takes two different types of normal maps for generating glass glares and performs synthetic-to-real adaption by training an albedo-refine network based on the observation that domain gap between synthetic and real data mainly comes from the diversity of the subject albedo.

### methodology
1. 合成数据（具有不同的hairstyles、clothing、accessories、face materials，主要的face scan和mesh来源于购买的商业数据）。
2. 将total relighting最后的rendering network decompose开来，使其不至于成为一个black-box（将最后的relight结果分为coarse和fine，fine是在coarse的基础上加上一个residual）。并且采用了
   1. 利用一个netwrok $f_R$学一个diffuse coefficients和一个specular coefficients以及一个relit residual（$C_d,C_s,\delta R=f_R(I,A,L_d,L_s)$）将diffuse light map和specular light map通过它们对应的coefficients 线性结合得到一个最终的light map，然后将这个light map与albedo 进行element-wise的相乘得到coarse relit结果
   2. 第一步中的specular light map $L_s$也与total relighting不一样。TR中采取specular lobe n=1，16，32，64几种，但是在Lumos中还加上了n=1024专门为了处理眼睛镜片上的glare。在TR中将几种不同n的specular light map结合是直接采用network结合的（black-box），但在Lumos中是学习各个分量的weight然后线性相加。此外n=1024的specular light map的生成不是通过query TR中的normal map得到的，而是query Lumos中独有的一个normal 得到的。
   3. Lumos通过$f_N$估计两个normal map，一个是有眼镜镜片的normal $N_l$，一个是没有的normal $N$，当label没有眼镜时，控制这两个normal一样，有镜片时可以根据gt监督生成不同的normal。step 2中就是用这个$N_l$去生成lobe n=1024的light map。
   4. 将1中的residual$\delta R$加到coarse结果上，就可以生成fine relit结果。（在算loss时对这个residual加了一个regularization loss使其逼近0，防止学出来的residual替代原来的coarse结果直接对fine结果起了最主要作用）
3. synthetic-to-real adaption
   1. 所有之前的network的参数都保持不变，在此基础上加一个优化albedo的network去实现这个adaption（优化albedo是因为观察到synthetic和real之间的gap主要在于real data中的albedo具有多样性，而synthetic中难以覆盖到这种多样性，所以需要用real data来训练这个albedo network）。
   2. 将这个albedo network嵌入到原来的pipeline中，是通过将原来的albedo和image以及normal输入到网络估计一个residual，然后加到albedo上形成新的albedo。然后将这个新的albedo用于后续的pipeline中实现。（$\delta \overline{A}=f_{\overline{A}}(I,N,A);\overline{A}=A+\delta \overline{A}$）
4. train albedo net时的Loss分析
   1. Lighting consistency loss: 在light stage中认为，两个光源同时照明下得到的image，可以通过分别采用这两个光源得到的image相加获得。这一点在物理场景（light stage）中是遵守的，即这一property是符合现实世界的，因此**为了让model更加接近现实理论，以及能够在后续rendering中使用单个点光源（只有一个pixel是有光的environment map）也能进行rendering（在project video中可以看到point light下的rendering结果）**，加上了这个loss。这个loss通过在每一个iteration中，任意取两张只有一个pixel亮的environment map（模拟light stage中的点光源）分别去rendering，然后再取一张这两个pixel都亮的environment map去rendering，优化它们之间的loss。（将environment map中某个pixel的值保持不变，其余的都设置为0去模拟点光源）
   2. Relative lighting consistency loss：这样来理解：为了使得这个network只学习albedo细节部分的优化，而不去改变light作用下的效果（如果不去考虑这个regularization，为了保持最后relit的结果与gt尽量相似，network可能会改变rendering时本来单独属于light部分的作用以去逼近gt），考虑保持两个environment map作用下的差异不变（用两个environment map去渲染，这两个map渲染下的image，除了light变了，其它都没变，故将它们做差，这个差异展现的仅仅只是rendering时light改变后的差异，其余部分不变）。在优化albedo之后再算这个loss，可以保证network优化的只是albedo的细节，但是保证rendering时light对整张image所发挥的那部分作用不变。（可以认为其余network已经学出了在rendering时，单纯改变light时所能呈现的不同效果；因此优化后仍要保持这种效果）。（在rendering中，各部分参数都发挥各自的作用，这个loss能够使得rendering equation中属于light的那部分作用不被baked into albedo中，使得它们的作用entangle在一起，故起到了disentangle的作用）
   3. Similarity loss：为了防止优化后的albedo与原来的偏差太大，加一个regularization。其利用VGG去获得$A 和\overline{A}$的高维semantic信息算loss（高维semantic可以忽略detail）
   4. Identity loss：使用一个face recgonition net获取脸部的high-level features，保证脸部不变（similarity是对整个albedo算，其中既含有脸部，也含有clothing等）。
   5. GAN loss：使得relit结果更photo-realistic。
5. train video：
   1. 在本方法中造成video不连续（flickering）的原因是pipeline中的某些部分在frame之间变化的过快（本文注意到主要是normal和albedo变化过快）。因此分别加一个albedo net和一个normal net来对这种差异进行smooth（rendering时其余部分的参数都不变）
   2. albedo net和normal net都是通过输入上一个frame的$A$以及$N$和当前frame的这两个分量来估计一个residual $\delta A$和$\delta N$，然后将这个residual加到当前frame的结果得到最终的值。
   3. Loss分析（仅针对这两个net）:
      1. Similarity loss：同上使用VGG，然后算loss，防止deviation过多。
      2. Identity loss：同上使用face recognition net算loss。
      3. Warping loss：在两个frame最终rendering后的image之间用外部方法算一个optical flow$W$（可以理解为一种transformation），然后将其用于上一个frame，获得当前frame应该得到的结果，在实际rendering出来的结果和这个warping后的结果之间算loss。这个loss可以保证在motion上的连续性。
6. 

### details
1. 3D face scans在https://triplegangers.com/ 上获得（可以获得4K diffuse texture和face geometry）。因为这个face mesh是unstructured的，所以需要用ICP将其对齐到一个坐标系中，获得其精确的坐标。

### 思考
Lumos使用了非常多的loss，比如在synthetic-to-real adaption中使用的对residual加一个regularization loss，本来或许可以用于所有residual，但是可能在experiment中作者发现某些地方加可能对实际效果有提升，于是最终就加了上去，有些地方没有提升，所以就没有加。

---

## Neural Video Portrait Relighting in Real-time via Consistency Modeling

### innovation

#### problems in previous work
1. 对于video portrait relighting，之前要么用昂贵的设备去做，要么就是过于费时，没办法做到real-time。
2. 许多基于monocular RGB inverse rendering的learning方法，它们的效果过于low-quality。
3. 一些image-to-image的方法只基于single image做，无法在video上保证连续。
4. 某些方法没办法做到environment light的任意editing。

#### improvements
1. real-time
2. temporally coherent
3. new dataset(OLAT)
4. enable dynamic environment map（scene editing）


### brief summary

### methodology

### 

---

## Single Image Portrait Relighting

### innovation

#### problems in previous work
1. 之前生成不同lighting的image需要资深摄影师控制相机参数，非常复杂。
2. physical-model-based inverse rendering方法，需要假设material（例如Lambertian）或者illumination（SH、SG），这不一定符合真实的情况。
3. 

#### improvements
1. 因为不需要假设material和illumanation，本方法不会受到physical model的限制。

### brief summary
SIPR takes an encoder-decoder architecture to generate relit images from a single input image, which enables both estimating light and editing light in the bottleneck.

### methodology
1. 运用light stage生成OLAT dataset，然后利用HDR environment map去生成数据。（HDR environment map的每个pixel代表light stage中的某个光源，而基于light可以相加的特性，可以把environment map中的每个pixel代表的光源映射到light stage的light上，然后将那些image线性相加）。
2. 网络结构为encoder-decoder，输入为single image，然后在bottleneck中输出estimated light并加入target light。在decoder处可以获得relit image。
3. Loss
   1. image loss：relit image和gt算loss（利用mask只算foreground）
   2. illumination loss：将estimated light与gt算loss（需要乘上每个pixel的solid angle）
   3. re-rendering loss：将estimated light作为target light重新输入bottleneck，然后算relit image和input的loss。
   4. 

---

## Single Image Portrait Relighting via Explicit Multiple Reflectance Channel Modeling

### innovation

#### problems in pervious work
1. light stage之类采集数据的方法对用户不友好。虽然也有方法采用data distribution transfer的方法生成数据，但弊病是computational cost和poor performance wrt. specular and shadow。
2. 有一些end-to-end的training方法，并没有explicitly consider specular and shadow，没法学出比较好的效果。
3. 缺少explicitly考虑specular和shadow的supervision。

#### improvements
1. 针对1和2，explicitly modeling specular和shadow等reflectance。
2. 针对3，创建了一个dataset。

### brief summary
Different from previous works using end-to-end neural networks, this paper explicitly models multiple reflectance channels(normal, albedo, parsing, shadow, specular) for single image portrait relighting, which is demonstrated to be effective for challenging effects like specular and shadow.

### methodology
1. 将input image输入De-lighting network，得到parsing、albedo、normal和predicted lighting。parsing不参与后续pipeline，这样做是模仿single image portrait relighting，为了使得网络能够更好地区分face的各个region，以获得更好的albedo；predicted light的生成方式也同single image portrait relighting，在bottleneck采用一个weighted average得到lighting，这个lighting也不用于后续的pipeline，作者说这样做可以增强训练稳定性。（**在这里，albedo和normal（geometry）被认为是intrinsic channels**）。
2. step 1得到的normal和target lighting被输入SS network去获得shadow和specular（因为shadow和specular主要是由light和geometry决定的）。在SS network中为了确保light和geometry能够更好地发生作用，采用了一个LFM module（本质是一个self-attention）。
3. 最后将albedo、normal、shadow、specular、target light输入composition network得到最终的relight结果。

### detail
1. 采用均匀光照照射脸部（洗干净没有油），以此直接拍摄multi-view images（拍到的就是albedo）。
2. 利用商业软件*PhotoScan*和1中拍摄的images生成geometry（mesh）和texture map（albedo）。
3. 利用template将mesh align到确定的坐标系下获得确定的pose信息。(在MVS中是用过feature point将新的camera registrate到已经获得的世界坐标系中；但是在这里并不是以image的feature points去registrate，因为这里没有image，只有mesh，故只是用了一种mesh的registrate方法获得当前mesh的camera-to-world pose)
4. 利用*Blender*中的*Cycles rendering engine*和*BSDF shader*做渲染：
   1. texture map被直接当成albedo输入其中。
   2. 根据经验设置roughness=0.6，specular=0.5以渲染gt。
   3. 在2的基础上设置specular=0进行渲染，然后将gt与没有specular的image相减得到specular map；在2的基础上设置shadow visibility=False进行渲染，与gt相减，然后转为grayscale获得shadow map（visualization时取inverse）。
   4. 在texture map的基础上，manually选取不同的8个区域作为semantic（parsing） map。
5. 在training时separately train各个network：
   1. warm-up：先根据supervision train De-lighting net，然后train SS和composition。在train SS和composition时，SS的input使用的是gt albedo和light，composition的input使用全是gt。
   2. 对De-lighting进行fine-tune：输入SS和composition的都是network predict的结果。首先对De-lighting net进行fine-tune（保持SS和composition不变）。（这样理解：在warm-up中，SS和composition的输入都是gt，所以它们在优化后都已经收敛了，只有De-lighting还未完全收敛，因此只需要对其进行fine-tune。换句话说，如果一开始将De-lighting的结果直接输入后续的net，训练就会较慢，或者说难以收敛。）
   3. Note：
      1. 作者将同一个light下的multiple reflectance channel称为一个group($G=\{I, l, I_n, I_{sp}, I_{sh}, N, P\}$)。在warm up时只需要一个group的内容即可：I是用l relighting得到的，因此将I输入De-lighting net后，得到的estimated light, parsing, albedo, normal都可以直接用group中的内容进行监督；而在train SS时，输入SS的light为group中的gt light l和gt normal N, 得到的shadow和specular用group中的内容监督；train composition net时，输入其中的albedo, normal, shadow, specular, light（没有其余的target light，只有input的gt light）全部来源于group。
      2. 但是在fine tune阶段，由于需要target light，因此需要两个group（除了light及其导致的不同的shadow和specular，其余都相同）来train。
   4. 

---

## Total Relighting: Learning to Relight Portraits for Background Replacement

### innovation

#### problems in previous work

#### improvements

### brif summary

### introduction 
1. 介绍relighting的背景：什么是relighting（Compositing a person into a scene to look like they are really there）；其应用（smartphone photography、video conferencing、film-making）；介绍了film-making的具体做法，指出其无法保证light-consistency（新背景下的object看起来跟真的处于背景之中一样）。
2. 列举了一些工作：relighting、estimate alpha matting and foreground colors、consider both foreground estimation and composite。指出它们由于lack explicit relighting step而无法得到photorealistic的结果。
3. 介绍light stage做relighting的方法。指出其hardware成本高。（每个object都需要扫一组，但是TR只需要用synthetic数据train一个net就可以做到对每个object的估计）
4. 介绍自己的工作。

### related work
1. image-based relighting
2. portrait relighting
3. alpha matting
4. our approach

### methodology

---

## Acquiring the Reflectance Field of a Human Face

### methodology
1. 用light stage拍摄OLAT照片。（light stage的灯覆盖了整个球面，其相当于一条按序点亮的灯带，按照经度绕完一圈后就按维度往下一点。每个灯的位置以$\theta \phi$表示，最终获得64*32个灯照明下的image）
![light_stage](./images/light_stage.png)
2. 对于每个camera view，记录当前camera view下每个pixel在64\*32个灯下的值，组成一张64*32的image，代表在当前view下，这个pixel对应的3D point在每个light下的reflectance。因为我们不知道3D点的normal，所以没办法用physical model来建模，但是我们在当前pixel看到的value是下面式子中除了$L_i$外的所有内容，而$L_i$是一个scale，所以相当于我们记录下了其它所有值的信息（在当前camera view下保持不变）。因此可以通过控制$L_i$的大小获得最终结果。
$$
L_o=\int{L_i \cdot f \cdot cos\theta \cdot \omega_{i}}
$$
总的来说，这张64*32的image代表了对于某个pixel来说，每个光照下的reflectance，其乘以illumination即可得到每个光照下的rendering结果，若全相加，代表是所有灯下的rendering结果。
3. 上述可以做到在某个view下的relighting结果，但是做不到在novel view下的结果。此方法后续通过用cross-polarization将specular和diffuse分开来，然后再进行novel view下的relighting。

---

## DPR：Deep Single-Image Portrait Relighting

### innovation

#### problems in previous work
1. 现有的image-based relighting方法估计的face geometry和reflectance details可能不准，导致后续的relighting结果not photo-realistic。
2. 

#### improvements
1. 对于1利用合成数据集先让network学习如何relighting，然后利用GAN去生成realistic的结果。


### brief summary
DPR first generates a relighting dataset, then performs image-based relighting using an encoder-decoder architecture which manipulates SH-based lighting at the bottleneck.

### methodology
1. 利用Ratio Image-based Face Relighting方式生成relighting数据集（每张source image生成5张不同lighting下的relighting image）。
![ratio_image](./images/ratio_image.png)
其中R是albedo（文章假设人脸材质为albedo），f代表SH lighting和normal作用下的结果。
   1. 其中的normal根据自己的算法获得。
   2. source image的SH lighting通过SfSNet估计获得。
   3. target image的lighting通过在一个SH lighting数据集里面sample获得。
2. 网络使用Unet结构，在bottleneck处估计两个latent code，一个为$z_f$代表face information，$z_s$代表lighting feature（经过一个regression得到estimated SH lighting）。然后在bottleneck处输入target lighting，与$z_f$ concatenate输入decoder。
![dpr](./images/DPR.png)
3. loss：
   1. image loss：predicted image和gt之间的L1 loss以及两张image的Laplacian（二阶梯度代表high frequency内容，包括edges）之间的loss。
   2. GAN loss：由ratio image trick生成的dataset可能由于normal不准产生artifact，导致数据层面就有问题。注意到artifacts出现在local patch，因此加了patch GAN的loss去refine local detail。
   3. latent code loss：给任意两张source image（face相同，lighting不同）经过encoder得到$z_f$，算两个$z_f$之间的loss。这个loss代表face相同的两张image的face information应该相同。
4. training strategy：作者发现，如果一开始training就用skip connection，会导致被encoded在$z_f$中的face information减少，减少的information都是由于skip connection被分流出去。因此采用skip training方式，一开始先不用skip，然后随着iteration的增加逐步加skip connection直到最后全部加上去。

### new knowledge
1. 在同一个真实lighting condition下，相机过长的exposure time会导致image过亮，这可能导致估计SH时不准确（其根据image估计），即其无法反映真实的lighting condition。
---

## PhotoApp: Photorealistic Appearance Editing of Head Portraits

### innovation

#### problems in previous work
1. dense light stage成本高，且无法捕捉in-the-wild images中的variations。
2. 大多数supervised方法无法支持novel view，能支持novel view的方法并不photo-realistic。
3. 虽然styleGAN可以没有direct supervision（没有算gt和predicted的直接的pixel-wise loss，而是用别的网络提了feature然后算loss），但是存在loss of quality。
4. 基于OLAT的deep learning方法和Debevec那样的方法只能在inner face region实现好的效果，但是没办法在eyes和haircut方面实现好的效果。
5. synthetic（利用albedo什么的去生成，类似Lumos）方法影响了relighting结果的photo-realistic。

#### improvements
1. 针对1没有用dense light stage，只用到了8个camera和150个RGB light，然后利用environment map去生成数据集。
2. 针对2本方法实现了novel view。
3. 针对5，本方法使用的不是合成数据集。

### brief summary

### methodology
1. 将input image、target illumination、camera pose、binary input p（记录camera pose是否与input相同，用来控制相同pose下的内容一样）输入pspNet（pretrained，参数保持不变）得到18\*512的latent code。
2. 然后将得到的latent code分成18个512维的latent code，分别代表不frequency的features，针对每个separated latent code，有一个独立的PhotoApp net（以target illumination、camera pose、binary input p一起作为输入），得到不同frequency下的latent code，然后将它们concatenate输入styleGAN（pretrained，参数保持不变）得到relit image。（这里因为light stage用的是150个RGB light，所以environment map也应该是150*3的rgb image，这里将其flatten，成为450维的vector）。
3. loss：
   1. latent code loss：PhotoApp net将input image的latent representation映射为target image的latent representation，因此要与gt的latent representation算loss。
   2. perceptual loss：predicted relit image与gt之间首先经过Alexnet算一个feature，然后算loss。
4. 

---

## High-Res Facial Appearance Capture from Polarized Smartphone Images

### innovation

#### problems in previous work
1. light stage with polarizers采集数据来恢复材质的方法非常unfriendly to amateurs。

#### improvements
1. 只用一个手机采用cross-polarization和parallel-polarization的方法获得diffuse和specular分量（disentangle）。capture setting非常友好。

### introduction
1. 第一段：介绍背景。硬件发展地比较好，激发了做数字人的热潮。但是relighting under arbitrary viewpoints with different lighting conditions非常难。指出传统的用light stage的方法非常复杂（引出问题），因此本文章要在保证relighting结果较好的情况下简化capture process（最主要的创新点，简化了，且证明可行）。
2. 第二段：第一段只是用一句话说了light stage的复杂，第二段细化了这种setting是怎么做的，然后再次说明很复杂。
3. 第三段：引出自己的方法（拍摄简述，后续要得到albedo、specular等），首先就是强调拍摄简单（only a smart phone）。最后说明自己的方法产生的各种material参数，可以很好地支持一些工作，如editable等。
4. In summary，总结一下contributions：
   1. capture setting使得能够separate diffuse和specular。
   2. co-located camera和light（感觉不像contributions）
   3. coarse-to-fine optimization for texture of different resolution using mipmap。（可以提高texture的sharpness，即可以获得high-resolution的texture）

### related work
1. polarizaton：最大创新点。指出polarization的方法主要基于specular不改变polarization方向这个事实。然后列举了几篇文章的方法。
2. lightstage capture systems：先介绍light stage来源。然后指出问题：拍摄时间长；给muitiple camera和light设置polarizer非常challenging。然后列举一些利用light stage的方法。最后还是强调拍摄设备非常复杂。
3. differentiable rendering：指出recently work极大推动了relighting的发展。列举几篇这个方面的工作，指出他们侧重于shape reconstruction，但是本篇文章可以基于一些效果非常好的reconstruction方法去重建shape，可以更侧重于材质的恢复。再指出一些工作用complex lighting setting，不利于恢复材质。
4. deep learning-based approaches：列举一些用deep learning的方法。

summary：在介绍各种technics时，除了要介绍其基本原理，还要引出一些文章，最后能批判地要批判他们。

### brief summary
Due to the high cost of previous hardware, PolFace proposes to capture face data only via a smartphone with polarizers, which not only simplifys the cpature process, but ensures the accurate disentanglement of diffuse and specular material attributes.

### methodology
1. Data capture:
   1. 拍摄两段视频（一段cross-polarization、一段parrallel-polarization）和一些photographs，它们一起用于shape reconstruction。拍摄photograph是因为作者认为，photograph的质量更高，要用它来恢复材质，而video主要用于shape reconstruction。且所有拍摄的light均只来自于flashlight（近似于point light）。
   2. 用这些数据首先重建一个coarse mesh。然后将其fit到一个FLAME model上获得更精确的geometry信息，进而获得更加精确的UV parameterization。
   3. 作者认为polarizers会对不同wavelength的light产生不同的attenuation，因此使用一个colorchecker board去colibrate color（affine transformation）。
   4. 此外作者认为flashlight不完全是一个point light，因为hardware原因其在某些方向会被遮挡，导致有些surface point接收的光不能完全用一个point light来计算（实际上没有接收到所有的光，只接收到一些特定方向的光），所以作者使用一个per-pixel的light attenuation map来建模（与render结果相乘来近似，因为render结果是用point light获得的，而实际场景非point light，所以要在render结果基础上attenuate）。为什么可以这样来model：因为对于image plane，每个pixel对应的surface point，其跟光源的相对位置是固定的（除去distance因素），所以每次拍摄的image，其同一个pixel上的surface point都接收了同样的光，乘以attenuation只是将光的比例缩小，来建模那些实际point light没有射到点上的光线。
   5. video中每10-frame抽取a frame（基于sharpness，其由laplacian衡量）。由于photograph时的light比video亮，因此要把iso和光圈调小，使得它们的亮度相似。
2. geometry reconstruction：用Agisoft Metashape重建。
3. BRDF：考虑了SSS
4. optimization：
   1. train两个部分都采用coarse-to-fine的方式，每次从512*512 resolution先train，然后resize继续train直到4096。所以train的内容全部在texture space，即texture map的内容是trainable 参数。首先train albedo部分，收敛后固定，然后train specular部分。
   2. 作者考虑到某些拍摄的image中某些pixel对应的部分刚好是在grazing angle下得到的，这样会影响image的quality，因此这些pixel的train weight应该被降低，即view和normal夹角越大的pixel权重越小，据此可以获得一张image，其每个pixel代表weight。此外，作者还考虑拍摄的distance问题，如果对于一个pixel，其对应的mipmap的level比较大（resolution比较低），代表其距离较远，实际的detail较少，那么其不应该被用于优化。实际过程中，作者只取了level 0来算weight map。

### limitation
1. 虽然只用一个手机拍摄条件比lightstage要好，但是要保证environment illumination近乎全黑实际上条件较为苛刻。此外，还需要保证拍摄的地点不能有一些反光强的物体，比如不能有玻璃、镜子等，否则没办法保证每个pixel的color都是来自于surface point。


## Deferred Neural Rendering: Image Synthesis using Neural Textures

### innovation

#### problems in pervious work
1. 一般的rendering pipeline都需要输入良好的3D geometry、material、light等，这些以前通常由skilled artist来制作。虽然3D reconstruction可以重建这些内容，但是imperfect（noisy、over-smoothed、holes），这会导致rendering结果not photo-realistic。

#### improvements
1. 本篇文章最大创新点（经常重复）就是在imperfect geometry（几乎可以认为所有基于重建得到的geometry都是imperfect的）的前提下rendering效果不错。作者认为texture中包含可以弥补geometry imperfections的信息，但是traditional texture特性（normal、albedo etc）的information非常low-dimensional，因此其提出使用learnable feature vectors。（作者在ablation证明了即使geometry resolution非常小，rendering结果也不错）。此外由于Neural Rederer是采用conv的，会利用到周围pixel的信息，因此也可以弥补geometry的imperfection（以前的方法有用per-pixel的conv来得到pixel的value）。

### introduction
1. 第一段：介绍graphic rendering pipeline需要well-defined data（geometry、material、illumination等），以前通常通过skilled artist manually获得。现在通过3D重建可以获得，但是geometry not perfect。
2. 第二段：引出算法的启蒙。与其解决geometry的imperfection，不如修改rendering pipeline使rendering结果更好。然后介绍neural texture的意思（与传统texture map的区别）。
3. 第三段：说明本方法可以有很多application，比如view synthesis、scene editing。然后说明自己做了哪些实验证明这些application的效果不错。
4. 第四段：说明本方法在video中效果好，即temporally coherent。因为本方法基于3D space进行而不是image进行，因此可以获得时序上连续的结果。

### related work
1. Novel-view Synthesis from RGB-D Scans
2. Image-based Rendering（Debevec那种image blending）
3. Light-field Rendering（PBR model-based）
4. Image Synthesis using Neural Networks
5. View Synthesis using Neural Networks

### brief summary


### methodology
1. 视频抽帧用于colmap重建，得到pose和mesh以及texture mapping关系。
2. 对于每个view，先做rasterize。然后train一个texture map，其中存储了feature vectors。对于rasterize的image处的每个pixel，通过mapping关系获得其feature vector，然后将整张feature image输入Deferred Render得到最终的预测结果。
3. 对于texture map的resolution：traditional graphics中使用mipmap来解决over-sampling和under-sampling问题，这里也参考了mipmap。首先取low-resolution（512*512）的texture map，然后通过upsampling得到high-resolution的map，对于每张image，在每个level上都做sample然后输入renderer计算color值，最终的color值是由所有level的color值相加。low-resolution的map成为coarse level，high resolution成为finer level。
4. 在Renderer中，同时输入view direction（其由SH encode）可以控制view-dependent内容。

### limitation
1. 没办法做relighting（appearance editing）

---

## Deferred Neural Rendering: Image Synthesis using Neural Textures

### innovation

#### problems in previous work
1. model-based solutions没法exactly recover each component（material、geometry、lighting）。
2. image-based solutions（特指Debevec的方法）需要dense and accurate samplings，难以获得。

#### improvements
1. 针对1的问题，作者选择使用image-based方法去做。针对2的问题，作者用了另一种image-based方法，不需要dense sampings。

### introduction
1. 第一段：介绍背景。简述model-based approach是怎么做的，指出其问题。简述image-based approach怎么做，指出其问题。
2. 第二段：首先一句话介绍自己的方法优越性。然后按first、second、finally介绍自己方法并指出radiance cues的重要性。
3. 第三段：说明如果geometry太差或者light transport effects过于complex，会导致neural textures需要太多channel或者neural renderer需要特别多的parameter来保证学习到这种information，然后引出自己的方法；指出自己做的实验证明了自己的方法有效性；进而引出自己的augmentation方法。
4. 第四段（contributions）：
   1. a novel system（能够整体上实现什么）
   2. a neural renderer（能够解决不同的lighting conditions）
   3. a novel acquisition（easy to do）
   4. an augmentation method（能够以不同的light做relighting）

### related work
1. model-based solutions：分为shape modeling、appearance modeling和joint modeling of shape and appearance。
   1. shape modeling没有model view dependent appearance。
   2. appearance modeling只能基于accurate geometry。
   3. joint modeling的精确度局限于输入数据的accuracy和当前材质下的model是否适用。
2. image-based solutions：分为image-based rendering和image-based relighting。

### methodology
1. 利用一个手机和一个相机作为acquisition setup，手机的flashlight作为point light。在获得proxy geometry时在natural light下用相机拍摄，然后用COLMAP重建mesh。然后在做relighting时，将手机camera和相机camera register上去，以手机的pose作为point light的位置，以相机拍摄的image来train。
2. neural textures有30个channel，material basis有5个（一个Lambertian，4个不同的roughness）。首先在某个pose下的view做一个rasterize，然后对每个pixel取texture，获得一张（H\*W\*30）的内容。（neural textures还过一个net获得mask）
3. 然后用path tracer，对不同的material和已知的geometry、point light做rendering（当前view）得到5张light map。
4. neural texture一共30-channel，分为5组，每组6 channels（实际为两张rgb image）。6个channel中每3个channel与light map相乘（element-wise），因此每组channel能获得两张image。将相乘后的一共10张image输入neural renderer获得最终结果。
5. 解决问题
   1. 为了解决GPU memory的问题，作者在view sphere上uniformly分布13个point（每个point对应一个net），然后point相连获得triangle。对于落在某个triangle内的view，其内容需经过这个triangle的3个vertices对应的net，得到三个不同的结果，然后将这三个结果以重心方式算weighted sum。
   2. 作者是用point light做的train，为了generalize to novel lighting，作者将light做了一个augmentation，使model可以用于environment map。  
6. 作者对拍摄到的用于train的image做了一个gamma correction（2.2）使得pixel value radiometrically linear。（相机在把拍摄到的内容存成image file时就做了一个gamma=1/2.2的correction，使image的成像更接近于人眼，因此做一个gamma=2.2的correction能够变回去）。
7. 作者对training gt和predicted image首先做了一个log，使它们在log域算loss，这保证training gt的dynamic range也能够被学习到。 

### limitation
1. 在设计网络时，没有explicitly考虑view direction。作者认为一个renderer没办法model所有view的结果，因此只是将所有的view分为13个partition，然后对每个partition train一个renderer。（或许是作者试过view direction的model，但是效果不好）。

## Rapid Acquisition of Specular and Diffuse Normal Maps from Polarized Spherical Gradient Illumination


### normal derivation
1. 对于位于light stage中心的human or object来说，从light stage各个方向射向其时，若某个方向为$\omega=(\omega_{x}), \omega_{y}, \omega_{z}$，那么从这个方向射出的光的intensity调整为$\omega_x$或$\omega_y$或$\omega_z$，至于调整为哪个，视其为4个pattern中的哪一个，若为$P_x$则调整为$\omega_x$，依次类推。
2. 整个light stage所有光源的最大亮度若为c，则pattern=c时代表将所有light的强度调整为c；若pattern=X，代表将每个光源的强度调整为$c\omega_x$（其中$\omega=(\omega_{x}, \omega_{y}, \omega_{z}) \& ||\omega||_2=1$）;pattern=Y或Z时类似。
3. diffuse normal推导
![diffuse](./images/diffuse_normal.png.JPG)

由于在公式推导时入射光强度$P_i(\omega)$定义在[-1, 1]上，即入射的light intensity定义在[-1, 1]，因此获得的reflectance（$L_x(\vec{v})$、$L_y(\vec{v})$、$L_z(\vec{v})$）也分布在[-1, 1]，这才能使得上述推导获得的normal取值在[-1, 1]之间，代表真正的坐标。由于我们实际的光照intensity位于[0, 1]之间，无法满足公式推导时的[-1, 1]要求，因此我们需要将得到的$L_i$映射回[-1, 1]。我们实际光照的intensity$P_i'$满足$$P_i' =\frac{1}{2}(P_i+P_c), P_c=1$，那么用这个实际的$P_i'$作为intensity得到的reflectance为$L_i'$，用推导时位于[-1, 1]之间的$P_i$得到的reflectance为$L_i$，用实际的满光照$L_c$得到的reflectance为$L_c$，则根据线性相加原则，$L_i'=\frac{1}{2}(L_i+L_c)$，为了将[0, 1]之间的$L_i'$映射回[-1, 1]，因此做反变换得到$L_i=2L_i'-L_c$。即我们要将在gradient illumination下得到的image $L_i'$的pixel value先乘上2，然后再减去在full illumination下得到的image $L_c$，即可得到位于[-1, 1]之间的$L_i$。根据上面的推导，将得到的$(L_x, L_y, L_z)$做normalization即可得到normal坐标，其位于[-1, 1]。为了做visualization，可以将其归一化到[0, 1]。文中的Figure 2就做了这个过程。

4. specular normal推导
![]


## Learning to Reconstruct Shape and Spatially-Varying Reflectance from a Single Image

### class
image-to-image

### setting
1. light: 
   1. point light collocated with the camera
   2. environment light(SH)

### brief summary
xxx trains an image-to-image pipeline which first gets the coarse results from three bounces, then refines it by cascade structure.. 

## De-rendering 3D Objects in the Wild

### class
image-to-image
unsupervised

### innovation
1. 第一次以unsupervised方法在in-the-wild数据上做inverse rendering。

### brief summary
First estimate the coarse geometry, material, light, then use the coarse information as the pseudo supervision to bootstrap the decomposition.

### thinking process
1. inverse rendering是一个ambiguous的问题，light、material和geometry的不同组合都能得到相同的结果，因此想要decompose必须要一些hints去引导学习。直接使用supervised的当然更好，但是要在in-the-wild上做需要特别的设计。
2. 作者首先进行一些假设简化rendering过程，然后预估coarse的各个components，然后以它们作为一些监督去引导decompose，但是由于之前的假设以及简化，渲染的结果可能不够好，因此将足够的flexibility赋予网络克服。

### methodology
1. 通过off-the-shell方法获得depth，然后由depth获得normal。
2. 简化rendering过程为phong-model（每张image share同一个scalar ambient light intensity、同一个scalar specularity intensity、同一个scalar shiness $\alpha$，建模一个directional light优化其light intensity和direction），light被建模为ambient light和directional light。
3. 这样设计GAN的原因：在PBR中，即使已知light的所有信息，还是有可能在normal（geometry）和material之间产生ambiguity（$f(\omega_i, \omega_o) <\vec{n} \cdot l>$），$f$和$cos\theta$之间不同的组合都可以产生相同的结果。直接对图像加image loss无法克服这种ambiguity，导致material和geometry无法较好地decomposition，一种比较明显地反映decomposition不好的现象就是当改变light时，渲染出来的图像有很多artifacts。作者发现改变light方向是导致这种artifacts的最主要因素，因此他从light direction入手。以estimated light direction和randomly sampled light direction分别做渲染，用GAN使得任意方向的渲染更像是resonable渲染结果。这就使得material和geometry能够估计地更加准确，因为任意方向的light都能生成好的结果的唯一可能就是它们估计地准确。作者考虑到用自己定义的rendering pipeline可能本来就会引入artifacts，所以他不用gt image来作为positive样本，防止discriminator将pipeline本身的不足作为判别是否为好的样本的依据。

