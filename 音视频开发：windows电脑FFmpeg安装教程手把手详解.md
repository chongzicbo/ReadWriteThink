# windows电脑FFmpeg安装教程手把手详解

本文以 `Windows 64` 位操作系统为例演示.

## 一、下载&解压

打开 [FFmpeg 官网]([Builds - CODEX FFMPEG @ gyan.dev](https://www.gyan.dev/ffmpeg/builds/))，选择下载。

![image-20241226161917589](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20241226161917589.png)

你可以选择下载上面红色圈中的 `release-full` 版本，或者选择下面红色圈中的前一个稳定版本 `xxx-full_build`。

`release-full` 版本会比下面的 `xxx-full_build` 版本更新，选择哪一个都可以，看你个人喜好。

至于你是想选择带 `shared` 的还是不带 `shared` 的版本，其实都是可以的。因为同一个版本带 `shared` 的和不带 `shared` 的，功能是完全一样的。

> 带 `shared` 的里面，多了 `include`、`lib` 目录。把 FFmpeg 依赖的模块包单独的放在的 `lib` 目录中。`ffmpeg.exe`，`ffplay.exe`，`ffprobe.exe` 作为可执行文件的入口，文件体积很小，他们在运行的时候，如果需要，会到 `lib` 中调用相应的功能。

> 不带 `shared` 的里面，`bin` 目录中有 `ffmpeg.exe`，`ffplay.exe`，`ffprobe.exe` 三个可执行文件，每个 `exe` 的体积都稍大一点，因为它已经把相关的需要用的模块包编译到`exe`里面去了。

![image-20241226162005708](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20241226162005708.png)



![image-20241226162118231](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20241226162118231.png)

## 二、配置环境变量



1、在电脑桌面上，打开我的电脑
![在这里插入图片描述](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/21ab46b8cdb18d51ff65bb89bfd94ff3.png)

2、在空白处，右键，选择[属性]
![在这里插入图片描述](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/d8748cb2829b112b567c7bace215a2d0.png)
3、选择 高级系统设置 -> 高级 -> 环境变量
![在这里插入图片描述](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/591cc61c254455abc09d96d467e25992.png)

4、在系统变量中，选择 `Path`，然后编辑：

![image-20241226162247589](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20241226162247589.png)

## 三、安装成功验证

打开windows terminal或者cmd

- ffmpeg

![image-20241226162436758](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20241226162436758.png)

- ffplay

![image-20241226162559847](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20241226162559847.png)