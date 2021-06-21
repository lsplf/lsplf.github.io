### 编译petalinux的步骤
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
4. 编译：执行petalinux-build。可以编译内核，uboot和根文件系统等。
5. 打包固件：执行以下命令：   
   petalinux-package --boot --fsbl images/linux/zynqmp_fsbl.elf --u-boot=images/linux/u-boot.elf --pmufw --atf –fpga images/linux/system.bit   
   然后在工程目录下image/linux中可以找到BOOT.bin,image.ub,rootfs.tar.gz，这三个文件就是我们要用到的制作启动SD卡的文件。
6. 制作SD卡启动镜像：
*  进行SD卡分区，一个分区为fat32格式，大小约为1G；另一个分区为ext4格式，大小约为14G。  
*  将生成的BOOT.bin与image.ub文件复制到fat32分区，rootfs.tar.gz解包到ext4分区。解包命令为：  
   sudo tar -xvf petalinux/hello/xilinx-zcu102-2020.1/images/linux/rootfs.tar.gz -C /media/ariza/挂载目录
7. 拔出sd卡，设置zcu102从sd卡启动。sw6为off off off on(4321)。串口为COM9。linux系统账户密码为root和root
