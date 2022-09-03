[toc]
# Notes for papers

## IRON: Inverse Rendering by Optimizing Neural SDFs and Materials from Photometric Images

### problems in previous works

---
### innovation

---
### limitation

---
### some conceptions
1. **edge pixel & interior pixel & subpixel**
   * **edge pixel**: where shading colors is a combination if colors at disconnected surface pieces.

    ![edge_pixels](./images/edge_pixels.jpg)

   * **interior pixel**: pixels whose content comes from smooth and continuous surface

   * **subpixel**: a small area inside a pixel
2. 
---

### tricks

### problems
1. equ(1): co-located flashlight and camera have the same direction?
2. figure 2(a)?

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
2. figure 3: visible and invisible surface
3. why this way can solve the problems caused by abrupt depth change and occlusion and why nerf can handle abrupt depth problems
4. why $\alpha$ takes the *max* operation? According to the feature of $\rho(t)$, $\alpha$ naturally obeys this rule.
5. mask loss
6. renderer.py line 230: dirs are unit vectors, but gradients may be not though the gradient loss forces them to be like unit vectores
7. cos anneal ratio at renderer.py line 230

---
### mathematical dederivation
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