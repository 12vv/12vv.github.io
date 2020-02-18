---
title: OPENGL-Xcode配置
date: 2019-09-29
tag: [OPENGL]
categories: [配置]
---

<!--more-->

### 概述

> 配置了好久，主要是跟着网上别人的步骤做，不全面，总是报错，终于排查除问题了，编译成功，总结一下。


### 准备

首先安装`glew`和`glfw`，在已有`homebrew`的情况下。

```
brew install glew
brew install glfw3
```

会自动安装在`/usr/local/Cellar`目录。

如果要安装`GLTOOLS`，

```
brew install cmake
git clone https://github.com/HazimGazov/GLTools
cd GLTools/build
cmake ..
make
sudo make install
```

然后，打开`GLAD`[在线服务](http://glad.dav1d.de/)，更改`Profile`为`Core`，`API`中`gl`根据自己版本选择，其他的默认设置即可(生成加载器(Generate a loader)的选项选中)。

![](/images/opglconfig/4.png)

点击`Generate`，下载压缩文件`glad.zip`，解压后将目录`glad`和`KHR`复制到`/usr/local/include/`路径下，并添加`glad.c`到工程目录中(和`main.cpp`在同一目录)。


### 配置

`XCODE -> Preferences -> Locations -> Custom Path`中作如下配置。

![](/images/opglconfig/2.png)

这一步是为了之后配置路径引用方便。


然后，在项目的`Build phases`导入`framework`。

![](/images/opglconfig/1.png)


然后，在`Build settings`搜索`Header Search Paths`添加

```
$(gltools_header)
$(glew_header) 
$(glfw_header)
```


搜索`Library Search Paths`添加

```
$(gltools_lib)
$(glew_lib) 
$(glfw_lib)
```



整个工程目录如下，已经把一些不用的文件删除了。

![](/images/opglconfig/5.png)

### 测试代码

```c++
#include <iostream>
#include <GL/glew.h>
#include <GLFW/glfw3.h>

void Render(void)
{
    glClearColor(0.0f, 0.0f, 0.0f, 1.0f);
    glClear(GL_COLOR_BUFFER_BIT);
    glBegin(GL_TRIANGLES);
    {
        glColor3f(1.0,0.0,0.0);
        glVertex2f(0, .5);
        glColor3f(0.0,1.0,0.0);
        glVertex2f(-.5,-.5);
        glColor3f(0.0, 0.0, 1.0);
        glVertex2f(.5, -.5);
    }
    glEnd();
}

int main(int argc, const char * argv[]) {
    GLFWwindow* win;
    if(!glfwInit()){
        return -1;
    }
    win = glfwCreateWindow(640, 480, "OpenGL Base Project", NULL, NULL);
    if(!win)
    {
        glfwTerminate();
        exit(EXIT_FAILURE);
    }
    if(!glewInit())
    {
        return -1;
    }
    glfwMakeContextCurrent(win);
    while(!glfwWindowShouldClose(win)){
        Render();
        glfwSwapBuffers(win);
        glfwPollEvents();
    }
    glfwTerminate();
    exit(EXIT_SUCCESS);
    return 0;
}

```

编译成功。

![](/images/opglconfig/3.png)


### 补充
如果引入`glad.h`头文件，必须在`GLFW`之前。

```
#include <glad/glad.h>  //glad 必须在GLFW之前添加
#include <GLFW/glfw3.h>
```