[toc]
# Daily summary
## 2022.8.24
### finished 
1. learning github

### assigned
1. lecture
   Games101: chapter 8 & 9
2. three videos and one paper
   1. [3d vision base](https://www.youtube.com/watch?v=CtyhRGq74js)
   2. [device principle](https://www.youtube.com/watch?v=GJ2gtQ0WxTU)
   3. [IRON](https://kai-46.github.io/IRON-website/): about relighting
3. ppt 
   [Differentiable Inverse Rendering](./files/Differentiable%20Inverse%20Rendering.pdf)

### notes
![draft](./images/IMG_0684.jpg)
1. 原理
   1. 自然光拍摄的图片，既有阴影，又有反光，影响了物体本身的颜色。
   2. 可能会产生阴影的部分采用其他光源照射即可消除
   3. 不同颜色的光照到物体表面会产生不同的颜色和效果，实际采集照片时可以利用设备控制环境光照恒定
   4. 自然光都是原偏光照，只有某一偏振方向的光为线偏光照
   5. 物体表面会发生镜面反射和漫反射。镜面的反射光与入射光偏振方向一致，只是传播方向发生改变；漫反射
   6. 进入镜片的光=漫反射分量+反射分量
2. 设备原理
   1. 在光源前面放置x方向的偏振片，使射出的只有x方向偏振的光，照射到物体表面产生反射光和漫反射光，在相机镜片处也放置x方向偏振的光；在光源前面放置y方向的偏振光可以得到
   2. 设备上的光源可以设置偏振，镜头也可以设置偏振。光源用来照射阴影部分消除阴影。
3. 目的
   1. 使用设备获得漫反射颜色对应的图像
   2. 用现有模型建立基于这些图像的模型，观察是否符合物体原色（无阴影和反光），符合原色说明得到的漫反射分量是正确的，只有其正确才能去解耦出正确的反射分量
   3. 利用手机拍摄的图像（漫反射分量+反射分量）通过nerf重建颜色，并利用这个颜色和得到的真实漫反射颜色解耦出（利用物理光照模型）漫反射分量和反射分量，目的即是获得最终的反射分量。