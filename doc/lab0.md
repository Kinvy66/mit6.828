# Lab0

操作系统：Ubuntu-18.04.5-64bit（VM虚拟机）/ Ubuntu20.04(WSL2)

课程主页：[mit6.828（2018）](https://pdos.csail.mit.edu/6.828/2018/schedule.html)

环境配置 [参考](https://pdos.csail.mit.edu/6.828/2018/tools.html)

注：mit6.828 课程从2019年开始使用 RISC_V 处理器，不再使用x86(IA-32)架构。

## 编译工具链

### 1.测试编译工具

```shell
$objdump -i
```

<img src="https://kinvy-images.oss-cn-beijing.aliyuncs.com/Images/image-20210727123132004.png" title="" alt="image-20210727123132004" data-align="center">

```shell
$gcc -m32 -print-libgcc-file-name         #测试gcc
```

上面这条命令是测试gcc的，一般系统是没有gcc的，需要安装

安装gcc , gdb, git, vim

```shell
$sudo apt-get install -y build-essential gdb git vim
```

安装32位的支持库

```shell
$sudo apt-get install gcc-multilib
```

### 2. 编译安装工具链

#### 2.1下载以下工具包

- [ftp://ftp.gmplib.org/pub/gmp-5.0.2/gmp-5.0.2.tar.bz2](ftp://ftp.gmplib.org/pub/gmp-5.0.2/gmp-5.0.2.tar.bz2)
- https://www.mpfr.org/mpfr-3.1.2/mpfr-3.1.2.tar.bz2
- http://www.multiprecision.org/downloads/mpc-0.9.tar.gz
- http://ftpmirror.gnu.org/binutils/binutils-2.21.1.tar.bz2
- http://ftpmirror.gnu.org/gcc/gcc-4.6.4/gcc-core-4.6.4.tar.bz2
- http://ftpmirror.gnu.org/gdb/gdb-7.3.1.tar.bz2

以上链接课程官网给出的，部分链接已失效，可复制链接前面部分网址，下载相应版本的软件包。

所有工具包也可在网盘下载：

https://pan.baidu.com/s/13vcPrmgT8p0zF-GD-d-cJQ 
提取码：yr1r

#### 2.2 编译安装

为了方便，将以上6个压缩包放在一个文件夹下 ，`~/download/mit6.828`

文件夹结构

![image-20210727131718755](https://kinvy-images.oss-cn-beijing.aliyuncs.com/Images/image-20210727131718755.png)

> 以下的操作都是在 `~/download/mit6.828` 目录下

1. 安装gmp-5.0.2

   ```shell
   $tar xjf gmp-5.0.2.tar.bz2
   $cd gmp-5.0.2
   $./configure --prefix=/usr/local   # 可能的错误：No usable m4 in $PATH or /usr/5bin (see config.log for reasons).
   $make
   $sudo make install             
   $cd ..
   ```

   逐条执行命令，每执行一条后，输出无 `error` 就可往下执行，后面几个安装包也是一样的

   > 可能的错误是第3个命令，如果报错，执行以下命令，然后再次执行第3行命令

   ```shell
   $sudo apt install m4
   ```

2. 安装mpfr-3.1.2

   ```shell
   $tar xjf mpfr-3.1.2.tar.bz2
   $cd mpfr-3.1.2
   $./configure --prefix=/usr/local
   $make
   $sudo make install           
   $cd ..
   ```

3. 安装mpc-0.9

   ```shell
   $tar xzf mpc-0.9.tar.gz
   $cd mpc-0.9
   $./configure --prefix=/usr/local
   $make
   $sudo make install            
   $cd ..
   ```

4. 安装binutils-2.21.1

   ```shell
   $tar xjf binutils-2.21.1.tar.bz2
   $cd binutils-2.21.1
   $./configure --prefix=/usr/local --target=i386-jos-elf --disable-werror
   $make
   $sudo make install             # This step may require privilege (sudo make install)
   $cd ..
   
   #测试
   $i386-jos-elf-objdump -i
   # 成功安装会输出类似下面的信息
   # BFD header file version (GNU Binutils) 2.21.1
   # elf32-i386
   #  (header little endian, data little endian)
   #   i386...
   ```

5. 安装gcc-core-4.6.4

   ```shell
   $tar xjf gcc-core-4.6.4.tar.bz2
   $cd gcc-4.6.4
   $mkdir build           
   $cd build
   $../configure --prefix=/usr/local \
       --target=i386-jos-elf --disable-werror \
       --disable-libssp --disable-libmudflap --with-newlib \
       --without-headers --enable-languages=c MAKEINFO=missing
   $make all-gcc
   $sudo make install-gcc         
   $make all-target-libgcc        #可能会报错 [configure-target-libgcc] Error 1
   $sudo make install-target-libgcc   
   $cd ../..
   
   #测试
   $i386-jos-elf-gcc -v
   # 成功安装会输出类似下面的信息
   # Using built-in specs.
   # COLLECT_GCC=i386-jos-elf-gcc
   # COLLECT_LTO_WRAPPER=/usr/local/libexec/gcc/i386-jos-elf/4.6.4/lto-wrapper
   # Target: i386-jos-elf
   ```

   > 执行11行命令可能会报错，如果报错，执行以下命令，然后再次执行第11行命令

   ```shell
   $export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib 
   ```

6. 安装gdb-7.3.1

   ```shell
   $tar xjf gdb-7.3.1.tar.bz2
   $cd gdb-7.3.1
   $./configure --prefix=/usr/local --target=i386-jos-elf --program-prefix=i386-jos-elf- \
       --disable-werror
   $make all            #可能的错误 no termcap library found
   $sudo make install         
   $cd ..
   ```

   > 可能报错的命令第5个，如果出现错误，执行以下命令，然后再执行该命令

   ```shell
   $wget http://ftp.gnu.org/gnu/termcap/termcap-1.3.1.tar.gz
   $tar -zxv -f termcap-1.3.1.tar.gz
   $cd termcap-1.3.1
   $ ./configure 
   $make
   $sudo make install
   $cd ..
   ```

## 安装 QEMU

### 1. 安装工具包

```shell
$sudo apt install libsdl1.2-dev libtool-bin libglib2.0-dev  libz-dev  libpixman-1-dev
$sudo apt install python2
```

安装python2可能会出错，参考https://blog.csdn.net/u010879745/article/details/125115600 解决

```bash
$ cd /usr/local/lib
$ sudo rm libgmp*
$ sudo apt --fix-broken install
$ sudo apt update
```

完成以上步骤回到之前的工作目录继续



### 2. 下载qemu

qemu需要用6.828定制的

```shell
$git clone https://github.com/mit-pdos/6.828-qemu.git qemu
```

### 3. 编译安装

```shell
$./configure --disable-kvm --disable-werror --prefix=/usr/local  \
	--target-list="i386-softmmu x86_64-softmmu" --python=python2
$make
$sudo make install
```

可能的错误：

1. 缺少一个头文件，错误如下

   ```shell
   qga/commands-posix.c: In function ‘dev_major_minor’:
   qga/commands-posix.c:633:13: error: In the GNU C Library, "major" is defined
    by <sys/sysmacros.h>. For historical compatibility, it is
    currently defined by <sys/types.h> as well, but we plan to
    remove this soon. To use "major", include <sys/sysmacros.h>
    directly. If you did not intend to use a system-defined macro
    "major", you should undefine it after including <sys/types.h>. [-Werror]
            *devmajor = major(st.st_rdev);
                ^~~~~~~~~~~~~~~~~~~~~~~~~~   
   ```

   > 解决：在 qga/commands-posix.c文件中的 #include <sys/types.h> 下面增加#include <sys/sysmacros.h>即可

   ![image-20230331114207290](https://kinvy-images.oss-cn-beijing.aliyuncs.com/Images/image-20230331114207290.png)

### 4.测试

```shell
#下载实验源码
$git clone https://pdos.csail.mit.edu/6.828/2018/jos.git lab
$cd lab
$make
$make qemu-noxl
```

make 可能报错 
```bash
/usr/local/libexec/gcc/i386-jos-elf/4.6.4/cc1: error while loading shared libraries: libmpc.so.2:
```

解决方法参考：https://stackoverflow.com/questions/19625451/cc1-error-while-loading-shared-libraries-libmpc-so-2-cannot-open-shared-objec

测试成功

![image-20210727171159202](https://kinvy-images.oss-cn-beijing.aliyuncs.com/Images/image-20210727171159202.png)
