# This is the notes for lecture First Pinciple of Computer Vision

## Image Formation | Image Sensing | Binary Images

### Overview | Image Formation
1. Pinhole camera gets little light which makes the image look dark.

### Pinhole and Perspective Projection | Image Formation
1. f is called Effective Focal Length which is different from $f_x$ and $f_y$ in camera intrinsic matrix K.
2. The image of a line in 3D is a line in 2D.
3. Magnification(m): the ratio of the length of a segment in the image plane to the length of the corresponding segment in the scene. The magnification finally turns out to be (note that the m here assumes that the image distance i equals to effective focal length, actually, which means object is exactly projected onto the image plane whose distance to the lens is $f$):
$$
|m| = \frac{||\overline{d}_i||}{||\overline{d}_o||}=|\frac{f}{z_o}|
$$
m can be assumed to be a constant if the range of scene depth $\Delta z$ is much smaller than the average scene depth $\widetilde{z}$. Besides, we can know from m that the ratio of two areas is as follows:
$$
\displaystyle\frac{Area_i}{Area_o}=m^2
$$
4. Vanishing Point: parallel straight lines converge at a single image point which is called vanishing point. 
   * Location of vanishing point depends on the orientation of parallel straight lines.
   * All the parallel lines converge to the same vanishing points. So, to find a vanishing point for a group of parallel lines, we can find a line passes through the pinhole which is parallel to these lines.
   * Human tends to focus on the vanishing points of an image. So photographers always put objects at the vanishing points to attact attention.
5. False Perspective: The objects in the image look further away than they actually do. 
6. The ideal pinhole size:
![pinhole](./images/pinhole_size.png)
   * When the pinhole size is very large, every scene point is projected to many points in the image plane, so the image is blurry. And when the size is small enough as it is close to the wave length, diffraction happens, which causes the blur again.
   * The ideal pinhole diameter is:
   $$
   d\approx 2\sqrt{f\lambda}
   $$
7. Pinholes pass less light and hence require long exposures to capture bright images. And images captured by pinholes are focused everywhere within the image.
![comparison](./images/comparison.png)
In the image, pinhole camera only gets the light denoted by the yellow line, but lens get all the light. So lens solve the problem that pinhole gets less light.

### Image Formation using Lenses | Image Formation
1. Gaussian Lens (Thin Lens) Law: Image distance i and object distance o obeys the following law:
$$
\displaystyle\frac{1}{i}+\frac{1}{o}=\frac{1}{f}
$$
To find the focal length, make the o -> $\infin$, then $f=i$. It means that focal length is the distance at which incoming rays that are parallel to the optical axis converge.
![focal](./images/focal.png)
2. The magnification here is:
$$
\displaystyle m = \frac{h_i}{h_o}=\frac{i}{o}
$$
The segment $h_o$ is projected as the segment $h_i$, which doen not mean that it is on the image plane, what we actually need to do is to make the object projected onto the image plane where the focal length is.
3. Without changing the distance between the object and the image(here the image means the one which we can see by setting the image plane here), we can change the magnification by adjusting the distance of two lens which lie bewteen the object and the image.
![two_lens](./images/two_lens.png)
4. f-number $N$: The f-number is defined by the following equation.
$$
N=\frac{f}{D}
$$
where $f$ is the effective focal length and the D is the diameter of the aperture.
5. Lens Defocus: the image does not lie on the camera image plane whose distance to lens is the focal length $f$.
   ![defocus](./images/defocus.png)
   * In this case, one point from object is actually projected onto the image plane as a Blur Circle.
   * The diameter of the blur circle is calculated as follows:
   ![blur_circle](./images/blur_circle.png)
   * Theorectical ways to focus the object:
     * Move the image plane
     * Move the lens
     * Move both image plane and lens
     * Move the camera itself towards or backwards the object
     ![ways](./images/ways_focus.png)

### Depth of Field | Image Formation
1. When object focus at certain depth, then all the objects at these depth are focused. Similarly, when an object defocus at a certain depth, all the objects at these depth defocus.
2. What does "focus" mean: the blur circles $b$ of all the points of an object lie within a pixel.
3. Depth of Field (DoF): the minus of two boundary where the object exactly focus.
   ![dof](./images/dof.png)
   * Hiperfocal Distance: the closest distance $o=h$ the lens must be focused to keep objects at infinity ($o_2=\infin$) accaptably sharp (blur circle $\le$ c)
   ![hyperfocal](./images/hyperfocal.png)
   The hyperfocal distance is calculated by:
   $$ 
   \begin{aligned}
   c&=\lim_{o_2\to \infty}\frac{f^2(o_2-o)}{No_2(o-f)}\\
   &=\frac{f^2o_2}{No_2(o-f)}\\
   &=\frac{f^2}{N(o-f)}
   \end{aligned}\\
   \begin{aligned}
    \frac{Nc}{f^2}=\frac{1}{o-f}
   \end{aligned}\\
   \begin{aligned}
    o-f=\frac{f^2}{Nc}
   \end{aligned}\\
   \begin{aligned}
    h=o=\frac{f^2}{Nc}+f
   \end{aligned}\\
   $$
   It means that with certain effective focal length and certain f-number and pixel size, objects whose depth is larger than focused point $o\cong$ h are all focused because their blur circles are all with a pixel.
   ---
   According to the equation which defines the hyperfocal distance, we can know that with the aperture decreasing (i.e., the f-number $N$ increasing), the h decreases, which means that the focused objects increase. However, the images are darker.
   
   ---
   Thus:
   Large Aperture (Small f-number):
     * Bright Image or short Exposure time (in case of the over-exposure)
     * Shallow Depth of Field
   Small Aperture (Large f-Number)
     * Dark Image or Long Exposure Time (in case of the under-exposure)
     * Large Depth of Field
4. When there are objects sticking to the lens tightly, we won't see them because there are no lights reflected by the objects going into the lens. What we can notice is that the image gets darker, because less light goes into the lens. But when the objects leave a little away, there might be light reflected by the objects going into the lens, so you can see the objects. This phenomenon tells us that when there are dusts sticking to the lens, it won't cause any bad effect but darker image.
![block](./images/block.png)
5. Tilting the Lens
When we don't want images that shows the objects of the same depth but images with objects located at different depth, we can use tilt camera.
![tilting](./images/tilting.png)
The object within the plane of focus is shown in the image.

### Lens Related Issues | Image Formation
1. Compound Lenses: a single lens fails to form a image which looks clear across the whole content, so we need compound lenses to compensate the bad effects to form a satisfying image.
![compound_lenses](./images/compound_lenses.png)
2. Bad effects
   * Vignetting(|vi'netin|)
   Reflected light from some points of the object is less than that of other points, so some parts look darker.
   ![vignetting](./images/vignetting.png)
   ![example](./images/example_vignetting.png)
   * Chromatic(|kromatic|) Aberration(|abrei|)
   Because of refraction, light of different colors which have different wavelength focus on different image plane. 
   ![chromatic_aberration](./images/chromatic_aberration.png) 
   * Geometric Distortion: in edge area, content in the image becomes distortion.
   ![distortion](./images/distortion.png)
   ![example](./images/distortion_example.png)
3. 








