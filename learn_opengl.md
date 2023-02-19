
# Notes for learning opengl

## GLFW and GLAD
1. GLFW是一个用于帮助opengl从os独立出来的library，使得opengl的各种函数不受os的限制，走向更底层硬件。
2. opengl只是指定了渲染的方式，定义了context，要实现rendering，需要硬件去执行。opengl的drivers有很多，每个drivers的function存放位置不同，容易导致不兼容而找不到渲染函数。GLAD就是为了找到这些driver function而诞生的，它也是独立于os的。

## Hello Window
1. include时GLAD必须在GLFW之前，因为GLFW需要用到其中的内容。
2. window中设置的size指的是整个opengl运行窗口的大小，而viewport中设置的渲染窗口size指的是在整个opengl运行窗口中用于显示渲染部分的窗口大小（opengl运行窗口除了渲染窗口还可以有一些setting的部分）。
3. 在opengl运行时所有的坐标都位于[-1, 1]，因此为了让渲染结果正确地出现在屏幕上，指定了viewport的width和height，使得[-1, 1]能够线性地映射到[width, height]上，正确地在屏幕上显示。
4. double buffer：应用程序使用单缓冲绘图时可能会存在图像闪烁的问题。 这是因为生成的图像不是一下子被绘制出来的，而是按照从左到右，由上而下逐像素进行渲染并显示在屏幕上。最终图像不是在瞬间显示给用户，而是通过一步一步生成并显示的，这会导致渲染的结果很不真实。为了规避这些问题，我们应用双缓冲渲染窗口应用程序。front buffer保存着最终输出的图像，它会在屏幕上显示；而所有的的渲染指令都会在back buffer进行。当所有的渲染指令执行完毕后，我们交换(Swap)前缓冲和后缓冲，这样图像就立即呈显出来，之前提到的不真实感就消除了。本质上是用渲染完后再显示替代边渲染边显示。glfwSwapBuffers就是交换这两个buffer。

## Hello Triangle
1. primitives：图元。用其来告诉opengl我们要画什么形状（triangle、points等），primitives有*GL_POINTS*, *GL_TRIANGLES*, *GL_LINE_STRIP*。
2. rendering pipeline
   1. 宏观：
      1. 将3D坐标转换为2D坐标。
      2. 将2D坐标转换为带颜色的pixels。
   2. 微观
      1. pipeline：vertex shader -> shape assembly -> geometry shader -> rasterization -> fragment shader -> test and blending。
      2. 概念
         1. 每一步用到上一步的输出。
         2. 每一步都可以并行（parallel）运行。
         3. 每一步内运行的小programs被称为shader。
      3. 作用
         1. vertex shader：一个vertex代表着这个vertex的所有attributes（坐标，颜色或者是neural textures都可以）。vertex shader用于对vertex的坐标进行处理（如将坐标归一化到NDC，opengl只处理坐标值位于[-1, 1]的坐标），并对其attributes做一些处理。**vertex shader每次只接收一个vertex并处理**。
         2. shape assembly：根据选择的primitives将vertices进行分类并assembly。
         3. geometry shader：vertex shader的输出optionally输入其。其用于将之前的primitives进行加工形成新的shape。
         4. rasterization：将NDC坐标进行CLIP并project到screen上。
         5. fragment shader：rasterization只是获得了每个pixel内对应着哪些3D vertices，但是每个vertices都有很多attributes。在这个stage，就是利用这些vertices的attributes计算最终的color（如PBR在这个stage进行）。
         6. test and blending：根据pixels内所有点的depth和alpha选择显示的内容，opaque时只显示最前面（depth最小）的内容，否则要将vertices的内容进行blending。
   3. code层面的流程
      1. init window。
      2. 写出vertex shader和fragment shader的GLSL code。
      3. 定义vertex shader和fragment shader -> 将写的GLSL code送入定义的shader -> 对shader进行编译 -> 检查编译是否成功。
      4. 定义program，将vertex shader和fragment shader attach到program上，然后对program进行link（使得前一个shader的输出能够输入到后一个shader），检查link是否成功。link完没有error后，delete之前定义的两个shader。
      5. 定义VAO和VBO，先绑定VAO到vertex array上，然后绑定VBO到当前绑定到vertex array的VAO上 -> 将数据送入VBO中 -> 定义VAO理解VBO中数据的方式（有了pointer就是有了数据的真实内存地址，因此后续可以unbind VBO，不需要VBO再提供什么信息） -> 使得刚定义的pointer使能 -> unbind VBO -> unbind VAO。
      6. 在loop中use program -> bind VAO以获得想要使用的数据 -> draw
      7. 完成project后，delete VAO、VBO、program
   4. 注意事项：所有申请的buffer，定义的object在project结束的时候必须全部释放。一些申请的buffer绑定到array上并定义pointer后就可以从array上unbind，以供后续的buffer bind并定义pointer。

3. 在opengl中，vertex shader和fragment shader必须由我们定义，其没有default one。
4. opengl的screen coordinates的y轴向上，x轴向右，原点在screen中心。
5. 任何buffer object都有一个唯一的buffer ID，可以由我们来指定。通过调用*glGenBuffers*来生成指定ID的buffer。通过buffer ID生成buffer后，我们无法直接通过buffer ID去操纵这个memory，必须将这个buffer bind（捆绑）到opengl中的buffer容器中，opengl共有8种不同的buffer容器，对应不同的数据类型。通过调用*glBindBuffer*函数可以实现捆绑，这样我们就可以通过直接调用这个容器来操纵buffer。一种类型的buffer容器每次只能容纳一组数据，每次调用bind时，就是将一组数据捆绑到上面，之前捆绑的数据失效，因此**每次需要用到新的数据时都要重新捆绑**。当捆绑VBO捆绑完成后，我们就可以用*glBufferData*将CPU中的数据送入这个buffer。
6. 将数据送入vertex shader就是在GPU中为数据创造buffer，其称为vertex buffer objects（VBO）。从CPU传递数据到GPU比较慢，这个buffer能够保证数据立马能够被GPU取到。通过调用*glBufferData*将定义在CPU中的数据装入指定的bound buffer。

## shaders
1. 输入vertex shader的vertex attributes有最大数量限制，不过一般至少大于16个。
2. 可定义的数据类型：
首先是C语言中的基础变量都可以定义
可用.x .y .z .w去query vec各个分量
``` 
vecn # 默认的float类型向量，长度为n
bvecn # 长度为n的bool类型向量
ivecn # 长度为n的int类型向量
uvecn # 长度为n的uint类型向量
dvecn # 长度为n的类型向量，每个分量含有两个值
```
3. swizzling功能，自由定义vec
``` 
vec2 someVec;
vec4 differentVec = someVec.xyxx;
vec3 anotherVec = differentVec.zyw;
vec4 otherVec = someVec.xxxx + anotherVec.yxzy;
```
4. 初始化定义向量
``` 
vec2 vect = vec2(0.5, 0.7);
vec4 result = vec4(vect, 0.0, 0.0);
vec4 otherResult = vec4(result.xyz, 1.0);
```
5. 前一个shader的out变量若与后一个shader的in变量在定义时候的变量名称一样，opengl会自动将值传递。
6. 可以用uniform来自由改动一个整个shader program中定义的变量。在一个shader program中定义的uniform变量，可以在这个program中的任何地方调用，但不能被其他program调用。
7. **注意：如果自定义了一个uniform变量，但是并没在program的任何一处地方调用它，则在编译时会默认将这个定义的变量移除**
8. opengl的内核是C语言，不支持函数重载（一个函数可以接收多种输入组合，输出多种输出组合，即同名函数定义多种用法）。因此我们在设置不同类型的值时，必须用不同的function，无法用一个function。以下为多种数据设置function。

```
glf() # 设置float类型变量的值
gli() # 设置int类型变量的值
glui() # 设置uint类型变量的值
gl3f() # 设置3 float类型变量的值
glfv() # 设置float vector类型变量的值
```
9. 如果定义了不同的顶点颜色，则在设置模式为只画线时，每条线上的颜色会被做插值处理；如果设置模式为整个多边形进行颜色填充时，整个多边形会被做插值处理。