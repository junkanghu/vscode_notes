## Cosine Lobe Based Relighting from Gradient Illumination Photographs

### usage
1. 以coloar gradient illumination获得imape per-pixel phong-model parameters、diffuse和specular normal、shadow的方法。

### innovation
1. 将spherical gradient illumination的四次imaging通过color gradient illumination减到两次。
2. 能够较好地explicitly fit出per-pixel phong exponent，而之前的方法要么需要manually select exponent，要么需要更多的光照条件（这里只需要两种）来估计per-pixel exponent。

### methodology
1. spherical gradient illumination是用4种gradient illumination（全白光，三通道值一样）去拍摄照片来计算normal等，但是这篇文章将gradient illumination的强度放入到rgb的不同通道中，因此只需要用一个spherical color gradient illumination和full illumination照射两次，就能够通过ratio获得spherical gradient中需要的三张image。
2. 然后通过Phong model的cosine lobe将每个pixel的的BRDF fit出来，最终可以得到这个pixel对应的lobe中心direction、shiness以及反射系数k。因此只要有了光照就可以逐pixel进行基于phong model的relighting。
3. 由于已经得到了每个pixel的lobe direction，那么可以根据view direction和lobe direction（perfect reflection direction）的half vector获得specular normal，并直接以lobe direction作为diffuse normal。
4. 得到normal后就可以利用类似poisson重建的方法获得coarse geometry，然后以这个coarse geometry来model self-occlution从而model shadow。


## Deep reflectance fields: high-quality facial reflectance field inference from color gradient illumination

### usage
1. 为了克服OLAT拍摄时被摄者的移动，每拍摄11帧时使灯光全亮拍摄一张“tracking frame”，最后用所有的tracking frame做个optical flow。然后所有帧的flow都可以通过时间进行插值获得。算法引用*Jump: Virtual Reality Video*。
2. 提供生成random light direction下的OLAT image的方式。
3. OLAT拍摄dynamic scenes的解决方案。

### problems in pervious work
1. OLAT配合envmap生成relighting image的方式不用explicitly model material和geometry就能够做。但是无法capture time-varying dynamic scenes；硬件复杂（高速相机、同步设备等）；需要做optical flow alignment应对人体微小变化。
2. 为了capture dynamic scenes，关键就是每次只需要不多的images去做relighting。在这样的setting下，有些方法选择性地只对portrait的某一部分做relighting（skin、clothes、glasses， etc.）；也有些只在synthetic数据集上有效果。

### innovation
1. 在inference阶段，即对某个新拍摄的人做relighting时，只需要坐下来拍摄两张color gradient images即可，拍摄过程非常简单，避免了做optical flow，以及对被摄者静止的限制，因此也就可以用于拍摄dynamic的scenes。
2. 可以生成任意light direction的OLAT image，这种dense结果能够在利用envmap时效果更好。

### thinking process
1. spherical color gradient images包含有丰富的信息，包括reflectance、shadowing等，因此与其像之前explicitly去获得各种material，不如直接用网络去预测OLAT images。能够说得通的地方就是spherical color gradient images含有丰富的信息，可以用DP decodes直接获得OLAT images。
2. task-specific VGG loss很具有启发性。

### brief summary
Given the two gradient images and light direction, xxx predicts the OLAT image relit by light from that direction. Then, OLAT images from dense light can be easily obtained and used with environment map to get the relit result.

### methodology
1. 在inference阶段可以只拍摄两张spherical color gradient images，但是在training阶段为了获得数据，肯定不可能这样复杂地去获得training数据。因此在拍摄training image时还是用常规的静止方式，并每11帧拍摄一张全亮的tracking frame用来算optical flow。
2. 每次输入网络的数据为两张gradient images和light direction的堆叠，因此为HxWx9。经过Unet来预测在这个输入的light direction下的OLAT结果。
3. loss：
   1. pretrained VGG loss
   2. specific VGG loss：作者认为pretrained VGG是在natural images上train的，无法捕捉当前task下的high frequency细节（specularity等），而specularity heavily depend on入射光方向，因此其想法是让整个network变得direction-aware。作者以VGG的network architecture预训练了一个网络，输入为gt image patches，输出为gt image拍摄时的light direction，即做了一个regression。可以理解为这个训练好的网络能够从输入image的specularity预测出light direction，而没有specularity的image难以预测出这个方向，那么用这个网络回归的方向去监督OLAT image的生成能够促使生成的image带有正确的specularity，这样才能保证回归的direction准确。
   3. sliding window pooling loss：作者认为即使在对training image进行optical flow校正后，还会存在noise，这种noise会导致网络无法较好地学习。因此首先用这个loss将gt image的optical flow校准到最佳。
