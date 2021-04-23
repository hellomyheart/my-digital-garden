# 配置Java开发环境

### 安装JDK

因为Java程序必须运行在JVM之上，所以，我们第一件事情就是安装JDK。

> 关于jdk版本问题，现在大部分的Java教程都是采用1.8的版本。廖雪峰老师的教程是基于jdk14的（目前最新），有些激进。但也有一些好处。jdk1.8之后有很多新的特性，在Java se阶段廖雪峰老师都提到了。
>
> 本教程使用jdk1.14。遇到新特性的地方，尽可能标注。
>
> 如果之前安装了jdk8,可以下载[jdk14](https://www.oracle.com/java/technologies/javase-jdk14-downloads.html)的zip包（jdk-14.0.1_windows-x64_bin .zip这个，win下的，其他系统下载对应压缩包，先解压到一个位置，在安装IDE这里会讲如何切换jdk）

搜索JDK 14，确保从[Oracle的官网](https://www.oracle.com/java/technologies/javase-downloads.html)下载最新的稳定版JDK：

![jdk14-download](https://zeroone-bucket.oss-cn-beijing.aliyuncs.com/hexo-client/20200629200820.png)

找到Java SE 14的下载链接，下载安装即可。

### 设置环境变量

安装完JDK后，需要设置一个`JAVA_HOME`的环境变量，它指向JDK的安装目录。在Windows下，它是安装目录，类似：

```shell
C:\Program Files\Java\jdk-14
```

在Mac下，它在`~/.bash_profile`或`~/.zprofile`里，它是：

```shell
export JAVA_HOME=`/usr/libexec/java_home -v 14`
```

然后，把`JAVA_HOME`的`bin`目录附加到系统环境变量`PATH`上。在Windows下，它长这样：

```shell
Path=%JAVA_HOME%\bin;<现有的其他路径>
```

在Mac下，它在`~/.bash_profile`或`~/.zprofile`里，长这样：

```shell
export PATH=$JAVA_HOME/bin:$PATH
```

把`JAVA_HOME`的`bin`目录添加到`PATH`中是为了在任意文件夹下都可以运行`java`。打开命令提示符窗口，输入命令`java -version`，如果一切正常，你会看到如下输出：

```
┌────────────────────────────────────────────────────────┐
│Command Prompt                                    - □ x │
├────────────────────────────────────────────────────────┤
│Microsoft Windows [Version 10.0.0]                      │
│(c) 2015 Microsoft Corporation. All rights reserved.    │
│                                                        │
│C:\> java -version                                      │
│java version "14" ...                                   │
│Java(TM) SE Runtime Environment                         │
│Java HotSpot(TM) 64-Bit Server VM                       │
│                                                        │
│C:\>                                                    │
│                                                        │
│                                                        │
└────────────────────────────────────────────────────────┘
```

如果你看到的版本号不是`14`，而是`12`、`1.8`之类，说明系统存在多个JDK，且默认JDK不是JDK 14，需要把JDK 14提到`PATH`前面。

如果你得到一个错误输出：

```
┌────────────────────────────────────────────────────────┐
│Command Prompt                                    - □ x │
├────────────────────────────────────────────────────────┤
│Microsoft Windows [Version 10.0.0]                      │
│(c) 2015 Microsoft Corporation. All rights reserved.    │
│                                                        │
│C:\> java -version                                      │
│'java' is not recognized as an internal or external comm│
│and, operable program or batch file.                    │
│                                                        │
│C:\>                                                    │
│                                                        │
│                                                        │
│                                                        │
└────────────────────────────────────────────────────────┘
```

这是因为系统无法找到Java虚拟机的程序`java.exe`，需要检查JAVA_HOME和PATH的配置。

可以参考[如何设置或更改PATH系统变量](https://www.java.com/zh_CN/download/help/path.xml)。

### JDK

细心的童鞋还可以在`JAVA_HOME`的`bin`目录下找到很多可执行文件：

- java：这个可执行程序其实就是JVM，运行Java程序，就是启动JVM，然后让JVM执行指定的编译后的代码；
- javac：这是Java的编译器，它用于把Java源码文件（以`.java`后缀结尾）编译为Java字节码文件（以`.class`后缀结尾）；
- jar：用于把一组`.class`文件打包成一个`.jar`文件，便于发布；
- javadoc：用于从Java源码中自动提取注释并生成文档；
- jdb：Java调试器，用于开发阶段的运行调试。