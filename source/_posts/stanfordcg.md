---
title: Stanford CS148 Assignment01
date: 2020-02-17
tag: [Computer Graphics]
categories: [Stanford CS148]
---

本文的配置基于MacOS。

### 环境

- C++编译器，支持C++11 : 可使用XCode
- OpenGL 4.1 : Apple对OpenGL的支持可见[这里](https://developer.apple.com/opengl/capabilities/)
- CMake 2.8+ : 最新版本的CMake在[这里](http://www.cmake.org/download/)
- Git

### 资源
现将[这个](https://github.com/b3h47pte/cs148opengl-public)工程克隆到本地目录。

```
git remote add source https://github.com/b3h47pte/cs148opengl-public.git
git fetch source
git merge source/master
```


### Mac OSX Setup
保证了上述的环境，进入上面fetch下来的目录，在`external/SDL2/osx`子目录下，双击`SDL2.dmg`文件，安装后将`SDL2.framework`文件拷贝回`external/SDL2/osx`目录。

打开`CMake`，将`Where is the source code`设置为`external/assimp`，创建子目录`external/assimp/build`并将`Where to build the binaries`设置为`external/assimp/build`，如下图所示。

![](/images/stcg/1.jpg)

![](/images/stcg/2.jpg)

点击`Configure`后选择`Xcode`->`Done`，点击`Generate`。


报错`xcode-select: error: tool 'xcodebuild' requires Xcode, but active developer directory '/Library/Developer/CommandLineTools' is a command line tools instance`。

解决：`sudo xcode-select -s /Applications//Xcode.app/Contents/Developer/`。

不报错成功后就会在`external/assimp/build`目录下找到`“Assimp.xcodeproj`可用XCode打开项目并运行。在XCode的`“Edit Scheme`可选择`Release`或`Debug`。在`external/assimp/build/code/Release`中拷贝`libassimp.dylib`到`external/assimp/distrib/osx`目录下。

### 自定义
依上所述，可以建立自己的工程。
1. 在项目的根目录下新建`cs148opengl4`文件夹，并使用命令行进入(`cd /Users/jaythan/workspace/stanfordCS148/cs148opengl4`)。

2. `cmake ../ -G "Xcode"`。

完成后可使用`ls`查看该目录，生成了一些cmake文件还有工程`cs148-opengl4.xcodeproj`。

![](/images/stcg/5.jpg)


用Xcode打开并运行，左上角选择自己的工程。

![](/images/stcg/4.jpg)

生成下面图片，作业完成。

![](/images/stcg/3.jpg)


### 后记
这个作业是完全根据instruction来做的，有些步骤不是很明白原因，比如cmake的作用机理，有空再补充。

### 参考资料

[Stanford CS148 Introduction to Computer Graphics and Imaging (Fall 2019)](http://web.stanford.edu/class/cs148/lectures.html)