# Linux内核regulator架构和编写
### 电源种类介绍
      （百度百科）LDO是low dropout regulator，意为低压差线性稳压器，是相对于传统的线性稳压器来说的。传统的线性稳压器，如78xx系列的芯片都要求输入电压要比输出电压高出2v~3V以上，否则就不能正常工作。但是在一些情况下，这样的条件显然是太苛刻了，如5v转3.3v,输入与输出的压差只有1.7v，显然是不满足条件的。针对这种情况，才有了LDO类的电源转换芯片。Ldo适合电压要求比较稳,但是功率不是很大的设备。
BUCK电路，降压式变换电路。就是一种DC-DC转换器，简单的讲就是通过震荡电路将一直流电压转变为一高频电源，然后通过脉冲变压器、整流滤波回路输出需要的直流电压，类似于开关电源
### 数据结构
 structregulator_dev {
       structregulator_desc *desc;
       structlist_head list; // regulator通过此结构挂到regulator_list链表中
       structlist_head consumer_list; //此regulator负责供电的设备列表
 
       structregulation_constraints *constraints;
       structregulator *supply;   父regulator的指针
};
regulator_list全局变量 每注册一个regulator都会挂到这里
regulator_map_list全局变量 每注册一个consumer都会挂到这里
### 编写驱动的步骤
概述
       内核里pmu驱动和regulator驱动大多是混合在一起写的，很不好。比如把regulator_init_data放到platform的driver data里传进去。
一般来讲，一款soc会有配套有限数量的pmu。regulator作为pmu的抽象层。供电线路分为pmu直接供电和mtcmos供电。mtcmos也是pmu的一路作为父电源进行供电，但是mtcmos是soc片上的供电线路。比如mtcmos用pmu的一路buck供电，但是usb，sd，pcie分别用了3路mtcmos进行供电。
 	每路供电节点都是一个regulator。如果不考虑mtcmos情况，通常都是一级供电。也就是说，从pmu出来一个regulator下面挂接的就是consumer，而不是多个regulator级联。
      如果出现级联的regulator，可以在regulator_desc的supply_name字段设置上级regulator。
 	推荐的写法是，regulator本身用什么实质内容都没有的platform dev和driver注册，在probe里注册regulator。只有regulator ops才会调用真正的pmu驱动。这样实现了适配层和具体驱动分离的原则。
定义regulator id格式
regulator_desc有一个成员是id。这个id在区分不同的regulator里没有什么作用，因为regulator都是通过name字符串来查找和区分的。id的作用集中表现在ops函数指针数组里。一个PMU芯片通常有多路供电，但是一类供电（如buck）使用同一组ops函数指针数组通常是相同的。但是实际不同的regulator设置的寄存器和方法是有差异的。所以通过id进行区分。通常把id作为offset使用。把寄存器和操作方法放到数组里，这样利用id从数组得到信息，避免了使用大量的if和switch case语句。
struct regulator_desc {
       const char*name;
       const char*supply_name;
       int id;
…
};
       更优化的一些方法可以是，id的有些bit表示pmu或者regulator本身的信息索引，而另外一些bit表示偏移。这样，一个系列的多个PMU芯片可能复用同一套regulator适配层代码。操作信息可以按照 func[pmu_id][offset]二位数组进行。这些使用方法都是灵活的。
获得id的函数
int rdev_get_id(struct regulator_dev *rdev)
{
       returnrdev->desc->id;
}
step1 准备ops
         虽然arm soc pmu通常多达几十路供电，但是ops数量不多，一般一类设备同用一组ops函数，常见的类别有buck ops和ldo ops。ops函数的入参是(struct regulator_dev *)，所以可以根据regulator_dev得到具体的id信息，进行不同的寄存器操作。
int offset = rdev_get_id(rdev);
理论上讲，一个pmu所有的regulator可以用同一组ops，然后用id区分操作。另一个极端是每个regulator用一组不同的ops。不过为了代码复用和解耦合，通常是一类regulator用一组ops。
 例子：
 static structregulator_ops max8660_ldo5_ops = {
        .list_voltage = max8660_ldo5_list,
        .set_voltage = max8660_ldo5_set,
        .get_voltage = max8660_ldo5_get,
};
step2 准备consumer
代码例子：
static struct regulator_consumer_supplylp3974_buck3_consumer[] = {
   REGULATOR_SUPPLY("vdet", "s5p-sdo"),
   REGULATOR_SUPPLY("vdd_reg", "0-003c"),
};
consumer指的是regulator树上的叶子节点。regulator_consumer_supply数组作为为regulator_init_data的成员。
      consumer的名字是需要提供给各模块使用的，如上面代码的“vdet”。所以如果是系列soc的代码，最好这个名字不能带有pmu芯片型号或者soc型号，因为同样的模块，如usb控制器，很可能系列soc上的都一样，驱动也是复用的。不可能每换个soc就全盘适配regulator。所以这种情况下最好命名为”myregulator-usb”之类。这样换了pmu或者soc，只要用regulator_get(dev, ”myregulator-usb”)总能适应各种情况，就不需要修改驱动。
 step3 准备regulator_desc
注意regulator_register的时候，regulator_desc和regulator_init_data是一一匹配的。所以regulator_desc数组和regulator_init_data数组的顺序要保持一致。
 static structregulator_desc regulators[] = {
       {
              .name         = "LDO2",
              .id         = MAX8998_LDO2,
              .ops             = &max8998_ldo_ops,
              .type            =REGULATOR_VOLTAGE,
              .owner        = THIS_MODULE,
       }, {
              .name         = "LDO3",
              .id         = MAX8998_LDO3,
              .ops             = &max8998_ldo_ops,
              .type            = REGULATOR_VOLTAGE,
              .owner        = THIS_MODULE,
       }, {
 另外还有一种写法，可以用用定义的枚举作为index，按顺序填写数组。只要regulator_desc数组和regulator_init_data数组都用同样的枚举做index，那肯定是一一匹配的。
 static structregulator_desc regulators[] = {
       [REG0] = {
              .name         = "LDO2",
              .id         = MAX8998_LDO2,
              .ops             = &max8998_ldo_ops,
              .type            = REGULATOR_VOLTAGE,
              .owner        = THIS_MODULE,
       },
[REG1] = {
              .name         = "LDO3",
              .id         = MAX8998_LDO3,
              .ops             = &max8998_ldo_ops,
              .type            = REGULATOR_VOLTAGE,
              .owner        = THIS_MODULE,
       }, {
 
step4  准备regulator_init_data
有的驱动程序将 regulator_init_data放入platform_data，不推荐。Regulator应该和pmu的驱动程序分开。只需要在ops里调用pmu的操作函数就可以。
regulator_init_data应该也是结构体数组，和regulator_desc结构体数组一一匹配。
  注意constraints的name字段在查找regulator的时候，有比regulator_desc更高的优先级。不推荐在constraints写name。
其中的一段如下。
static struct regulator_init_data lp3974_buck3_data ={
   .constraints  = {
      .name      = "VCC_1.8V",
      .min_uV       = 1800000,
      .max_uV       = 1800000,
      .apply_uV  = 1,
      .always_on = 1,
      .state_mem = {
          .enabled   = 1,
       },
    },
   .num_consumer_supplies = ARRAY_SIZE(lp3974_buck3_consumer),
   .consumer_supplies = lp3974_buck3_consumer,
};
step5  准备注册regulator
最简单的platform注册就可以，只需要在probe里把regulator都注册上。
static int my_regulator_probe (struct platform_device*pdev)
{
       int i；
       for(i = 0;i < MAX ; i ++)
{
regulator_register(&regulator_desc[i], &pdev->dev,regulator_init_data[i], NULL, NULL);
}
}
 
static struct platform_device my_regulator_dev = {
       .name =“my_regulator”,
};
static struct platform_driver my_regulator_driver = {
       .driver ={
              .name= " my_regulator ",
              .owner= THIS_MODULE,
       },
       .probe =my_regulator_probe,
       .remove =__devexit_p(my_regulator__remove),
};
在module_init里注册device和driver就可以。
platform_device_register(&my_regulator_dev)
platform_driver_register(&my_regulator_driver);
regulator使用
总体来说就是先调用regulator_get供电节点的名字，得到regulator指针，然后调用ops函数集。举例如下：
regulator = regulator_get(dev, “my_usb”)；
就会返回usb名字供电的regulator。
打开和关闭校准器（regulator）API如下。
int regulator_enable(regulator);
int regulator_disable(regulator);
还有其他一些函数，参见struct regulator_ops。常见的有
struct regulator_ops {
       /* get/setregulator voltage */
       int(*set_voltage) (struct regulator_dev *, int min_uV, int max_uV,
                         unsigned *selector);
       int(*get_voltage) (struct regulator_dev *);
 
       /* get/setregulator current  */
       int(*set_current_limit) (struct regulator_dev *,
                             int min_uA, int max_uA);
       int(*get_current_limit) (struct regulator_dev *);
       /*enable/disable regulator */
       int(*enable) (struct regulator_dev *);
       int(*disable) (struct regulator_dev *);
       int(*is_enabled) (struct regulator_dev *);
 
};
内核代码分析
注册regulator，包括设置supply和consumer
/**
 *regulator_register - register regulator
 *@regulator_desc: 就是regulator_desc
 * @dev:
 * @init_data: 就是regulator_init_data
 * @driver_data:用户私有信息，不推荐，设置为NULL就可以
 * @of_node:device tree可以设置regulator的树状结构，先不考虑).
 */
struct regulator_dev *regulator_register(structregulator_desc *regulator_desc,
       structdevice *dev, const struct regulator_init_data *init_data,
       void*driver_data, struct device_node *of_node)
{
 
// 编写驱动只要提供regulator_desc和regulator_init_data就可以，分配regulator_dev结构在这里
       rdev =kzalloc(sizeof(struct regulator_dev), GFP_KERNEL);
 
//初始化regulator_dev结构
 
// 设置constraints
       ret =set_machine_constraints(rdev, constraints);
 
// 如果此regulator有父regulator，设置父regulator. 优先选init_data
       if(init_data && init_data->supply_regulator)
              supply= init_data->supply_regulator;
       else if(regulator_desc->supply_name)
              supply= regulator_desc->supply_name;
 
       if(supply) {
              //用name字符串在regulator_list查找， 如果找到，就把本regulator加入到上级的consumer_list
              r =regulator_dev_lookup(dev, supply);
              ret= set_supply(rdev, r);
              ...
       }
 
       /*
       首先检查regulator_map_list全局变量里有没有冲突或者重复的。如果没有为consumer申请struct regulator_map，
       然后设置regulator_map的regulator设置为本regulator，同时用regulator_map记录父子信息，
       并且把regulator_map加入到regulator_map_list全局变量里。
        */
       if(init_data) {
              for(i = 0; i < init_data->num_consumer_supplies; i++) {
                     ret= set_consumer_device_supply(rdev,
                            init_data->consumer_supplies[i].dev_name,
                            init_data->consumer_supplies[i].supply);
              }
       }
 
       //将本regulator加入到全局regulator_list
       list_add(&rdev->list,&regulator_list);
}
 
regulator_desc和regulator_init_data的name重复字段
Regulator的name在struct regulator_init_data的constraints->name字段可以定义，也可以在regulator_desc的name字段定义。但是constraints->name优先级更高。
 
regulator_register-》regulator_dev_lookup-》rdev_get_name里代码如下：
 
static const char *rdev_get_name(struct regulator_dev*rdev)
{
    if(rdev->constraints && rdev->constraints->name)
       returnrdev->constraints->name;
    else if(rdev->desc->name)
       returnrdev->desc->name;
    else
       return"";
}
 
static LIST_HEAD(regulator_list);     //所有regulator都注册在这个链表里
上级电源supply重复字段
除了name，regulator_desc->supply_name和regulator_init_data->supply_regulator也是重复的，都是指向上级regulator的指针，regulator_init_data优先级更高，可以在regulator_register里看到代码如下
      if (init_data &&init_data->supply_regulator)
              supply= init_data->supply_regulator;
       else if(regulator_desc->supply_name)
              supply= regulator_desc->supply_name;
 
 

[回到页首](index.md)