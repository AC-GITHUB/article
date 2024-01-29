# OpenGL快速入门 - 个人文章 - SegmentFault 思否
概述
--

### OpenGL

OpenGL是渲染2D、3D矢量图形硬件的一种软件接口。本质上说，它是一个3D图形和模型库，具有高度的可移植性，并且具有非常快的渲染速度。OpenGL并不是一种语言，而是更像一个C运行时函数库。它提供了一些预包装的功能，帮助开发人员编写功能强大的三维应用程序。 OpenGL可以再多种操作系统平台上运行，例如各种版本的Windows、UNIX/Linux、Mac OS 和 OS/2等。如今，OpenGL广泛流行于游戏、医学影像、地理信息、气象模拟等领域，是高性能图像和交互性场景处理的工业标准。  
OpenGL的高效实现（利用了图形加速硬件）存在于Windows，部分UNIX平台和Mac OS。这些实现一般由显示设备厂商提供，而且非常依赖于该厂商提供的硬件。

### OpenGL ES与WebGL

OpenGL ES (OpenGL for Embedded Systems) 是 OpenGL 三维图形 API 的子集，针对手机、PDA和游戏主机等嵌入式设备而设计

WebGL（全写Web Graphics Library）是一种3D绘图协议，这种绘图技术标准允许把JavaScript和OpenGL ES 结合在一起，通过增加OpenGL ES 的一个JavaScript绑定，WebGL可以为HTML5 Canvas提供硬件3D加速渲染，这样Web开发人员就可以借助系统显卡来在浏览器里更流畅地展示3D场景和模型了，还能创建复杂的导航和数据视觉化。

### OpenGL发展史

OpenGL是个开放的标准，虽然它由SGI(美国硅图公司)首创，但它的标准并不是控制在SGI的手中，而是由OpenGL体系结构审核委员会(ARB)所掌管。 ARB由SGC、DEC、IBM、Intel和Microsoft等著名公司1992年创立，后来又陆续添加了nVidia、ATI等图形芯片领域的巨擎。 ARB每隔4年开一次会，对OpenGL规范进行维护和改善，并出台计划对OpenGL标准进行升级，使OpenGL一直保持与时代的同步。

2006年，SGIG公司把OpenGL标准的控制从ARB移交给一个新的工作组：Khronos小组(www.khronos.org)。 Khronos是一个由成员提供资金的行业协会，专注于开放媒体标准的创建和维护。

软件安装
----

在正式开始学习OpenGL之前，我们需要先配置好OpenGL的软件环境。

### IDE

支持OpenGL的IDE有很多，OpenGL的开发环境我们选择的是Visual Studio，可以从[Visual Studio官网](https://link.segmentfault.com/?enc=T4AKGABbvLg%2F%2FjiqsZfY%2Bg%3D%3D.DUv04L8pLpsJFxQPr7BoSmwTz74ClEY1roxglBcBbMKa22RC3AU%2FRbms88pm4epD)下载最新的版本。

### GLFW

OpenGL是一个图形库，而要画图，就需要先创建一个窗口。不幸的是，OpenGL并没有提供创建窗口的功能，必须自己创建窗口。而创建窗口在每一个操作系统上都不同的（在Windows上代码量也不少），为了方便，我们会使用一个窗口库来简化这一过程。常用的OpenGL窗口库有GLUT、GLFW和SDL，此处为我们选择使用得比较多的GLFW。

Visual Studio对于OpenGL（gl.h）只支持到1.1，而我们使用的是OpenGL 3.3。但是，OpenGL是由显卡支持的，显卡已经提供了我们需要的OpenGL函数。因此就需要在运行程序时动态地获取函数地址。在Windows下，以glGenBuffers为例，大概是这样的：

```
#include <windows.h>
#include <GL/gl.h>
...

typedef void \* (*WGLGETPROCADDRESS)(const char *);
typedef void (*GLGENBUFFERS)(GLsizei, GLsizei *);

HMODULE hDll = LoadLibrary("opengl32.dll");
WGLGETPROCADDRESS wglGetProcAddress = (WGLGETPROCADDRESS)GetProcAddress(hDll, "wglGetProcAddress");

GLGENBUFFERS glGenBuffers = (GLGENBUFFERS)wglGetProcAddress("glGenBuffers");


GLuint vbo;
glGenBuffers(1, &vbo);
```

当然，GLFW可以从它的[官方网站](https://link.segmentfault.com/?enc=fG3q5ViMOMmkFIe2JrkuZA%3D%3D.y5htKJP318ljUrX21aLxFAGBfAdk%2FffInv%2BuYJNNSk0%3D)上下载。然后，你可以直接下载它的binaries，或者自己使用CMake编译。如果自己使用CMake编译，可以参考下面的文章： [GLFW 环境配置](https://link.segmentfault.com/?enc=4Fs7y%2BsJF3sHbuqhZJ3kAQ%3D%3D.kSLRp8tAXkKtkkWCiW3xiW5NJzL1ogsKpgnI8q5HYXGg7D%2BypfAuCcedOKysN%2BqDySZ3pSHUMP9bVJrDxFjZwczITd1wvTabJ96%2B%2FjpWjyv6PgtsuhcqN%2FqVsp2nF3AI)，[创建窗口](https://link.segmentfault.com/?enc=Gnyo%2BbgkUk1YTQnduS14xw%3D%3D.9gRJ%2FTNwOuJYrCKdDJniG%2BE2qWiHn54xIHOFl16mQ%2BtyyflTx9lafJd0Chm%2BNhsbCIEJmr4hWoKnpu%2Bhx4zdLo%2BFi%2BtZUJ%2FK8hql3tAbSthZewPlvZcECmb7KZW12RIi)

如果下载已经编译好的binaries，解压并打开，可以找到一个include文件夹和若干lib-xxxx文件夹（xxxx是编译器名）。include文件夹里含有一个GLFW文件夹，里面有glfw3.h（还有一个glfw3native.h不用管）

详细文档可以参考官方的介绍，或者直接从[GLFW官方](https://link.segmentfault.com/?enc=fuyDERi%2F15qAdYTxBUaYeQ%3D%3D.T4I5ltYy%2F8tHkCzXZCMujMRNHf22mKbLv70UCmMZH6NG4h5BGVVaZggSHZioAtaz)网站的下载页上获取源代码包。

OpenGL基础知识
----------

### 数据类型和函数名

OpenGL的数据类型定义可以与其它语言一致，但建议在ANSI C下最好使用以下定义的数据类型，例如GLint、GLfloat等。

| 前缀 | 数据类型 | 相应C语言类型 | OpenGL类型 |
| --- | --- | --- | --- |
| b | 8-bit integer | signed char | GLbyte |
| s | 16-bit integer | short | GLshort |
| i | 32-bit integer | long | GLint,GLsizei |
| f | 32-bit floating-point | float | GLfloat,GLclampf |
| d | 64-bit floating-point | double | GLdouble,GLclampd |
| ub | 8-bit unsigned integer | unsigned char | GLubyte,GLboolean |
| us | 16-bit unsigned integer | unsigned short | GLushort |
| ui | 32-bit unsigned integer | unsigned long | GLuint,GLenum,GLbitfield |

从上表可以看出，OpenGL的库函数命名方式很有规律，了解这种规律后阅读和编写程序都比较容易方便。  
首先，每个库函数有前缀gl、glu、glx或aux，表示此函数分属于基本库、实用库、X窗口扩充库或辅助库，其后的函数名头字母大写，后缀是参数类型的简写，取i、f。例如：

```
glVertex2i(2,4);
glVertex3f(2.0,4.0,5.0);
```

如上，有的函数参数类型后缀前带有数字2、3、4。其中，2代表二维，3代表三维，4代表alpha值。

除此之外，有些OpenGL函数最后带一个字母v，表示函数参数可用一个指针指向一个向量（或数组）来替代一系列单个参数值。下面两种格式都表示设置当前颜色为红色，二者等价。

```
glColor3f(1.0,0.0,0.0);
float color_array\[\]={1.0,0.0,0.0};
glColor3fv(color_array);

```

除了以上基本命名方式外，还有一种带“_”星号的表示方法，例如glColor_()，它表示可以用函数的各种方式来设置当前颜色。同理，glVertex*v()表示用一个指针指向所有类型的向量来定义一系列顶点坐标值。

### 示例

例如有下面一个示例程序，也是一个初学者学习的第一个示例程序。源码如下：

```

 
#include <GLUT/GLUT.h>
#include <OpenGL/OpenGL.h>
 

void init() {
    glClearColor(0.1, 0.1, 0.4, 0.0);
    glShadeModel(GL_SMOOTH);
}
 

void display() {

    
    glClear(GL\_COLOR\_BUFFER_BIT);
 
    
    glBegin(GL_TRIANGLES);
    glColor3f(1, 0, 0);
    glVertex3f(-1, -1, -5);
    glColor3f(0, 1, 0);
    glVertex3f(1, -1, -5);
    glColor3f(0, 0, 1);
    glVertex3f(0, 1, -5);
    glEnd();
    
    glFlush();
}
 

void reshape(int w, int h) {
    glViewport(0, 0, w, h);
    glMatrixMode(GL_PROJECTION);
    glLoadIdentity();
    gluPerspective(60.0, (GLfloat)w/(GLfloat)h, 0.1, 100000.0);
    glMatrixMode(GL_MODELVIEW);
    glLoadIdentity();
}
 
int main(int argc, const char \* argv\[\]) {
    
    glutInit(&argc, const_cast<char **>(argv));
    glutInitDisplayMode(GLUT\_SINGLE | GLUT\_RGB); 
 
    
    glutInitWindowSize(500, 500);
    glutInitWindowPosition(100, 100);
    glutCreateWindow(argv\[0\]);
 
    init();
    glutReshapeFunc(reshape);
    glutDisplayFunc(display);
 
    
    glutMainLoop();
    return 0;
}
```

运行效果如下：  
![](https://segmentfault.com/img/remote/1460000017246737?w=368&h=350)

### 几何图形绘制

在空间直角坐标系中，任意一点可用一个三维坐标矩阵\[x y z\]表示。如果将该点用一个四维坐标的矩阵\[Hx Hy Hz H\]表示时，则称为齐次坐标表示方法。在齐次坐标中，最后一维坐标H称为比例因子。  
　　在OpenGL中，二维坐标点全看作三维坐标点，所有点都用齐次坐标来描述，统一作为三维齐次点来处理。每个齐次点用一个向量(x, y, z, w)表示，其中四个元素全不为零。齐次点具有下列几个性质：  
　　1）如果实数a非零，则(x, y, x, w)和(ax, ay, az, aw)表示同一个点，类似于x/y = (ax)/( ay)。  
　　2）三维空间点(x, y, z)的齐次点坐标为(x, y, z, 1.0)，二维平面点(x,y)的齐次坐标为(x, y, 0.0, 1.0)。  
　　3）当w不为零时，齐次点坐标(x, y, z, w)即三维空间点坐标(x/w, y/w, z/w)；当w为零时，齐次点(x, y, z, 0.0)表示此点位于某方向的无穷远处。  
　　注意：OpenGL中指定w大于或等于0.0。

#### 几何图形

在集合图形中，会涉及到几个概念：

##### 点

用浮点值表示的点称为顶点（Vertex）。所有顶点在OpenGL内部计算时都作为三维点处理，用二维坐标(x, y)定义的点在OpenGL中默认z值为0。所有顶点坐标用齐次坐标(x, y, z, w) 表示，如果w不为0.0，这些齐次坐标表示的顶点即为三维空间点(x/w, y/w, z/w)。编程者可以自己指定w值，但很少这样做。一般来说，w缺省为1.0。

##### 线

在OpenGL中，线代表线段（Line Segment），不是数学意义上的那种沿轴两个方向无限延伸的线。这里的线由一系列顶点顺次连结而成，有闭合和不闭合两种。  
![](https://segmentfault.com/img/remote/1460000017246738)

##### 多边形

OpenGL中定义的多边形是由一系列线段依次连结而成的封闭区域。这些线段不能交叉，区域内不能有空洞，多边形必须在凸多边形，否则不能被OpenGL函数接受。  
![](https://segmentfault.com/img/remote/1460000017246739?w=568&h=190)

#### 绘制图元

##### 定义顶点

在OpenGL中，所有几何物体最终都由有一定顺序的顶点集来描述的。函数glVertex{234}{sifd}v可以用二维、三维或齐次坐标定义顶点。例如：

```
glVertex2s(2,3);
glVertex3d(0.0,1.0,3.1414926535);
glVertex4f(2.4,1.0,-2.2,2.0);
GLfloat pp\[3\]={5.0,2.0,10.2};
glVertex3fv(pp);
```

第一例子表示一个空间顶点(2, 3, 0)，第二个例子表示用双精度浮点数定义一个顶点，第三个例子表示用齐次坐标定义一个顶点，其真实坐标为(1.2, 0.5, -1.1)，最后一个例子表示用一个指针（或数组）定义顶点。

##### 几何图元

在实际应用中，通常用一组相关的顶点序列以一定的方式组织起来定义某个几何图元，而不采用单独定义多个顶点来构造几何图元。在OpenGL中，所有被定义的顶点必须放在glBegain()和glEnd()两个函数之间才能正确表达一个几何图元或物体，否则，glVertex*()不完成任何操作。例如：

```
glBegin(GL_POLYGON);
　　　　glVertex2f(0.0,0.0);
　　　　glVertex2f(0.0,3.0);
　　　　glVertex2f(3.0,3.0);
　　　　glVertex2f(4.0,1.5);
　　　　glVertex2f(3.0,0.0);
glEnd();
 
```

以上这段程序定义了一个多边形，如果将glBegin()中的参数GL\_POLYGON改为GL\_POINTS，则图形变为一组顶点（5个）。  
![](https://segmentfault.com/img/remote/1460000017246740)

##### 图元标志

点函数glBegin(GLenum mode)标志描述一个几何图元的顶点列表的开始，其参数mode表示几何图元的描述类型。所有类型及说明见下表：

| 类型 | 说明 |
| --- | --- |
| GL_POINTS | 单个顶点集 |
| GL_LINES | 多组双顶点线段 |
| GL_POLYGON | 单个简单填充凸多边形 |
| GL_TRAINGLES | 多组独立填充三角形 |
| GL_QUADS | 多组独立填充四边形 |
| GL\_LINE\_STRIP | 不闭合折线 |
| GL\_LINE\_LOOP | 闭合折线 |
| GL\_TRAINGLE\_STRIP | 线型连续填充三角形串 |
| GL\_TRAINGLE\_FAN | 扇形连续填充三角形串 |
| GL\_QUAD\_STRIP | 连续填充四边形串 |

上面表用几何图形表示的化，如下图。  
![](https://segmentfault.com/img/remote/1460000017246741?w=559&h=502)

在glBegin()和glEnd()之间最重要的信息就是由函数glVertex*()定义的顶点，必要时也可为每个顶点指定颜色、法向、纹理坐标或其他，即调用相关的函数，如下表。

| 函数 | 说明 |
| --- | --- |
| glVertex*() | 设置顶点坐标 |
| glColor*() | 设置当前颜色 |
| glIndex*() | 设置当前颜色表 |
| glNormal*() | 设置法向坐标 |
| glCallList(),glCallLists() | 执行显示列表 |
| glTexCoord*() | 设置纹理坐标 |
| glEdgeFlag*() | 控制边界绘制 |
| glMaterial*() | 设置材质 |

看一个示例：

```
glBegin(GL_POINTS);
　　　　glColor3f(1.0,0.0,0.0); 
　　　　glVertex(...);
　　　　glColor3f(0.0,1.0,0.0); 
　　　　glColor3f(0.0,0.0,1.0); 
　　　　glVertex(...);
　　　　glVertex(...);
　　glEnd();
```

#### 示例

为了更好的理解OpenGL几何图形的绘制，下面看一个综合的示例。

```
#include <GLUT/GLUT.h>
#include <OpenGL/OpenGL.h>


void init() {
    glClearColor(0.1, 0.1, 0.4, 0.0);
    glShadeModel(GL_SMOOTH);
}
 
 
void DrawMyObjects(void){
    
    glBegin(GL_POINTS);
        glColor3f(1.0,0.0,0.0);
        glVertex2f(-10.0,11.0);
        glColor3f(1.0,1.0,0.0);
        glVertex2f(-9.0,10.0);
        glColor3f(0.0,1.0,1.0);
        glVertex2f(-8.0,12.0);
    glEnd();
 
 
    
    glBegin(GL_LINES);
        glColor3f(1.0,1.0,0.0);
        glVertex2f(-11.0,8.0);
        glVertex2f(-7.0,7.0);
        glColor3f(1.0,0.0,1.0);
        glVertex2f(-11.0,9.0);
        glVertex2f(-8.0,6.0);
    glEnd();
     
 
    
    glBegin(GL\_LINE\_STRIP);
        glColor3f(0.0,1.0,0.0);
        glVertex2f(-3.0,9.0);
        glVertex2f(2.0,6.0);
        glVertex2f(3.0,8.0);
        glVertex2f(-2.5,6.5);
    glEnd();
 
 
    
    glBegin(GL\_LINE\_LOOP);
        glColor3f(0.0,1.0,1.0);
        glVertex2f(7.0,7.0);
        glVertex2f(8.0,8.0);
        glVertex2f(9.0,6.5);
        glVertex2f(10.3,7.5);
        glVertex2f(11.5,6.0);
        glVertex2f(7.5,6.0);
    glEnd();
 
 
    
    glBegin(GL_POLYGON);
        glColor3f(0.5,0.3,0.7);
        glVertex2f(-7.0,2.0);
        glVertex2f(-8.0,3.0);
        glVertex2f(-10.3,0.5);
        glVertex2f(-7.5,-2.0);
        glVertex2f(-6.0,-1.0);
    glEnd();
 
 
    
    glBegin(GL_QUADS);
        glColor3f(0.7,0.5,0.2);
        glVertex2f(0.0,2.0);
        glVertex2f(-1.0,3.0);
        glVertex2f(-3.3,0.5);
        glVertex2f(-0.5,-1.0);
        glColor3f(0.5,0.7,0.2);
        glVertex2f(3.0,2.0);
        glVertex2f(2.0,3.0);
        glVertex2f(0.0,0.5);
        glVertex2f(2.5,-1.0);
    glEnd(); 
 
    
    glBegin(GL\_QUAD\_STRIP);
        glVertex2f(6.0,-2.0);
        glVertex2f(5.5,1.0);
        glVertex2f(8.0,-1.0);
        glColor3f(0.8,0.0,0.0);
        glVertex2f(9.0,2.0);
        glVertex2f(11.0,-2.0);
        glColor3f(0.0,0.0,0.8);
        glVertex2f(11.0,2.0);
        glVertex2f(13.0,-1.0);
        glColor3f(0.0,0.8,0.0);
        glVertex2f(14.0,1.0);
    glEnd();
 
 
    
 
    glBegin(GL_TRIANGLES);
        glColor3f(0.2,0.5,0.7);
        glVertex2f(-10.0,-5.0);
        glVertex2f(-12.3,-7.5);
        glVertex2f(-8.5,-6.0);
        glColor3f(0.2,0.7,0.5);
        glVertex2f(-8.0,-7.0);
        glVertex2f(-7.0,-4.5);
        glVertex2f(-5.5,-9.0);
    glEnd();
 
    
    glBegin(GL\_TRIANGLE\_STRIP);
        glVertex2f(-1.0,-8.0);
        glVertex2f(-2.5,-5.0);
        glColor3f(0.8,0.8,0.0);
        glVertex2f(1.0,-7.0);
        glColor3f(0.0,0.8,0.8);
        glVertex2f(2.0,-4.0);
        glColor3f(0.8,0.0,0.8);
        glVertex2f(4.0,-6.0);
    glEnd();
 
  
    
    glBegin(GL\_TRIANGLE\_FAN);
        glVertex2f(8.0,-6.0);
        glVertex2f(10.0,-3.0);
        glColor3f(0.8,0.2,0.5);
        glVertex2f(12.5,-4.5);
        glColor3f(0.2,0.5,0.8);
        glVertex2f(13.0,-7.5);
        glColor3f(0.8,0.5,0.2);
        glVertex2f(10.5,-9.0);
    glEnd();
}
 

void display() {
    
    glClear(GL\_COLOR\_BUFFER_BIT);
    DrawMyObjects();
    
    glFlush();
}
 
 

void reshape(int w, int h) {
    glViewport(0, 0, w, h);
    glMatrixMode(GL_PROJECTION);
    glLoadIdentity();
    gluPerspective(60.0, (GLfloat)w/(GLfloat)h, 0.1, 100000.0);
    glMatrixMode(GL_MODELVIEW);
    glLoadIdentity();
    gluLookAt(0, 0, 25, 0, 0, -1, 0, 1, 0);
}
 
int main(int argc, const char \* argv\[\]) {
    
    glutInit(&argc, const_cast<char **>(argv));
    glutInitDisplayMode(GLUT\_SINGLE | GLUT\_RGB);
 
    
    glutInitWindowSize(500, 500);
    glutInitWindowPosition(100, 100);
    glutCreateWindow(argv\[0\]);
 
    init();
    glutReshapeFunc(reshape);
    glutDisplayFunc(display);
 
    
    glutMainLoop();
    return 0;
}
```

运行效果如下图：  
![](https://segmentfault.com/img/remote/1460000017246742?w=1002&h=996)

### 坐标系及坐标变换

#### 右手坐标系

openGL采用右手坐标系，关于左右手坐标系区别可参考下图。  
![](https://segmentfault.com/img/remote/1460000017246743?w=220&h=165)

#### 坐标空间

openGL 空间分为：

*   局部空间(Local Space，或者称为物体空间(Object Space))
*   世界空间(World Space)
*   观察空间(View Space，或者称为视觉空间(Eye Space))
*   裁剪空间(Clip Space)
*   屏幕空间(Screen Space)

##### 局部空间

局部空间是指物体所在的坐标空间，即对象最开始所在的地方。想象你在一个建模软件中创建了一个立方体。你创建的立方体的原点有可能位于(0, 0, 0)，即便它有可能最后在程序中处于完全不同的位置。甚至有可能你创建的所有模型都以(0, 0, 0)为初始位置。所以，你的模型的所有顶点都是在局部空间中，它们相对于你的物体来说都是局部的。

##### 世界空间

如果我们将我们所有的物体导入到程序当中，它们有可能会全挤在世界的原点(0, 0, 0)上，这并不是我们想要的结果。我们想为每一个物体定义一个位置，从而能在更大的世界当中放置它们。世界空间中的坐标正如其名：是指顶点相对于世界的坐标。如果你希望将物体分散在世界上摆放（特别是非常真实的那样），这就是你希望物体变换到的空间。物体的坐标将会从局部变换到世界空间；该变换是由模型矩阵(Model Matrix)实现的。  
模型矩阵是一种变换矩阵，它能通过对物体进行位移、缩放、旋转来将它置于它本应该在的位置或朝向。你可以将它想像为变换一个房子，你需要先将它缩小（它在局部空间中太大了），并将其位移至郊区的一个小镇，然后在y轴上往左旋转一点以搭配附近的房子。你也可以把上一节将箱子到处摆放在场景中用的那个矩阵大致看作一个模型矩阵；我们将箱子的局部坐标变换到场景/世界中的不同位置。

##### 观测空间

观察空间经常被人们称之OpenGL的摄像机(Camera)（所以有时也称为摄像机空间(Camera Space)或视觉空间(Eye Space)）。观察空间是将世界空间坐标转化为用户视野前方的坐标而产生的结果。因此观察空间就是从摄像机的视角所观察到的空间。而这通常是由一系列的位移和旋转的组合来完成，平移/旋转场景从而使得特定的对象被变换到摄像机的前方。这些组合在一起的变换通常存储在一个观察矩阵(View Matrix)里，它被用来将世界坐标变换到观察空间。

##### 裁剪空间

在一个顶点着色器运行的最后，OpenGL期望所有的坐标都能落在一个特定的范围内，且任何在这个范围之外的点都应该被裁剪掉(Clipped)。被裁剪掉的坐标就会被忽略，所以剩下的坐标就将变为屏幕上可见的片段。这也就是裁剪空间(Clip Space)名字的由来。  
因为将所有可见的坐标都指定在−1.0 −1.0到1.0 1.0的范围内不是很直观，所以我们会指定自己的坐标集(Coordinate Set)并将它变换回标准化设备坐标系，就像OpenGL期望的那样。  
为了将顶点坐标从观察变换到裁剪空间，我们需要定义一个投影矩阵(Projection Matrix)，它指定了一个范围的坐标，比如在每个维度上的−1000 −1000到1000 1000。投影矩阵接着会将在这个指定的范围内的坐标变换为标准化设备坐标的范围(−1.0,1.0) (−1.0,1.0)。所有在范围外的坐标不会被映射到在−1.0 −1.0到1.0 1.0的范围之间，所以会被裁剪掉。在上面这个投影矩阵所指定的范围内，坐标(1250,500,750) (1250,500,750)将是不可见的，这是由于它的x x坐标超出了范围，它被转化为一个大于1.0 1.0的标准化设备坐标，所以被裁剪掉了。  
如果只是图元(Primitive)，例如三角形，的一部分超出了裁剪体积(Clipping Volume)，则OpenGL会重新构建这个三角形为一个或多个三角形让其能够适合这个裁剪范围。  
由投影矩阵创建的观察箱(Viewing Box)被称为平截头体(Frustum)，每个出现在平截头体范围内的坐标都会最终出现在用户的屏幕上。将特定范围内的坐标转化到标准化设备坐标系的过程（而且它很容易被映射到2D观察空间坐标）被称之为投影(Projection)，因为使用投影矩阵能将3D坐标投影(Project)到很容易映射到2D的标准化设备坐标系中。

##### 屏幕空间

最终的坐标将会被映射到屏幕空间中（使用glViewport中的设定），并被变换成片段。

#### 空间变换

为了将坐标从一个坐标系变换到另一个坐标系，我们需要用到几个变换矩阵，最重要的几个分别是模型(Model)、观察(View)、投影(Projection)三个矩阵。物体顶点的起始坐标再局部空间（Local Space）,这里称它为局部坐标（Local Coordinate），它在之后会变成世界坐标（world Coordinate）,观测坐标（View Coordinate）,裁剪坐标（Clip Coordinate）,并最后以屏幕坐标（Screen Corrdinate）的形式结束。

下面这张图阐释了 空间变换过程中的具体过程和结果。  
![](https://segmentfault.com/img/remote/1460000017246744?w=1618&h=814)

#### 相关API

空间变化相关的API有：

模型矩阵变换

```
void glTranslate{fd}(TYPE x,TYPE y,TYPE z) void glRotate{fd}(TYPE angle,TYPE x,TYPE y,TYPE z)
void glScale{fd}(TYPE x,TYPE y,TYPE z) 
```

视图矩阵变换

```
void gluLookAt(GLdouble eyex,GLdouble eyey,GLdouble eyez,GLdouble centerx,GLdouble centery,GLdouble centerz,GLdouble upx,GLdouble upy,GLdouble upz);


```

投影变换

```
void glOrtho(GLdouble left,GLdouble right,GLdouble bottom,GLdouble top, GLdouble near,GLdouble far)
void gluOrtho2D(GLdouble left,GLdouble right,GLdouble bottom,GLdouble top)
void glFrustum(GLdouble left,GLdouble Right,GLdouble bottom,GLdouble top, GLdouble near,GLdouble far);
void gluPerspective(GLdouble fovy,GLdouble aspect,GLdouble zNear, GLdouble zFar);

```

视口变换

```
glViewport(GLint x,GLint y,GLsizei width, GLsizei height);

```

通用变换

```
void glLoadMatrix{fd}(const TYPE *m)
void glMultMatrix{fd}(const TYPE *m)

```

OpenGL纹理
--------

在三维图形中，纹理映射（Texture Mapping）的方法运用得很广，尤其描述具有真实感的物体。比如绘制一面砖墙，就可以用一幅真实的砖墙图像或照片作为纹理贴到一个矩形上，这样，一面逼真的砖墙就画好了。如果不用纹理映射的方法，则墙上的每一块砖都必须作为一个独立的多边形来画。另外，纹理映射能够保证在变换多边形时，多边形上的纹理图案也随之变化。例如，以透视投影方式观察墙面时，离视点远的砖块的尺寸就会缩小，而离视点 较近的就会大些。此外，纹理映射也常常运用在其他一些领域，如飞行仿真中常把一大片植被的图像映射到一些大多边形上用以表示地面，或用大理石、木材、布匹等自然物质的图像作为纹理映射到多边形上表示相应的物体。

### 纹理分类

按照纹理的使用场景和表现形式来分，纹理主要分为以下几类：

*   *   一维纹理，例如，程序所绘制的带纹理的镶条的所有变化可能发生在同一个方向，一维纹理就像一个高度为1的二维纹理。
    *   二维纹理，其实是最容易理解的，也是最常用的，具有横向和纵向纹理坐标的，通常一个图片可以用作一个二维纹理。
    *   三维纹理，最常见的应用是医学和地球科学领域的渲染。在医学应用程序中，三维纹理可以用于表示一系列的断层计算成像系统(CT)或者核磁共振(MRI)图像。对于石油和天然气研究人员，三维纹理可以用来对岩石底层进行建模。三维纹理可以看成一层层二维子图像矩形构成的。
    *   球体纹理， 也就是环境纹理，目标是渲染具有完美反射能力的物体，它的表面颜色就是反射到人眼周围环境的颜色。
    *   立方体纹理，是一种特殊的纹理技术，它用6幅二维纹理图像构成一个以原点为中心的纹理立方体。立方体纹理非常适用于实现环境、反射和光照效果。
    *   多重纹理，多重纹理允许应用几个纹理，在纹理操作管线中把它们逐个应用到同一个多边形上。

*   。。。

### 纹理定义

##### 一维纹理

```
void glTexImage1D(GLenum target,GLint level,GLint components,GLsizei width,
　GLint border,GLenum format,GLenum type,const GLvoid *pixels);
```

定义一个一维纹理映射，除了第一个参数target应设置为GL\_TEXTURE\_1D外，其余所有的参数与函数TexImage2D()的一致，不过纹理图像是一维纹素数组，其宽度值必须是2的幂，若有边界则为2m+2。

##### 二维纹理

```
void glTexImage2D(GLenum target,GLint level,GLint components,
　　　　　　　　　　　GLsizei width, glsizei height,GLint border,
　　　　　　　　　　　GLenum format,GLenum type, const GLvoid *pixels);
```

　定义一个二维纹理映射。其中参数target是常数GL\_TEXTURE\_2D。参数level表示多级分辨率的纹理图像的级数，若只有一种分辨率，则level设为0。  
　　参数components是一个从1到4的整数，指出选择了R、G、B、A中的哪些分量用于调整和混合，1表示选择了R分量，2表示选择了R和A两个分量，3表示选择了R、G、B三个分量，4表示选择了R、G、B、A四个分量。  
　　参数width和height给出了纹理图像的长度和宽度，参数border为纹理边界宽度，它通常为0，width和height必须是2m+2b，这里m是整数，长和宽可以有不同的值，b是border的值。纹理映射的最大尺寸依赖于OpenGL，但它至少必须是使用64x64（若带边界为66x66），若width和height设置为0，则纹理映射有效地关闭。  
　　参数format和type描述了纹理映射的格式和数据类型，它们在这里的意义与在函数glDrawPixels()中的意义相同，事实上，纹理数据与glDrawPixels()所用的数据有同样的格式。参数format可以是GL\_COLOR\_INDEX、GL\_RGB、GL\_RGBA、GL\_RED、GL\_GREEN、GL\_BLUE、GL\_ALPHA、GL\_LUMINANCE或GL\_LUMINANCE\_ALPHA（注意：不能用GL\_STENCIL\_INDEX和GL\_DEPTH\_COMPONENT）。类似地，参数type是GL\_BYPE、GL\_UNSIGNED\_BYTE、GL\_SHORT、 GL\_UNSIGNED\_SHORT、GL\_INT、GL\_UNSIGNED\_INT、GL\_FLOAT或GL\_BITMAP。  
　　参数pixels包含了纹理图像数据，这个数据描述了纹理图像本身和它的边界。

### 纹理控制函数

OpenGL中的纹理控制函数如下：

```
void glTexParameter{if}\[v\](GLenum target,GLenum pname,TYPE param);
```

第一个参数target可以是GL\_TEXTURE\_1D或GL\_TEXTURE\_2D，它指出是为一维或二维纹理说明参数；后两个参数的可能值见下表。

| 参数 | 对应的值 |
| --- | --- |
| GL\_TEXTURE\_WRAP_S | GL\_CLAMP ，GL\_REPEAT |
| GL\_TEXTURE\_WRAP_T | GL\_CLAMP，GL\_REPEAT |
| GL\_TEXTURE\_MAG_FILTER | GL\_NEAREST，GL\_LINEAR |
| GL\_TEXTURE\_MIN_FILTER | GL\_NEAREST，GL\_LINEAR，GL\_NEAREST\_MIPMAP\_NEAREST ，GL\_NEAREST\_MIPMAP\_LINEAR ，GL\_LINEAR\_MIPMAP\_NEAREST ，GL\_LINEAR\_MIPMAP\_LINEAR |

一般来说，纹理图像为正方形或长方形。但当它映射到一个多边形或曲面上并变换到屏幕坐标时，纹理的单个纹素很少对应于屏幕图像上的象素。根据所用变换和所用纹理映射，屏幕上单个象素可以对应于一个纹素的一小部分（即放大）或一大批纹素（即缩小）。下面用函数glTexParameter*()说明放大和缩小的方法：

```
glTexParameter*;
　　glTexParameter*;
```

实际上，第一个参数可以是GL\_TEXTURE\_1D或GL\_TEXTURE\_2D，即表明所用的纹理是一维的还是二维的；第二个参数指定滤波方法，其中参数值GL\_TEXTURE\_MAG\_FILTER指定为放大滤波方法，GL\_TEXTURE\_MIN\_FILTER指定为缩小滤波方法；第三个参数说明滤波方式，其值见表12-1所示。  
　　若选择GL\_NEAREST则采用坐标最靠近象素中心的纹素，这有可能使图像走样；若选择GL\_LINEAR则采用最靠近象素中心的四个象素的加权平均值。GL\_NEAREST所需计算比GL\_LINEAR要少，因而执行得更快，但GL_LINEAR提供了比较光滑的效果。

同时，纹理坐标可以超出(0, 1)范围，并且在纹理映射过程中可以重复映射或约简映射。在重复映射的情况下，纹理可以在s，t方向上重复。例如：

```
　glTexParameterfv(GL\_TEXTURE\_2D,GL\_TEXTURE\_WRAP\_S,GL\_REPEAT);
　glTexParameterfv(GL\_TEXTURE\_2D,GL\_TEXTURE\_WRAP\_T,GL\_REPEAT);
```

### 纹理坐标

在绘制纹理映射场景时，不仅要给每个顶点定义几何坐标，而且也要定义纹理坐标。经过多种变换后，几何坐标决定顶点在屏幕上绘制的位置，而纹理坐标决定纹理图像中的哪一个纹素赋予该顶点。并且顶点之间的纹理坐标插值与基础篇中所讲的平滑着色插值方法相同。  
　　纹理图像是方形数组，纹理坐标通常可定义成一、二、三或四维形式，称为s，t，r和q坐标，以区别于物体坐标(x, y, z, w)和其他坐标。一维纹理常用s坐标表示，二维纹理常用(s, t)坐标表示，目前忽略r坐标，q坐标象w一样，一半值为1，主要用于建立齐次坐标。OpenGL坐标定义的函数是：  
　　

```
void gltexCoord{1234}{sifd}\[v\](TYPE coords);
```

设置当前纹理坐标，此后调用glVertex_()所产生的顶点都赋予当前的纹理坐标。对于gltexCoord1_()，s坐标被设置成给定值，t和r设置为0，q设置为1；用gltexCoord2_()可以设置s和t坐标值，r设置为0，q设置为1；对于gltexCoord3_()，q设置为1，其它坐标按给定值设置；用gltexCoord4*()可以给定所有的坐标。使用适当的后缀（s，i，f或d）和TYPE的相应值（GLshort、GLint、glfloat或GLdouble）来说明坐标的类型。注意：整型纹理坐标可以直接应用，而不是象普通坐标那样被映射到\[-1, 1\]之间。

#### 示例

```
#include <GLUT/GLUT.h>
#include <OpenGL/OpenGL.h>
#include "BMPLoader.h"
GLuint tex2D;
GLfloat angle;
 

void init() {
    glEnable(GL\_DEPTH\_TEST);
    glDepthFunc(GL_LESS);
    glClearColor(0.1, 0.1, 0.4, 0.0);
    glShadeModel(GL_SMOOTH);
    CBMPLoader bmpLoader;
    bmpLoader.LoadBmp("/123-bmp.bmp");
     
 
    
    glGenTextures(1, &tex2D);
    glBindTexture(GL\_TEXTURE\_2D, tex2D);

 
    
    glTexParameteri(GL\_TEXTURE\_2D, GL\_TEXTURE\_MIN\_FILTER, GL\_LINEAR);
    glTexParameteri(GL\_TEXTURE\_2D, GL\_TEXTURE\_MAG\_FILTER, GL\_LINEAR);
    glTexParameteri(GL\_TEXTURE\_2D, GL\_TEXTURE\_WRAP\_S, GL\_CLAMP);
    glTexParameteri(GL\_TEXTURE\_2D, GL\_TEXTURE\_WRAP\_T, GL\_CLAMP);
 
 
    
    glTexImage2D(GL\_TEXTURE\_2D, 0, GL_RGB, bmpLoader.imageWidth, bmpLoader.imageHeight, 0, GL\_RGB, GL\_UNSIGNED_BYTE, bmpLoader.image);
    angle = 0;
}
 
 

void DrawBox(){
    glEnable(GL\_TEXTURE\_2D);
 
    
    glBindTexture(GL\_TEXTURE\_2D, tex2D);

    
    glBegin(GL_QUADS);
 
    
    glNormal3f(0.0f, 0.0f, 1.0f);                               
    glTexCoord2f(0.0f, 0.0f); glVertex3f(-1.0f, -1.0f, 1.0f);
    glTexCoord2f(1.0f, 0.0f); glVertex3f(1.0f, -1.0f, 1.0f);
    glTexCoord2f(1.0f, 1.0f); glVertex3f(1.0f, 1.0f, 1.0f);
    glTexCoord2f(0.0f, 1.0f); glVertex3f(-1.0f, 1.0f, 1.0f);
 
    
    glNormal3f(0.0f, 0.0f, -1.0f);                              
    glTexCoord2f(0.0f, 0.0f); glVertex3f(-1.0f, -1.0f, -1.0f);
    glTexCoord2f(1.0f, 0.0f); glVertex3f(-1.0f, 1.0f, -1.0f);
    glTexCoord2f(1.0f, 1.0f); glVertex3f(1.0f, 1.0f, -1.0f);
    glTexCoord2f(0.0f, 1.0f); glVertex3f(1.0f, -1.0f, -1.0f);
 
    
    glNormal3f(0.0f, 1.0f, 0.0f);                               
    glTexCoord2f(0.0f, 0.0f);glVertex3f(-1.0f, 1.0f, 1.0f);
    glTexCoord2f(1.0f, 0.0f);glVertex3f(1.0f, 1.0f, 1.0f);
    glTexCoord2f(1.0f, 1.0f);glVertex3f(1.0f, 1.0f, -1.0f);
    glTexCoord2f(0.0f, 1.0f);glVertex3f(-1.0f, 1.0f, -1.0f);
 
    
    glNormal3f(0.0f, -1.0f, 0.0f);                              
    glTexCoord2f(0.0f, 0.0f);glVertex3f(-1.0f, -1.0f, 1.0f);
    glTexCoord2f(1.0f, 0.0f);glVertex3f(1.0f, -1.0f, 1.0f);
    glTexCoord2f(1.0f, 1.0f);glVertex3f(1.0f, -1.0f, -1.0f);
    glTexCoord2f(0.0f, 1.0f);glVertex3f(-1.0f, -1.0f, -1.0f);
 
    
    glNormal3f(1.0f, 0.0f, 0.0f);                               
    glTexCoord2f(0.0f, 0.0f); glVertex3f(1.0f, -1.0f, -1.0f);
    glTexCoord2f(1.0f, 0.0f); glVertex3f(1.0f, 1.0f, -1.0f);
    glTexCoord2f(1.0f, 1.0f); glVertex3f(1.0f, 1.0f, 1.0f);
    glTexCoord2f(0.0f, 1.0f); glVertex3f(1.0f, -1.0f, 1.0f);
    
    
    glNormal3f(-1.0f, 0.0f, 0.0f);                              
    glTexCoord2f(0.0f, 0.0f); glVertex3f(-1.0f, -1.0f, -1.0f);
    glTexCoord2f(1.0f, 0.0f); glVertex3f(-1.0f, 1.0f, -1.0f);
    glTexCoord2f(1.0f, 1.0f); glVertex3f(-1.0f, 1.0f, 1.0f);
    glTexCoord2f(0.0f, 1.0f); glVertex3f(-1.0f, -1.0f, 1.0f);
    glEnd();
    glDisable(GL\_TEXTURE\_2D);
}
 


void display() {
    
    glClear(GL\_COLOR\_BUFFER\_BIT|GL\_DEPTH\_BUFFER\_BIT);
    glPushMatrix();
    glTranslatef(0.0f, 0.0f, -5.0f);
    glRotated(angle, 1, 1, 0);
    DrawBox();
    glPopMatrix();
 
    
    glFlush();
    angle ++;
    glutPostRedisplay();
}

 

void reshape(int w, int h) {
    glViewport(0, 0, w, h);
    glMatrixMode(GL_PROJECTION);
    glLoadIdentity();
    gluPerspective(60.0, (GLfloat)w/(GLfloat)h, 0.1, 100000.0);
    glMatrixMode(GL_MODELVIEW);
    glLoadIdentity();
}

int main(int argc, const char \* argv\[\]) {
    
    glutInit(&argc, const_cast<char **>(argv));
    glutInitDisplayMode(GLUT\_SINGLE | GLUT\_RGB|GLUT_DEPTH);

 
    
    glutInitWindowSize(500, 500);
    glutInitWindowPosition(100, 100);
    glutCreateWindow(argv\[0\]);
 
    init();
    glutReshapeFunc(reshape);
    glutDisplayFunc(display);
 
    
    glutMainLoop();
    return 0;
}
```

运行效果如下：  
![](https://segmentfault.com/img/remote/1460000017246745?w=994&h=990)

OpenGL光照和材质
-----------

当光照射到一个物体表面上时，会出现三种情形。首先，光可以通过物体表面向空间反射，产生反射光。其次，对于透明体，光可以穿透该物体并从另一端射出，产生透射光。最后，部分光将被物体表面吸收而转换成热。在上述三部分光中，仅仅是透射光和反射光能够进入人眼产生视觉效果。这里介绍的简单光照模型只考虑被照明物体表面的反射光影响，假定物体表面光滑不透明且由理想材料构成，环境假设为由白光照明。  
　　一般来说，反射光可以分成三个分量，即环境反射、漫反射和镜面反射。环境反射分量假定入射光均匀地从周围环境入射至景物表面并等量地向各个方向反射出去，通常物体表面还会受到从周围环境来的反射光（如来自地面、天空、墙壁等的反射光）的照射，这些光常统称为环境光（Ambient Light）；漫反射分量表示特定光源在景物表面的反射光中那些向空间各方向均匀反射出去的光，这些光常称为漫射光（Diffuse Light）；镜面反射光为朝一定方向的反射光，如一个点光源照射一个金属球时会在球面上形成一块特别亮的区域，呈现所谓“高光（Highlight）”，它是光源在金属球面上产生的镜面反射光（Specular Light）。对于较光滑物体，其镜面反射光的高光区域小而亮；相反，粗糙表面的镜面反射光呈发散状态，其高光区域大而不亮。

#### 光组成

　在OpenGL简单光照模型中的几种光分为：辐射光（Emitted Light）、环境光（Ambient Light）、漫射光（Diffuse Light）、镜面光（Specular Light）。  
　　

*   辐射光是最简单的一种光，它直接从物体发出并且不受任何光源影响。
*   环境光是由光源发出经环境多次散射而无法确定其方向的光，即似乎来自所有方向。一般说来，房间里的环境光成分要多些，户外的相反要少得多，因为大部分光按相同方向照射，而且在户外很少有其他物体反射的光。当环境光照到曲面上时，它在各个方向上均等地发散（类似于无影灯光）。
*   漫射光来自一个方向，它垂直于物体时比倾斜时更明亮。一旦它照射到物体上，则在各个方向上均匀地发散出去。于是，无论视点在哪里它都一样亮。来自特定位置和特定方向的任何光，都可能有散射成分。
*   镜面光来自特定方向并沿另一方向反射出去，一个平行激光束在高质量的镜面上产生100%的镜面反射。光亮的金属和塑料具有很高非反射成分，而象粉笔和地毯等几乎没有反射成分。因此，三某种意义上讲，物体的反射程度等同于其上的光强（或光亮度）。

#### 创建光源

光源有许多特性，如颜色、位置、方向等。选择不同的特性值，则对应的光源作用在物体上的效果也不一样，这在以后的章节中会逐步介绍的。下面详细讲述定义光源特性的函数glLight*()：

```
void glLight{if}\[v\](GLenum light , GLenum pname, TYPE param)
```

创建具有某种特性的光源。其中第一个参数light指定所创建的光源号，如GL\_LIGHT0、GL\_LIGHT1、...、GL_LIGHT7。第二个参数pname指定光源特性，这个参数的辅助信息见表1-3所示。最后一个参数设置相应的光源特性值。

| pname 参数名 | 默认值 | 说明 |
| --- | --- | --- |
| GL_AMBIENT | (0.0, 0.0, 0.0, 1.0) | RGBA模式下环境光 |
| GL_DIFFUSE | (1.0, 1.0, 1.0, 1.0) | RGBA模式下漫反射光 |
| GL_SPECULAR | (1.0,1.0,1.0,1.0) | RGBA模式下镜面光 |
| GL_POSITION | (0.0,0.0,1.0,0.0) | 光源位置齐次坐标（x,y,z,w） |
| GL\_SPOT\_DIRECTION | (0.0,0.0,-1.0) | 点光源聚光方向矢量（x,y,z） |
| GL\_SPOT\_EXPONENT | 0.0 | 点光源聚光指数 |
| GL\_SPOT\_CUTOFF | 180.0 | 点光源聚光截止角 |
| GL\_CONSTANT\_ATTENUATION | 1.0 | 常数衰减因子 |
| GL\_LINER\_ATTENUATION | 0.0 | 线性衰减因子 |
| GL\_QUADRATIC\_ATTENUATION | 0.0 | 平方衰减因子 |

以上列出的GL\_DIFFUSE和GL\_SPECULAR的缺省值只能用于GL\_LIGHT0，其他几个光源的GL\_DIFFUSE和GL_SPECULAR缺省值为(0.0,0.0,0.0,1.0)。另外，表中后六个参数的应用放在下一篇中介绍。在上面例程中，光源的创建为：

```
GLfloat light_position\[\] = { 1.0, 1.0, 1.0, 0.0 };
glLightfv(GL\_LIGHT0, GL\_POSITION, light_position);
```

其中light_position是一个指针，指向定义的光源位置齐次坐标数组。其它几个光源特性都为缺省值。同样，我们也可用类似的方式定义光源的其他几个特性值。例如：

```
　GLfloat light_ambient \[\] = { 0.0, 0.0, 0.0, 1.0 };
　GLfloat light_diffuse \[\] = { 1.0, 1.0, 1.0, 1.0 };
   GLfloat light_specular\[\] = { 1.0, 1.0, 1.0, 1.0 };
　glLightfv(GL\_LIGHT0, GL\_AMBIENT , light_ambient );
　glLightfv(GL\_LIGHT0, GL\_DIFFUSE , light_diffuse );
　glLightfv(GL\_LIGHT0, GL\_SPECULAR, light_specular);
```

#### 启动光照

在OpenGL中，必须明确指出光照是否有效或无效。如果光照无效，则只是简单地将当前颜色映射到当前顶点上去，不进行法向、光源、材质等复杂计算，那么显示的图形就没有真实感，如前几章例程运行结果显示。要使光照有效，首先得启动光照，启动光照需要用到如下函数。

```
glEnable(GL_LIGHTING);
```

若使光照无效，则调用gDisable(GL_LIGHTING)可关闭当前光照。然后，必须使所定义的每个光源有效，如果只用了一个光源。

```
glEnable(GL_LIGHT0);
```

其它光源类似，只是光源号不同而已。

#### 材质颜色

OpenGL用材料对光的红、绿、蓝三原色的反射率来近似定义材料的颜色。像光源一样，材料颜色也分成环境、漫反射和镜面反射成分，它们决定了材料对环境光、漫反射光和镜面反射光的反射程度。在进行光照计算时，材料对环境光的反射率与每个进入光源的环境光结合，对漫反射光的反射率与每个进入光源的漫反射光结合，对镜面光的反射率与每个进入光源的镜面反射光结合。对环境光与漫反射光的反射程度决定了材料的颜色，并且它们很相似。对镜面反射光的反射率通常是白色或灰色（即对镜面反射光中红、绿、蓝的反射率相同）。镜面反射高光最亮的地方将变成具有光源镜面光强度的颜色。例如一个光亮的红色塑料球，球的大部分表现为红色，光亮的高光将是白色的。材质的定义与光源的定义类似：

```
void glMaterial{if}\[v\](GLenum face,GLenum pname,TYPE param);
```

定义光照计算中用到的当前材质。face可以是GL\_FRONT、GL\_BACK、GL\_FRONT\_AND\_BACK，它表明当前材质应该应用到物体的哪一个面上；pname说明一个特定的材质；param是材质的具体数值，若函数为向量形式，则param是一组值的指针，反之为参数值本身。非向量形式仅用于设置GL\_SHINESS。另外，参数GL\_AMBIENT\_AND_DIFFUSE表示可以用相同的RGB值设置环境光颜色和漫反射光颜色。

| 参数名 | 默认值 | 说明 |
| --- | --- | --- |
| GL_AMBIENT | (0.2, 0.2, 0.2, 1.0) | 材料的环境光颜色 |
| GL_DIFFUSE | (0.8, 0.8, 0.8, 1.0) | 材料的漫反射光颜色 |
| GL\_AMBIENT\_AND_DIFFUSE |  | 材料的环境光和漫反射光颜色 |
| GL_SPECULAR | (0.0, 0.0, 0.0, 1.0) | 材料的镜面反射光颜色 |
| GL_SHINESS | 0.0 | 镜面指数（光亮度） |
| GL_EMISSION | (0.0, 0.0, 0.0, 1.0) | 材料的辐射光颜色 |
| GL\_COLOR\_INDEXES | (0, 1, 1) | 材料的环境光、漫反射光和镜面光颜色 |

#### 材质RGB值 与 光源RGB

　材质的颜色与光源的颜色有些不同。对于光源，R、G、B值等于R、G、B对其最大强度的百分比。若光源颜色的R、G、B值都是1.0，则是最强的白光；若值变为0.5，颜色仍为白色，但强度为原来的一半，于是表现为灰色；若R＝G＝1.0，B＝0.0，则光源为黄色。对于材质，R、G、B值为材质对光的R、G、B成分的反射率。比如，一种材质的R＝1.0、G＝0.5、B＝0.0，则材质反射全部的红色成分，一半的绿色成分，不反射蓝色成分。也就是说，若OpenGL的光源颜色为（LR、LG、LB），材质颜色为（MR、MG、MB），那么，在忽略所有其他反射效果的情况下，最终到达眼睛的光的颜色为（LR_MR、LG_MG、LB*MB）。  
　　同样，如果有两束光，相应的值分别为（R1、G1、B1）和（R2、G2、B2），则OpenGL将各个颜色成分相加，得到（R1+R2、G1+G2、B1+B2），若任一成分的和值大于1（超出了设备所能显示的亮度）则约简到1.0。

### 示例

下面的示例将演示光照和材质在OpenGL上的应用。

```
#include <GLUT/GLUT.h>
#include <OpenGL/OpenGL.h>
 
 

void init() {
    GLfloat ambient\[\] = { 0.0, 0.0, 0.0, 1.0 };
    GLfloat diffuse\[\] = { 1.0, 1.0, 1.0, 1.0 };

    GLfloat position\[\] = { 0.0, 0, -1.0, 0.0 };
    glEnable(GL\_DEPTH\_TEST);
    glDepthFunc(GL_LESS);
    glLightfv(GL\_LIGHT0, GL\_AMBIENT, ambient);
    glLightfv(GL\_LIGHT0, GL\_DIFFUSE, diffuse);

    glLightfv(GL\_LIGHT0, GL\_POSITION, position);
    glEnable(GL_LIGHTING);
    glEnable(GL_LIGHT0);
    glClearColor(0.0, 0.1, 0.1, 0.0) ；
}
 
 

void display() {
    GLfloat no_mat\[\] = { 0.0, 0.0, 0.0, 1.0 };
    GLfloat mat_ambient\[\] = { 0.7, 0.7, 0.7, 1.0 };
    GLfloat mat\_ambient\_color\[\] = { 0.8, 0.8, 0.2, 1.0 };
    GLfloat mat_diffuse\[\] = { 0.1, 0.5, 0.8, 1.0 };
    GLfloat mat_specular\[\] = { 1.0, 1.0, 1.0, 1.0 };
    GLfloat no_shininess\[\] = { 0.0 };
    GLfloat low_shininess\[\] = { 5.0 };
    GLfloat high_shininess\[\] = { 100.0 };
    GLfloat mat_emission\[\] = {0.3, 0.2, 0.2, 0.0};
    glClear(GL\_COLOR\_BUFFER\_BIT | GL\_DEPTH\_BUFFER\_BIT);
 
     
 
    
    glPushMatrix();
    glTranslatef (-3.75, 3.0, 0.0);
    glMaterialfv(GL\_FRONT, GL\_AMBIENT, no_mat);
    glMaterialfv(GL\_FRONT, GL\_DIFFUSE, mat_diffuse);
    glMaterialfv(GL\_FRONT, GL\_SPECULAR, no_mat);
    glMaterialfv(GL\_FRONT, GL\_SHININESS, no_shininess);
    glMaterialfv(GL\_FRONT, GL\_EMISSION, no_mat);
    glutSolidSphere(1.0, 20, 20);
    glPopMatrix();
 
     
 
    
    glPushMatrix();
    glTranslatef (-1.25, 3.0, 0.0);
    glMaterialfv(GL\_FRONT, GL\_AMBIENT, no_mat);
    glMaterialfv(GL\_FRONT, GL\_DIFFUSE, mat_diffuse);
    glMaterialfv(GL\_FRONT, GL\_SPECULAR, mat_specular);
    glMaterialfv(GL\_FRONT, GL\_SHININESS, low_shininess);
    glMaterialfv(GL\_FRONT, GL\_EMISSION, no_mat);
    glutSolidSphere(1.0, 20, 20);
 
    glPopMatrix();
 
     
 
    
    glPushMatrix();
    glTranslatef (1.25, 3.0, 0.0);
    glMaterialfv(GL\_FRONT, GL\_AMBIENT, no_mat);
    glMaterialfv(GL\_FRONT, GL\_DIFFUSE, mat_diffuse);
    glMaterialfv(GL\_FRONT, GL\_SPECULAR, mat_specular);
    glMaterialfv(GL\_FRONT, GL\_SHININESS, high_shininess);
    glMaterialfv(GL\_FRONT, GL\_EMISSION, no_mat);
    glutSolidSphere(1.0, 20, 20);
    glPopMatrix();
     
 
    
    glPushMatrix();
    glTranslatef (3.75, 3.0, 0.0);
    glMaterialfv(GL\_FRONT, GL\_AMBIENT, no_mat);
    glMaterialfv(GL\_FRONT, GL\_DIFFUSE, mat_diffuse);
    glMaterialfv(GL\_FRONT, GL\_SPECULAR, no_mat);
    glMaterialfv(GL\_FRONT, GL\_SHININESS, no_shininess);
    glMaterialfv(GL\_FRONT, GL\_EMISSION, mat_emission);
    glutSolidSphere(1.0, 20, 20);
    glPopMatrix();
     
 
    
    glPushMatrix();
    glTranslatef (-3.75, 0.0, 0.0);
    glMaterialfv(GL\_FRONT, GL\_AMBIENT, mat_ambient);
    glMaterialfv(GL\_FRONT, GL\_DIFFUSE, mat_diffuse);
    glMaterialfv(GL\_FRONT, GL\_SPECULAR, no_mat);
    glMaterialfv(GL\_FRONT, GL\_SHININESS, no_shininess);
    glMaterialfv(GL\_FRONT, GL\_EMISSION, no_mat);
    glutSolidSphere(1.0, 20, 20);
    glPopMatrix();
     
 
    
    glPushMatrix();
    glTranslatef (-1.25, 0.0, 0.0);
    glMaterialfv(GL\_FRONT, GL\_AMBIENT, mat_ambient);
    glMaterialfv(GL\_FRONT, GL\_DIFFUSE, mat_diffuse);
    glMaterialfv(GL\_FRONT, GL\_SPECULAR, mat_specular);
    glMaterialfv(GL\_FRONT, GL\_SHININESS, low_shininess);
    glMaterialfv(GL\_FRONT, GL\_EMISSION, no_mat);
    glutSolidSphere(1.0, 20, 20);
    glPopMatrix();
 
 
    
    glPushMatrix();
    glTranslatef (1.25, 0.0, 0.0);
    glMaterialfv(GL\_FRONT, GL\_AMBIENT, mat_ambient);
    glMaterialfv(GL\_FRONT, GL\_DIFFUSE, mat_diffuse);
    glMaterialfv(GL\_FRONT, GL\_SPECULAR, mat_specular);
    glMaterialfv(GL\_FRONT, GL\_SHININESS, high_shininess);
    glMaterialfv(GL\_FRONT, GL\_EMISSION, no_mat);
    glutSolidSphere(1.0, 20, 20);
    glPopMatrix();
  
 
    
    glPushMatrix();
    glTranslatef (3.75, 0.0, 0.0);
    glMaterialfv(GL\_FRONT, GL\_AMBIENT, mat_ambient);
    glMaterialfv(GL\_FRONT, GL\_DIFFUSE, mat_diffuse);
    glMaterialfv(GL\_FRONT, GL\_SPECULAR, no_mat);
    glMaterialfv(GL\_FRONT, GL\_SHININESS, no_shininess);
    glMaterialfv(GL\_FRONT, GL\_EMISSION, mat_emission);
    glutSolidSphere(1.0, 20, 20);
    glPopMatrix();
 
 
    
    glPushMatrix();
    glTranslatef (-3.75, -3.0, 0.0);
    glMaterialfv(GL\_FRONT, GL\_AMBIENT, mat\_ambient\_color);
    glMaterialfv(GL\_FRONT, GL\_DIFFUSE, mat_diffuse);
    glMaterialfv(GL\_FRONT, GL\_SPECULAR, no_mat);
    glMaterialfv(GL\_FRONT, GL\_SHININESS, no_shininess);
    glMaterialfv(GL\_FRONT, GL\_EMISSION, no_mat);
    glutSolidSphere(1.0, 20, 20);
    glPopMatrix();
     
 
    
    glPushMatrix();
    glTranslatef (-1.25, -3.0, 0.0);
    glMaterialfv(GL\_FRONT, GL\_AMBIENT, mat\_ambient\_color);
    glMaterialfv(GL\_FRONT, GL\_DIFFUSE, mat_diffuse);
    glMaterialfv(GL\_FRONT, GL\_SPECULAR, mat_specular);
    glMaterialfv(GL\_FRONT, GL\_SHININESS, low_shininess);
    glMaterialfv(GL\_FRONT, GL\_EMISSION, no_mat);
    glutSolidSphere(1.0, 20, 20);
    glPopMatrix();
     
 
    
    glPushMatrix();
    glTranslatef (1.25, -3.0, 0.0);
    glMaterialfv(GL\_FRONT, GL\_AMBIENT, mat\_ambient\_color);
    glMaterialfv(GL\_FRONT, GL\_DIFFUSE, mat_diffuse);
    glMaterialfv(GL\_FRONT, GL\_SPECULAR, mat_specular);
    glMaterialfv(GL\_FRONT, GL\_SHININESS, high_shininess);
    glMaterialfv(GL\_FRONT, GL\_EMISSION, no_mat);
    glutSolidSphere(1.0, 20, 20);
    glPopMatrix();
 
 
    
    glPushMatrix();
    glTranslatef (3.75, -3.0, 0.0);
    glMaterialfv(GL\_FRONT, GL\_AMBIENT, mat\_ambient\_color);
    glMaterialfv(GL\_FRONT, GL\_DIFFUSE, mat_diffuse);
    glMaterialfv(GL\_FRONT, GL\_SPECULAR, no_mat);
    glMaterialfv(GL\_FRONT, GL\_SHININESS, no_shininess);
    glMaterialfv(GL\_FRONT, GL\_EMISSION, mat_emission);
    glutSolidSphere(1.0, 20, 20);
    glPopMatrix();
    
    glFlush();
}
 
 

void reshape(int w, int h) {
    glViewport(0, 0, w, h);
    glMatrixMode(GL_PROJECTION);
    glLoadIdentity();
    gluPerspective(60.0, (GLfloat)w/(GLfloat)h, 0.1, 100000.0);
    glMatrixMode(GL_MODELVIEW);
    glLoadIdentity();
    gluLookAt(0, 0, 10, 0, 0, -1, 0, 1, 0);
}
 
 
int main(int argc, const char \* argv\[\]) {
    
    glutInit(&argc, const_cast<char **>(argv));
    glutInitDisplayMode(GLUT\_SINGLE | GLUT\_RGBA);

 
    
    glutInitWindowSize(500, 500);
    glutInitWindowPosition(100, 100);
    glutCreateWindow(argv\[0\]);

 
    init();
    glutReshapeFunc(reshape);
    glutDisplayFunc(display);
 
    
    glutMainLoop();
    return 0;
}
```

运行效果如下图：  
![](https://segmentfault.com/img/remote/1460000017246746?w=990&h=990)