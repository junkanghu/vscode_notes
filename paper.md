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

---

## NeuS: Learning Neural Implicit Surfaces by Volume Rendering for Multi-view Reconstruction

### advantages and limitations in previous works
* IDR:
  1. fail to reconstruct objects with complex structures that causes abrupt depth changes: The cause of this limitation is that the surface rendering method used in IDR only considers a single surface intersection point for each ray. Consequently, the gradient only exists at
  this single point, which is too local for effective back propagation and would get optimization stuck in a poor local minimum when there are abrupt changes of depth on images
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

### problems
1. Note that the standard deviation of $\phi_s(x)$ is given by $\frac{1}{s}$, which is also a trainable parameter, that is, $\frac{1}{s}$ approaches to zero as the network training converges.