## makefile学习
本文将讲解gcc编译与makefile相关基础

### GCC编译
1. 过程：预处理->汇编->编译->链接
2. 各步骤作用
* 预处理：处理include与#define等宏定义与头文件包含命令  
          输入是.c，输出是.i。命令是gcc -E xxx.c -o xxx.i   
* 汇编：  将预处理后的.i文件正式进行编译，分析语法错误   
         生成汇编代码.s文件。命令是gcc -S xxx.i -o xxx.s  
* 编译：  将汇编后的.s文件生成机器码，也就是二进制文件.o。  
         命令是gcc -c xxx.s -o xxx.o或者as xxx.s -o xxx.o。
* 链接：  将生成的机器码文件与所需要的库文件进行链接，  
         生成完整的可执行程序。命令是gcc xxx.o -o xxx.out。 
1. 当直接执行gcc xxx.c -o xxx.out时，会自动执行4个步骤，得到   
   最终的输出

### 库文件的生成
1. 静态库生成：ar crsv libxxx.a xxx.o。将源文件xx.c生成xxx.o。   
   执行这条命令就可以生成静态库。   
   假如生成的libxxx.a在/LIB_DIR/下。则可以执行gcc main.o -L/LIB_DIR/ -lxxx -o main.out。
   或者直接使用：gcc main.o libxxx.a -o main.out。
2. 动态库的生成：g++ xxx.cpp -fPIC -shared -Wl,-soname,libxxx.so -o libxxx.so.0.1。   
   假如生成的libxxx.so在/LIB_DIR/下。则可以执行gcc main.o -L/LIB_DIR/ -lxxx -o main.out。或者直接使用：gcc main.o libxxx.so -o main.out。
3. -L指定库的搜索目录，-l指定库文件名字。默认的搜索路径为环境变量LD_LIBRARY_PATH以及  
   /lib/usr/lib /usr/local/lib。
4. 如果链接路径下同时有.so和.a则优先选择链接.so。
5. 在动态链接库安装的时候总是复制库文件到某个目录，然后用软连接生成别名，   
   在库文件进行更新的时候仅仅更新软连接即可。使用sudo ln -s 被链接的文件 软链接文件。   
   如sudo ln -s libxxx.so.0.1 libxxx.so。

### 头文件搜索原则
1. <xxx.h>： 只在默认的系统包含路径搜索
2. 'xxx.h":  首先在当前目录以及-I指定的目录搜索，最后再到系统默认的路径搜索。
3. 指定头文件目录： -I/INC_DIR/
4. 指定头文件： -include xxx.h

### 参考
[A Simple Makefile Tutorial](https://www.cs.colby.edu/maxwell/courses/tutorials/maketutor/)