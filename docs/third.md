# linux驱动程序之电源管理 之linux休眠与唤醒-(2)

在Linux中，休眠主要分三个主要的步骤：
1. 冻结用户态进程和内核态任务；
1. 调用注册的设备的suspend的回调函数；
1. 按照注册顺序休眠核心设备和使CPU进入休眠态。
       冻结进程是内核把进程列表中所有的进程的状态都设置为停止，并且保存下所有进程的上下文。当这些进程被解冻的时候，他们是不知道自己被冻结过的，只是简单的继续执行。如何让Linux进入休眠呢？用户可以通过读写sys文件/sys /power/state 是实现控制系统进入休眠。
       比如： 
 ```
 # echo standby > /sys/power/state
 ```

命令系统进入休眠。也可以使用
```
 # cat /sys/power/state来得到内核支持哪几种休眠方式。
 ```

Linux Suspend 的流程。
相关的文件的路径: 
linux_soruce/kernel/power/main.c 
linux_source/kernel/arch/xxx/mach-xxx/pm.c
linux_source/driver/base/power/main.c 

(1）接下来让我们详细的看一下Linux是怎么休眠/唤醒的。
      用户对于/sys/power/state 的读写会调用到 main.c中的state_store()，用户可以写入 const char * const pm_state[] 中定义的字符串，比如"mem"、 "standby"。然后state_store()会调用enter_state()，它首先会检查一些状态参数，然后同步文件系统。 （2）准备冻结进程。
      当进入到suspend_prepare()中以后，它会给suspend分配一个虚拟终端来输出信息，然后广播一个系统要进入suspend的Notify，关闭掉用户态的helper进程，然后一次调用suspend_freeze_processes()冻结所有的进程，这里会保存所有进程 当前的状态，也许有一些进程会拒绝进入冻结状态，当有这样的进程存在的时候，会导致冻结失败，此函数就会放弃冻结进程，并且解冻刚才冻结的所有进程。
（3）让外设进入休眠。
      现在，所有的进程(也包括workqueue/kthread) 都已经停止了，内核态任务有可能在停止的时候握有一些信号量，所以如果这时候在外设里面去解锁这个信号量有可能会发生死锁，所以在外设的suspend()函数里面作lock/unlock锁要非常小心，这里建议设计的时候就不要在suspend()里面等待锁。
      最后会调用suspend_devices_and_enter()来把所有的外设休眠，在这个函数中，如果平台注册了suspend_pos(通常是在板级定义中定义和注册)，这里就会调用suspend_ops->begin()，
      然后driver/base/power/main.c 中的 device_suspend()->dpm_suspend() 会被调用，他们会依次调用驱动的suspend() 回调来休眠掉所有的设备。当所有的设备休眠以后，suspend_ops->prepare()会被调用，这个函数通常会作一些准备工作来让板机进入休眠。
      接下来Linux，在多核的CPU中的非启动CPU会被关掉，通过注释看到是避免这些其他的CPU造成race condion，接下来的以后只有一个CPU在运行了。       suspend_ops 是板级的电源管理操作，通常注册在文件 arch/xxx/mach-xxx/pm.c 中。接下来，suspend_enter()会被调用，这个函数会关闭arch irq，调用 device_power_down()，它会调用suspend_late()函数，这个函数是系统真正进入休眠最后调用的函数，通常会在这个函数中作最后的检查。如果检查没问题，接下来休眠所有的系统设备和总线，并且调用 suspend_pos->enter()  来使CPU进入省电状态。这时候，就已经休眠了，代码的执行也就停在这里了。
（4）Resume。
      如果在休眠中系统被中断或者其他事件唤醒，接下来的代码就会开始执行，这个唤醒的顺序是和休眠的顺序相反的，所以系统设备和总线会首先唤醒，使能系统中断，使能休眠时候停止掉的非启动CPU，以及调用suspend_ops->finish()，而且在suspend_devices_and_enter()函数中也会继续唤醒每个设备，使能虚拟终端。最后调用 suspend_ops->end()。再返回到enter_state()函数中的，当suspend_devices_and_enter()  返回以后，外设已经唤醒了，但是进程和任务都还是冻结状态，这里会调用suspend_finish()来解冻这些进程和任务，而且发出Notify来表示系统已经从suspend状态退出，唤醒终端。到这里，所有的休眠和唤醒就已经完毕了，系统继续运行了。
      
[回到页首](index.md)