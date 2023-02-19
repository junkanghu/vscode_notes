[toc]

# Chapter 9: Physically Based Shading
1. Physics of Light
   
   1. Particles
      1. 在正常的气压条件下，气体分子的分布是任意且无关的（称之为isolated particles）。在这样的条件下，不同的分子接收light energy后进行scatter后的light waves的phase都是random的，即incoherent。在这样的情况下，它们的总能量E(irradiance)是linearly added的，即有n个monochromatic wave，则它们的E为$nE_{single}$。
      2. 如果多个气体分子占据的空间**被压缩到小于light wavelength**，则不同particle scatter后的light waves是coherent的，即它们的phase都相同，此时为constructive interference。因此在一个单位立方体中，若particles的数量是一定的，将多个particles聚在一起形成多个clusters会增强scattering intensity；clusters中的particles越多（少几个clusters，增大单个cluster中的particle数量），scattering intensity也会越大。这解释了云和雾scatter非常厉害的原因，因为它们将water分子cluster起来。
      3. *particles*通常既指isolated particles，又指multi-molecule clusters。
      4. 空气（atmosphetic particles）造成的scattering称为*Rayleigh scattering*，固体particles造成的scattering称为*Tyndall scattering*。
   2. Media
      1. homogeneous medium：组成它的molecules uniformly分布（可以不是perfectly regular），例如crystal。还有一些液体（水）和非晶态固体，如果组成它们的所有molecules都是相同的且它们不存在缝隙或气泡，则它们在光学上也是homegeneous的，即也能够产生严格意义上的homogeneous medium产生的光学效果。
      2. homegeneous medium scatter后的light全部朝向original传播方向，看起来就好像没有发生scatter，因为除了这个方向，其他的方向都发生了destructive interference。
      3. nonhomogeneous media可被建模为homogeneous media与particles的混合，homogeneous media中的任何分布的改变都会导致scattering效应的改变。
      4. scattering和absorption都是scale-dependent的，在鱼缸里看不到水的absorption效应，但是海里面可以；在近处看不到air的scattering效应，但是远处可以。
      5. medium的appearance通常由combination of scattering and absorption造成。white color尤其是combination of high scattering and low absorption。
   3. 
2. 