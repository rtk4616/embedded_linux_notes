# 设备模型

为了降低设备多样性带来的Linux驱动开发的复杂度，以及设备热拔插处理、电源管理等，Linux内核提出了`设备模型`（也称作Driver Model）的概念。设备模型将硬件设备归纳、分类，然后抽象出一套标准的数据结构和接口。驱动的开发，就简化为对内核所规定的数据结构的填充和实现.

## 构成

### bus

总线是CPU和一个或多个设备之间信息交互的通道。而为了方便设备模型的抽象，所有的设备都应连接到总线上（无论是CPU内部总线、虚拟的总线还是“platform Bus”）

### class

在Linux设备模型中，Class的概念非常类似面向对象程序设计中的Class（类），它主要是集合具有相似功能或属性的设备，这样就可以抽象出一套可以在多个设备之间共用的数据结构和接口函数。

### device

抽象系统中所有的硬件设备，描述它的名字、属性、从属的Bus、从属的Class等信息。

### device driver

Linux设备模型用Driver抽象硬件设备的驱动程序，它包含设备初始化、电源管理相关的接口实现。而Linux内核中的驱动开发，基本都围绕该抽象进行（实现所规定的接口函数）

## 系统启动初始化设备模型结构

linux内核启动后,在init进程中对设备模型相关结构进行初始化配置.

初始化流程:
``` C
start_kernel
	|----> rest_init
				|----> kernel_thread(kernel_init, NULL, CLONE_FS | CLONE_SIGHAND);

kernel_init //1号进程
	|---> kernel_init_freeable
				|---> do_basic_setup
							|---> driver_init  //(drivers/base/init.c)
								     |--- > devices_init
								     |--- > buses_init
									 |--- > classes_init
									 |--- > platform_bus_init
```
设备模型核心文件:
>file: drivers/base/ 

### device_init

``` C
int __init devices_init(void)
{
	//创建并注册kset
	//结合sys文件系统, /sys/devices/
    devices_kset = kset_create_and_add("devices", &device_uevent_ops, NULL); //NULL表示不依赖与任何对象,挂载在sys根目录下
	...
	//相当于/sys/dev/
    dev_kobj = kobject_create_and_add("dev", NULL);
	...
	//相当于/sys/dev/block/
    sysfs_dev_block_kobj = kobject_create_and_add("block", dev_kobj);
	//相当于/sys/dev/char/
    sysfs_dev_char_kobj = kobject_create_and_add("char", dev_kobj);
}
```
>file: drivers/base/core.c 

deices主要是对系统硬件的抽象描述,在linux中所有的设备大体可以分为`字符设备`和`块设备`,因此设备模型抽象出`char`和`block`进行相应的描述

### buses_init

``` C
int __init buses_init(void)
{
	//相当于/sys/bus/
    bus_kset = kset_create_and_add("bus", &bus_uevent_ops, NULL);
	...
	//相当于/sys/devices/system/
    system_kset = kset_create_and_add("system", NULL, &devices_kset->kobj);
                                                                                             
    return 0;
}
```
>file: drivers/base/bus.c

bus_kset和system_kset

### class_init

``` C
int __init classes_init(void)
{
	//相当于/sys/class/
    class_kset = kset_create_and_add("class", NULL, NULL);
}
```

### platform_bus_init

``` C
int __init platform_bus_init(void)                                                     
{  
	//注册platform_bus设备, 相当于device
    error = device_register(&platform_bus);
	//注册platform总线
    error =  bus_register(&platform_bus_type);
}
```
>file: drivers/base/platform.c

在内核中platform_bus相当于一个设备,同样也是kset链表上的一个kobject

### 关系

在1号进程中对设备模型进行初始化完成后,创建量4个kset,也就是4条kobject的链表由于管理这三类抽象体(devices, bus, class),主要有deives_kset, bus_kset, system_kset, class_kset.

## 基本数据结构

### struct kset

``` C
struct kset {                                                            
    struct list_head list;    //所有kobject的链表
    spinlock_t list_lock;     //链表同步的锁
    struct kobject kobj;      //代表内嵌的每一个kobject
    const struct kset_uevent_ops *uevent_ops;   
};  
```
>file: include/linux/kobject.h

kset是一组kobject的集合, 而每一个kobject具有不同的`type`, 它主要描述的是对象集合与集合之前的关系,侧重点在于集合.S

### struct kobject

``` C
struct kobject {
    const char      *name;
    struct list_head    entry;			//将kobject结构添加到kset的list_head
    struct kobject      *parent;  		//将kobject以层次的关系组合
    struct kset     *kset;
    struct kobj_type    *ktype;                               
    struct sysfs_dirent *sd;
    struct kref     kref;  		  		//原子操作引用计数
    unsigned int state_initialized:1;	//是否初始化
    unsigned int state_in_sysfs:1;		//是否在sys文件系统中出现
    unsigned int state_add_uevent_sent:1;
    unsigned int state_remove_uevent_sent:1;
    unsigned int uevent_suppress:1;
};
```
>file: include/linux/kobject.h

1. 对象的引用计数
一个内核对象被创建时,不可能知道该对象的存活时间,跟踪此对象生命周期的一个方法是使用引用计数,当内核中没有代码持有该对象的引用时,该对象将结束自己的生命周期,并且被删除.
操作:
``` C
struct kobject *kobject_get(struct kobject *kobj)
void kobject_put(struct kobject *kobj)
```

2. 使用
在内核中很少单独使用kobject对象,kobject通常被用于控制对`大型域`(domain)相关对象的访问.因此,一般都是将kobject`嵌入`到其他的数据结构中.
>这里我们可以通过面向对象的思维去理解这一行为,kobject可以被认为时最顶层的基类,其他类读是它的派生产物.此时,可以将kobject`嵌入`到其他的数据结构中,理解为其他数据结构`继承`了kobject.


### struct kobj_type

``` C
struct kobj_type {                                                                 
    void (*release)(struct kobject *kobj); //release回调函数,用于释放kobject结构                     
    const struct sysfs_ops *sysfs_ops;     //sys文件系统的ops接口      
    struct attribute **default_attrs;      //kobject的attribute列表(相当于sys文件系统中的一个文件)                    
    const struct kobj_ns_type_operations *(*child_ns_type)(struct kobject *kobj);  
    const void *(*namespace)(struct kobject *kobj);                                
};                                                                                 
```
>file: include/linux/kobject.h

kobject的属性操作集合.它描述的重点是对象的类型,侧重点在于单个对象.


### 属性

``` C
struct attribute {                                           
    const char      *name;	//在kobject的sysfs目录中显示
    umode_t         mode;	//应用与属性的保护位
};
```
>file:include/linux/sysfs.h

| mode | 描述 |
| ---- | ---- |
| S_IRUGO | 只读 |
| S_IWUSR | 可写 |
详细描述:include/linux/stat.h 

## 操作

``` C
struct sysfs_ops {                                                                       
    ssize_t (*show)(struct kobject *, struct attribute *,char *); 
    ssize_t (*store)(struct kobject *,struct attribute *,const char *, size_t);
    const void *(*namespace)(struct kobject *, const struct attribute *);
};
```
>file: include/linux/sysfs.h


### 关系与总结

1. Kobject必须时动态申请和动态释放,当kobject维护的引用计数为0时,将自动释放Kobject所占的内存空间.
2. 

## 模型操作

### Add device

在内核中的大部分驱动都依赖与platform总线进行device和driver注册和初始化.

1. 依赖与platform总线的设备采用,`platform_device_register`
2. 直接进行添加时,必须先对device进行初始化.
``` C
	1. device_initialize(&pdev->dev);    //初始化device设备                                    
	
	2. device_add(&dev);
   
```

### Add bus

接口函数:
``` C
int bus_register(struct bus_type *bus)  
```
>在内核中所有的bus都是通过`bus_register`函数进行注册的,它们之间没有从属关系.

### Add class

接口函数:
``` C
#define class_register(class)           \
({                      \
    static struct lock_class_key __key; \
    __class_register(class, &__key);    \        
}) 

int __class_register(struct class *cls, struct lock_class_key *key) 
```

### Add driver

``` C
int driver_register(struct device_driver *drv) 
{
	
    other = driver_find(drv->name, drv->bus);	//查看当前总线上是否已注册该名字设备
   
    ret = bus_add_driver(drv);					//添加该驱动到总线bus
   
    ret = driver_add_groups(drv, drv->groups);	//
  
    kobject_uevent(&drv->p->kobj, KOBJ_ADD);
    return ret;
}
```

## 应用数据结构

### struct device

``` C
struct device {
    struct device       *parent;    //该设备的父设备,可能是一个总线或设备,也可以不存在
	struct device_private   *p;

    struct kobject kobj; 
    const struct device_type *type;

    struct bus_type *bus;       /* type of bus device is on */
	
	struct class        *class;    //The class of the device.
	...

}
```

``` C
struct device_private {                                                  
    struct klist klist_children;
    struct klist_node knode_parent;
    struct klist_node knode_driver;
    struct klist_node knode_bus;
    struct list_head deferred_probe;
    void *driver_data;
    struct device *device;
};   
``` 
>file: include/linux/device.h

### struct device_driver

``` C
struct device_driver {             
    const char      *name;
    struct bus_type     *bus;	 //The bus which the device of this driver belongs to

    struct module       *owner;  //The module owner.  
    const char      *mod_name;  /* used for built-in modules */

    bool suppress_bind_attrs;   /* disables bind/unbind via sysfs 是否允许驱动通过sysfs决定挂载还是卸载设备, 如mmc_test*/

    const struct of_device_id   *of_match_table;
    const struct acpi_device_id *acpi_match_table;

    int (*probe) (struct device *dev);
    int (*remove) (struct device *dev);
    void (*shutdown) (struct device *dev);
    int (*suspend) (struct device *dev, pm_message_t state);
    int (*resume) (struct device *dev);
    const struct attribute_group **groups;
   
    const struct dev_pm_ops *pm;
   
    struct driver_private *p;
};
```

``` C
struct driver_private {
    struct kobject kobj;			//设备模型节点,在sys文件系统中代表driver目录
    struct klist klist_devices;		//驱动的设备链表
    struct klist_node knode_bus;	//挂载在总线上的驱动链表节点
    struct module_kobject *mkobj;	//driver和module的联系
    struct device_driver *driver;	//指回device_driver
};
```
>file: include/linux/device.h

### struct bus_type

``` C
struct bus_type {
    const char      *name;
    const char      *dev_name;
    struct device       *dev_root;
    struct bus_attribute    *bus_attrs;
    struct device_attribute *dev_attrs;
    struct driver_attribute *drv_attrs;

    int (*match)(struct device *dev, struct device_driver *drv);
    int (*uevent)(struct device *dev, struct kobj_uevent_env *env);
    int (*probe)(struct device *dev);
    int (*remove)(struct device *dev);
    void (*shutdown)(struct device *dev);
	...
};
```
>file: include/linux/device.h

### struct class

``` C
struct class {                                                                                         
    const char      *name;
    struct module       *owner;

    struct class_attribute      *class_attrs;  //Default attributes of this class
    struct device_attribute     *dev_attrs;    //Default attributes of the devices belong to the class.
    struct bin_attribute        *dev_bin_attrs;// Default binary attributes of the devices belong to the class.
    struct kobject          *dev_kobj;

    int (*dev_uevent)(struct device *dev, struct kobj_uevent_env *env);
    char *(*devnode)(struct device *dev, umode_t *mode);  //Callback to provide the devtmpfs.

    void (*class_release)(struct class *class);
    void (*dev_release)(struct device *dev);
	...
};
```
>file: include/linux/device.h

### 设备属性

``` C
struct device_attribute {                                                                 
    struct attribute    attr;
    ssize_t (*show)(struct device *dev, struct device_attribute *attr,
            char *buf);
    ssize_t (*store)(struct device *dev, struct device_attribute *attr,
             const char *buf, size_t count);
};
```
这两个接口主要实现了sys文件系统下,对相关属性节点的读写操作,即`echo` 和 `cat`

驱动中的应用接口:
``` C
#define DEVICE_ATTR(_name, _mode, _show, _store) \                                         
    struct device_attribute dev_attr_##_name = __ATTR(_name, _mode, _show, _store)
```

## 设备与驱动

驱动为设备的软件逻辑,在内核中将设备和驱动分别抽象出两个结构体:`struct device`和`struct device_driver`

### 驱动的匹配

在进行驱动的加载时,需要将device和driver进行绑定,绑定成功后驱动才能拿到所需的数据.

``` C
static int __init test_init(void)
{
	int error;

    error = platform_driver_register(&driver);	//注册驱动                            
	...
    platform_device_register(&device);			//注册设备
	...
    return error;
}
```
在设备或驱动最后注册完成时,设备模型将通过platform总线进行二者的匹配,匹配成功后将执行驱动的接口函数`probe`.
``` C
/** 
 * bus_probe_device - probe drivers for a new device
 * @dev: device to probe                                                                    
 *  
 * - Automatically probe for a driver if the bus allows it.
 */ 
void bus_probe_device(struct device *dev)
```

## 热插拔事件

当系统配置出现变化时,从内核空间发送到用户空间的通知,为热插拔事件

>无论kobject被创建还是被删除,都会产生热插拔事件



## 附:

相关API含义

### kset_create_and_add
>创建并注册一个kset
### kobject_create_and_add
>创建并注册一个kobject
### device_register
>注册一个设备
### bus_register
>注册一条总线
### device_add
>添加设备
### bus_add_device
>为一个总线添加设备

