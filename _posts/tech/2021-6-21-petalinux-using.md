### petalinux学习

#### 一、编译步骤

1. 创建工程目录,如hello。拷贝zcu102.bsp到hello目录。  
   利用zcu102.bsp创建工程：petalinux-create -t project -s zcu102.bsp
2. 配置离线编译环境，运行petalinux-config。进行如下设置：
*  设置sstate。进入菜单“Yocto Settings ->Local sstate feeds settings ->local sstate feeds url” ，  
   按Enter键，提供sstate目录。对于我的arm64的路径，目录是/home/ariza/software/petalinux/2020.1/sstate/aarch64。   
*  设置本地download。进入菜单“Yocto Settings ->Add pre-mirror url” 里，按Enter键，以格式“file://”提供上述download目录，   
   比如我的路径为“/home/ariza/software/petalinux/2020.1/downloads”。   
*  进入菜单“Yocto Settings -> [] BB NO NETWORK”，按Enter键，选择“BB NO NETWORK”。  
这样设置过后保存退出，petalinux会进行相应的config编译流程。但是最后会报一个error说failed to add user_layer。  
此时运行sudo sysctl -n -w fs.inotify.max_user_watches=524288。然后再重新运行petalinux-config使其编译即可。
4. 编译：执行petalinux-build -x mrproper，然后petalinux-build。可以编译内核，uboot和根文件系统等。
5. 打包固件：执行以下命令：
   petalinux-package --boot --fsbl images/linux/zynqmp_fsbl.elf --u-boot=images/linux/u-boot.elf --pmufw --fpga images/linux/system.bit --force  
   然后在工程目录下image/linux中可以找到BOOT.bin,image.ub,rootfs.tar.gz，这三个文件就是我们要用到的制作启动SD卡的文件。
6. 制作SD卡启动镜像：
*  进行SD卡分区，一个分区为fat32格式，大小约为1G；另一个分区为ext4格式，大小约为14G。  
*  将生成的BOOT.bin与image.ub文件复制到fat32分区，rootfs.tar.gz解包到ext4分区。解包命令为：  
   sudo tar -xvf images/linux/rootfs.tar.gz -C /media/ariza/rootfs。注意一定要在命令行   
   再输入sync这个命令，否则SD卡启动不起来。
7. 拔出sd卡，设置zcu102从sd卡启动。sw6为off off off on(4321)。串口为COM9。linux系统账户密码为root和root   


#### 二、个别文件编译
1. device_tree文件: petalinux-build后生成的设备树文件在 /components/plnx_workspace/device-tree/device-tree下面。  
   单独修改device_tree后使用命令：     
   petalinux-build -c device-tree -x cleanall   
   petalinux-build -c device-tree
   然后重新将新生成的device_tree打包进image.ub使用命令：
   petalinux-build -x package
2. 修改文件系统从SD卡挂载：运行petalinux-config命令，选择Image Packaging Configration --> Root filesystem type，  
   选择SD卡。然后保存退出，再执行petalinux-build。
3. 重新编译rootfs:
   petalinux-config -c rootfs
   petalinux-build -c rootfs -x do_cleanall   
   petalinux-build -c rootfs
4. 重新编译kernel:
   petalinux-build -c kernel -x cleanall   
   petalinux-build -c kernel
5. 常用命令： https://www.programmersought.com/article/51752047495/
6. 系统启动后，使用ifconfig查看ip地址。如果不是192.168.1.10那么，将其修改为这个。  
   设置本地的网卡地址为192.168.1.11。然后使用filezila进行传输文件。命令为  
   ifconfig eth0 192.168.1.11 netmask 255.255.255.0  
7. 编译petalinux sdk: 使用以下命令：  
   petalinux-build --sdk
8. boot参数：console=ttyPS0,115200 root=/dev/mmcblk0p2 rw earlyprintk rootfstype=ext4 rootwait devtmpfs.mount=0 
#### 三、Open Source Linux移植
1. rootfs从SD卡启动只有boot.scr和设备树有关，与Boot.BIN和kernel无关。

#### 四、参考
1. [PetaLinux 工程的离线编译](https://www.cnblogs.com/hankfu/p/14074595.html)
2. [[米尔FZ3深度学习计算卡]petalinux环境搭建与petalinux编译](https://www.cirmall.com/bbs/thread-197656-1-1.html)


​     

   
