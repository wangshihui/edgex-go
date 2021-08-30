# window下编译 edgex

golang 本事就是一个支持跨平台编译的语言, 所以完全可以支持在windows下进行edgex的开发,构建.


[edgex开发环境需求](https://docs.edgexfoundry.org/2.0/getting-started/Ch-GettingStartedDevelopers/)

## 硬件
edgex 虽然无关平台,但是对 内存,磁盘,性能较好的cpu还是由一定的需求

## 软件

- git

- redies,

  windows下可以用docker 运行redis镜像 或者 [Memurai](https://www.memurai.com/) 作为替代品
  
- MongoDB （自Geneva 日内瓦版本起已经标记为弃用）

- ZeroMQ  
  必选,运行时可选的MQ 组件有redis和ZeroMQ,但是编译时必须的。这个点也时edgex在不同平台下编译的最大区别
- Docker 可选

- 一门开发语言 c 或者 golang

所以edgex 在不同平台下编译唯一的差别就是 解决zeroMQ 依赖的方式不一样.

windows： [link](https://github.com/edgexfoundry/edgex-go/blob/master/ZMQWindows.md)

linux: [link](https://gist.github.com/katopz/8b766a5cb0ca96c816658e9407e83d00.)

mac: 
```shell
brew install zeromq
```

## windows下安装 zeromq的动态库

解决zeroMq动态库的安装问题,即可开始在windows下编译运行edgex的源码.

1. 从github上下载edgex源码到本地. edgex已经支持go的模块化. 所以可以下载源码到本地的任意位置

2. 下载下载安装 TDM-GCC, http://tdm-gcc.tdragon.net/download.  安装完成后,在 tdm-gcc的安装目录中的bin文件内找到
  `mingw32-make.exe` copy一份修改为 `make.exe`
3. 执行命令 `go get -v -x github.com/pebbe/zmq4`
  
  如果没有安装zero MQ 则会有一个可以预见的失败
  这个步骤可以跳过,但是 这个步骤会创建后续操作所必须的一些目录结构，如果跳过需要手动创建这些目录.
  
4. 下载平台依赖的zmq的库: https://ci.appveyor.com/project/zeromq/libzmq. 确保选择的版本是构建成功的且平台架构和本地的环境一致.
  
  比如: `Environment: platform=x64, configuration=Release, WITH_LIBSODIUM=ON, ENABLE_CURVE=ON, NO_PR=TRUE`. 

  提取压缩包中的文件, 

  拷贝`libsodium.dll` 和 `libzmq-v120-mt-4_x_x.dll` 这两个文件到 `%GOPATH%\pkg\mod\github.com\pebbe\zmq4@v1.0.0\lib`
  如果没有lib目录手动创建即可. 重命名 文件 `libzmq-v120-mt-4_x_x.dll` 为 `libzmq.dll`.
  
  如果这一步没有配置正确那么，会类似如下的错误提示
  
  ```shell
  C:/TDM-GCC-64/bin/../lib/gcc/x86_64-w64-mingw32/5.1.0/../../../../x86_64-w64-mingw32/bin/ld.exe:cannot find -lzmq
  ```
5.  下载zeromq源码
  
  ```shell
  git clone https://github.com/zeromq/libzmq.git
  ```
  reset 版本到上一步选择的zeromq的构建记录
  ```shell
  git reset --hard a84ffa12
  ```
  拷贝头文件`include`到`%GOPATH%\pkg\mod\github.com\pebbe\zmq4@v1.0.0\`.
6. 设置环境变量

powershell
```shell
$Env:CGO_CFLAGS="-I$Env:GOPATH\pkg\mod\github.com\pebbe\zmq4@v1.0.0\include"
$Env:CGO_LDFLAGS="-L$Env:GOPATH\pkg\mod\github.com\pebbe\zmq4@v1.0.0\lib"
```

cmd
```cmd
set CGO_CFLAGS="-I$Env:GOPATH\pkg\mod\github.com\pebbe\zmq4@v1.0.0\include"
set CGO_LDFLAGS="-L$Env:GOPATH\pkg\mod\github.com\pebbe\zmq4@v1.0.0\lib"
```
或者通过控制面板配置系统环境变量


以上步骤完成后,`go get -v -x github.com/pebbe/zmq4` 命令应该可以执行成功,否则需要重新操作上述3-6的步骤

当然推荐步骤6的环境变量设置为全局或者配置到IDE里,以避免因为忘记设置环境变量而导致的编译失败.

7. 接下来可以尝试编译了

进入到 `cmd\core-data` 目录

执行 `go get -v` 安装依赖
执行 `go build -v` 进行构建，如果构建过程提示类似如下错误,那很有可能是步骤6的环境变量没有正确配置,重新检查下环境变量配置,如 `go get env`
```
not finding zmq.h from auth.go
```

8. 准备

如果编译成功, 那么应该有一个构建产物`core-data.exe`.在步骤4中解压缩提取到的文件 中找到`libsodium.dll` 和 `libzmq-v120-mt-4_x_x.dll`, 病拷贝到到 `exe` 文件所在的目录

9. 执行

双击`exe` 文件.

如果运行报错, 那么 `$Env:GOCACHE="off"` 重新找一个合适版本的zeroMq的lib 步骤4-8


[win10下测试可用的zeroMq](lib-zeromq-win)