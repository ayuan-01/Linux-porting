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

  - 编译完成之后会生成一个u-boot.bin文件。必须向u-boot.bin添加头部信息。u-boot编译最后会通过./tools/mkimage软件添加头部信息，生成u-boot.imx文件
  - 如果配置过uboot，那么一定要注意shell脚本会清除整个工程。那么配置的文件也会被删除。配置项也会被删除掉
  - 为了方便开发，建议在uboot顶层makefile里面设置好ARCH和CROSS_COMPILE两个变量的值。

  