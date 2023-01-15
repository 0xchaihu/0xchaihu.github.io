---
title: 剥洋葱：从RT-Thread的串口到内核对象
date: 2018-01-04
tags:
---
　　*写在前面：如何阅读和学习一个较大的工程，每个人都有不同的方法。我倾向于首先“不求甚解”地使用API，将整个工程跑起来，对该工程建立一个整体的认识，然后在使用中，从最感兴趣的模块入手，从上层开始，像剥洋葱一样，逐层分析。本文就是按照这样的方法，为研究一个小功能，从RT-Thread 中bsp的串口开始，最终深入到RT-Thread的内核，初步探究内核中的基本元素——内核对象。由于作者手中只有stm32f103的开发板，所以本文在涉及到bsp的代码部分，都是指RT-Thread的github仓库中最新的bsp/stm32f10x内的代码。*

　　使用RT-Thread的第一步就是通过ENV工具和scons来构建一个工程。我们可以看到在rtconfig.h中，RT-Thread通过一个宏`RT_CONSOLE_DEVICE_NAME`（在stm32f10x内，其默认值为`“uart1”`），就可以完成console所使用串口的设置。这个宏本质就是一个字符串，也就是串口的名字，将这个宏修改为`“uart1”`，console就会使用串口1。RT-Thread是如何通过串口的名字就可以实际在硬件上控制该串口了呢？

　　在stm32f10x的bsp中，main函数内的`rtthread_startup()`将会完成RT-Thread的初始化。根据代码，整理出与我们此次研究有关的代码层次结构图，如下图所示，在该图中，下级表示被上级调用的子函数，同级之间表示并列关系，即同级的函数都是被上级函数所调用的子函数。
　　![main （startup.c）](\img\main （startup.c）.png)

　　根据上图，我们可以推断出在`rt_hw_usart_init`中，就完成了字符串（即串口名称）与硬件串口的绑定，所以在接下来调用`rt_console_set_device`时，就可以直接通过`RT_CONSOLE_DEVICE_NAME`使用该串口。

　　下面我们先分析`rt_hw_usart_init`，其中核心代码的调用结构如下图所示：
　　![rt_hw_usart_init](\img\rt_hw_usart_init.png)

　　串口设备数据结构如下图所示。
　　![struct rt_serial_device](\img\struct rt_serial_device.png)

　　从调用层次和串口设备的数据结构中可以发现，RT-Thread将串口封装成一个结构体，其名字（`char *name`，在stm32f10x中，uart1的名字为`“uart1”`）最终赋值给结构体子成员的`rt_object parent`的name数组中。

　　根据上述分析，当`rt_hw_usart_init`运行完毕后，串口设备就被注册至内核了。实际上，只是串口设备的“孙”成员（子成员的子成员）`rt_object parent`，被注册到了内核中。而所谓注册到内核，就是指内核将其地址存入一个链表中。为什么只需要注册其中一个成员呢？这里运用了一个C语言的小技巧，即结构体首个成员的地址就是该结构体的地址，所以当我们获取到了结构体首个成员的地址时，也就相当于我们获取到了该结构体的地址。

　　下面我们分析`rt_console_set_device`，其核心代码调用结构如下图所示：
　　![rt_console_set_device](\img\rt_console_set_device.png)
　　
　　其代码如下所示：
```
rt_device_t rt_console_set_device(const char *name)
{
    rt_device_t new, old;

    /* save old device */
    old = _console_device;
    
    /* find new console device */
    new = rt_device_find(name);
    if (new != RT_NULL)
    {
        if (_console_device != RT_NULL)
        {
            /* close old console device */
            rt_device_close(_console_device);
        }
    
        /* set new console device */
        rt_device_open(new, RT_DEVICE_OFLAG_RDWR | RT_DEVICE_FLAG_STREAM);
        _console_device = new;
    }
    
    return old;
}
```

　　与我们之前的分析一致。`rt_console_set_device`中维护了一个全局变量`_console_device`，保存console所使用的串口设备句柄（即串口设备结构体指针）。通过`rt_device_find`遍历内核中相应的链表，匹配与传入的字符串名称(`char *name`，在stm32f10x中，默认传入为`“uart1”`)一致的对象，即可获取到相应的句柄，将其赋值给`_console_device`，然后console模块就可以通过`_console_device`控制和使用该串口了。
　　
　　至此，RT-Thread通过名字就可以控制实际的硬件设备的原理已分析完毕。可以看出，RT-Thread通过分层和对象化思想，将串口封装成了`串口`——》`rt_device`——》`rt_object`的关系，在RT-Thread内核的其他模块中，我们也能大量的发现`rt_object`和`rt_device`的身影，这样的封装形式极大的提高了代码的复用率，在RT-Thread中，其派生关系如下图所示。本文分析的串口设备就是一种字符设备。
　　![派生关系](\img\派生关系.png)

　　*后记：C语言的对象化和分层思想并不是为了“炫技”和模仿C++，而是为了用最少的代码干最多的事儿，并且减少耦合，更利于大规模的多人开发。Linux、RT-Thread以及其他大的C语言项目中，都运用了这种思想。不论使用什么语言做程序设计，这种思想都是开发大工程的一种最优解（当然这句话在目前“函数式编程”蓬勃发展的情况下显得比较主观）。在阅读优秀的代码时，相比于读懂具体的代码实现，我认为弄清楚为什么要这样实现反而更重要。*

---
<font size="1">本文已被RT-Thread官方公众号“RTThread物联网操作系统”审核通过并发布。</font> 