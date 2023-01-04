[TOC]
# Experience for Solving Problems

## about codes
1. 在安装oh-my-zsh时，先安装oh-my-zsh，再安装conda，否则conda的环境变量会找不到。
2. 执行python文件时报错：Failed to establish dbus connection。
可以执行 /etc/init.d/dbus
3. RuntimeError: Detected that PyTorch and torch_scatter were compiled with different CUDA versions. PyTocatter has CUDA version 10.1. Please reinstall the torch_scatter that matches your PyTorch install.（即package与package之间的cuda version不同）：
   在安装时指定cuda version。进入（https://pytorch-geometric.com/whl/torch-1.x.x+cuxxx.html）（cu111代表cuda version 11.1），下载对应的whl文件到本地目录，然后pip install xxx.whl即可

4. 当torch或者numpy出现nan时，首先考虑是不是除0，查看代码所有含有/的地方，在分母处加上一个小的数，如1e-5，防止除0.

## about system
1. 文件名不要带空格：首先，带空格无法在sell中表示；其次，带空格无法在代码中引用；最后，带空格的文件若是参与pc之间的共享，会造成传输识别错误或极度延迟。
2. 在shell中打开名字含有空格的file时，需要以"\ "代替那个空格

example：
``` shell
open /Applications/ReadPaper\ (Beta).app
```
3. 

# 3d vision
1. Iterate the faces or vertices of a mesh: using openmesh.
   Documentation: https://openmesh-python.readthedocs.io/_/downloads/en/latest/pdf/
2. Exported ".obj" files by blender can be opened as txt files:
   vs: 3D coordinates of vertices.
   vts: uv coordinates of vertices.
   f: a/b. a is the vertex index of mesh triangles. b is the texture-vertex index of texture triangles.
   vs和f的第一个数字代表着原来的mesh；当所有triangles被展开到uv坐标后，原来的mesh上的一个顶点可能对应着uv平面上的多个点，原来的mesh上的triangles都成为了uv平面上对应的triangles。新的triangles有新的vertex坐标，只不过从3D变为了2D。这个vertex坐标就是vts。因为mesh上的一个顶点展开到uv后可能对应多个顶点，所以vts的数量往往会比vs多。f的第二个数字就对应着，在uv平面的triangles的vertex索引。

3. open3d读取mesh.obj格式的mesh时，vertices的顺序与mesh.obj中一行行下来的顺序不一样。(大坑)
4. nvidiffrat专题：
   1. 关于坐标系
      1. 相机坐标系各坐标轴朝向：在opengl中，在相机坐标系中x朝右，y朝上，z朝向观察者（观察者从-z方向看物体）。从相机坐标系变换到clip space后，x、y的朝向不变，z的朝向取反，即朝向远离观察者的方向。而在nvdiffrast中的相机坐标系朝向和clip space朝向都遵循opengl；但在colmap中，相机坐标系的x朝右，y朝下，z朝向远离观察者的方向。
      2. 像素坐标系朝向：一般我们认为像素坐标系的原点在左上角，w朝右，h朝下(top-down)。但在nvdiffrast中认为像素坐标系的原点在左下角，w朝右，h朝上(bottom-up)。因此在texture_sampling中，给定image（tensor H*W*3）、uv坐标后，nvdiffrast认为image的坐标是nvdiffrast给的那样，所以用uv进行sample的时候遵循那样的坐标系，但是我们的uv坐标对应的是原来的那种坐标系，这样就会出现错误。我们需要做的就是把image的H维度进行flip。即对H*W*3的image进行，*torch.flip(image, [0])*
      3. 的y轴与nvdiffrast的y轴方向相反，我们可以将colmap的y坐标取反来align。但是又因为nvdiffrast的像素坐标系的y轴与colmap的y轴相反，故我们可以有两种方案：
         1. 在做clip space变换时，不对y轴操作。那么做完rasterize之后得到的rast_out的H轴则不用做flip，即其就是top-down系的坐标轴。
         ![principle](./images/coordinate.jpg)
         在colmap中一个3D点投影到像素平面后在$(x_0,y_0)$，但是由于没有对y轴取负，所以将这个坐标值放到opengl中时，会对应在$(x_0', y_0')$，即在图像空间中的位置改变了。但是opengl在存储图像时，因为坐标轴是bottom-up，所以在内存中存储时，按照坐标轴的顺序逻辑（第一行，也就是图像的最下面那一行），图像的最下面那一行被放到内存中的最前面，而图像的最上面那一行被放到了内存中的最后面。在把这个内存空间的数据转换为tensor时，根据tensor的存储逻辑，内存中最前面的那一行就是对应图像的最上面那一行，故这个转换之后，相当于对图像做了一个flip。所以在将opengl的图像object转换为tensor的图像object时，opengl的第一行（最下面）会被放到tensor的第一行（最上面），以此类推，opengl的最后一行（最上面）会被放到tensor的最后一行（最下面）。因此，最终得到的图像也就是标准的图像，与我们的输入对应。
         2. 在做clip space变换时，直接把y轴坐标取反，以适应nvdiffrast的规则。然后做完rasterize后得到的rast_out(batch, H, W, 4)，我们需要对H轴做一下flip，以得到我们常用的top-down坐标系。
      4.  

5.  total-rendering相关（数据main文件夹下的内容）
    1. pipeline会根据pcd.ply中的点云的范围划一个单元球以包住这个pcd.ply，也即包住我们感兴趣的区域（ROI， region of interest）。后续的渲染都会针对这个单元球内的content进行渲染，因此当我们不想渲染除了ROI外的区域时，可以对pcd.ply进行裁剪（用meshlab），把那些不感兴趣的部分的point cloud删除点，使单元球包不住，也就不会渲染了。而pcd.npz是根据pcd.ply生成的。pcd.npz中含有sparse volume，它首先通过在pcd.ply中的每个点处生成一个voxel（voxel的size由yaml中的configuration控制），然后在这个点周围再生成26个voxel（size也由yaml中的configuration控制，一般选择与刚才的voxel size相等）（为什么是26？想象一个立方体被分成27个小立方体，含有point的那个voxel相当于最中心的那个立方体）。生成了voxel后，就可以根据voxel的八个顶点的sdf来判断在这个voxel上的面片的形状，然后形成mesh。
    2. sparse/0文件夹里包含的是colmap跑出来的结果，包括points（世界坐标系下），camera intrinsic和image信息（不包含图片内容本身，只包含图像的名称，对应的intrinsic的id等）；sfm_ws_scaled里面包含的是对colmap跑出的结果进行scale恢复真实空间尺度（以m为单位）后的上述对应数据。
    3. results/.../mesh里面的以“mcc”为结尾的mesh就是最终重建完的mesh，它会被cp到main文件夹中，并重新命名为recon_mesh.ply。
    4. 
6. 感觉看到的scene比预想的要近，有两种可能性的bug：
   1. pose出错导致相机的位置离scene更近了
   2. 相机的焦距出错

7. non-manifold意思：没办法把mesh上所有triangle平铺到一个平面上（如果triangle之间两两以边相连，可以通过绕着这条边旋转让triangle位于同一个平面）。可能的情况有两种：
   1. mesh中存在多个连通分量，即有两块mesh不相连，这就无法判定两块mesh怎么铺到一个平面。
   2. 某两个triangles不是以edge相连而是以某个point（triangle的某个vertex）相连，这就无法以旋转edge的方式使其位于一个平面。
8. 


## When installing colmap on mac
After installing qt5 by ```brew install qt5```, the path of qt5 is at */opt/homebrew/Cellar/qt@5/5.15.5_1*. However, when compiling according to the colmap authority, the command is ```cmake .. -DQt5_DIR=/usr/local/opt/qt/lib/cmake/Qt5```, in which the qt5 path is not aligned with the actual path of our pc. This makes the cmake not capture the qt5, so when doing cmake, errors occur as *By not providing "FindQt5.cmake" in CMAKE_MODULE_PATH this project has asked CMake to find a package configuration file provided by "Qt5", but CMake did not find one.*. As a result, *colmap gui* cannot be used. The true step is:
``` shell
export CMAKE_PREFIX_PATH=/opt/homebrew/Cellar/qt@5/5.15.5_1  
cmake ../
```

## ubuntu解压分卷压缩文件
![zip](./images/zip.png)
1. 运行
```shell
cat 368_OK.zip* > data.zip # 星号前是分卷压缩的文件名相同部分
unzip -q data.zip # -q实现解压时不显示信息，提高压缩速度，当然可以去掉
```
2. 如果在unzip时出现```Damaged Zip archive```：
```shell
zip -FF broken.zip --out fixed.zip
unzip fixed.zip
```


## ubuntu设置系统代理
1. 打开zshrc
```shell
vi ~/.zshrc
```
2. 将以下内容加到zshrc中
```shell
export http_proxy=http://dnas.idr.ai:7890
export https_proxy=http://dnas.idr.ai:7890
export HTTP_PROXY=http://dnas.idr.ai:7890
export HTTPS_PROXY=http://dnas.idr.ai:7890
export no_proxy=http://gitlab.idr.ai
```

## ubuntu下载google driver上的public文件
一、利用wget（可能会下不了）
1. 右键点击想要下载的文件，点击*获取链接*，然后点击*复制链接*。
2. 链接的格式一般如```https://drive.google.com/file/d/10clmresw01amm1PaYeGjA8zsEBew-eX3/view?usp=share_link```，我们需要的是fileid=```10clmresw01amm1PaYeGjA8zsEBew-eX3```。
3. 然后在terminal运行：
```shell
wget --no-check-certificate 'https://docs.google.com/uc?export=download&id=FILEID' -O FILENAME
wget --load-cookies /tmp/cookies.txt "https://docs.google.com/uc?export=download&confirm=$(wget --quiet --save-cookies /tmp/cookies.txt --keep-session-cookies --no-check-certificate 'https://docs.google.com/uc?export=download&id=FILEID' -O- | sed -rn 's/.*confirm=([0-9A-Za-z_]+).*/\1\n/p')&id=FILEID" -O FILENAME && rm -rf /tmp/cookies.txt
```
4. 上面的命令一般用于下载小文件，下面的命令一般用于下载大文件（数据集之类的）。注意，两个命令中都存在FILEID（第一个命令有一处，第二个命令有两处）和FILENAME（两个命令分别各有一处），FILEID就是2中指的内容，FILENAME就是本地想要存取的路径和文件名（后缀一定要与源文件一样，名字可以改）。

二、利用curl
1. 打开google api[link](https://developers.google.com/oauthplayground/)。
2. 找到Drive API v3，点击https://...，然后点击Authorize APIs。
![api1](./images/api1.jpg)
3. 出现这个页面后，点击*Exchange authorization code for tokens*
![api2](./images/api2.jpg)
4. 点击后会出现access_token，这个就是我们需要使用的东西
![api3](./images/api3.jpg)
5. 在shell输入以下命令
``` shell
curl -H "Authorization: Bearer YYYYY" https://www.googleapis.com/drive/v3/files/XXXXX?alt=media -o ZZZZZ 
```
将YYYYY替换为上面的access_token（引号里面的内容）；将XXXXX替换为google drive的文件ID；将ZZZZZ替换为想要保存的文件名。

例如
``` shell
 curl -C - -H "Authorization: Bearer ya29.a0AX9GBdWFPjrWOcfNgEKaho4azox3ifXBfTSaOeO8LxMXJx_AgwumRCLTDIwL0xFBIQx4D3LdqkDRMrSW2qgeq9NyJTEhTA5nzU07REjPkrpo5J3_r1G7NddxQ37mbWfOC66Mq-Gi5eoGjBwqaDvIxbbM0VYVaCgYKAdYSARISFQHUCsbCkw2SFqJf7z2hbq5-wzVuEQ0163" https://www.googleapis.com/drive/v3/files/1aVhXp2TPBNCPCEoIdPxvbxc841NEzxdG?alt=media -o   hro.zip
```