# How does Lumos generate dataset

1. 已经有mesh（含有texture albedo）、haircut、clothing and accessories。
2. 每次randomly sample一组face scan（获得其含有albedo的mesh）、a haircut、a garment、a pair of eyeglasses(以0.2的概率决定是否要取眼镜)、a headset（以0.05的概率判断是否要选择耳机）。将这些attributes加到3D mesh上（生成一个synthetic human），然后sample camera pose（horizontally -45-45度，vertically -22.5-22.5度）。对于每个camera pose下的rendering结果，分别采用两个environment map在Arnold下基于PBR生成一对paired-illuminated image（512*512，hdr format），此外，也生成该view下的normal map（两张，1张with lenses，一张w/o lenses）及albedo map。
- 关于如何用environment light做relighting：已知mesh的确切位置，也已知camera pose，那么就可以根据每个pixel进行ray marching得到intersection，然后在intersection处进行PBR（light为environment map）得到relighting结果。
- 关于如何在relighted image中增加specular分量：利用Arnold中的aiStandardSurface shader去manipulate facial material，可以在face的不同部位使用不同程度的specular和roughness，然后进行PBR rendering。
3. 2中用到的HDR images搜集到的有1536（training）+70（testing）张。此外，为了增加lighting的variation，对HDR images进行了random rotation和flips（都是horizontally）。
4. 每一个data sample包含：一对paired images（512*512，hdr format；一张用作net的input，这个input和target environment map输入net后得到一张predicted image，另一张就是gt，与predicted算loss），normal map，albedo map，一张target environment map。（在算relative lighting consistency loss时，从environment map sets中任意取两张去relighting然后算loss，跟input和gt的environment map无关）


Code:
1. specular light map的weight都是single channel；rendering residual的weight也都是single channel。
2. albedo、normal net的output layer用的都是Relu，不是sigmoid。
3. gan loss没有用TR中的方式，直接用了cyclegan的patch。
4. network中的pooling方式没有与TR一样
5. syn2real中的gan loss，G方面通过两个完整的env map计算，D方面通过计算两个predicted与input data来计算。