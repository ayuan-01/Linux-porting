# 正点原子官方镜像系统烧写实验

## windows下使用OTG烧写系统

- 在windows下使用NXP提供的mfgtoll来向开发板烧写系统，需要用线将开发板的USB_OTG接口连接到电脑上
- OTG（**On-The-Go**）接口是一种增强型的USB接口，允许设备在没有主机（如PC）参与的情况下实现主从角色的动态切换。换句话说，支持OTG的设备可以在主设备（Host）和从设备（Device）之间切换，取决于连接的另一端设备的角色。OTG的关键特点包括：
  1. **主从角色切换**：
     - **Host**：当设备作为主机时，它可以控制连接到它的其他USB设备（如鼠标、键盘、U盘等）。
     - **Device**：当设备作为从设备时，它可以被另一个主机设备（如PC）控制。
  2. **动态切换**：
     - OTG设备可以根据连接的情况自动切换角色。例如，一部支持OTG的智能手机可以作为USB驱动器连接到PC上（此时作为从设备），也可以作为主机连接并控制USB键盘或U盘。
  3. **OTG电缆**：
     - OTG设备通常使用特殊的OTG线缆连接，OTG线缆的一端为标准的USB接口，另一端为微型或迷你USB接口。
- mfgtool先向板子上的DDR下载一个linux系统。然后用这个Linux系统完成后续的烧写工作。
- 切记！使用OTG烧写的时候要先把SD卡拔出来，等USB OTG与电脑连接成功以后就可以再将SD卡插进去了。
- 烧写系统都是烧写到NAND或者eMMC中
- eMMC：10100110
- SD：10000010
- 连接OTG后先下载的系统是这里的E:\mfgtool\Profiles\Linux\OS Firmware\firmware
- 最终烧写进去的系统是这里的E:\mfgtool\Profiles\Linux\OS Firmware\files

## 使用Linux下脚本烧写系统

- 首先向SD烧写一个系统，然后使用SD卡启动，启动以后再Linux中执行烧写到emmc或nand中
- 先将系统烧写到Sd卡中，烧写完成后SD卡会出现3个分区，其中u-boot分区看不到。然后将files复制到SD卡系统目录的rootfs/home/root中
- 使用Linux图形界面拷贝不确定是不是传输完成，可以在命令行中执行**sync**命令等待传输完成 

1. 将正点原子mftool中的files/文件移动到Linux下

2. 使用files目录下的脚本文件./imx6mksdboot.sh烧写系统到SD卡(首先脚本文件权限需要修改`chmod 777 imx6mksdboot.sh`,所有sh文件的权限都修改一下)

3. ```makefile
   sudo ./imx6mksdboot.sh -device /dev/sdb -flash emmc -ddrsize 512
   ```

4. 烧写到SD卡中会出现3个分区`boot`,`rootfs`,`uboot`

5. 将`files`复制到SD卡系统目录的`rootfs/home/root`中

6. 开发板连接串口，拨码开关调整到从SD卡启动。

7. 现在开发板启动的系统是SD里面的，随后使用串口连接的CRT的命令行换到上面保存在SD卡系统下的files文件路径

8. 利用`./imx6mkemmcboot.sh`将系统烧写到emmc中

9. 查找emmc的挂载点`/dev/mmcblk1`

   ```makefile
   fdisk -l
   ```

10. ```
    ./imx6mkemmcboot.sh -device /dev/mmcblk1 -ddrsize 512
    ```

11. 烧写完成后将拨码开关调节到emmc启动。

# 正点原子官方u-boot编译

## 什么是u-boot

- u-boot是一个裸机程序，比较复杂
- uboot就是一个bootloader，作用就是启动Linux或其它系统。uboot最主要的工作就是初始化DDR。因为Linux的运行是运行在DDR里面的。Linux的镜像如果不裁剪的话大概有四五M字节，内部的RAM是放不下的，所以要放到DDR里面去运行。
- 对于6U来讲，DDR的初始化是内部的bootroom完成的
- 但是对于其他的大部分的cotex-A系列的芯片的bootroom是不会初始化DDR的。这时就必须启动uboot在uboot中完成DDR的初始化。
- 初始化完成之后，Linux的*系统镜像（zImage或uImage）加设备树（.dtb)*是要存储在某个地方，一般存放在SD卡，emmc，nand，spi flash等外置的存储区域。
- 这里牵扯到一个问题，需要将Linux镜像从外置的介质拷贝到DDR中。再去启动。这个驱动是uboot来做的。提供外置flash的读写工作
- uboot的主要目的就是为系统的启动做准备。uboot工作完成之后，自己挂掉，将CPU的使用权交给Linux系统。
- uboot不仅仅能启动Linux。比如VxWorks。
- Linux不仅仅能通过uboot启动，可能还有其他的bootloader
- uboot是个通用的bootloader，它支持多种架构。
- uboot是一个程序，需要编译生成uboot.bin文件，用这个bin文件启动开发板。

## Uboot获得

- uboot官网
- soc厂商会从uboot官网下载某一个版本的uboot，然后在这个uboot加入相应的soc以及驱动。这就是soc厂商定制版的uboot。比如NXP官方的i.mx6ull EVK板子。
- 做开发板的厂商，开发板会参考SOC厂商的板子

# 正点原子官方uboot编译

- /home/syw/linux/IMX6ULL/uboot/alientek_uboot

  将正点原子提供的uboot压缩包放在这里面

- 解压文件

- 编译uboot的时候需要先配置

  - 清理

    ```makefile
    make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- distclean
    ```

  - 配置uboot---正点原子默认配置文件

    ```makefile
    make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- mx6ull_14x14_ddr512_emmc_defconfig
    ```

  - 配置完成后会生成.config文件，将各配置项保存在这个文件中。

  - 编译(V=1表示编译的时候打印出详细的编译过程)

    ```makefile
    make V=1 ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf-
    ```

  - 编译完成之后会生成一个u-boot.bin文件。必须向u-boot.bin添加头部信息。对于mx6u板子u-boot编译最后会通过./tools/mkimage软件会添加头部信息，生成u-boot.imx文件
  
  - 将u-boot.imx文件拷贝到win

  - 将该文件拷贝到mftool中的firmware文件夹中，重命名并替换掉原本的uboot文件u-boot-imx6ull-14x14-emmc.imx。firmware是先下载到DDR中的系统，最终烧录到emmc中的系统是在files文件夹中，所以需要将刚才的u-boot.imx也重命名u-boot-imx6ull-14x14-ddr512-emmc.imx拷贝到files文件夹中。
  
  - 使用mftool工具烧写
  
  - CRT软件中在板子系统启动时在第一行会显示u-boot的编译时间，以此判断当前的u-boot是不是我们最新编译的。
  
  - 为了方便编译，可以将以上三行代码写成shell脚本。如果配置过uboot，那么一定要注意shell脚本会清除整个工程。那么配置的文件也会被删除。配置项也会被删除掉
  
    这里说的是通过图形界面配置了uboot的话要注意shell脚本会清除配置项。如果是通过修改uboot源码来配置的话就不用担心。
  
  - 为了方便开发，建议在uboot顶层makefile里面设置好ARCH和CROSS_COMPILE两个变量的值。

# uboot命令使用

在调试过程中都使用SD卡烧写程序，因为我们只需要修改uboot，如果用mftool工具每次都要烧写全部系统太慢了。

- 将裸机实验中使用的imxdown软件复制到`/linux/IMX6ULL/uboot/alientek_uboot/`用来烧录u-boot.imx

## help命令

- 直接输入help（或者？）会打印出当前的uboot支持的所有的命令
- 查看某一个命令的帮助信息，`? <命令名>`

## 信息查询

- bdinfo---打印板子的信息结构体
- printenv---查看当前板子的环境变量。

## setenv

- 设置环境变量`setenv <环境变量> <值>`
- 自定义环境变量，格式同上
- 删除环境变量，不设置值即可`setenv <要删除的环境变量>`
- 如果值是字符串，需要用单引号
- 设置完之后需要`saveenv`

## saveenv

- 保存环境变量.

## 内存操作相关命令

### 1.md

用于显示内存值

```
md[.b .w .l] addr [# of objects]
```

命令中`.b`表示以一个字节来显示内存值，`.w`代表一个字，`.l`代表四个字节。addr为要显示内存值的起始地址，最后一个表示要显示的数据长度，**注意：这里的数据长度是根据前面设置的.b等来的，表示n个字节，n个字，n个双字**。

**另外注意！！！uboot命令中的数字都是十六进制的。**

### 2.nm

用于修改指定地址的内存值

```
nm[.b .w .l] addr
nm.l 80000000
=>80000000: fffaffff ? 12345678
=>80000000: 12345678 ? q
```

修改完之后输入**q**退出

### 3.mm

自增

```
mm[.b .w .l] addr
mm.l 80000000
=>80000000: 12345678 ? 05050505
=>80000004: affafffa ? q
```

### 4.mw

使用一个指定的数据来填充一段内存

```
mm[.b .w .l] addr value [count]
value要填充的值，count要填充的数量
```

### 5.cp

用于将DRAM中的数据从一段内存拷贝到另一段内存中。或者把nor flash中的数据拷贝到DRAM中。

```
cp[.b .w .l] source target count
源地址 目标地址 数据长度
```

### 6. cmp

用来比较两段内存的数据是否相等

```
cmp[.b...] addr1 addr2 count
```

## 网络命令

### 1.ping

- 网线插到ENEP2上保证开发板和电脑处于同一个网段内

- 具体环境设置见正点原子文档

- 设置开发板网络

  ```
  setenv ipaddr 192.168.10.50
  setenv ethaddr 00:04:9f:04:d2:35
  setenv gatewayip 192.168.10.1
  setenv netmask 255.255.255.0
  setenv serverip 192.168.10.100
  saveenv
  ```

### 2.dhcp

自动获取ip地址

通过DHCP获取到的IP地址仅有本次有效，不会修改环境变量中ipaddr的值。下次重启以后依旧用的ipaddr里面的IP地址

### 3.nfs

网络文件系统，通过nfs可以在计算机之间通过网络来分享资源，比如我们可以将Linux的镜像和设备树文件放到Ubuntu中，然后在uboot中使用nfs命令将其下载到开发板的DRAM中。

目的一般是为了调试代码。

Linux服务器创建一个nfs文件夹，用于存放将要使用nfs传输到开发板的文件。

```
nfs 80800000 192.168.1.66:/home/syw/linux/nfs/zImage
```

### 4.tftp

作用和nfs命令一样，都是你通过网络下载东西到DRAM中，只是tftp命令使用的是TFTP协议，Ubuntu主机作为TFTP服务器，因此需要在Ubuntu上搭建TFTP服务器

在ping命令的时候设置了`setenv serverip 192.168.10.100`

在使用tftp命令时不需要设置服务器地址

```
tftp 80800000 zImage
```

```
安装tftp-hpa,tftpd-hpa
sudo apt-get install tftp-hpa tftpd-hpa
创建tftpboot目录
修改权限 chmod 777
sudo touch /etc/xinetd.d/tftp
vi tftp
输入：server tftp
{
    socket_type     = dgram
    protocol        = udp
    wait            = yes
    user            = root
    server          = /usr/sbin/in.tftpd
    server_args     = -s /home/syw/linux/tftpboot
    disable         = no
    per_source      = 11
    cps             = 100 2
    flags           =IPV4
}
sudo service tftpd-hpa start

sudo vi /etc/default/tftpd-hpa

输入：  1 # /etc/default/tftpd-hpa
  2 
  3 TFTP_USERNAME="tftp"
  4 TFTP_DIRECTORY="/home/syw/linux/tftpboot"
  5 TFTP_ADDRESS=":69"
  6 TFTP_OPTIONS="-l -c -s"

syw@syw-virtual-machine:/etc/xinetd.d$ cd /home/syw/linux/tftpboot/
syw@syw-virtual-machine:~/linux/tftpboot$ ls
syw@syw-virtual-machine:~/linux/tftpboot$ cp ../nfs/zImage ./
syw@syw-virtual-machine:~/linux/tftpboot$ ls
zImage
syw@syw-virtual-machine:~/linux/tftpboot$ sudo service tftpd-hpa restart
在使用tftp命令传输之前，需要给要传输的文件一个大的权限
sudo chmod 777 zImage
```

## MMC

1. mmc info

   当前使用设备的信息Sd卡或者eMMC

   ```
   mmc info
   Device: FSL_SDHC
   Manufacturer ID: 9f
   OEM: 5449
   Name: SD32G 
   Tran Speed: 50000000
   Rd Block Len: 512
   SD version 3.0
   High Capacity: Yes
   Capacity: 29.1 GiB
   Bus Width: 4-bit
   Erase Group Size: 512 Bytes
   ```

2. mmc rescan----mmc list

   mmc rescan用来扫描当前板子的mmc设备

   mmc list用来显示出扫描出的mmc设备

3. mmc dev <n>

   设置当前的mmc设备

   mmc dev 1 0---设备1的分区0

4. mmc part

   当前的mmc设备分区

5. mmc read

   mmc dev 1 0-----------------------------切换mmc设备-块

   mmc read 80800000 600 10-------8080000是读取到DRAM上的地址。读取第600个块，读取10个块。一个块相当于一个扇区即512字节

6. mmc write命令

   - version当前u-boot版本

   - 将编译好的u-boot通过tftp下载到80800000

     tftp 80800000 u-boot.imx

     `Bytes transferred = 363520 (58c00 hex)`

   - 上面的命令可以看出，u-boot.imx的大小为363520Byte(2C6)

     ```
     => mmc write 80800000 2 2C6
     //写入第2个块，写710个块。
     MMC write: dev # 0, block # 2, count 710 ... 710 blocks written: OK
     ```

   - 重启---reset，查看u-boot编译时间。

7. mmc erase

   擦除mmc设备的指定块

   `mmc erase <起始blk> <cnt>`

   尽量不要擦除mmc

## FAT格式文件系统操作命令

对于6u来说。SD/eMMC分为三个分区

- 第一个存放u-boot
- 第二个存放zImage，.dtb。FAT格式
- 第三个存放根文件系统。EXT4格式。

### fatinfo

查询指定mmc设置指定分区的文件系统信息。

```
=> fatinfo mmc 0:1    //设备0的分区1
Interface:  MMC
  Device 0: Vendor: Man 00009f Snr 70017b01 Rev: 7.5 Prod: SD32Ga
            Type: Removable Hard Disk
            Capacity: 29819.0 MB = 29.1 GB (61069312 x 512)
Filesystem: FAT32 "boot   
```

### fatls

用于查询FAT格式设备的目录和文件信息

```
=> fatls mmc  1:1
  6785480   zimage 
    39459   imx6ull-14x14-emmc-4.3-480x272-c.dtb 
    39459   imx6ull-14x14-emmc-4.3-800x480-c.dtb 
    39459   imx6ull-14x14-emmc-7-800x480-c.dtb 
    39459   imx6ull-14x14-emmc-7-1024x600-c.dtb 
    39459   imx6ull-14x14-emmc-10.1-1280x800-c.dtb 
    40295   imx6ull-14x14-emmc-hdmi.dtb 
    40203   imx6ull-14x14-emmc-vga.dtb 

8 file(s), 0 dir(s)
```

### fstype

查看分区文件格式

```
fstype mmc 0:1
fat
```

### fatload

用于将指定的文件读取到DDR中

u-boot启动就是使用该命令将系统读取到DDR中，这个命令很重要。

```
fatload mmc 1:1 80800000 zImage		//将mmc设备分区1中的zImage文件读取到DDR的80800000地址中 
```

### fatwrite

将DDR中的数据写入到mmc设备中。

```
fatwrite mmc 1:1 80800000 zImage 0x5c2720(zImage大小)
```

## EXT文件

见教程

## BOOT操作命令

### bootz

`bootz addr initrd[:size] fdt`

第一项表示镜像文件地址；第二项一般不用使用`-`代替，第三项为设备树地址。

要启动linux，必须将zImage，dtb放到DDR。放上去之后就通过bootz来启动。

- 可以通过网络`tftp`将zImage传输到80800000，将`dtb`传输到83000000。

- 可以通过mmc的fat命令将zImage和dtb文件从MMC设备读取到DRAM。

1. 通过网络启动

​	将镜像文件和设备树文件放到tftpboot文件夹中

```
tftp 80800000 zImage
tftp 83000000 imx6ull-14x14-emmc-7-1024x600-c.dtb
bootz 80800000 - 83000000
```

2. 通过MMC启动

   ```
   fatls mmc 1:1		//查看分区文件，zImage，dtb
   fatload mmc 1:1 80800000 zImage
   fatload mmc 1:1 830000000 imx6ull-14x14-emmc-7-1024x600-c.dtb
   bootz 80800000 - 83000000
   ```

3. boot命令

   boot命令会执行`bootcmd`变量中所定义的所有的命令

   ```
   setenv bootcmd 'tftp 80800000 zImage;tftp 83000000 imx6ull-14x14-emmc-7-1024x600-c.dtb;bootz 80800000 - 83000000;'
   saveenv
   boot
   ```

## 其他命令

### reset

复位

### go

跳转到指定的地址执行命令，可以使用这条命令来执行裸机程序。

下载裸机代码的时候要下载没有头部信息的bin文件

tftp 87800000 printf.bin

go 87800000

### run

用于运行环境变量中定义的命令，可以在环境变量中自定义一些命令，使用run来运行

```
setenv mybootemmc 'fatls mmc 1:1;fatload mmc 1:1 80800000 zImage;fatload mmc 1:1 830000000 imx6ull-14x14-emmc-7-1024x600-c.dtb;bootz 80800000 - 83000000;'
saveenv
run mybootemmc
```

### mtest

测试内存

mtest <start> <end>

# u-boot源码目录分析

因为uboot会使用到一些经过编译才会生成的文件，因此我们在分析uboot的时候，需要先编译一下uboot。

- arch\arm\cpu\u-boot.lds就是整个uboot的连接脚本
- board\freescale\mx6ullevk重点
- comfigs目录是uboot的默认配置文件目录。此目录下都是以_defconfig结尾的。这些配置文件对应不同的板子。
- 移植uboot的时候要重点关注board\freescale\，\configs，主要是_defconfig
- 当我们执行`make xxx_defconfig`以后就会生成.config文件，此文件保存了详细的uboot配置信息。
- 顶层README很重要，建议阅读，介绍uboot。
- u-boot.*文件都是编译生成的文件。
- u-boot这个就是编译出来带elf信息的uboot可执行文件。
- .map映像文件。

# uboot顶层makefile分析

DDR的起始地址为0x80000000，0x87800000是u-boot的起始地址。后来uboot从低地址区域移到高地址区域。所以将代码下载到0x87800000这个地址跟uboot并不冲突

芯片上电后运行uboot---初始化DDR等外设---将linux内核从flash拷贝到DDR中---启动Linux内核

u-boot是一个裸机代码，可以看作是一个综合的裸机历程，有C源码，Makefile，.lds连接脚本文件。

## u-boot的文件目录

下面是u-boot的文件目录

```
u-boot/
├── api/                 # U-Boot提供的API接口定义
├── arch/                # 与CPU架构相关的代码，如arm、avr32、mips、x86等
│   └── <arch>/          # 特定架构的源代码目录
│       ├── include/     # 架构相关的头文件
│       └── cpu/         # 架构特定的CPU代码
├── board/               # 支持的各个开发板相关的文件
│   └── <vendor>/<board>/ # 特定供应商的开发板目录
│       ├── include/     # 开发板相关的头文件
│       └── Kconfig      # 开发板的配置文件
├── cmd/                 # U-Boot命令实现代码
├── common/              # 通用的代码，不依赖于特定硬件平台
├── config/              # 默认的配置文件
├── disk/                # 磁盘分区和I/O代码
├── doc/                 # 文档，包括设计理念和用户指南
├── drivers/             # 设备驱动代码
│   ├── clk/             # 时钟驱动
│   ├── gpio/            # GPIO驱动
│   ├── i2c/             # I2C驱动
│   ├── mmc/             # SD卡/MMC驱动
│   └── net/             # 网络驱动
├── dts/                 # 设备树源文件
├── examples/            # 示例代码
├── fs/                  # 文件系统代码
├── include/             # 公共头文件
├── lib/                 # 库文件
├── Licenses/            # 许可证文件
├── net/                 # 网络协议栈代码
├── post/                # 上电自检代码
├── scripts/             # 编译脚本和工具
├── test/                # 测试代码
├── tools/               # 工具代码，如二进制格式转换工具
└── u-boot.lds           # U-Boot链接器脚本

```

arch/arm/cpu/中有u-boot.lds文件这个就是arm芯片所使用的连接脚本文件。uboot 根目录下的 u-boot.lds 链接脚本就是来源于 arch/arm/cpu/u-boot.lds文件。 

arch/arm/cpu/armv7是我们所使用的架构的文件夹。

configs 文件夹，可配置启动方式之类的，xxx_defconfig文件。使用“make xxx_defconfig”命令即可配置uboot，比如：make mx6ull_14x14_ddr512_emmc_defconfig

在编译 uboot 之前一定要使用 defconfig 来配置 uboot！ 

U-Boot 源码中的 `scripts` 文件夹主要包含以下功能：

- 提供配置系统支持 (`Kconfig` 工具)。
- 自动化处理构建过程中的各种任务。
- 生成工具链和构建环境相关的文件。
- 支持生成和处理可执行镜像、内存文件系统等。

## 顶层makefile分析

见文档，参考博客https://www.cnblogs.com/jliuxin/p/14129260.html

# uboot启动流程

U-Boot 的启动流程是嵌入式系统中常见的启动引导流程，主要负责初始化硬件、加载操作系统内核等。以下是 U-Boot 的典型启动流程的各个阶段：

### 1. **硬件复位**（Hardware Reset）

当嵌入式设备加电或重置时，处理器会从预定义的复位地址开始执行。在这个阶段，U-Boot 会被加载到内存中，通常从 ROM、NOR Flash 或 NAND Flash 等非易失性存储器中执行。

### 2. **第一阶段（Stage 1）：硬件初始化**

U-Boot 的第一阶段通常称为 **SPL**（Secondary Program Loader），其主要职责是初始化最基本的硬件（比如 DRAM），为第二阶段 U-Boot 提供执行环境。SPL 通常是一个小型引导程序，以下是主要步骤：

- **CPU 初始化**：设置处理器模式，初始化必要的寄存器。
- **堆栈设置**：为后续执行过程设置堆栈空间。
- **时钟、DDR 初始化**：通常需要对板级的时钟和 DDR（内存）进行初始化，为引导提供运行所需的内存环境。
- **加载第二阶段的 U-Boot**：将主 U-Boot 从 Flash 或其他存储介质加载到内存中（DRAM）。

SPL 加载和执行完整的 U-Boot 镜像，通常这个镜像更大且功能更完整。

### 3. **第二阶段（Stage 2）：U-Boot 主程序**

U-Boot 主程序负责更多复杂的硬件初始化和操作系统内核加载。流程如下：

#### （1）板级初始化（Board Initialization）

这一步会执行各类板级相关初始化，比如：

- 设置串口（UART）以便进行调试输出。
- 设置 GPIO（通用输入输出端口）。
- 可能涉及其他外设的初始化，比如网卡、USB 等。

#### （2）内存检测（Memory Detection）

U-Boot 还可能会对系统的 DRAM 或其他内存进行检测，以确保内存配置正确。它会为内核和设备驱动分配合适的内存。

#### （3）设备树（Device Tree）加载

如果使用设备树（Device Tree）来描述硬件，U-Boot 会将它加载到内存，并传递给操作系统。设备树是内核用来理解硬件配置的一个数据结构。

#### （4）环境变量加载

U-Boot 有一套环境变量（如 `bootcmd`, `bootargs`），这些变量可以存储引导配置、内核启动参数等。U-Boot 会从 Flash 或其他非易失性存储器中加载这些环境变量。

#### （5）网络、存储设备初始化

U-Boot 会根据需要初始化存储设备（如 NAND、MMC、SD 卡、eMMC）或网络设备（如以太网、TFTP）。这一步为 U-Boot 提供访问存储介质或网络的能力，加载操作系统。

### 4. **启动命令执行**

U-Boot 会根据环境变量中的 `bootcmd` 指定的命令执行。常见的引导命令包括：

- **加载内核**：从存储介质（如 NAND Flash、MMC）或通过网络（TFTP）加载 Linux 内核镜像。
- **加载设备树和内存盘（initrd）**：如果系统需要，U-Boot 也会加载设备树文件和 initrd 镜像。
- **启动内核**：最终，U-Boot 通过 `bootm` 或 `bootz` 命令启动内核，并将控制权移交给操作系统。

### 5. **操作系统引导**

U-Boot 完成上述所有操作后，它会通过 `bootm` 或 `bootz` 将内核镜像加载到内存中，并跳转到内核的入口点，启动操作系统（如 Linux）。从这一步开始，系统将由内核接管，U-Boot 的引导过程结束。

------

### U-Boot 启动流程总结图解：

1. **SPL 阶段**
   - CPU 初始化
   - 内存初始化
   - 加载完整 U-Boot
2. **U-Boot 阶段**
   - 硬件、设备初始化
   - 环境变量加载
   - 加载内核、设备树、initrd
3. **启动操作系统**

在 i.MX6ULL EVK 开发板上，U-Boot 的启动流程与一般的 U-Boot 启动类似，但由于其特定的硬件平台，初始化步骤会有一些针对 i.MX6ULL 的特定操作。以下是 i.MX6ULL EVK 开发板的 U-Boot 启动流程的详细说明：

### 1. **ROM 引导阶段（BootROM）**

i.MX6ULL 处理器的启动流程从 ROM 代码开始。ROM 代码在加电或复位后自动执行，它的职责是决定从哪种介质启动。具体的启动介质可以通过板上的启动引脚（BOOT pins）配置，这些引脚通常选择以下几种启动模式之一：

- NAND Flash
- SD 卡
- eMMC
- 串行下载（用于开发和调试）

ROM 引导代码会从指定的存储介质中加载并执行 U-Boot 的 SPL 或 U-Boot 镜像。

### 2. **SPL 阶段（Secondary Program Loader）**

在 i.MX6ULL 平台上，SPL 是一个精简版的引导程序，它通常位于 Flash 存储的头部。其职责包括：

- **初始化 DDR**：在 i.MX6ULL 上，SPL 的主要工作之一是初始化 DDR 内存控制器，因为 U-Boot 主程序需要依赖 DDR 内存。
- **加载主 U-Boot**：SPL 会将主 U-Boot 镜像从存储介质（如 NAND 或 SD 卡）加载到 DDR 中，然后将控制权转交给主 U-Boot。

在 i.MX6ULL 上，SPL 通常位于启动存储设备的前几块（blocks）中。ROM 代码加载 SPL 后执行它，SPL 负责初始化 DDR，并加载 U-Boot 主程序。

### 3. **U-Boot 主程序（U-Boot）**

SPL 成功加载并初始化 DDR 后，U-Boot 主程序（stage 2）开始执行。在 i.MX6ULL EVK 上，U-Boot 主程序负责以下几个关键步骤：

#### （1）CPU 和内存初始化

虽然大部分 CPU 初始化由 SPL 处理，但在 U-Boot 主程序中，仍会进行一些额外的初始化步骤，比如缓存设置等。同时，U-Boot 会检查和配置系统内存（DDR）。

#### （2）串口初始化

U-Boot 初始化串口（通常是 UART），并输出调试信息，这使得开发者可以通过串口终端监控启动过程。

#### （3）板级初始化

i.MX6ULL EVK 开发板特有的外设（如 GPIO、LED、网卡、I2C、SPI、MMC/SD 控制器等）会被初始化。通常根据开发板的硬件配置文件（board-specific code）来进行。

#### （4）环境变量加载

U-Boot 会从 Flash 或其他存储设备中读取环境变量（如果存在）。这些变量定义了设备的启动配置，例如：

- `bootcmd`：默认的启动命令。
- `bootargs`：传递给内核的命令行参数。
- `bootdelay`：在启动前等待的时间，允许用户通过串口输入命令。

#### （5）存储和网络初始化

U-Boot 初始化与启动设备相关的存储接口，如 SD 卡、NAND Flash 或 eMMC。如果需要通过网络启动，也会初始化以太网接口。

### 4. **启动命令执行**

根据环境变量 `bootcmd` 的定义，U-Boot 将执行特定的启动命令。在 i.MX6ULL EVK 开发板上，常见的引导方法包括：

- **从 SD 卡或 eMMC 启动**：从存储设备上加载 Linux 内核（zImage 或 uImage）、设备树（DTB）和根文件系统（通常为 ext4 或其他文件系统类型）。
- **从网络启动**：通过 TFTP 加载 Linux 内核和设备树。

典型的 `bootcmd` 可能会类似于：

```
bash


复制代码
bootcmd=mmc dev 0; ext2load mmc 0:1 ${loadaddr} /boot/zImage; ext2load mmc 0:1 ${fdtaddr} /boot/imx6ull-14x14-evk.dtb; bootz ${loadaddr} - ${fdtaddr}
```

这条命令将：

1. 将 SD 卡作为存储设备进行操作。
2. 加载内核镜像到内存地址 `${loadaddr}`。
3. 加载设备树文件到内存地址 `${fdtaddr}`。
4. 使用 `bootz` 启动内核。

### 5. **启动操作系统（内核）**

U-Boot 加载内核镜像（如 `zImage` 或 `uImage`），并将设备树传递给内核，然后通过 `bootm` 或 `bootz` 命令启动内核。在启动内核的过程中，U-Boot 将控制权交给 Linux 内核。此时，U-Boot 的任务完成，系统的控制权由内核接管。

------

### i.MX6ULL EVK 开发板 U-Boot 启动流程总结：

1. **ROM Boot**：根据引脚配置，从 NAND、SD 卡等加载 SPL。

2. **SPL**：初始化 DDR，加载 U-Boot 主程序。

3. U-Boot 主程序

   ：

   - CPU 和内存初始化。
   - 外设初始化（串口、网卡、SD 卡等）。
   - 读取环境变量并执行 `bootcmd`。

4. **启动 Linux 内核**：加载内核、设备树，启动操作系统。

这个流程可以根据具体开发需求调整，比如修改启动设备、网络引导配置等。如果你需要针对 i.MX6ULL EVK 开发板的 U-Boot 配置进行更深层次的修改，可以深入分析 `board/freescale/mx6ull_14x14_evk` 目录中的板级初始化代码。

# uboot移植实验

## NXP官方uboot编译测试

- 将NXP的官方uboot复制到服务器，并重命名

- ~/linux/IMX6ULL/uboot/uboot-imx-rel_imx_4.1.15_2.1.0_ga_alientek$

- nxp的配置文件为mx6ull_14x14_evk_emmc_defconfig

- 写**mx6ull_14x14_evk_emmc.sh**脚本文件编译uboot。

  ```shell
  #!/bin/bash
  make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- distclean
  make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- mx6ull_14x14_evk_emmc_defconfig
  make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- V=1
  ```

- 将*ARCH*和*CROSS_COMPILE*的值写入Makefile文件中

- 编译，烧录到SD卡中，烧录之前格式化SD卡，清除环境变量

## 在uboot中添加ALPHA开发板

流程见教程文档

- 复制NXP的xxx_defconfig文件重命名mx6ull_alientek_emmc_defconfig作为板子的配置文件

  ```shell
  CONFIG_SYS_EXTRA_OPTIONS="IMX_CONFIG=board/freescale/mx6ull_alientek_emmc/imximage.cfg,MX6ULL_EVK_EMMC_REWORK" # 更改
  CONFIG_ARM=y
  CONFIG_ARCH_MX6=y
  CONFIG_TARGET_MX6ULL_ALIENTEK_EMMC=y # 更改
  CONFIG_CMD_GPIO=y
  ```

- 添加板子对应的头文件

  不同的板子，有一些需要配置的信息。一般是在一个头文件中配置

  对于NXP**/include/configs/mx6ullevk.h**

  复制为自己的

- 添加开发板对应的板级文件夹

  /board/freescale/mx6ullevk

  拷贝为/board/freescale/mx6ull_alientek_emmc

  修改板级文件夹中的文件，详情见文档。

- 修改uboot图形界面配置文件

  arch/arm/cpu/armv7/mx6/Kconfig

## LCD驱动修改

**一般uboot中修改驱动基本都是在mx6ull_alientek_emmc.c和mx6ull_alientek_emmc.h中修改的**

- 修改IO

  c文件728行

  屏蔽758行复位

  背光

- LCD参数

  780行
  
  ```c
  struct display_info_t const displays[] = {{
  	.bus = MX6UL_LCDIF1_BASE_ADDR,
  	.addr = 0,
  	.pixfmt = 24,
  	.detect = NULL,
  	.enable	= do_enable_parallel_lcd,
  	.mode	= {
  		.name			= "TFT43AB",
  		.xres           = 480,
  		.yres           = 272,
  		.pixclock       = 108695,
  		.left_margin    = 8,
  		.right_margin   = 4,
  		.upper_margin   = 2,
  		.lower_margin   = 4,
  		.hsync_len      = 41,
  		.vsync_len      = 10,
  		.sync           = 0,
  		.vmode          = FB_VMODE_NONINTERLACED
  } } };
  ```
  
  `pixclock`（像素时钟）是一个非常重要的参数。它定义了显示器每秒钟可以处理的像素数量，通常以皮秒（picoseconds，1皮秒 = 10^-12秒）为单位。
  
  panel环境变量表示屏幕ID，在mx6ull_alientek_emmc.h中定义

## 网络驱动修改

修改PHY

见教程

## uboot启动Linux内核

mmc启动

```sh
mmc dev 1 //切换到 EMMC
fatload mmc 1:1 0x80800000 zImage //读取 zImage 到 0x80800000 处
fatload mmc 1:1 0x83000000 imx6ull-14x14-emmc-7-1024x600-c.dtb //读取设备树到 0x83000000 处
bootz 0x80800000 - 0x83000000 //启动 Linux
```

配置两个环境变量

```sh
setenv bootargs 'console=ttymxc0,115200 root=/dev/mmcblk1p2 rootwait rw'
setenv bootcmd 'mmc dev 1; fatload mmc 1:1 80800000 zImage; fatload mmc 1:1 83000000 imx6ull-alientek-emmc.dtb; bootz 80800000 - 83000000;'
saveenv
```

网络启动

```sh
tftp 80800000 zImage
tftp 83000000 imx6ull-14x14-emmc-7-1024x600-c.dtb
bootz 80800000 - 83000000
```

## 板子.h文件分析

文档32.2.2节

主要是参数bootargs和bootcmd

## DDR初始化参数修改

编译u-boot可以得到

```sh
./tools/mkimage -n board/freescale/mx6ull_alientek_emmc/imximage.cfg.cfgtmp -T imximage -e 0x87800000 -d u-boot.bin u-boot.imx 
```

uboot使用/tools/mkimage工具，向u-boot.bin添加board/freescale/mx6ull_alientek_emmc/imximage.cfg.cfgtmp文件信息，从而得到u-boot.imx。

board/freescale/mx6ull_alientek_emmc/这个路径中默认只有imximage.cfg文件。此文件里面保存的是DCD数据。DDR初始化也在此文件里面。

我们要修改DDR初始化代码，就需要修改imximage.cfg文件。此文件默认拷贝的是NXP给EVK开发板写的，默认是给512MDDR3L写的。

将裸机实验中生成的.inc文件中的校验值修改到imximage.cfg文件中去

```
//Read DQS Gating calibration			
setmem /32	0x021b083c =	0x01400140	// MPDGCTRL0 PHY0
			
//Read calibration			
setmem /32	0x021b0848 =	0x40403236	// MPRDDLCTL PHY0
			
//Write calibration                     			
setmem /32	0x021b0850 =	0x40403834	// MPWRDLCTL PHY0

DATA 4 0x021B083C 0x01400140
DATA 4 0x021B0848 0x40403236
DATA 4 0x021B0850 0x40403834
```

#  uboot图形化界面配置

## 图形化配置操作

menuconfig 是一套图形化的配置工具，需要 ncurses 库支持。ncurses 库提供了一系列的 API 函数供调用者生成基于文本的图形界面，因此需要先在 Ubuntu 中安装 ncurses 库，命令如下：

```sh
sudo apt-get install build-essential
sudo apt-get install libncurses5-dev
```

在打开图形化配置界面之前，要先使用“make xxx_defconfig”对 uboot 进行一次默认配置， 只需要一次即可。如果使用“make clean”清理了工程的话就那就需要重新使用“make xxx_defconfig”再对 uboot 进行一次配置。进入 uboot 根目录，输入如下命令： 

```sh
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- mx6ull_alientek_emmc_defconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig
```

如果已经在 uboot 的顶层 Makefile 中定义了 ARCH 和 CROSS_COMPILE 的值，那么上述命令可以简化为： 

```sh
make mx6ull_alientek_emmc_defconfig
make menuconfig
```

选中子菜单以后按下“Y”键就会将相应的代码编译进 Uboot 中，菜单前面变为“< * >”。

按下“N”键不编译相应的代码

按下“M”键就会将相应的代码编译为模块，菜单前面变为“< M >”。**linux内核里面用到**

按下“?”键查看此菜单的帮助信息

按下“/”键打开搜索框

- 配置DNS域名解析命令

- 使用如下命令编译 uboot：

  ```sh
  make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j16
  ```

  千万不能使用如下命令：

  ```sh
  ./mx6ull_alientek_emmc.sh
  ```

  因为 mx6ull_alientek_emmc.sh 在编译之前会清理工程，会删除掉.config 文件！通过图形化界面配置所有配置项都会被删除，结果就是竹篮打水一场空。 

  编译完成以后烧写到 SD 卡中，重启开发板进入 uboot 命令模式，输入“?”查看是否有“dns”命令

  要先设置一下 dns 服务器的 IP 地址，也就是设置环境变量 dnsip 的值，命令如下：

  ```sh
  setenv dnsip 114.114.114.114
  saveenv
  ```

  设置好以后就可以使用 dns 命令查看百度官网的 IP 地址了，输入命令：

  ```
  dns www.baidu.com
  ```

  **开发板直连电脑以太网口无法访问外网的解决办法**

  1. 启用Internet连接共享

     - 在Windows上，打开 "网络和共享中心"。

     - 找到无线网络（Wi-Fi）连接，右键点击选择 "属性"。

     - 在“共享”选项卡中，勾选“允许其他网络用户通过此计算机的Internet连接进行连接”。

     - 在“家庭网络连接”中，选择与开发板连接的以太网接口。

     - 设置好之后弹出窗口，一定要注意！！！

       ```
       Internet 连接共享被启用时，你的 LAN 适配器将被设置成使用 IP 地址192.168.137.1。计算机可能会失去与网络上其他计算机的连接，如果这些计算机有静态IP 地址，你应该将它们设置成自动获取 IP 地址。你确定要启用Internet 连接共享吗?
       ```

       Windows以太网的网段被改为192.168.137.1，虚拟机和开发板必须跟Windows处于同一个网段内才可以访问互联网

  2. 如果网络共享无法让开发板访问外网，可以尝试通过虚拟机设置NAT网络，并在虚拟机中启用IP转发：

     ```bash
     sudo sysctl -w net.ipv4.ip_forward=1
     sudo iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE  # wlan0 是虚拟机中连接外网的接口
     ```

     ```bash
     # 使用下面命令来查看虚拟机中连接外网的网口是哪个
     syw@syw-virtual-machine:~/linux/IMX6ULL/uboot/uboot-imx-rel_imx_4.1.15_2.1.0_ga_alientek$ ip addr
     1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
         link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
         inet 127.0.0.1/8 scope host lo
            valid_lft forever preferred_lft forever
         inet6 ::1/128 scope host 
            valid_lft forever preferred_lft forever
     2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
         link/ether 00:0c:29:92:ea:4b brd ff:ff:ff:ff:ff:ff
         inet 192.168.137.100/24 brd 192.168.137.255 scope global dynamic ens33
            valid_lft 603953sec preferred_lft 603953sec
         inet6 fe80::c98b:dca1:b056:4a2a/64 scope link 
            valid_lft forever preferred_lft forever
     3: ens38: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
         link/ether 00:0c:29:92:ea:55 brd ff:ff:ff:ff:ff:ff
     syw@syw-virtual-machine:~/linux/IMX6ULL/uboot/uboot-imx-rel_imx_4.1.15_2.1.0_ga_alientek$ 
     
     ```

当我们配置好之后需要保存自己的配置文件

图形化配置界面中save

```
./configs/test_config
```

导入时使用图形化配置界面的load

## Kconfig语法

Kconfig文件是图形化界面的配置文件

### source

和 makefile 一样，Kconfig 也可以调用其他子目录中的 Kconfig 文件，调用方法如下：

```sh
source "xxx/Kconfig" //xxx 为具体的目录名，相对路径
```

### menu/endmenu

menu 用于生成菜单，endmenu 就是菜单结束标志，这两个一般是成对出现的。这两个之间是子菜单。

### choice/endchoice

choice/endchoice 代码段定义了一组可选择项，将多个类似的配置项组合在一起，供用户单选或者多选。示例代码 34.2.2.7 就是选择处理器架构，可以从 ARC、ARM、AVR32 等这些架构中选择，这里是单选。在 uboot 图形配置界面上选择“Architecture select”

### config 

顶层 Kconfig 中的“General setup”子菜单。可以看出，在 menu/endmenu 代码块中有大量的“config xxxx”的代码块，也就是 config 条目。config 条目就是“General setup”菜单的具体配置项，“config LOCALVERSION”对应着第一个配置项，“config LOCALVERSION_AUTO”对应着第二个配置项，以此类推 。

假如我们使能了 LOCALVERSION_AUTO这个功能，那么就会下.config 文件中生成`ONFIG_LOCALVERSION_AUTO=y`由此可知，.config 文件中的“CONFIG_xxx” (xxx 就是具体的配置项名字)就是 Kconfig 文件中 config 关键字后面的配置项名字加上“CONFIG_”前缀

string "Local version - append to U-Boot release"string 是变量类型，也就是“CONFIG_ LOCALVERSION”的变量类型。可以为：bool、tristate、string、hex 和 int，一共 5 种。最常用的是 bool、tristate 和 string 这三种.

bool 类型有两种值：y 和 n，当为 y 的时候表示使能这个配置项，当为 n 的时候就禁止这个配置项。tristate 类型有三种值：y、m 和 n，其中 y 和 n 的涵义与 bool 类型一样，m 表示将这个配置项编译为模块。string 为字符串类型，所以 LOCALVERSION 是个字符串变量，用来存储本地字符串，选中以后即可输入用户定义的本地版本号

第 18 行，help 表示帮助信息，告诉我们配置项的含义，当我们按下“h”或“?”弹出来的帮助界面就是 help 的内容。 

### depends on和select

```sh
config SYS_GENERIC_BOARD
	bool
	depends on HAVE_GENERIC_BOARD

config ARC
	bool "ARC architecture"
	select HAVE_PRIVATE_LIBGCC
	select HAVE_GENERIC_BOARD
	select SYS_GENERIC_BOARD
	select SUPPORT_OF_CONTROL

```

“depends on”说明“SYS_GENERIC_BOARD”项依赖于“HAVE_GENERIC_BOARD”, 也就是“HAVE_GENERIC_BOARD”被选中以后“SYS_GENERIC_BOARD”才能被选中。 

“select”表示方向依赖，当选中“ARC”以后，“HAVE_PRIVATE_LIBGCC”，“HAVE_GENERIC_BOARD”，“SYS_GENERIC_BOARD”和“SUPPORT_OF_CONTROL”这四个也会被选中。 

### menuconfig

menuconfig 和 menu 很类似，但是 menuconfig 是个带选项的菜单。前面有“[ ]”说明这个菜单是可选的，当选中这个菜单以后就可以进入到子选项中

```
示例代码 34.2.2.8 menuconfig 用法
1 menuconfig MODULES 
2 		bool "菜单"
3 	if MODULES
4 	...
5 	endif # MODULES
```

### comment

comment用于注释，也就是在图形化界面中显示一行注释 .



Kconfig文件最后一行需要空一行

# 正点原子官方Linux内核体验

```sh
  1 #!/bin/sh
  2 make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- distclean
  3 make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- imx_v7_defconfig
  4 make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig
  5 make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- all
```

编译内核需要安装**lzop**

```sh
sudo apt-get install lzop
```

编译好生成的zImage的路径**./arch/arm/boot/zImage**

```SH
cp ./arch/arm/boot/zImage /home/syw/linux/tftpboot/ -f	#-f的意思是替换
```

.dtb文件的路径**./arch/arm/boot/dts/*.dtb**

编译.dts文件为.dtb文件需要在Linux目录下进行./

```sh
make imx6ull-14x14-emmc-7-1024x600-c.dtb
```

随后使用网络启动

# Linux源码目录分析

见文档

主要是./arch/arm/boot/

# 顶层Makefile

xxx_def生成.cofig

make   所有built-in.o连接生成vmlinux

built-in.o是由各种.o文件连接生成的

zImage, uImage等各种镜像是由vmlinux生成的

# Linux内核启动流程简介

看启动流程肯定要分析第一行代码是从哪里开始执行的。也就是要先分析链接脚本arch/arm/kernel/vmlinux.lds

init进程 1

# Linux系统移植

- 将NXP官方的Linux内核拷贝到服务器

- 编译NXP官方开发板对应的Linux系统

- 设置顶层Makefile的ARCH和CROSS_COMPILE

- 配置编译Linux内核

  - 默认配置文件arch/arm/configs 
  - imx_v7_defconfig 和imx_v7_mfg_defconfig 都可作为 I.MX6ULL EVK 开发板所使用的默认配置文件。但是这里建议使用 imx_v7_mfg_defconfig 这个默认配置文件，首先此配置文件默认支持 I.MX6UL 这款芯片， 而且重要的一点就是此文件编译出来的 zImage 可以通过 NXP 官方提供的 MfgTool 工具烧写！！imx_v7_mfg_defconfig 中的“mfg”的意思就是 MfgTool。
  - 配置内核**make imx_v7_mfg_defconfig**
  - 编译 make
  - Linux 内核编译完成以后会在 arch/arm/boot 目录下生成 zImage 镜像文件，如果使用设备树的话还会在 arch/arm/boot/dts 目录下开发板对应的.dtb(设备树)文件，比如 imx6ull-14x14-evk.dtb就是 NXP 官方的 I.MX6ULL EVK 开发板对应的设备树文件。

- 内核启动测试

  - 在测试之前确保 uboot 中的环境变量 bootargs 内容如下：

    ```sh
    console=ttymxc0,115200 root=/dev/mmcblk1p2 rootwait rw
    ```

  - 将上一小节编译出来的 zImage 和 imx6ull-14x14-evk.dtb 复制到 Ubuntu 中的 tftp 目录下

    ```sh
    cp arch/arm/boot/zImage /home/syw/linux/tftpboot/ -f
    cp arch/arm/boot/dts/imx6ull-14x14-evk-emmc.dtb ../../../tftpboot/ -f
    ```

  - 网络启动

- 根文件系统出现错误

  - Linux 内核启动以后是需要根文件系统的，根文件系统存在哪里是由 uboot 的 bootargs 环境变量指定，bootargs会传递给Linux内核作为命令行参数。比如设置**root=/dev/mmcblk1p2**，也就是说根文件系统存储在EMMC的分区2中。我们开发板的这个位置已经有了根文件系统，所以可以启动，如果将这个变量删了，系统将会出错，启动不了。



- **在Linux中添加自己的开发板**
  - 添加开发板默认配置文件

    ```sh
    cd arch/arm/configs
    cp imx_v7_mfg_defconfig imx_alientek_emmc_defconfig
    ```

    打开 imx_alientek_emmc_defconfig 文件，找到“CONFIG_ARCH_MULTI_V6=y”这一行，将其屏蔽掉。因为 I.MX6ULL 是 ARMV7 架构的，因此要屏蔽掉 V6 相关选项，否则后面做驱动实验的时候可能会遇到驱动模块无法加载的情况。

  - 添加开发板对应的设备树文件

    ```sh
    cd arch/arm/boot/dts
    cp imx6ull-14x14-evk.dts imx6ull-alientek-emmc.dts
    ```

    imx6ull-alientek-emmc.dts创建好以后我们还需要修改文件arch/arm/boot/dts/Makefile ，找到 “ dtb-$(CONFIG_SOC_IMX6ULL)”配置项，在此配置项中加入“imx6ull-alientek-emmc.dtb” 

  - 创建一个编译脚本imx6ull_alientek_emmc.sh
  - 执行
  - 网络启动开发板

- **CPU主频和网络驱动修改**

  设置板子默认从网络启动：
  
  ```
  setenv bootcmd 'tftp 80800000 zImage;tftp 83000000 imx6ull-alientek-emmc.dtb;bootz 80800000 - 83000000;'
  ```
  
  设置根文件系统位置，emmc设备的第二个分区
  
  ```
  setenv bootargs console=ttymxc0,115200 root=/dev/mmcb1k1p2
  ```
  
  - 启动开发板进入命令行系统
  
  - 输入如下命令查看 cpu 信息：
  
    ```
    cat /proc/cpuinfo
    ```
  
    BogoMIPS 是 Linux 系统中衡量处理器运行速度的一个“尺子”，处理器性能越强，主频越高，BogoMIPS 值就越大。BogoMIPS 只是粗略的计算 CPU 性能，并不十分准确。但是我们可以通过 BogoMIPS 值来大致的判断当前处理器的性能。
  
  - 一种方法查看当前 CPU 的工作频率。进入到目录/sys/bus/cpu/devices/cpu0/cpufreq 中，此目录下会有很多文件。
  
    - 此目录中记录了 CPU 频率等信息，这些文件的含义如下： 
  
      **cpuinfo_cur_freq**：当前 cpu 工作频率，从 CPU 寄存器读取到的工作频率。 
  
      **cpuinfo_max_freq**：处理器所能运行的最高工作频率(单位: KHz）。 
  
      **cpuinfo_min_freq** ：处理器所能运行的最低工作频率(单位: KHz）。 
  
      **cpuinfo_transition_latency**：处理器切换频率所需要的时间(单位:ns)。 
  
      **scaling_available_frequencies**：处理器支持的主频率列表(单位: KHz）。 
  
      **scaling_available_governors**：当前内核中支持的所有 governor(调频)类型。 
  
      **scaling_cur_freq**：保存着 cpufreq 模块缓存的当前 CPU 频率，不会对 CPU 硬件寄存器进行检查。 
  
      **scaling_driver**：该文件保存当前 CPU 所使用的调频驱动。 
  
      **scaling_governor**：governor(调频)策略，Linux 内核一共有 5 中调频策略， 
  
      ①、Performance，最高性能，直接用最高频率，不考虑耗电。 
  
      ②、Interactive，一开始直接用最高频率，然后根据 CPU 负载慢慢降低。 
  
      ③、Powersave，省电模式，通常以最低频率运行，系统性能会受影响，一般不会用这个！ 
  
      ④、Userspace，可以在用户空间手动调节频率。 
  
      ⑤、Ondemand，定时检查负载，然后根据负载来调节频率。负载低的时候降低 CPU 频率， 
  
      这样省电，负载高的时候提高 CPU 频率，增加性能。 
  
      **scaling_max_freq**：governor(调频)可以调节的最高频率。 
  
      **cpuinfo_min_freq**：governor(调频)可以调节的最低频率。 
  
      stats 目录下给出了 CPU 各种运行频率的统计情况，比如 CPU 在各频率下的运行时间以及 
  
      变频次数。
  
      使用如下命令查看当前 CPU 频率：
  
      ```
      cat cpuinfo_cur_freq
      ```
  
      *查看 stats 目录下的 time_in_state 文件可以看到 CPU 在各频率下的工作时间*
  
  - 修改频率
  
    imx_alientek_emmc_defconfig 文件
  
    46 CONFIG_CPU_FREQ_GOV_ONDEMAND=y
  
  - 图形化界面配置频率
  
    ```
    CPU Power Management 
     -> CPU Frequency scaling 
     -> Default CPUFreq governor
    ```
  
    配置后编译
  
  - 超频操作，修改设备树，**见文档**.
  
- 修改EMMC驱动

  - 使能8线emmc驱动

    正点原子EMMC核心板上采用的是8位的数据线，Linux内核中默认是4线的，4线肯定没有8线模式速度快，所以本节要将EMMC的驱动修改为8线，修改方式很简单，直接修改设备树即可。

    相关模块改为如下代码（arch/arm/boot/dts/imx6ull-14x14-evk-emmc.dts中的配置）

    ```sh
    734 &usdhc2 {
    735 	pinctrl-names = "default", "state_100mhz", "state_200mhz";
    736 	pinctrl-0 = <&pinctrl_usdhc2_8bit>;
    737 	pinctrl-1 = <&pinctrl_usdhc2_8bit_100mhz>;
    738 	pinctrl-2 = <&pinctrl_usdhc2_8bit_200mhz>;
    739 	bus-width = <8>;
    740 	non-removable;
    741 	status = "okay";
    742 };
    ```

  - 关闭emmc1.8V供电选项

    &usdhc2 {

    ​    pinctrl-names = "default", "state_100mhz", "state_200mhz";

    ​    pinctrl-0 = <&pinctrl_usdhc2_8bit>;

    ​    pinctrl-1 = <&pinctrl_usdhc2_8bit_100mhz>;

    ​    pinctrl-2 = <&pinctrl_usdhc2_8bit_200mhz>;

    ​    bus-width = <8>;

    ​    non-removable;

    ​    **no-1-8-v;**

    ​    status = "okay";

    };

- 网络驱动修改

1. 修改LAN8720复位以及时钟引脚驱动

   主要是修改设备树文件里面的相关引脚（SNVS_TAMPER7 和 SNVS_TAMPER8）。见文档

   修改 fec1 和fec2节点的pinctrl-0属性

   修改LAN8720A的PHY地址

   ```
   ethphy0: ethernet-phy@0 {
   			compatible = "ethernet-phy-ieee802.3-c22";
   			reg = <0>;
   		};
   
   		ethphy1: ethernet-phy@1 {
   			compatible = "ethernet-phy-ieee802.3-c22";
   			reg = <1>;
   		};
   ```

   修改内核源码中的fec_main.c文件如果要在 I.MX6ULL 上使用 LAN8720A 就需要设置ENET1 和 ENET2 的 TX_CLK 引脚复位寄存器的 SION 位为 1。

   ```c
   	/* 3452行 设置 MX6UL_PAD_ENET1_TX_CLK 和 MX6UL_PAD_ENET2_TX_CLK
   	* 这两个 IO 的复用寄存器的 SION 位为 1。
   	*/
   	void __iomem *IMX6U_ENET1_TX_CLK;
   	void __iomem *IMX6U_ENET2_TX_CLK;
   
   	IMX6U_ENET1_TX_CLK = ioremap(0X020E00DC, 4);
   	writel(0X14, IMX6U_ENET1_TX_CLK);
   	
   	IMX6U_ENET2_TX_CLK = ioremap(0X020E00FC, 4);
   	writel(0X14, IMX6U_ENET2_TX_CLK);
   ```

   配置Linux内核，使能LAN8720驱动

   ```
   -> Device Drivers 
   	-> Network device support 
   		-> PHY Device support and infrastructure 
   			-> Drivers for SMSC PHYs
   ```

   修改smsc.c文件

   编译内核，网络启动

   打开网卡

   ```sh
   ifconfig eth0 up
   ifconfig eth1 up
   ```

   其中eth0 对应于 ENET2，eth1 对应于 ENET1

   设置IP后pingLinux服务器

   ```
   ifconfig eth0 192.168.137.50
   ping 192.168.137.100
   ```

   