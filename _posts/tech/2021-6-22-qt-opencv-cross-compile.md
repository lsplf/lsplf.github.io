#### ZCU102移植QT和OpenCV

#### 一、zcu102交叉编译工具链的安装
* zynq-7000系列为arm-linux-gnueabihf-前缀，而zynq_mpsoc系列为aarch64-linux-gnu-前缀。   
  如果安装了vitis或者petalinux就会自带有。如我安装了vitis 2020.1。则其zynq_mpsoc的工具链路径为：Vitis/2020.1/gnu/aarch64/lin/aarch64-linux/bin/。   
  值得注意的是Vitis/2020.1/gnu/aarch64/lin目录下还有一个aarch64-none文件夹，里面的aarch64-none-elf-是编译裸机程序的。   
  如fsbl、arm_trust_firmware、pmufw等。而uboot和linux内核，opencv，qt等，都需要用aarch64-linux-gnu-去编译。安装完vitis后，
  source Vitis/2020.1/settings64.sh这个文件就会导入编译器环境变量。
#### 二、zcu102移植opencv
1. 下载opencv源码。vitis 2020.1对应的opencv源码为3.4.3版本
2. 安装cmake: sudo apt-get install cmake   
3. 建立zynqmp.toolchain.cmake文件，内容如下
   ```cmake
    set( CMAKE_SYSTEM_NAME Linux )
    set( CMAKE_SYSTEM_PROCESSOR arm )
    set( CMAKE_C_COMPILER aarch64-linux-gnu-gcc )
    set( CMAKE_CXX_COMPILER aarch64-linux-gnu-g++ )
    set( CMAKE_INSTALL_PREFIX /home/ariza/software/opencv3.4.3_zynq )
    ```
   保存，并且拷贝到opencv-3.4.3/platforms/linux目录下。
4. 建立安装目录：mkdir ~/software/opencv3.4.3_zynq。
5. 在opencv-3.4.3根目录下创建build目录，并进入build目录。
6. 在build目录下执行如下命令：
   ```shell
   cmake -DBUILD_EXAMPLES=1 -DBUILD_ZLIB=1 -DBUILD_JPEG=1   
   -DCMAKE_TOOLCHAIN_FILE=../platforms/linux/zynqmp.toolchain.cmake    
   -DWITH_FFMPEG=0 -DWITH_PNG=0 -DWITH_1394=0 -DWITH_GTK=0 -DWITH_GTK_2_X=0   
   -DWITH_GSTREAMER=0 -DWITH_GSTREAMER_0_10=0 ..
   ```
7. 然后make -j4
8.  make install

#### 三、zcu102移植qt
1. vitis 2020.1对应的qt版本是5.13.2，需要下载源码用于交叉编译以及桌面版本qt用于编写程序。   
   分别为qt-everywhere-src-5.13.2和qt-opensource-linux-x64-5.13.2.run。
2. 安装桌面版本qt:  
   chmod +x qt-opensource-linux-x64-5.13.2.run
   ./qt-opensource-linux-x64-5.13.2.run
3. 交叉编译:
* 创建编译和安装目录:   
  ```shell
  mkdir ~/software/Qt5.13.2_zynq/build   
  mkdir ~/software/Qt5.13.2_zynq/install
  ```
* 设置编译的变量,在QT源码根目录下执行： 
  ```shell
  export CROSS_COMPILE=aarch64-linux-gnu-a
  export PATH=/home/ariza/software/vitis/2020.1/Vitis/2020.1/gnu/aarch64/lin/aarch64-linux/bin:$PATH 
  export ZYNQ_QT_BUILD=/home/ariza/software/Qt5.13.2_zynq/build
  export ZYNQ_QT_INSTALL=/home/ariza/software/Qt5.13.2_zynq/install
  export PATH=$ZYNQ_QT_INSTALL/bin:$PATH
  ```
* 下载编译配置文件：
  在[Qt & Qwt Build Instructions (Qt 5.4.2, Qwt 6.1.2)](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842110/Qt+Qwt+Build+Instructions+Qt+5.4.2+Qwt+6.1.2#Qt&QwtBuildInstructions(Qt5.4.2,Qwt6.1.2)-BuildandInstall.1)   
  下载qmake.conf和qplatformdefs.h。原链接文件带有版本号，更改名字去掉版本号，并且修改qmke.conf里面的内容，   
  里面为交叉编译工具链的前缀。内容如下：   
  ```make
    #
    # qmake configuration for building with aarch64-linux-gnu-g++
    #

    MAKEFILE_GENERATOR      = UNIX
    CONFIG                 += incremental
    QMAKE_INCREMENTAL_STYLE = sublib

    include(../common/linux.conf)
    include(../common/gcc-base-unix.conf)
    include(../common/g++-unix.conf)

    # modifications to g++.conf
    QMAKE_CC                = aarch64-linux-gnu-gcc
    QMAKE_CXX               = aarch64-linux-gnu-g++
    QMAKE_LINK              = aarch64-linux-gnu-g++
    QMAKE_LINK_SHLIB        = aarch64-linux-gnu-g++

    # modifications to linux.conf
    QMAKE_AR                = aarch64-linux-gnu-ar cqs
    QMAKE_OBJCOPY           = aarch64-linux-gnu-objcopy
    QMAKE_NM                = aarch64-linux-gnu-nm -P
    QMAKE_STRIP             = aarch64-linux-gnu-strip
    load(qt_config)
  ```
  创建存放上述文件的文件夹：
  mkdir -p qtbase/mkspecs/aarch64-linux-gnu-g++
  拷贝qmake.conf和qplatformdefs.h到这个目录下。
* 进行配置：
  ./configure -xplatform aarch64-linux-gnu-g++ -opensource -no-opengl -prefix $ZYNQ_QT_INSTALL
* 编译： make -j4
* 安装： make install
