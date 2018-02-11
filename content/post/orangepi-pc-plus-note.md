---
title: "Orangepi Pc Plus 研究笔记"
date: 2018-02-11T11:27:31+08:00
tags: ["iot","orangepi"]
topics: ["iot"]
---

本文主要是对在使用OrangePi PC Plus开发板的过程中遇到的一些问题的记录。

### 查看系统信息 

由于移植的系统都是进行过裁剪的，所以很多在Linux中常用的工具命令可能都被删除掉了，从而查看一些系统的信息方式需要另找工具或者入口。特别是对于Android系统的使用上更是如此。以下是一些额外的途径去获取需要的系统信息。

1. 内核版本信息：uname 命令不存在时，可用 cat /proc/version
2. 内存使用信息：free 命令不存在时，可用 cat /proc/meminfo
3. 网络接口信息：ifconfig 命令不可用时, 可用 netcfg命令，或者 ip addr命令

    ```
    root@dolphin-fvd-p1:/ # ip addr
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN 
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
        inet6 ::1/128 scope host 
           valid_lft forever preferred_lft forever
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
        link/ether 02:ae:79:67:9b:30 brd ff:ff:ff:ff:ff:ff
        inet 192.168.0.180/24 brd 192.168.0.255 scope global eth0
        inet6 fe80::ae:79ff:fe67:9b30/64 scope link 
           valid_lft forever preferred_lft forever
    3: tunl0: <NOARP> mtu 1480 qdisc noop state DOWN 
        link/ipip 0.0.0.0 brd 0.0.0.0
    4: gre0: <NOARP> mtu 1476 qdisc noop state DOWN 
        link/gre 0.0.0.0 brd 0.0.0.0
    5: sit0: <NOARP> mtu 1480 qdisc noop state DOWN 
        link/sit 0.0.0.0 brd 0.0.0.0
    6: ip6tnl0: <NOARP> mtu 1452 qdisc noop state DOWN 
        link/tunnel6 :: brd ::
    7: wlan0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DORMANT qlen 1000
        link/ether 00:ae:79:67:9b:31 brd ff:ff:ff:ff:ff:ff
    8: p2p0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DORMANT qlen 1000
        link/ether 02:ae:79:67:9b:31 brd ff:ff:ff:ff:ff:ff
    root@dolphin-fvd-p1:/ # 
    root@dolphin-fvd-p1:/ # netcfg                                                 
    sit0     DOWN                                   0.0.0.0/0   0x00000080 00:00:00:00:00:00
    p2p0     UP                                     0.0.0.0/0   0x00001003 02:ae:79:67:9b:31
    gre0     DOWN                                   0.0.0.0/0   0x00000080 00:00:00:00:00:00
    eth0     UP                               192.168.0.180/24  0x00001043 02:ae:79:67:9b:30
    lo       UP                                   127.0.0.1/8   0x00000049 00:00:00:00:00:00
    wlan0    UP                                     0.0.0.0/0   0x00001003 00:ae:79:67:9b:31
    tunl0    DOWN                                   0.0.0.0/0   0x00000080 00:00:00:00:00:00
    ip6tnl0  DOWN                                   0.0.0.0/0   0x00000080 00:00:00:00:00:00
    root@dolphin-fvd-p1:/ #
    ```

### Android应用安装

1. 通过sd卡拷贝进行apk包安装注意事项

    - apk包需要放置在data目录下（root用户具有读写权限的目录下）
    - apk包的权限需要设置为644: chmod 644 UsbModeSwitch.apk
    
2. 交叉编译的可执行文件拷贝到android/linux系统下运行，提示/system/bin/sh: : No such file or directory错误

    这一般是系统的动态链接器与XXX这个程序中的动态链接器的名字或路径不对,通过readelf命令查看解释器。
    
    ```
    bruce@ubuntu:~/orangepi/i2c-tools/tools$ arm-linux-gnueabihf-readelf -l i2cdetect

    Elf file type is EXEC (Executable file)
    Entry point 0x90ed
    There are 8 program headers, starting at offset 52
    
    Program Headers:
      Type           Offset   VirtAddr   PhysAddr   FileSiz MemSiz  Flg Align
      EXIDX          0x0021a4 0x0000a1a4 0x0000a1a4 0x00008 0x00008 R   0x4
      PHDR           0x000034 0x00008034 0x00008034 0x00100 0x00100 R E 0x4
      INTERP         0x000134 0x00008134 0x00008134 0x00019 0x00019 R   0x1
          [Requesting program interpreter: /lib/ld-linux-armhf.so.3]
      LOAD           0x000000 0x00008000 0x00008000 0x021b0 0x021b0 R E 0x8000
      LOAD           0x0021b0 0x000121b0 0x000121b0 0x001a8 0x001bc RW  0x8000
      DYNAMIC        0x0021bc 0x000121bc 0x000121bc 0x000e8 0x000e8 RW  0x4
      NOTE           0x000150 0x00008150 0x00008150 0x00044 0x00044 R   0x4
      GNU_STACK      0x000000 0x00000000 0x00000000 0x00000 0x00000 RW  0x10
    
     Section to Segment mapping:
      Segment Sections...
       00     .ARM.exidx 
       01     
       02     .interp 
       03     .interp .note.ABI-tag .note.gnu.build-id .hash .dynsym .dynstr .gnu.version .gnu.version_r .rel.dyn .rel.plt .init .plt .text .fini .rodata .ARM.exidx .eh_frame 
       04     .init_array .fini_array .jcr .dynamic .got .data .bss 
       05     .dynamic 
       06     .note.ABI-tag .note.gnu.build-id 
       07     
    ```
    
    其中的
    
          INTERP         0x000134 0x00008134 0x00008134 0x00019 0x00019 R   0x1
              [Requesting program interpreter: /lib/ld-linux-armhf.so.3]
    
    说明了缺少的文件，拷贝到android对应目录即可！
    
3. 交叉编译的可执行文件在Android执行提示：Permission Denied！动态编译不行，静态编译可以(-static选项)。

    当然前提条件是check可执行文件是否有执行权限: chmod 777 i2cdetect(示例)

    ```
    root@dolphin-fvd-p1:/data/tmp # chmod 777 hello_*                          
    root@dolphin-fvd-p1:/data/tmp # ls -l
    -rw-r--r-- root     root        24627 2017-11-17 11:49 UsbModeSwitch.apk
    -rwxrwxrwx root     root         6050 2017-11-17 14:08 hello_dynamic
    -rwxrwxrwx root     root       507980 2017-11-17 14:08 hello_static
    drwx------ root     log               2017-11-17 11:50 i2ctools
    -rw------- system   system         16 2017-11-17 09:51 ota_uuid.txt
    root@dolphin-fvd-p1:/data/tmp # ./hello_dynamic                                
    sh: ./hello_dynamic: Permission denied
    1|root@dolphin-fvd-p1:/data/tmp # ./hello_static                               
    hello world!
    ```

4. 测试Android应用程序

    - UsbModeSwitch APP: 用来打开usb的adb调试选项，点开后选择enable usb debug option，每次关机后都要重新操作以下才能用！！
    - SuperSu APP： 用来对APP的root权限进行授权操作，当前主要是对OpiGpioTest授权
    - OpiGpioTest APP： 用来测试OrangPi开发板上的GPIO管脚功能，支持获取，设置方向，设置高低电平，设置需要root权限授权
    - SerialTool APP： 用来测试OrangePi开发板上的串口通讯功能，需要通过adb shell手动设置对应串口节点权限为0666，因为APP没有获取root权限
    - I2C测试工具： 在/data/tmp/i2ctools目录下有i2cdetect,i2cset,i2cget等工具可以用来检测i2c设备，获取设置对应设备的值，只能在adb shell下执行测试

### Android源码编译步骤

参考官方操作手册(虽然有坑)： OrangePi_PCPlus_H3_用户手册_v0.1.pdf

1. 安装Ubuntu 16.04LTS 64bit系统版本，使用虚拟机，硬件配置建议如下：
    
    - CPU：2, 内存: 4G, SWAP: 2G, 硬盘空间: 至少50GB,建议100GB
    - 需要能够连接Internet来进行依赖包的安装，因此建议使用NAT网络
    
2. 系统编译需要的依赖工具和环境包检查与安装：
    
    - Python 的2.7.3 版本(系统一般默认满足，检查： python --version)
    - GNU Make 的3.81-3.82 版本(android 编译时的make版本必须是3.81, 3.82,需要手动安装)
        
        ```
        $ wget http://ftp.gnu.org/pub/gnu/make/make-3.81.tar.gz
        $ tar zxvf make-3.8.1.tar.gz ; cd make-3.8.1;
        $ ./configure --prefix=/usr/ ; make ; make install
        ```
    
    - git 的1.7 或更高版本(检查：git --version;安装: sudo apt-get install git)
    - Java 1.6 版本, 官网下载jdk-6u31-linux-x64.bin(环境变量的配置建议添加到profile中，这样可以避免每次重新添加)
        
        ```
        $ ./jdk-6u31-linux-x64.bin
        $ export JAVA_HOME=$PWD/jdk1.6.0_31
        $ export PATH=$PATH:/$JAVA_HOME/bin
        $ export CLASSPATH=.:$JAVA_HOME/lib
        $ export JRE_HOME=$JAVA_HOME/jre
        $ java # 测试是否成功配置
        ```
    
    - mkimage工具构建uImage: sudo apt-get install uboot-utils
    - gawk工具编译Android需要： sudo apt-get install gawk
    - libncurses5包，make menuconfig配置内核需要
    
            $ sudo apt-get install libncurses5 libncurses5-dev
        
    - 其他需要的构建依赖包安装：
    
        ```
        $ sudo apt-get install git gnupg flex bison gperf build-essential \
        zip curl libc6-dev libncurses5-dev:i386 x11proto-core-dev \
        libx11-dev:i386 libreadline6-dev:i386 libgl1-mesa-glx:i386 \
        libgl1-mesa-dev g++-multilib mingw32 tofrodos \
        python-markdown libxml2-utils xsltproc zlib1g-dev:i386
        $ sudo ln -s /usr/lib/i386-linux-gnu/mesa/libGL.so.1 /usr/lib/i386-linux-gnu/libGL.so
        ```
        
3. orangepi 官网资源下载页面下载PC Plus板的Android 源码，解压到orangepi目录(可能需要二十分钟左右)：

        mkdir orangepi; tar xf qin2-sync-yunos-v1.0.tar.gz -C orangepi/
    
    解压后得到两个目录： android lichee

4. 编译lichee内核

    ```
    $ cd lichee
    $ ./build.sh lunch # 选择 2 - sun8iw7p1 - android - dolphin
    ```
    
    等待编译完成，成功的打印信息： build sun8iw7p1 android dolphin lichee ok
    
5. Android源码编译

    ```
    $ cd android
    $ source ./build/envsetup.sh
    $ lunch dolphin_fvd_p1-eng  #选择方案号
    $ extract-bsp #拷贝内核及驱动模块
    $ make # 等待编译完成，大概需要6个小时左右
    $ pack #打包生成固件, 位置在 orangepi/lichee/tools/pack/
    $ ls -lh orangepi/lichee/tools/pack/sun8iw7p1_android_dolphin-p1_uart0.img
    -rwxrwxr-x 1 bruce bruce 333M Nov 14 23:00 orangepi/lichee/tools/pack/sun8iw7p1_android_dolphin-p1_uart0.img
    ```
    
6. Android镜像烧录到microSD卡

    - 通过共享文件夹功能拷贝镜像到Windows主机系统中：烧录只有Windows的工具
    - OrangePi官网资源下载页面下载烧录软件工具PhoenixCard
    - 解压进入文件夹，以管理员身份运行PhoenixCard工具
    - 选择插入的microSD卡，选择编译出来的镜像文件，选择卡启动，点击烧录进行烧写
    - 打印信息中：magic完成，烧写结束... 表示烧写成功，弹出卡插入开发板启动
    
    Note: PhoenixCard工具使用不太稳定，出现以下问题请多试几次：
    
    - 提示"处理出错..."： 多试几次烧录，或者更换microSD卡再尝试
    - 一直卡在"正在格式化卡..."不动: 更换microSD卡再试(卡在Windows中是可以正常访问的，但就是不行)
    
    
### Android镜像构建分析

参照上面的步骤可以编译出Android镜像，但是在PC Plus上启动后有以下问题：

- 界面停在"连接无线网络": 重启系统就可以出现Android界面了【PASS】
- 鼠标无法操作问题：重新插拔一次鼠标就可以使用【PASS】
- 不支持无线网卡：检查发现系统没有加载wlan驱动，没有wlan0网络接口生成，型号是Realtek RTL8189FTV SDIO-based WiFi
- 不支持eMMC： 没有找到eMMC的设备节点
- 不支持I2C设备节点

#### 1. lichee内核构建

构建结果主要存放在两个位置，目录中的内容是一样的，所以应该是拷贝的结果：

    - linux-3.4/output
    - out/sun8iw7p1/android/common/
    
其中bImage文件和lib目录下的内核modules文件会被之后的Android构建命令extract-bsp拷贝到device/softwinner/dolphin-fvd-p1/目录下，bImage文件重命名为kernel。

遗憾的是lichee中构建出来的boot.img文件并没有拷贝过去，android是自行构建打包的boot.img文件，对比如下：

    bruce@ubuntu:~/orangepi$ find . -name boot.img |xargs md5sum
    e0dc3bde0e5987ec52b129da51e48413  ./lichee/out/sun8iw7p1/android/common/boot.img
    e0dc3bde0e5987ec52b129da51e48413  ./lichee/linux-3.4/output/boot.img
    9dcb37696cf08001a8a7c8127906d23e  ./android/out/target/product/dolphin-fvd-p1/boot.img

此外，工程配置文件sys_config.fex是从构建目标的configs文件中拷贝出来使用的，源文件是：

    lichee/tools/pack/chips/sun8iw7p1/configs/dolphin-p1/sys_config.fex
    
因此如果需要修改配置，可以通过修改该文件然后重新编译构建实现，当然如果能直接替换当前系统的script.bin那就更好了。
    
#### 2. Android镜像构建

镜像主要是三个文件：boot.img,system.img,recovery.img

    bruce@ubuntu:~/orangepi/android$ ls out/target/product/dolphin-fvd-p1/*.img -lh
    -rw-r--r-- 1 bruce bruce  12M Nov 14 22:05 out/target/product/dolphin-fvd-p1/boot.img
    -rw-rw-r-- 1 bruce bruce 2.2M Nov 14 22:05 out/target/product/dolphin-fvd-p1/ramdisk.img
    -rw-rw-r-- 1 bruce bruce 3.1M Nov 14 22:05 out/target/product/dolphin-fvd-p1/ramdisk-recovery.img
    -rw-r--r-- 1 bruce bruce  13M Nov 14 22:05 out/target/product/dolphin-fvd-p1/recovery.img
    -rw-r--r-- 1 bruce bruce 299M Nov 14 22:05 out/target/product/dolphin-fvd-p1/system.img

Android 产品中，内核格式是Linux标准的zImage(这里好像用的是bImage)，根文件系统采用ramdisk格式(android构建目标)。这两者在Android下是直接合并在一起取名为boot.img,会放在一个独立分区当中。这个分区格式是Android自行制定的格式。

上面也许解释了为什么Android需要自行重新构建boot.img文件，而不使用lichee构建的。因此我们不能直接用lichee构建出来的boot.img来简单替换Android的，行不通。

Android开发时，最标准的做法是重新编译于内核和根文件系统，然后调用Android给的命令行文件mkbootimg(out/host/linux-x86/bin/)来打包。

在制作手机ROM时，有时会单独编译内核或抽出根文件进行修改内容，比如我只编译内核，其余的地方不变。这样重新安装巨大的Android开发环境实在不划算。因此很多boot.img解包工具被人开发出来，这一些工具都是把内核和根文件系统从一个现成的boot.img抽取出来，修改后再次打包还原。 【相比整个重新编译的话可以节省很多时间】

boot.img的基本格式：boot header(1 page) + kernel(n pages) + ramdisk(m pages) + second stage( x pages)

关于镜像文件的打包与解包参考： [Linux下解包/打包Android映像文件](https://www.cnblogs.com/cute/p/4140230.html)

可惜在Android源码中没有找到unpackbootimg工具，只能下载开源的自行编译： [GitHub unpackbootimg](https://github.com/huaixzk/unpackbootimg.git)

    bruce@ubuntu:~/orangepi$ git clone https://github.com/huaixzk/unpackbootimg.git
    bruce@ubuntu:~/orangepi$ cd unpackbootimg; make
    bruce@ubuntu:~/orangepi/unpackbootimg$ ./unpackbootimg 
    usage: unpackbootimg
    	-i|--input boot.img
    	[ -o|--output output_directory]
    	[ -p|--pagesize <size-in-hexadecimal> ]
    bruce@ubuntu:~/orangepi/unpackbootimg$ ./unpackbootimg -i boot.img -o ./out/
    BOARD_KERNEL_CMDLINE 
    BOARD_KERNEL_BASE 40000000
    BOARD_PAGE_SIZE 2048
    bruce@ubuntu:~/orangepi/unpackbootimg$ ls out/ -lh
    total 12M
    -rw-rw-r-- 1 bruce bruce    9 Nov 15 23:50 boot.img-base
    -rw-rw-r-- 1 bruce bruce    1 Nov 15 23:50 boot.img-cmdline
    -rw-rw-r-- 1 bruce bruce    5 Nov 15 23:50 boot.img-pagesize
    -rw-rw-r-- 1 bruce bruce 2.2M Nov 15 23:50 boot.img-ramdisk.gz
    -rw-rw-r-- 1 bruce bruce 9.7M Nov 15 23:50 boot.img-zImage
    
比较一下解包出来的zImage与打包进去的kernel/bImage镜像文件是否一样：

    bruce@ubuntu:~/orangepi$ md5sum lichee/linux-3.4/output/bImage 
    693cf939b8b6fd7d46d9b0e78ac3c6f0  lichee/linux-3.4/output/bImage
    bruce@ubuntu:~/orangepi$ md5sum unpackbootimg/out/boot.img-zImage 
    693cf939b8b6fd7d46d9b0e78ac3c6f0  unpackbootimg/out/boot.img-zImage
    bruce@ubuntu:~/orangepi$ md5sum android/out/target/product/dolphin-fvd-p1/kernel 
    693cf939b8b6fd7d46d9b0e78ac3c6f0  android/out/target/product/dolphin-fvd-p1/kernel

发现三个文件是一样的，说明解包没问题。我们还需要验证下重新打包的话能否恢复：

    bruce@ubuntu:~/orangepi/unpackbootimg$ ./mkbootimg 
    error: no output filename specified
    usage: mkbootimg
           --kernel <filename>
           --ramdisk <filename>
           [ --second <2ndbootloader-filename> ]
           [ --cmdline <kernel-commandline> ]
           [ --board <boardname> ]
           [ --base <address> ]
           [ --pagesize <pagesize> ]
           [ --ramdiskaddr <address> ]
           -o|--output <filename>
    bruce@ubuntu:~/orangepi/unpackbootimg$ ./mkbootimg --kernel out/boot.img-zImage --ramdisk out/boot.img-ramdisk.gz --base 40000000 --pagesize 2048 -o boot.img.repack
    bruce@ubuntu:~/orangepi/unpackbootimg$ ls -lh boot.img*
    -rw-r--r-- 1 bruce bruce 12M Nov 15 23:49 boot.img
    -rw-r--r-- 1 bruce bruce 12M Nov 15 23:58 boot.img.repack
    bruce@ubuntu:~/orangepi/unpackbootimg$ md5sum boot.img boot.img.repack 
    9dcb37696cf08001a8a7c8127906d23e  boot.img
    9dcb37696cf08001a8a7c8127906d23e  boot.img.repack

对比MD5发现重新打包是完全一致的，说明这种解包/修改/再打包解决方法可行。但是有一个要求：我们修改内核重新编译后应该只有内核映像文件，而不会有内核模块等其它文件，否则就需要更新system.img文件了，因为modules是存放在system/vendor/modules目录下的。因此裁剪内核时，必须使用built-in的方式构建功能模块。

#### 3. Android系统打包

通过对pack打包脚本的初步分析，文件位于lichee/tools/pack/pack,可以得出以下结论：

- 打包时会拷贝lichee/平台下的工程配置文件sys_config.fex到目录out中进行打包构建
- 打包时会拷贝Android目录下的boot.img, recovery.img,system.img文件进行打包构建

因此我们对工程配置文件sys_config.fex的修改，和对boot.img的更新之后，重新执行pack命令应该能够生成更新之后的Android系统镜像文件，从而避免耗时的完整标准Android编译过程。

1. 修改linux内核配置：make ARCH=arm menuconfig; cp .config arch/arm/configs/sun8iw7p1_android_defconfig

    - 添加I2C chardev支持
    - 添加SPI chardev支持
    
    重新编译内核镜像,参考手册上的lichee编译流程: ./build.sh lunch，选择2
    
2. 将编译出来的lichee/linux-3.4/output/bImage文件与之前Android boot.img解包出来的其他文件重新打包成boot.img文件
3. 将重新打包的boot.img文件替换掉Android目录out/target/product/dolphin-fvd-p1/下的boot.img文件
4. 修改lichee/tools/pack/pack/chips/sun8iw7p1/configs/dolpin-p1/sys_config.fex文件
5. 在Android目录下按照前面的构建流程配置好环境变量,然后直接打包pack就可以了

        $ source ./build/envsetup.sh
        $ lunch dolphin_fvd_p1-eng  #选择方案号
        $ pack
6. 重新烧录新生成的img镜像系统到SD卡中


    
