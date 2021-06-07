---
layout: post
title: Vitis Library环境搭建与使用
category: 技术
tags: Vitis
keywords: Vitis,Vivado,HLS,FPGA,ISP
---
## 一、 概述
Xilinx在2019的版本提出了Vitis平台，其实是将原来的SDSoC、SDAccl、SDK等嵌入式平台统一起来，统一在Vitis里面。这样Xilinx  
以后的开发工具就只有三个:  
1) Vivado:用于开发底层硬件平台  
2) Vivado HLS:用于将高层次语言转换成HDL
3) Vitis：用于给硬件平台开发驱动  
Xilinx提供了一个Vitis库，里面包含各种C的图像处理算法，包括传统的ISP算法：demosaic, LSC, BPC等，还包括OpenCV库，里面  
有Harri、Stereo等算法其实就是以前它提出的xfOpencv。  这些可以通过HLS快速的转换成硬件加速器，大大地提高开发效率。本文  
描述如何使用这个Vitis Library。

## 二、 建立环境
1. Vitis软件安装：在Ubantu 18.04下安装好Vitis 2020.1。其中会包括HLS,Vitis,Vivado。
2. 安装Xilinx Runtime库：此库用于提供运行时环境，编译Vitis Library需要。  
   在官网上下载对应版本的安装包deb文件，然后直接安装这个deb时会出错。此时报错  
   是因为缺少相关的依赖。此时可以将XRT源码repo下载下来，然后去找里面的一个依赖  
   安装脚本，安装完依赖后再安装deb文件。参考链接：https://www.yuque.com/fujz/zynq/fyt1zn
3. 安装OpenCV 3.4.11：   
   1）下载OpenCV源码  
   2）在源码文件夹根目录下建立build目录，用于存放编译出来的文件  
   3）进入build目录，执行命令:  
      cmake -D CMAKE_BUILD_TYPE=Release -D CMAKE_INSTALL_PREFIX=/usr/local ..       
      表示安装到/usr/local目录。   
   4）执行make -j8  
   5）执行make install。执行完该命令可以在/usr/local/bin下可以找到opencv的可执行文件，可以在     
      /usr/local/lib下找到opencv的.so库文件。可以在/usr/local/include下找到opencv的头文件。   
   6）配置环境: sudo gedit /etc/ld.so.conf.d/opencv.conf，在这个文件里面添加编译出来的库的路径，   
      本次安装为默认的路径:/usr/local/lib。保存退出，执行命令：sudo ldconfig用于更新系统共享链接库。   
   7）修改bash.bashrc文件，在末尾加入下面两句命令，并保存退出：     
      PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/local/lib/pkgconfig     
      export PKG_CONFIG_PATH   
   8）执行source /etc/bash.bashrc使配置生效   
 4. 下载Vitis Library源码：https://github.com/Xilinx/Vitis_Libraries/tree/2020.1  
    下载对应的Vitis版本，该链接我已经切换2020.1分支。

## 三、编译例程
1. Vitis Library的图像处理算法代码都放在源码根目录下的vision文件夹。里面分为L1,L2,L3。代表使用的不同层级。  
L1代表是一个kernel，在HLS里面综合出一个IP核，L2和L3类似原来在SDx使用的流程，直接集成到嵌入式工程里面。   
本文评估L1里面的例程，使用HLS综合得到IP，在Vivado中调用。
2. L1目录下含有examples和tests目录。在tests目录下对应例子进行编译会自动调用examples下面的代码。  
本文编译一个图像resize IP，因此进入Vitis_Libraries/vision/L1/tests/resize/resize_DOWN_AREA_NO_RGB目录。
里面有一个Makefile文件，还有一个xf_config_params.h用于提供生成IP时的参数。  
3. 在编译时，需要执行以下命令，提高相应的变量给Makefile。命令如下：  
source /home/ariza/software/vitis/2020.1/Vitis/2020.1/settings64.sh  
source /opt/xilinx/xrt/setup.sh  
export XPART='xczu9eg-ffvb1156-2-e'   
export OPENCV_LIB=/usr/local/lib  
export LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:/usr/local/lib 
然后执行make命令：make run CSIM=1 CSYNTH=1 COSIM=0     
注意在这样执行的时候会报check_opecv出错。需要将makefile的第77行内容：ifeq (,$(XPART))  
剪切到第177行内容：check_part: check_platform check_vpp前面，这样才能编译通过。原因是当  
设置了XPART后，原有的makefile由于在第77行执行了ifeq(, $(XPART))后，后面的很多目标都没有被执行。  
里面上XPART应该只需要影响check_part就可以，不应该影响其他。所以将77行剪切到177行check_part目标前。  
（针对我个人的环境，我的文件夹路径为"/home/ariza/project/revision/test/Vitis_Libraries"，且备份了一个  
修改后的可以直接编译的makefile在"/home/ariza/project/revision/test/编译能通过的makefile" 目录下面，  
直接拷贝到工作目录就可以.注意，工作目录一定要是vitis_libraries下面的tests目录，否则会报错，如我本次例程   
的工作目录为："/home/ariza/project/revision/Vitis_Libraries/Vitis_Libraries-master/vision/L1/tests/resize/resize_DOWN_BILINEAR_RO")
4. 编译完成会在当前目录生成一个resize.proj,这是一个hls的工程文件，使用hls打开就可以看到相应的结果。
在hls界面上选择Run->export rtl就可以生成IP文件。


## 四、参考
- [XRT 2020.1安装](https://www.yuque.com/fujz/zynq/fyt1zn)
- [Vitis Vision Library User Guide](https://xilinx.github.io/Vitis_Libraries/vision/2020.2/index.html)
- [Vitis HLS 2020.2使用Vitis Vision实例代码实现图像处理sobelfilter](https://blog.csdn.net/u011747505/article/details/112612215)
