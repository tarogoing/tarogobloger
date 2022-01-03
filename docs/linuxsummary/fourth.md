# linux驱动程序之电源管理之regulator机制流程-(1)

下面通过下面三个过程分析regulartor供电机制：
1. 分析regulator结构体
2. regulator 注册过程
3. 设备使用regulator过程
 
### 一.分析regulator结构体
Regulator模块用于控制系统中某些设备的电压/电流供应。在嵌入式系统（尤其是手机）中，控制耗电量很重要，直接影响到电池的续航时间。所以，如果系统中某一个模块暂时不需要使用，就可以通过regulator关闭其电源供应；或者降低提供给该模块的电压、电流大小。

```
Regulator的文档在KERNEL/Documentation/Power/Regulator中。
```
 

Regulator与模块之间是树状关系。父regulator可以给设备供电，也可以给子regulator供电：

```
父Regulator
--> 子Regulator  --> [supply]
--> 设备[Consumer]
具体细节可参考内核文档machine.txt。
 
regulator_dev
regulator_dev代表一个regulator设备。
struct regulator_dev {
struct regulator_desc *desc; // 描述符，包括regulator的名称、ID、regulator_ops等
int use_count; // 使用计数
 
struct list_head list; // regulator通过此结构挂到regulator_list链表中
struct list_head slist; // 如果有父regulator，通过此域挂到父regulator的链表
 
struct list_head consumer_list; // 此regulator负责供电的设备列表
struct list_head supply_list; //此regulator负责供电的子regulator
struct blocking_notifier_head notifier; // notifier，具体的值在consumer.h中，比如REGULATOR_EVENT_FAIL
struct mutex mutex;
struct module *owner;
struct device dev; // device结构，属于class regulator_class
struct regulation_constraints *constraints; // 限制，比如最大电压/电流、最小电压/电流
struct regulator_dev *supply; // 父regulator的指针，即由此regulator  供电
void *reg_data;
};
 
regulator_init_data
 
regulator_init_data在初始化时使用，用来建立父子regulator、受电模块之间的树状结构，以及一些regulator的基本参数。
struct regulator_init_data {
struct device *supply_regulator_dev; // 父regulator的指针
struct regulation_constraints constraints;
int num_consumer_supplies;
struct regulator_consumer_supply *consumer_supplies; // 负责供电的设备数组
 
int (*regulator_init)(void *driver_data); // 初始化函数，在regulator_register被调用
void *driver_data;
};
 
```

其它结构体自己可以看看～如

```
struct regulator　　　            -------> 设备驱动直接操作的结构体
struct regulation_constraints　　　　　  ----->regulator限制范围，其它信息，在于
　　　　　　　　　　　　　　　　　　　　　　　struct regulator_init_data，用于初始化
struct regulator_consumer_supply　　　----->consumer信息
struct regulator_desc                           ----->这个多关注些，内有正真操作设备函数结构体～
struct regulator_map                           ----->这个为consumers与regulator对应表
```

___

### 二.　注册regulator过程
先说明下两具在regulator的core中有两个关键的全局变量链表：
regulator_list		 每注册一个regulator都会挂到这里
regulator_map_list　　每一个regulator都会为多个consumer供电，此表为挂consumer
 
regulator注册过程是通过platform平台注册，当然一个电源管理芯片可以供几十个设备供电，所以不可能每个regulator一个驱动文件，它们是通过，在电源管理芯片驱动中一块注册到regulato的core中。
对于电源管理芯片驱动的注册则通过Ｉ２Ｃ注册的。接下来以中星微方案过下，
首先，在平台设备文件中，有关struct regulator_init_data XX定义～
如：

```

static struct regulator_init_data va7882_ldo13_data = {
.constraints = {
.name = "LDO13-HDMI1V2", //Default: 1.5V , Powered By DCDC5, C-class
.min_uV = 1200000,
.max_uV = 1800000,
.apply_uV = 1,
// TEMP_ON
.valid_ops_mask = REGULATOR_CHANGE_VOLTAGE | REGULATOR_CHANGE_STATUS | REGULATOR_CHANGE_MODE,
.initial_mode = REGULATOR_MODE_NORMAL,
.valid_modes_mask = REGULATOR_MODE_NORMAL | REGULATOR_MODE_STANDBY,
},
.supply_regulator = supply_regulator_name_arrary[ID_VA7882_LDO13],
.num_consumer_supplies = 2,
.consumer_supplies = (struct regulator_consumer_supply []) {
{ .supply = regulator_name_arrary[ID_VA7882_LDO13][0] },
{ .supply = regulator_name_arrary[ID_VA7882_LDO13][1] },
{ .supply = regulator_name_arrary[ID_VA7882_LDO13][2] },
},
};

```

其实这些结构体又会被同一文件中的自定义文件初始化函数 v8_va7882_init调用，其实就是
va7882_register_regulator---->为每个regulator分配对应的struct platform_device---->再platform_device_add　
那platform_driver在哪？

```
static struct platform_driver va7882_regulator_driver = {
.driver = {
.name = "va7882-regulator",
.owner = THIS_MODULE,
},
.probe = va7882_regulator_probe,
.remove = __exit_p(va7882_regulator_remove),
.suspend = va7882_regulator_suspend,　　　　//可见休眠唤醒是使用同一个,
.resume = va7882_regulator_resume,　　　　  //再次说明它们是一统管理，它们也是为了节省.
.shutdown = va7882_regulator_shutdown,
};

```

这个platform_driver是每个regulator共用的的，因为name都是" va7882-regulator"。
在这个platform_driver中probe中就干了一件事regulator_register，从而获得regulator_dev。
 
接下来分析下regulator_register注册
Regulator的注册由regulator_register完成。
一般来说，为了添加regulator_dev，需要实现一个设备驱动程序，以及在板子的设备列表中增加一个该驱动对应的设备（比如platform_device）。
在这个设备的struct device->platform_data域，需要设置regulator_init结构，填写该regulator的相关信息。
另外，还需要定义一个regulator_desc结构。这样，在这个物理设备的驱动程序中，就可以通过regulator_register函数登记生成一个regulator_dev。

```
struct regulator_dev *regulator_register(struct regulator_desc *regulator_desc, struct device *dev, void *driver_data)
struct regulator_init_data *init_data = dev->platform_data;// 得到init_data
// 完整性检查
…
// 分配regulator_dev结构
struct regulator_dev *rdev = kzalloc (sizeof(struct regulator_dev), GFP_KERNEL);
// 初始化regulator_dev结构
…
// 执行regulator_init，该函数中实现regulator代表的硬件设备的初始化
if (init_data->regulator_init)
ret = init_data->regulator_init(rdev->reg_data);
rdev->dev.class = &regulator_class; // 指定class为regulator_class
rdev->dev.parent = dev;
device_register(&rdev->dev); // 注册设备
// 设置constraints，其中可能会包括供电状态的初始化（设置初始电压，enable/disable等等）
set_machine_constraints(rdev, &init_data->constraints);
add_regulator_attributes (rdev);
// 如果此regulator有父regulator，设置父regulator
if (init_data->supply_regulator_dev) {
ret = set_supply(rdev,
dev_get_drvdata(init_data->supply_regulator_dev));
if (ret < 0)
goto scrub;
}
// 设置此regulator与其负责供电的设备之间的联系
for (i = 0; i < init_data->num_consumer_supplies; i++)
ret = set_consumer_device_supply(rdev, init_data->consumer_supplies[i].dev,
init_data->consumer_supplies[i].supply);
// 将regulator加入一个链表，该链表包含所有regulator
list_add(&rdev->list, &regulator_list);
```

___


那个regulator 根据在regulator_list中用init_data->supply_regulator来匹配
匹配成功用set_supply()来设置注册的regulator是由那个regulator供电，rdev->supply = supply_rdev; list_add(&rdev->slist, &supply_rdev->supply_list);
多个consumer用set_consumer_device_supply()，先检查
list_for_each_entry(node, &regulator_map_list, list) 后添加
list_add(&node->list, &regulator_map_list);当然node已经在
　　　　　　　　　node->regulator = rdev;
　　　　　　　　　node->supply = supply;
形成 对于每一个regulator_dev—comsumer_dev的配对
最后在把regulator通过list_add(&rdev->list, &regulator_list);加到 regulator_list链表中。
 
### 三.设备使用regulator过程
 
　在设备驱动使用regulator对其驱动的设备供电时，首先要确保设备与对应regulator之间的匹配关系已经被登记到regulator框架中。
之后，设备驱动通过regulator_get函数得到regulator结构，此函数通过前文所述regulator_map_list找到对应regulator_dev，再生成regulator结构给用户使用。
通过regulator_enable / regulator_disable打开、关闭regulator，这两个函数最终都是调用struct regulator_ops里的对应成员。
除此之外，还有regualtor_set_voltage / regulator_get_voltage等等。
Regulator能够支持的所有功能列表都在struct regulator_ops中定义，具体可参考代码中的注释。

```
struct regulator_ops {
 
int (*set_voltage) (struct regulator_dev *, int min_uV, int max_uV);
int (*get_voltage) (struct regulator_dev *);
 
int (*set_current_limit) (struct regulator_dev *,
int min_uA, int max_uA);
int (*get_current_limit) (struct regulator_dev *);
 
int (*enable) (struct regulator_dev *);
int (*disable) (struct regulator_dev *);
int (*is_enabled) (struct regulator_dev *);
 
int (*set_mode) (struct regulator_dev *, unsigned int mode);
unsigned int (*get_mode) (struct regulator_dev *);
 
unsigned int (*get_optimum_mode) (struct regulator_dev *, int input_uV,
int output_uV, int load_uA);
 
 
int (*set_suspend_voltage) (struct regulator_dev *, int uV);
 
int (*set_suspend_enable) (struct regulator_dev *);
int (*set_suspend_disable) (struct regulator_dev *);
 
int (*set_suspend_mode) (struct regulator_dev *, unsigned int mode);
};
 
```

接下来我就以中星型的ＨＤＭＩ驱动使用regulator过一遍。
vc088x_hdmi.c文件：
ＨＤＭＩ驱动也是通过platform平台注册上去，所以在platform_driver的probe中有这个句

```
ret = v8hdmi_pwr_get(&pdev->dev);
regulator_get(dev,id);
_regulator_get(dev, id, 0);
{
…........
…...........
if (dev)
devname = dev_name(dev);
list_for_each_entry(map, &regulator_map_list, list) {
 
if (map->dev_name &&(!devname || strcmp(map->dev_name, devname)))
continue;
if (strcmp(map->supply, id) == 0) {//查找配对
rdev = map->regulator;
goto found;
}
}
```
___

//这个用于在sys目录下建立对应的regulator文件，用于用户空间操作

```
regulator = create_regulator(rdev, dev, id); //regulator -->struct regulator
….............................
return regulator;
}
 
```

返回的regulator会赋给全局变量，如 hdmi_core_consumer = regulator,//这只是例子，不同方案处理不一样。
在恰当的时候使能它，如

```
ret = v8hdmi_pwr_enable();
regulator_enable(hdmi_io_consumer);
struct regulator_dev *rdev = regulator->rdev;
ret = _regulator_enable(rdev);
….................
if (rdev->use_count == 0) {
 
if (rdev->supply) {
mutex_lock(&rdev->supply->mutex);
ret = _regulator_enable(rdev->supply);//使能父regulator
mutex_unlock(&rdev->supply->mutex);
If (ret < 0) {
rdev_err(rdev, "failed to enable: %d\n", ret);
return ret;
}
}
}
…...............................
ret = rdev->desc->ops->enable(rdev);//调用真正使能操作.
….......................

```

使能函数到此结束.
 
总的来看，使用也是通过regulator_get()-->regulator_enable()就可以了
想关时，regulator_disable()--->regulator_put(),反操作～
 
 
其实我疑惑是真正操作电源管理芯片那些操作，存放在struct regulator_ops 结构体内
而这个结构体包含在于struct regulator_desc内，这个结构体，在执行注册regulator时，使用到，被赋到regulator_dev-->desc中～
对于struct regulator_ops中的操作方法，就涉及到电源芯片驱动，下面是va7882的操作方法

```
static struct regulator_ops va7882_dcdc_ops = {
.set_voltage = va7882_dcdc_set_voltage,
.get_voltage = va7882_dcdc_get_voltage,
.list_voltage = va7882_dcdc_list_voltage,
.enable = va7882_dcdc_enable,
.disable = va7882_dcdc_disable,
.is_enabled = va7882_dcdc_is_enabled,
.get_mode = va7882_dcdc_get_mode,
.set_mode = va7882_dcdc_set_mode,
.get_optimum_mode = va7882_dcdc_get_optimum_mode,
.set_suspend_voltage = va7882_dcdc_set_suspend_voltage,
.set_suspend_enable = va7882_dcdc_set_suspend_enable,
.set_suspend_disable = va7882_dcdc_set_suspend_disable,
.set_suspend_mode = va7882_dcdc_set_suspend_mode,
.enable_time = va7882_enable_time,
};
```

[回到页首](../index.md)