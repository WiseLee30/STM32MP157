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
1. 通信 
2. 读时序
3. 写时序
#### I2C子系统总体框架
1. 裸机I2C驱动
    - I2C主机驱动：SoC的I2C控制器对应的驱动程序，一旦编写完成就不需要再做修改，其他I2C设备直接调用主机驱动提供的API函数完成读写操作即可。
    - I2C设备驱动：挂在I2C总线下的具体设备对应的驱动程序
### 10、Linux SPI
2. Linux
    使用I2C总线框架，主要分为
    - I2C核心
    - I2C总线驱动
    - I2C设备驱动
### 11、Linux UART
### 12、Linux CAN
### 13、Linux WiFi