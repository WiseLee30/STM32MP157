# STM32MP157开发文档（基于正点原子）
## 一、编译正点原子出厂TF-A、U-Boot、Linux内核
### 1、获取ST系统官方源码
1. **获取路径**
    `开发板光盘→1、程序源码→5、 ST 官方原版 Linux 源码→en.SOURCES-stm32mp1-openstlinux-5-4-dunfell-mp1-20-06-24.tar.xz`
2. **放置路径**
    `/home/lei/STM32MP157/linux/atk-mp1`
3. **文件名**
    `stm32mp1-openstlinux-5.4-dunfell-mp1-20-06-24`
4. **主要内容**：
   - uboot 源码：`u-boot-stm32mp-2020.01-r0`
   - linux 源码：`linux-stm32mp-5.4.31-r0`
   - tf-a 源码：`tf-a-stm32mp-2.2.r1-r0`
### 2、编译TF-A
1. **stm32wrapper4dbg 工具安装**
   路径为： `开发板光盘→5、开发工具→ stm32wrapper4dbg-master.zip`
   `unzip stm32wrapper4dbg-master.zip`
   `cd stm32wrapper4dbg-master
   make`
   `sudo cp stm32wrapper4dbg /usr/bin`
   验证：`stm32wrapper4dbg -s`
2. **安装设备树编译相关命令**
    `sudo apt-get install device-tree-compiler`
3. **源码路径**
    `开发板光盘→1、程序源码→ 1、正点原子 Linux 出厂系统源码→ tf-a-stm32mp-2.2.r1-g463d4d8-v1.0.tar.bz2`
    放到路径`/home/lei/STM32MP157/linux/atk-mp1/alientek_tf-a`并解压
    `tar -xvf tf-a-stm32mp-2.2.r1-g463d4d8-v1.0.tar.bz2`
4. **修改Makefile.sdk**
    将`CROSS_COMPILE`设定为`arm-none-linux-gnueabihf-`
5. **进入目录进行编译**
    `cd tf-a-stm32mp-2.2.r1/`
    `make -f ../Makefile.sdk all`
6. **生成TF-A固件**
    所用固件为： `tf-a-stm32mp157d-atk-trusted.stm32`
    位于：`/home/lei/STM32MP157/linux/atk-mp1/alientek_tf-a/build/trusted`
### 3、编译U-Boot
1. **安装相关库**
    `sudo apt-get install libncurses5-dev bison flex`
2. **源码路径**
    `开发板光盘→1、程序源码→1、正点原子 Linux 出厂系统源码→u-bootstm32mp-2020.01-xxxxxxxx-v1.0.tar.bz2`
    放到路径`/home/lei/STM32MP157/linux/atk-mp1/alientek_uboot`并进行解压
    `tar -vxf u-boot-stm32mp-2020.01-gdb8d2374-v1.0.tar.bz2`
3. **修改Makefile**
    - `ARCH = arm`
    - `CROSS_COMPILE = arm-none-linux-gnueabihf-`
4. **在源码目录进行编译**
    ```
    make distclean
    make stm32mp157d_atk_defconfig
    make V=1 DEVICE_TREE=stm32mp157d-atk all
    ```
5. **生成U-Boot固件**
    所用固件为：`u-boot.stm32`
    目录：`/home/lei/STM32MP157/linux/atk-mp1/alientek_uboot`
### 4、编译Linux内核
1. **第三方库和工具安装**
    ```
    sudo apt-get update
    sudo apt-get install lzop
    sudo apt-get install libssl-dev
    sudo apt-get install u-boot-tools
    ```
2. **源码路径**
    `开发板光盘→1、程序源码→1、正点原子 Linux 出厂系统源码→linux-5.4.31-gb8d3ec3acv1.1.tar.bz2`
    放到路径`/home/lei/STM32MP157/linux/atk-mp1/alientek_linux`并进行解压
    `tar -vxjf linux-5.4.31-gb8d3ec3ac-v1.1.tar.bz2`
3. **修改Makefile**
    - `ARCH = arm`
    - `CROSS_COMPILE = arm-none-linux-gnueabihf-`
4. **编译脚本**
    Linux 源码根目录下新建shell 脚本`:stm32mp157d_atk.sh`内容为：
    ```
    #!/bin/sh
    make distclean
    make stm32mp1_atk_defconfig
    make menuconfig
    make uImage dtbs LOADADDR=0XC2000040 -j16
    ```
    给予权限并编译：
    ```
    chmod 777 stm32mp157d_atk.sh
    ./stm32mp157d_atk.sh
    ```
5. **生成固件**
    Linux 镜像文件：`uImage`
    所在目录：`/home/lei/STM32MP157/linux/atk-mp1/alientek_linux/arch/arm/boot`
    设备树文件：`stm32mp157d-atk.dtb`
    所在目录：`/home/lei/STM32MP157/linux/atk-mp1/alientek_linux/arch/arm/boot/dts`
6. **单独编译命令**
    - 单独编译uImage：`make uImage LOADADDR=0XC2000040`
    - 设备树：`make dtbs`
## 二、Buildroot根文件系统构建
### 1、构建根文件系统
#### 源码路径
`1、例程源码-》7、buildroot 源码-》buildroot-2020.02.6.tar.bz2`
放到路径`/home/lei/STM32MP157/linux/tool`并解压
`tar -vxjf buildroot-2020.02.6.tar.bz2`
#### 配置buildroot
在`buildroot-2020.02.6`目录中
通过图形化配置：`make menuconfig`
1. 配置Target options
```
Target options
    -> Target Architecture = ARM (little endian)
    -> Target Binary Format = ELF
    -> Target Architecture Variant = cortex-A7
    -> Target ABI = EABIhf
    -> Floating point strategy = NEON/VFPv4
    -> ARM instruction set = ARM
```
2. 配置Toolchain
```
Toolchain
    -> Toolchain type = External toolchain
    -> Toolchain = Custom toolchain //用户自己的交叉编译器
    -> Toolchain origin = Pre-installed toolchain //预装的编译器
    -> Toolchain path = /usr/local/arm/gcc-arm-9.2-2019.12-x86_64-arm-none-linux-gnueabihf
    -> Toolchain prefix = $(ARCH)-none-linux-gnueabihf //前缀
    -> External toolchain gcc version = 9.x
    -> External toolchain kernel headers series = 4.20.x //交叉编译器的 linux 版本号
    -> External toolchain C library = glibc/eglibc
    -> [*] Toolchain has SSP support? (NEW) //选中
    -> [*] Toolchain has RPC support? (NEW) //选中
    -> [*] Toolchain has C++ support? //选中
    -> [*] Enable MMU support (NEW) //选中
```
3. 配置System configuration
```
System configuration
    -> System hostname = ATK-stm32mp1 //平台名字，自行设置
    -> System banner = Welcome to alientek STM32MP157 //欢迎语
    -> Init system = BusyBox //使用 busybox
    -> /dev management = Dynamic using devtmpfs + mdev //使用 mdev
    -> [*] Enable root login with password (NEW) //使能登录密码
    -> Root password = 123456 //登录密码为 123456
```
4. 配置Filesystem images
```
-> Filesystem images
    -> [*] ext2/3/4 root filesystem //如果是 EMMC 或 SD 卡的话就用 ext3/ext4
        -> ext2/3/4 variant = ext4 //选择 ext4 格式
        -> exact size =1G //ext4 格式根文件系统 1GB(根据实际情况修改)
    -> [*] ubi image containing an ubifs root filesystem //如果使用 NAND 的话就用 ubifs
```
5. 禁止编译 Linux 内核和 uboot
```
-> Kernel
    -> [ ] Linux Kernel //不要选择编译 Linux Kernel 选项！
```
```
-> Bootloaders
    -> [ ] U-Boot //不要选择编译 U-Boot 选项！
```
6. 配置Target packages
```
-> Target packages
    -> System tools
        -> [*] kmod //使能内核模块相关命令
```
7. 保存配置项
保存为：`./configs/stm32mp1_atk_defconfig`
重新配置buildroot：`make stm32mp1_atk_defconfig`
#### 编译buildroot
`sudo make`（sudo+不能多核编译）
将所需文件拷贝到目标目录
```
cd /home/lei/STM32MP157/linux/tool/buildroot-2020.02.6/output/images
cp rootfs.tar /home/lei/STM32MP157/linux/nfs/rootfs/ -f //拷贝 rootfs.tar
cd /home/lei/STM32MP157/linux/nfs/rootfs/ //进入到 rootfs 目录下
tar -vxf rootfs.tar //解压缩 rootfs.tar
rm rootfs.tar //删除 rootfs.tar
```
之后通过nfs挂载到开发板上
### 2、busybox配置
#### 源码路径
压缩包：`/home/lei/STM32MP157/linux/tool/buildroot-2020.02.6/dl/busybox`
解压结果：`/home/lei/STM32MP157/linux/tool/buildroot-2020.02.6/output/build/busybox-1.31.1`
#### 修改busybox 配置
在buildroot源码根目录输入，进入busybox配置界面：`sudo make busybox-menuconfig`
```
Location:
    -> Settings
        -> Build static binary (no shared libs)（不选）
```
```
Location:
    -> Settings
        -> vi-style line editing commands
```
```
Location:
    -> Linux Module Utilities
        -> Simplified modutils（不选）
```
```
Location:
    -> Linux System Utilities
        -> mdev (16 kb) //确保下面的全部选中，默认都是选中的
```
```
Location:
    -> Settings
        -> Support Unicode //选中
            -> Check $LC_ALL, $LC_CTYPE and $LANG environment variables //选中
```
#### 中文字符支持
#### 单独编译命令
`sudo make busybox`
#### 使能depmod 命令
 busybox 中使能
 ```
-> Linux Module Utilities
    -> [*] depmod //使能 depmod 命令
 ```
### 3、显示路径
在`./etc/profile.d`目录下新建shell 脚本文件`myprofile.sh`
```
cd etc/profile.d/
touch myprofile.sh
sudo chmod 777 myprofile.sh
```
脚本内容为：
```
#!/bin/sh

PS1='[\u@\h]:\w$ '
export PS1
```
### 4、使能sysfs debug 目录
在`./etc/init.d`目录下创建自启动文件`Sautorun`
```
cd etc/init.d/
touch Sautorun
chmod 777 Sautorun
```
脚本内容为：
```
#/bin/sh

mount -t debugfs none /sys/kernel/debug
```
### 5、修改Ubuntu的nfs版本配置
进入`/etc/default/nfs-kernel-server `
在最后面添加：`RPCNFSDOPTS="--nfs-version 2,3,4 --debug --syslog"`
重启NFS服务：`sudo /etc/init.d/nfs-kernel-server restart`
### 6、烧写到eMMC
```
**擦除原有环境变量**
eraseenv
**设置bootcmd环境变量**
setenv bootcmd 'ext4load mmc 1:2 c2000000 uImage;ext4load mmc 1:2 c4000000 stm32mp157d-atk.dtb;bootm c2000000 - c4000000'
**设置bootargs环境变量**
setenv bootargs 'console=ttySTM0,115200 root=/dev/nfs nfsroot=192.168.229.114:/home/lei/STM32MP157/linux/nfs/rootfs,proto=tcp rw ip=192.168.229.250:192.168.229.114:192.168.229.1:255.255.255.0::eth0:off'
**保存**
saveenv
**启动**
boot
```
## 三、驱动开发
### 1、字符设备驱动
以LED驱动为例
```
/*宏定义*/

/* 寄存器物理地址 */
基地址+偏移，例如：
#define PERIPH_BASE (0x40000000)
#define MPU_AHB4_PERIPH_BASE (PERIPH_BASE + 0x10000000)

/*映射后的寄存器虚拟地址指针*/
static void __iomem *MPU_AHB4_PERIPH_RCC_PI;

设备结构体
描述设备号、类等信息
struct newchrled_dev{
    dev_t devid; /* 设备号 */
    struct cdev cdev; /* cdev */
    struct class *class; /* 类 */
    struct device *device; /* 设备 */
    int major; /* 主设备号 */
    int minor; /* 次设备号 */
};

/*构建设备*/
struct newchrled_dev newchrled; /* led 设备 */

/*使用sta实现解析信号*/
void led_switch(u8 sta)
{
    u32 val = 0;
    if(sta == LEDON) {
        val = readl(GPIOI_BSRR_PI);
        val |= (1 << 16);
        writel(val, GPIOI_BSRR_PI);
    }else if(sta == LEDOFF) {
        val = readl(GPIOI_BSRR_PI);
        val|= (1 << 0);
        writel(val, GPIOI_BSRR_PI);
    }
}

/*取消映射*/
void led_unmap(void)
{
    iounmap(MPU_AHB4_PERIPH_RCC_PI);
    iounmap(GPIOI_MODER_PI);
    iounmap(GPIOI_OTYPER_PI);
    iounmap(GPIOI_OSPEEDR_PI);
    iounmap(GPIOI_PUPDR_PI);
    iounmap(GPIOI_BSRR_PI);
}

/*打开设备*/
static int led_open(struct inode *inode, struct file *filp)
{
    filp->private_data = &newchrled; /* 设置私有数据 */
    return 0;
}

/*读数据*/
static ssize_t led_read(struct file *filp, char __user *buf, size_t cnt, loff_t *offt)
{
    return 0;
}

/*写数据*/
static ssize_t led_write(struct file *filp, const char __user *buf,size_t cnt, loff_t *offt)
{
    int retvalue;
    unsigned char databuf[1];
    unsigned char ledstat;

    retvalue = copy_from_user(databuf, buf, cnt);//从用户空间获取输入
    if(retvalue < 0) {
        printk("kernel write failed!\r\n");
        return -EFAULT;
    }

    ledstat = databuf[0]; /* 获取状态值 */
    if(ledstat == LEDON) {
        led_switch(LEDON); /* 打开 LED 灯 */
    } e if(ledstat == LEDOFF) {
        led_switch(LEDOFF); /* 关闭 LED 灯 */
    }
    return 0;
}

/*关闭设备*/
static int led_release(struct inode *inode, struct file *filp)
{
    return 0;
}

/* 设备操作函数 */
static struct file_operations newchrled_fops = {
    .owner = THIS_MODULE,
    .open = led_open,
    .read = led_read,
    .write = led_write,
    .write = led_write,
    .release = led_release,
};

/*驱动初始化函数*/
static int __init led_init(void)
{
    u32 val = 0;
    int ret;

    /*初始化LED*/
    /* 1、寄存器地址映射 */
    MPU_AHB4_PERIPH_RCC_PI = ioremap(RCC_MP_AHB4ENSETR, 4);
    /* 2、使能 PI 时钟 */
    /* 3、设置 PI0 通用的输出模式。 */
    /* 4、设置 PI0 为推挽模式。 */
    /* 5、设置 PI0 为高速。 */
    /* 6、设置 PI0 为上拉。 */
    /* 7、默认关闭 LED */

    /* 注册字符设备驱动 */
    /* 1、创建设备号 */
    if (newchrled.major) { /* 定义了设备号 */
        newchrled.devid = MKDEV(newchrled.major, 0);
        ret = register_chrdev_region(newchrled.devid, NEWCHRLED_CNT, NEWCHRLED_NAME);
        if(ret < 0) {
            pr_err("cannot register %s char driver [ret=%d]\n",NEWCHRLED_NAME, NEWCHRLED_CNT);
            goto fail_map;
        }
    }else {/* 没有定义设备号 */
        ret = alloc_chrdev_region(&newchrled.devid, 0, NEWCHRLED_CNT, NEWCHRLED_NAME); /* 申请设备号 */
        if(ret < 0) {
            pr_err("%s Couldn't alloc_chrdev_region, ret=%d\r\n", NEWCHRLED_NAME, ret);
            goto fail_map;
        }
        newchrled.major = MAJOR(newchrled.devid);
        newchrled.minor = MINOR(newchrled.devid);
    }
    /* 2、初始化 cdev */
    newchrled.cdev.owner = THIS_MODULE;
    cdev_init(&newchrled.cdev, &newchrled_fops);
    /* 3、添加一个 cdev */
    ret = cdev_add(&newchrled.cdev, newchrled.devid, NEWCHRLED_CNT);
    if(ret < 0)
        goto del_unregister;
    /* 4、创建类 */
    newchrled.class = class_create(THIS_MODULE, NEWCHRLED_NAME);
    if (IS_ERR(newchrled.class)) {
        goto del_cdev;
    }
    /* 5、创建设备 */
    newchrled.device = device_create(newchrled.class, NULL, newchrled.devid, NULL, NEWCHRLED_NAME);
    if (IS_ERR(newchrled.device)) {
        goto destroy_class;
    }
    return 0;

destroy_class: 
    class_destroy(newchrled.class);
del_cdev:
    cdev_del(&newchrled.cdev);
del_unregister:
    unregister_chrdev_region(newchrled.devid, NEWCHRLED_CNT);
fail_map:
    led_unmap();
    return -EIO;
}

/*驱动出口函数*/
static void __exit led_exit(void)
{
    /* 取消映射 */
    led_unmap();

    /* 注销字符设备驱动 */
    cdev_del(&newchrled.cdev);/* 删除 cdev */
    unregister_chrdev_region(newchrled.devid, NEWCHRLED_CNT);
    device_destroy(newchrled.class, newchrled.devid);
    class_destroy(newchrled.class);
}

module_init(led_init);
module_exit(led_exit);
MODULE_LICENSE("GPL");
MODULE_AUTHOR("ALIENTEK");
MODULE_INFO(intree, "Y");
```
### 2、设备树
#### 概念
使用树形结构描述板级设备(开发板)上的设备信息，如CPU数量、内存基地址、IIC接口/SPI接口连接设备的DTS文件
#### 设备节点
每个设备都是一个节点，叫做设备节点，每个键点通过描述属性的键值对来描述节点信息
1. 每个设备树文件只有一个根节点/
2. 命名格式为：
   1. `node-name@unit-address`
      - `node-name`：节点名字，是ASCII字符串
      - `unit-address`：一般表示设备地址或寄存器首地址
    2. `label: node-name@unit-address`
    可以通过&label访问节点，相当于别名
3. 常见数据形式
    - 字符串
    - 32位无符号整数
    - 字符串列表
4. 标准属性
    - `compatible`：字符串列表，用于绑定设备和驱动，`"manufacturer,model"`
    - `model`：字符串，描述开发板名/设备模块信息
    - `status`：字符串，描述设备状态
    - `#address-cells`和`#size-cells`：无符号32位整形，描述子节点reg属性总，地址信息和长度信息所占用字长
    - `#address-cells`：表示(起始地址，地址长度)，描述外设寄存器地址范围信息/I2C设备地址
    - `ranges`：空或(子总线地址空间的物理地址,父总线地址空间的物理地址,子地址空间的长度)进行地址转换
    - `name`
    - `device_type`
    - `根节点compatible`：表示(硬件设备,SOC)，使用`DT_MACHINE_START`进行匹配
5. 向节点追加或修改内容
   只需要在硬件设备对应的设备树文件和库中添加即可，例如`stm32mp157d-atk.dts`和`stm32mp157d-atk.dtsi` 
#### 初始化LED GPIO
1. 修改设备树，添加相应节点(重点位设置reg属性)
2. 获取reg属性中的寄存器地址并初始化
3. 设置GPIO复用功能，`GPIOI_MODER`寄存器
4. 设置GPIO速度、上下拉、模式，`GPIOI_OTYPER`、 `GPIOI_OSPEEDR`和`GPIOI_PUPDR`等寄存器
### 3、pinctrl和gpio子系统
- pinctrl子系统是设置PIN的复用和电器属性，将PIN复用位GPIO；gpio子系统是初始化GPIO并提供相应的PGIO函数
- 二者驱动文件相同，都是`pinctrl-stm32mp157.c`，pinctrl驱动会顺便把gpio驱动一起注册，入口函数都是`stm32_pctl_probe`
#### pinctrl子系统
1. 主要工作内容
   1. 获取设备树中 pin 信息
   2. 根据获取到的 pin 信息来设置 pin 的复用功能
   3. 根据获取到的 pin 信息来设置 pin 的电气特性
2. 在设备树中配置PIN信息
    `stm32mp15-pinctrl.dtsi`中的`pinctrl子节点`
    1. `pinmux属性`:存放外设所要使用的所有IO，使用`STM32_PINMUX`配置GPIO组、该组第几个引脚、复用模式
    2. 电器属性配置(非必须)
3. 驱动流程
    1. 定义 pinctrl_desc 结构体
    2. 初始化结构体， 重点是 pinconf_ops、 pinmux_ops 和 pinctrl_ops 这三个结构体成员变量
    3. 调用 devm_pinctrl_register 函数完成 PIN 控制器注册
4. 实例
在pinctrl节点下添加一个“uart4_pins”节点
```C
pinctrl {
    uart4_pins: uart4-0 {//添加节点
        pins1{//添加pins属性
            pinmux = <STM32_PINMUX('G', 11, AF6)>; /* UART4_TX */   //具体配置
            bias-disable;
            drive-push-pull;
        };
    };
};
```
#### gpio子系统
1. 主要工作
    在设备树中添加GPIO相关信息，在驱动程序中使用gpio子系统提供的API函数来操作GPIO
2. 实例
在根节点“/”下创建led设备子节点
```C
led {
    compatible = "atk,led";
    gpio = <&gpioi 0 GPIO_ACTIVE_LOW>;
    status = "okay";
};
```
3. 驱动程序流程
    1. 修改设备树，`stm32mp157d-atk.dts`根节点下创建gpioled节点
        ```C
        gpioled {
            compatible = "alientek,led";
            status = "okay";
            led-gpio = <&gpioi 0 GPIO_ACTIVE_LOW>;
        };
        ```
    2. 具体驱动编写
        ```C
        1. 设备结构体
        2. 用创建设备
        3. open
        4. read
        5. write
        6. release
        7. 操作函数
        8. 驱动入口
            /*设置 LED 所使用的 GPIO*/
            1. 获取设备节点
            gpioled.nd = of_find_node_by_path("/gpioled");
            2. 读取status
            ret = of_property_read_string(gpioled.nd, "status", &str);
            3. 获取compatible属性并匹配
            ret = of_property_read_string(gpioled.nd, "compatible", &str);
            4. 获取设备树中gpio属性，得到LED编号
            gpioled.led_gpio = of_get_named_gpio(gpioled.nd, "led-gpio", 0);
            5. 向gpio子系统申请使用 GPIO
            ret = gpio_request(gpioled.led_gpio, "LED-GPIO");
            6. 设置输出，高电平，默认关闭
            ret = gpio_direction_output(gpioled.led_gpio, 1);
            /*注册字符设备驱动*/
        9. 驱动出口
        10. 模块描述
        ```
### 4、并发与竞争
### 5、Linux中断
### 6、阻塞和非阻塞
### 7、异步通知
### 8、platform设备驱动
#### 设备驱动分离
**驱动<-->总线<-->设备模型**
 - 常见总线有I2C、SPI、USB等总线，对于SOC中没有总线的外设，采用platform虚拟总线，相应的有platform_device和platform_driver
 - platform总线是bus_type的一个具体实例
 - match函数和用来根据注册的设备/驱动查找相应的驱动/设备
#### 匹配方式
 - OF类型匹配(设备树)
    device_driver结构体有名为of_match_table的成员变量，保存着驱动的compatible匹配表，可以与设备树中每个设备节点的compatible属性进行比较
 - ACPI匹配
 - id_table匹配
 - name字段
#### 驱动框架
```C
1. 设备结构体
2. 定义设备结构体变量
3. open
4. read
5. write
6. 字符设备驱动操作集
7. platform驱动的probe函数
8. remove
9. 匹配列表match
10. 平台驱动结构体
11. 驱动模块加载
12. 驱动模块卸载
13. 模块信息
```
#### 设备树下的platform驱动编写
1. 整体流程
    驱动设备匹配->根据设备树中pinctrl属性设置电气特性->probe函数->在probe函数中执行字符设备驱动->注销驱动模块时执行remove函数
2. 修改pinctrl-stm32.c
    ST针对STM32MP1提供 Linux系统中，其pinctrl 配置的电气属性只能在platform 平台下被引用pinctrl什么时候有效，不同的芯片厂商有不同的处理方法。imx6ull芯片在Linux系统启动运行过程中会自动解析设备树下的pinctrl配置，然后初始化引脚的电器属性，不需要platform驱动框架。
    对于STM32MP1来说，PI0这个IO已经被其他外设申请走，所以需要修改修改pinctrl-stm32.c，否则当木哦各引脚用作GPIO时会提示该引脚无法申请到
    修改`stm32_mpx_ops`下的`.srtict`为false
    完成之后需要重新编译内核
3. 创建设备的 pinctrl 节点
    在`stm32mp15-pinctrl.dtsi`文件(TM32MP1 的所有引脚 pinctrl 配置都是在这个文件里面完成)中添加设备节点
4. 在设备树中创建设备节点
    在`stm32mp157d-atk.dts` 中添加节点
5. 注意platform驱动中的兼容属性设置
    - 兼容表xxx_of_match
    - 声明设备匹配表
    - 设置platform_driver中of_match_table
6. 注意pinctrl配置，检查引脚有没有被复用为多个设备，或者GPIO有没有被占用
### 9、Linux I2C
#### 基本原理
1. 简介
    数据线：SCL(串行时钟线)和SDA(串行数据线)，接上拉电阻，空闲时处于高电平。
    标准模式下100Kb/S，快速模式下400Kb/S
    支持多个从机
2. 通信
    - 起始位：当SCL为高电平时，SDA出现下降沿表示为起始位
    - 停止位：当SCL为高电平时，SDA出现下降沿表示为停止位
    - 数据传输：进行时，保证SCL为高电平时，数据稳定；数据变化只发生在SCL为低电平时
    - 应答信号：I2C主机发送完8位数据后，将SDA设置为**输入状态**，等待I2C主机应达。**应到信号**由从机发出，主机提供应答信号所需的时钟，主机发完八位数据后紧跟着**一个时钟信号**就是给应答信号使用。从机通过**拉低SDA表示通信成功**。
3. 写时序
    1. 开始信号
    2. 主机发送要写的I2C设备地址，8位数据，其中高7位位设备地址
    3. 最后一位是读写位，1读，0写
    4. 从机发送ACK应答信号
    5. 主机重新发送开始信号
    6. 主机发送要写入数据的寄存器地址
    7. 从机发送ACK信号
    8. 主机发送要写入寄存器的数据
    9. 从机发送ACK信号
    10. 停止信号
4. 读时序
    1. 开始信号
    2. 主机发送要读的I2C设备地址
    3. 读写控制位，写信号
    4. 从机发送ACK应答信号
    5. 重新发送开始信号
    6. 主机发送要读取的寄存器地址
    7. 从机发送ACK应答信号
    8. 重新发送开始信号
    9. 重新发送要读取的I2C设备地址
    10. 读写控制位，读信号
    11. 从机发送ACK应答信号
    12. 从I2C器件里读到的数据
    13. 主机发出NO ACK信号，表示读取完成，不需要从机再发送ACK信号
    14. 主机发出STOP信号，停止I2C通信
#### I2C子系统总体框架
1. 裸机I2C驱动
    - I2C主机驱动：SoC的I2C控制器对应的驱动程序，一旦编写完成就不需要再做修改，其他I2C设备直接调用主机驱动提供的API函数完成读写操作即可。
    - I2C设备驱动：挂在I2C总线下的具体设备对应的驱动程序
2. Linux使用I2C总线框架，主要分为三大部分：
    1. I2C核心
        提供I2C总线驱动(适配器)和设备驱动的注册、注销方法，I2C通信方法(algorithm)与具体硬件无关的代码，以及探测设备地址的上层代码
    2. I2C总线驱动
        I2C适配器的软件实现，提供I2C适配器与设备间完成数据通信的能力。由i2c_adapter和i2c_algorithm来描述
    3. I2C设备驱动
        包含设备注册和驱动注册
        重点关注两个数据结构：
        - i2c_client：描述I2C总线下的设备
        - **i2c_driver**：描述I2C总线下的设备驱动
3. 驱动编写流程
    1. 在设备树中创建相应节点
        使用I2C器件AP3216C三合一环境传感器，挂在**I2C5总线接口**上，所以必须在**i2c5节点**下创建字节点来描述PA3216C设备
        ```C
        &i2c5 {
            pinctrl-names = "default", "sleep";
            pinctrl-0 = <&i2c5_pins_a>;
            pinctrl-1 = <&i2c5_pins_sleep_a>;
            status = "okay";

            ap3216c@1e {
            compatible = "alientek,ap3216c";
            reg = <0x1e>;
            };
        }；
        ```
        `ap3216c`是子节点名字，@后的`1e`是ap3216c的I2C器件地址，设置`compatible属性`是"alientek,ap3216c"，`reg属性`设置ap321c 的器件地址
    2. i2c设备数据收发流程处理
        I2C设备驱动首先要**初始化i2c_driver**并向**Linux内核注册**。设备和驱动匹配后i2c_driver里的**probe函数**会执行，里面是字符设备驱动。一般需要在probe函数里初**始化I2C设备**，需要对I2C设备寄存器进行读写，**i2c_msg结构体**来描述一个消息，使用**i2c_transfer**函数发送数据。最终调用**i2c_algorithm**里面的**master_xfer函数**.
4. 程序编写
   1. 修改设备树
      1. `stm32mp15-pinctrl.dtsi`，AP3216C使用了I2C5接口。I2C5使用IO为PA11和PA12，需要根据数据手册设置I2C5的pinmux配置。如果要使用AP3216C的中断功能还需要初始化AP_INT对应的PE4这个IO。
      2. `stm32mp157d-atk.dts`，在i2c5节点追加ap321c6子节点，修改完成后使用`make dtbs`，然后使用新的设备树启动Linux内核，如果修改正确会在/sys/bus/i2c/deveices目录下看到一个名为`0-001e`的子目录
   2. AP3216C驱动编写 
      1. ap3216creg.h 
        寄存器宏定义
      2. ap3216c.c
    - 定义设备结构体
    ```C
    struct ap3216c_dev {
        struct i2c_client *client; /* i2c 设备 */
        dev_t devid; /* 设备号 */
        struct cdev cdev; /* cdev */
        struct class *class; /* 类 */
        struct device *device; /* 设备 */
        struct device_node *nd; /* 设备节点 */
        unsigned short ir, als, ps; /* 三个光传感器数据 */
    };
    ```
    - 从多个寄存器读数据
    ```C
    static int ap3216c_read_regs(struct ap3216c_dev *dev, u8 reg, void *val, int len)
    {
        int ret;
        struct i2c_msg msg[2];
        struct i2c_client *client = (struct i2c_client *)dev->client;
        
        /* msg[0]为发送要读取的首地址 */
        msg[0].addr = client->addr; /* ap3216c 地址 */
        msg[0].flags = 0; /* 标记为发送数据 */
        msg[0].buf = &reg; /* 读取的首地址 */
        msg[0].len = 1; /* reg 长度 */

        /* msg[1]读取数据 */
        msg[1].addr = client->addr; /* ap3216c 地址 */
        msg[1].flags = I2C_M_RD; /* 标记为读取数据 */
        msg[1].buf = val; /* 读取数据缓冲区 */
        msg[1].len = len; /* 要读取的数据长度 */

        ret = i2c_transfer(client->adapter, msg, 2);
        if(ret == 2) {
            ret = 0;
        } else{
            printk("i2c rd failed=%d reg=%06x len=%d\n",ret, reg, len);
            ret = -EREMOTEIO;
        }
        return ret;
    }
    ```
    - 向多个寄存器写数据
    ```C
    static s32 ap3216c_write_regs(struct ap3216c_dev *dev, u8 reg, u8 *buf, u8 len)
    {
        u8 b[256];
        struct i2c_msg msg;
        struct i2c_client *client = (struct i2c_client *)dev->client;
       
        b[0] = reg; /* 寄存器首地址 */
        memcpy(&b[1],buf,len); /* 将要写入的数据拷贝到数组 b 里面 */

        msg.addr = client->addr; /* ap3216c 地址 */
        msg.flags = 0; /* 标记为写数据 */

        msg.buf = b; /* 要写入的数据缓冲区 */
        msg.len = len + 1; /* 要写入的数据长度 */

        return i2c_transfer(client->adapter, &msg, 1);
    }
    ```
    - 指定寄存器读数据
    ```C
    static unsigned char ap3216c_read_reg(struct ap3216c_dev *dev, u8 reg)
    {
        u8 data = 0;
        
        ap3216c_read_regs(dev, reg, &data, 1);
        return data;
    }
    ```
    - 指定寄存器写数据
    ```C
    static void ap3216c_write_reg(struct ap3216c_dev *dev, u8 reg, u8 data)
    {
        u8 buf = 0;
        buf = data;
        ap3216c_write_regs(dev, reg, &buf, 1);
    }
    ```
    - 读取AP3216C数据
    ```C
    void ap3216c_readdata(struct ap3216c_dev *dev)
    {
        unsigned char i =0;
        unsigned char buf[6];

        /* 循环读取所有传感器数据 */
        for(i = 0; i < 6; i++) {
            buf[i] = ap3216c_read_reg(dev, AP3216C_IRDATALOW + i);
        }

        if(buf[0] & 0X80) /* IR_OF 位为 1,则数据无效 */
            dev->ir = 0;
        else
            dev->ir = ((unsigned short)buf[1] << 2) | (buf[0] & 0X03)
        
        dev->als = ((unsigned short)buf[3] << 8) | buf[2];

        if(buf[4] & 0x40) /* IR_OF 位为 1,则数据无效 */
            dev->ps = 0;
        else /* 读取 PS 传感器的数据 */
            dev->ps = ((unsigned short)(buf[5] & 0X3F) << 4) | (buf[4] &);
    }
    ```
    - 打开设备
    ```C
    static int ap3216c_open(struct inode *inode, struct file *filp)
    {
        /* 从 file 结构体获取 cdev 指针， 再根据 cdev 获取 ap3216c_dev 首地址 */
        struct cdev *cdev = filp->f_path.dentry->d_inode->i_cdev;
        struct ap3216c_dev *ap3216cdev = container_of(cdev, struct ap3216c_dev, cdev);

        /* 初始化 AP3216C */
        ap3216c_write_reg(ap3216cdev, AP3216C_SYSTEMCONG, 0x04);
        mdelay(50);
        ap3216c_write_reg(ap3216cdev, AP3216C_SYSTEMCONG, 0X03)
        return 0;
    }
    ```
    - 从设备中读取数据
    ```C
    static ssize_t ap3216c_read(struct file *filp, char __user *buf, size_t cnt, loff_t *off)
    {
        short data[3];
        long err = 0;
        
        struct cdev *cdev = filp->f_path.dentry->d_inode->i_cdev;
        struct ap3216c_dev *dev = container_of(cdev, struct ap3216c_dev, cdev);

        ap3216c_readdata(dev);

        data[0] = dev->ir;
        data[1] = dev->als;
        data[2] = dev->ps;
        err = copy_to_user(buf, data, sizeof(data));
        return 0;
    }
    ```
    - 关闭设备
    ```C
    static int ap3216c_release(struct inode *inode, struct file *filp)
    {
        return 0;
    }
    ```
    - AP3216C操作函数
    ```C
    static const struct file_operations ap3216c_ops = {
        .owner = THIS_MODULE,
        .open = ap3216c_open,
        .read = ap3216c_read,
        .release = ap3216c_release,
    };
    ```
    - i2c驱动的probe函数
    ```C
    static int ap3216c_probe(struct i2c_client *client, const struct i2c_device_id *id)
    {
        int ret;
        struct ap3216c_dev *ap3216cdev;

        ap3216cdev = devm_kzalloc(&client->dev, sizeof(*ap3216cdev), GFP_KERNEL);

        if(!ap3216cdev)
            return -ENOMEM;
        
        /* 注册字符设备驱动 */
        /* 1、创建设备号 */
        ret = alloc_chrdev_region(&ap3216cdev->devid, 0, AP3216C_CNT, AP3216C_NAME);

        if(ret < 0) {
            pr_err("%s Couldn't alloc_chrdev_region, ret=%d\r\n", AP3216C_NAME, ret);
            return -ENOMEM;
        }

        /* 2、初始化 cdev */
        ap3216cdev->cdev.owner = THIS_MODULE;
        cdev_init(&ap3216cdev->cdev, &ap3216c_ops);

        /* 3、添加一个 cdev */
        ret = cdev_add(&ap3216cdev->cdev, ap3216cdev->devid, AP3216C_CNT);

        if(ret < 0) {
            goto del_unregister;
        }

        /* 4、创建类 */
        ap3216cdev->class = class_create(THIS_MODULE, AP3216C_NAME);
        if (IS_ERR(ap3216cdev->class)) {
            goto del_cdev;
        }

        /* 5、创建设备 */
        ap3216cdev->device = device_create(ap3216cdev->class, NULL, ap3216cdev->devid, NULL, AP3216C_NAME);

        if (IS_ERR(ap3216cdev->device)) {
            goto destroy_class;
        }
        ap3216cdev->client = client;
        /* 保存 ap3216cdev 结构体 */
        i2c_set_clientdata(client, ap3216cdev);

        return 0;
    destroy_class:
        device_destroy(ap3216cdev->class, ap3216cdev->devid);
    del_cdev:
        cdev_del(&ap3216cdev->cdev);
    del_unregister:
        unregister_chrdev_region(ap3216cdev->devid, AP3216C_CNT);
        return -EIO;
    }
    ```
    - remove函数
    ```C
    static int ap3216c_remove(struct i2c_client *client)
    {
        struct ap3216c_dev *ap3216cdev = i2c_get_clientdata(client);
        /* 注销字符设备驱动 */
        /* 1、删除 cdev */
        cdev_del(&ap3216cdev->cdev);
        /* 2、注销设备号 */
        unregister_chrdev_region(ap3216cdev->devid, AP3216C_CNT);
        /* 3、注销设备 */
        device_destroy(ap3216cdev->class, ap3216cdev->devid);
        /* 4、注销类 */
        class_destroy(ap3216cdev->class);
        return 0;
    }
    ```
    - 传统匹配方式ID列表
    ```C
    static const struct i2c_device_id ap3216c_id[] = {
        {"alientek,ap3216c", 0},
        {}
    };
    ```
    - 设备树匹配列表
    ```C
    static const struct of_device_id ap3216c_of_match[] = {
        { .compatible = "alientek,ap3216c" },
        { /* Sentinel */ }
    };
    ```
    - i2c驱动结构体
    ```C
    static struct i2c_driver ap3216c_driver = {
        .probe = ap3216c_probe,
        .remove = ap3216c_remove,
        .driver = {
            .owner = THIS_MODULE,
            .name = "ap3216c",
            .of_match_table = ap3216c_of_match,
        },
        .id_table = ap3216c_id,
    };
    ```
    - i2c驱动入口函数
    ```C
    static int __init ap3216c_init(void)
    {
        int ret = 0;
        ret = i2c_add_driver(&ap3216c_driver);
        return ret;
    };
    ```
    - i2c驱动出口函数
    ```C
    static int __init ap3216c_init(void)
    {
        i2c_del_driver(&ap3216c_driver);
    };
    ```
    - 模块信息
    module_init(ap3216c_init);
    module_exit(ap3216c_exit);
    MODULE_LICENSE("GPL");
    MODULE_AUTHOR("ALIENTEK");
    MODULE_INFO(intree, "Y");
### 10、Linux SPI
#### 基本原理 
1. SPI简介
    串行，高速，全双工，同步通信总线
    一般需要4根通信线(单向传输使用3根)：
    - CS/SS：片选信号线，通过拉低片信号选择相应从机
    - SCK：串行时钟
    - MOSI/SDO：主出从入信号线
    - MISO/SDI：主入从出信号线
2、四种工作模式：
    时钟极性：
    - CPOL = 0，串行时钟空间状态为低电平。
    - CPOL = 1，串行时钟空闲状态为高电平
    时钟相位
    - CPHA = 0，串行时钟第一个跳变沿采集数据
    - CPHA = 1，串行时钟第二个跳变沿采集数据
#### 总体框架
1. 总线框架
    - SPI核心层
    - SPI控制器驱动层
    - SPI设备驱动层
2. 设备驱动匹配
    由SPI总线`spi_bus_type`来完成，匹配函数为`spi_match_device`
3. 驱动编写流程
    - IO的pinctrl子节点创建和修改(检查有没有被使用)
    - SPI设备节点的创建与修改
    - 数据收发处理流程：
        1. 申请并**初始化 spi_transfer**，设置spi_transfer的tx_buf成员变量，tx_buf为要发送的数
据。然后设置rx_buf成员变量，rx_buf保存着接收到的数据。最后设置len 成员变量，也就是要进行数据通信的长度
        2. 使用 spi_message_init函数**初始化 spi_message**
        3. 使用spi_message_add_tail函数将前面设置好的**spi_transfer添加到spi_message队列中**
        4. 使用spi_sync函数完成SPI 数据同步传输
1. 程序编写
    1. 设备结构体创建
    2. spi_driver注册和注销
    ```C
    //传统ID列表
    //设备树匹配列表
    //SPI驱动结构体
    对.probe .remobe .driver .id_table进行赋值
    //驱动入口函数
    spi_register_driver
    //驱动出口函数
    spi_unregister_driver
    //模块信息
    ```
    3. probe和remove函数
    **probe**
    ```C
    //创建设备结构体对象
    //分配对象空间
    //注册设备字符驱动
        //创建设备号
        //初始化cdev
        //添加cdev
        //创建类
        //创建设备
        //初始化spi_device
        配置spi模式
    ```
    **remove**
    ```C
    //获取驱动数据
    //注销字符设备驱动
        //删除cdev
        //注销设备号
        //注销设备
        //注销类
    ```
    4. 寄存器读写和初始化
    ```C
    //读取多个寄存器数据
    //向多个寄存器写数据
    //读取一个寄存器
    //写入一个寄存器
    //读取ICM20608设备原始数据
    //内部寄存器初始化
        通过寄存器写入数据
    ```
    5. 字符驱动框架
    ```C
    //open
    //read
    //release
    //操作函数
    ```
### 11、Linux UART
#### UART驱动框架
1. 设备结构体创建
2. spi_driver注册和注销
3. 匹配成功后执行probe函数，重点是初始化uart_port，然后添加到对应的uart_driver中
   - **stm32_usart_of_get_port**函数，它主要是负责配置 stm32_ports 数组
   - **stm32_usart_init_port**函数，它主要是负责获取 SOC UART 外设首地址、中断号、注册中断函数同时还设置 uart_ops 为 stm32_uart_ops(STM32MP1最底层驱动哈数集合)
   - **uart_add_one_port**向 uart_driver 添加 uart_port
4. stm32_uart_ops 中的函数基本都是和 STM32MP1 的 UART 寄存器打交道
### 12、Linux CAN
### 13、Linux WiFi