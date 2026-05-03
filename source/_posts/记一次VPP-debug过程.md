---
title: 记一次VPP debug过程
date: 2020-02-23 09:26:09
tags:
- 技术日志
- C/C++
- VPP
categories:
- VPP
---

**关键词：** GDB调试  VPP  AVF

## 1. 前言

年初的时候接了一个任务，研究一下VPP在SMP服务器架构下创建AVF接口出现错误的bug。当时刚刚完成CSAPP的第三个attack lab的实验，对gdb的使用也更为熟练。本文就是对整个debug过程的整理和记录。

<!-- more -->

## 2. 背景知识

### 2.1 服务器架构

从系统架构来看，市面上的商用服务器基本可以分为三类：

- SMP（Symmetric Multi-Processor）对称多处理器结构
- NUMA（Non-Uniform Memory Access）非一致存储访问结构
- MPP（Massive Parallel Processing）海量并行处理结构

**SMP**结构是指服务器中多个CPU对称工作，各个CPU之间的关系平等无差别。所有的CPU共享全部资源，包括总线、内存和I/O系统等。**SMP**结构的优点是共享，缺点也显而易见，其扩展能力非常有限。每一个共享的环节都有可能造成瓶颈。试想一下，当**SMP**结构中的多个CPU访问同一块内存时，内存访问就会发生冲突，CPU的资源发生浪费。为了解决上述问题，**NUMA**结构应运而生。**NUMA**结构最基本的特征就是由多个CPU模块构成，每个CPU模块都含有多个CPU，并具有本地独立的缓存、I/O槽等。各个CPU模块之间可通过互联模块（Crossbar Switch）来进行信息交互，所以每个CPU都可以访问到整个系统的内存，但是访问模块内的本地内存的速度要远远高于访问模块外的远地内存，这也是**NUMA**为什么叫做非一致存储的缘由。相较于**NUMA**，**MPP**采取了另一种系统扩展的方式。本质上，**MPP**结构相当于把多个**SMP**服务器组合在一起，将每个**SMP**服务器视作一个节点，每个**SMP**节点之间可以通过节点互联网络进行信息交互，但是每个**SMP**节点内的CPU不能够直接访问另一个**SMP**节点的资源，也就是不能够进行异地内存访问。

### 2.2 VPP AVF插件

VPP里的AVF插件给intel 710系列网卡提供自适应的Virtual Function（VF），其设计的初衷是给虚拟机提供通用的VF。AVF（Adaptive Virtual Function），从其英文名字就可以理解，其功能即意味着不需要随着网卡的更新而更新VF的驱动，这样子就可以让虚机无需更新软硬件的情况下跑在新的网卡上。VPP AVF插件的使用方法在[链接](https://docs.fd.io/vpp/18.04/avf_plugin_doc.html)中有具体说明。

## 3. 调试过程

### 3.1 系统环境

出现上述bug的机器对应的是Qualcomm Centriq 2400，采用的是ARMv8-A微架构的CPU。

```
snowball@net-arm-c2400-02:~$ lscpu
Architecture:        aarch64
Byte Order:          Little Endian
CPU(s):              46
On-line CPU(s) list: 0-45
Thread(s) per core:  1
Core(s) per socket:  46
Socket(s):           1
NUMA node(s):        1
Vendor ID:           Qualcomm
Model:               1
Model name:          Falkor
Stepping:            0x0
CPU max MHz:         2600.0000
CPU min MHz:         600.0000
BogoMIPS:            40.00
L1d cache:           32K
L1i cache:           64K
L2 cache:            512K
L3 cache:            58880K
NUMA node0 CPU(s):   0-45
Flags:               fp asimd evtstrm aes pmull sha1 sha2 crc32 cpuid asimdrdm
snowball@net-arm-c2400-02:~$ sudo numactl -H
[sudo] password for snowball:
available: 1 nodes (0)
node 0 cpus: 0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45
node 0 size: 97946 MB
node 0 free: 59005 MB
node distances:
node   0
  0:  10
```

### 3.2 bug复现

按照2.2节中给出的链接完成VF的创建，用gdb起VPP debug镜像来创建VF接口，命令如下：

```
sudo gdb ./build-root/build-vpp_debug-native/vpp/bin/vpp
```

在gdb命令行中敲入以下命令起VPP CLI：

```
run -c ~/startup_avf.conf
```

`startup_avf.conf`是VPP的配置文件，使用AVF接口的话就要把原始的`startup.conf`文件中的`dpdk`插件注释掉。

在VPP DBGvpp命令行中创建VPP AVF接口，命令如下：

```
DBGvpp# create int avf 0000:01:02.0
```

上述命令的结果如下，bug复现。

```
0: /home/snowball/tasks/avf_plugin_test/vpp/src/vlib/buffer_funcs.h:165 (vlib_buffer_pool_get_default_for_numa) assertion `numa_node < VLIB_BUFFER_MAX_NUMA_NODES' fails

Thread 1 "vpp_main" received signal SIGABRT, Aborted.
```

### 3.3 调试bug

首先我用gdb中的`backtrace`命令来回溯函数调用栈，结果如下所示：

```
(gdb) bt
#0  __GI_raise (sig=sig@entry=6) at ../sysdeps/unix/sysv/linux/raise.c:51
#1  0x0000fffff6cc38b4 in __GI_abort () at abort.c:79
#2  0x0000aaaaaaaa8570 in os_panic () at /home/snowball/tasks/avf_plugin_test/vpp/src/vpp/vnet/main.c:366
#3  0x0000fffff6e1d9b4 in debugger () at /home/snowball/tasks/avf_plugin_test/vpp/src/vppinfra/error.c:84
#4  0x0000fffff6e1dd60 in _clib_error (how_to_die=2, function_name=0x0, line_number=0,
    fmt=0xffffb6814f88 "%s:%d (%s) assertion `%s' fails") at /home/snowball/tasks/avf_plugin_test/vpp/src/vppinfra/error.c:143
#5  0x0000ffffb67fd7cc in vlib_buffer_pool_get_default_for_numa (vm=0xfffff6fcd000 <vlib_global_main>, numa_node=4294967295)
    at /home/snowball/tasks/avf_plugin_test/vpp/src/vlib/buffer_funcs.h:165
#6  0x0000ffffb6801258 in avf_rxq_init (vm=0xfffff6fcd000 <vlib_global_main>, ad=0xffffb8eb2d80, qid=0, rxq_size=512)
    at /home/snowball/tasks/avf_plugin_test/vpp/src/plugins/avf/device.c:247
#7  0x0000ffffb6803d78 in avf_device_init (vm=0xfffff6fcd000 <vlib_global_main>, am=0xffffb6831dc8 <avf_main>, ad=0xffffb8eb2d80,
    args=0xffffb8f90a30) at /home/snowball/tasks/avf_plugin_test/vpp/src/plugins/avf/device.c:948
#8  0x0000ffffb6806304 in avf_create_if (vm=0xfffff6fcd000 <vlib_global_main>, args=0xffffb8f90a30)
    at /home/snowball/tasks/avf_plugin_test/vpp/src/plugins/avf/device.c:1443
#9  0x0000ffffb67fae0c in avf_create_command_fn (vm=0xfffff6fcd000 <vlib_global_main>, input=0xffffb8f90f18, cmd=0xffffb7c9ede0)
    at /home/snowball/tasks/avf_plugin_test/vpp/src/plugins/avf/cli.c:63
#10 0x0000fffff6f1c438 in vlib_cli_dispatch_sub_commands (vm=0xfffff6fcd000 <vlib_global_main>,
    cm=0xfffff6fcd2a0 <vlib_global_main+672>, input=0xffffb8f90f18, parent_command_index=18)
    at /home/snowball/tasks/avf_plugin_test/vpp/src/vlib/cli.c:568
#11 0x0000fffff6f1c2f8 in vlib_cli_dispatch_sub_commands (vm=0xfffff6fcd000 <vlib_global_main>,
    cm=0xfffff6fcd2a0 <vlib_global_main+672>, input=0xffffb8f90f18, parent_command_index=19)
    at /home/snowball/tasks/avf_plugin_test/vpp/src/vlib/cli.c:528
#12 0x0000fffff6f1c2f8 in vlib_cli_dispatch_sub_commands (vm=0xfffff6fcd000 <vlib_global_main>,
    cm=0xfffff6fcd2a0 <vlib_global_main+672>, input=0xffffb8f90f18, parent_command_index=0)
    at /home/snowball/tasks/avf_plugin_test/vpp/src/vlib/cli.c:528
#13 0x0000fffff6f1c7e0 in vlib_cli_input (vm=0xfffff6fcd000 <vlib_global_main>, input=0xffffb8f90f18,
    function=0xfffff6f784e0 <unix_vlib_cli_output>, function_arg=0) at /home/snowball/tasks/avf_plugin_test/vpp/src/vlib/cli.c:667
#14 0x0000fffff6f7e5b0 in unix_cli_process_input (cm=0xfffff6fcd9d8 <unix_cli_main>, cli_file_index=0)
    at /home/snowball/tasks/avf_plugin_test/vpp/src/vlib/unix/cli.c:2572
#15 0x0000fffff6f7f154 in unix_cli_process (vm=0xfffff6fcd000 <vlib_global_main>, rt=0xffffb8f10000, f=0x0)
    at /home/snowball/tasks/avf_plugin_test/vpp/src/vlib/unix/cli.c:2688
#16 0x0000fffff6f4436c in vlib_process_bootstrap (_a=281473750202792) at /home/snowball/tasks/avf_plugin_test/vpp/src/vlib/main.c:1475
#17 0x0000fffff6e2eb38 in clib_calljmp () at /home/snowball/tasks/avf_plugin_test/vpp/src/vppinfra/longjmp.S:763
```

可以看到，在调用异常处理函数之前，代码出错的地方在`vlib_buffer_pool_get_default_for_numa (vm=0xfffff6fcd000 <vlib_global_main>, numa_node=4294967295)`。找到`vlib_buffer_pool_get_default_for_numa`的定义，如下所示：

```c
always_inline u8
vlib_buffer_pool_get_default_for_numa (vlib_main_t * vm, u32 numa_node)
{
  ASSERT (numa_node < VLIB_BUFFER_MAX_NUMA_NODES);
  return vm->buffer_main->default_buffer_pool_index_for_numa[numa_node];
}
```

结合bug复现阶段看到的错误信息` assertion numa_node < VLIB_BUFFER_MAX_NUMA_NODES' fails`，再看此时的`numa_node`的值是`4294967295`，对应的数据类型是`u32`，而`VLIB_BUFFER_MAX_NUMA_NODES`宏定义的值为32，判断代码在此处断言错误导致程序出错。而`u32`类型的`numa_node`的值为`4294967295`，而这个值很有可能是`int`类型的`-1`隐式转换成`u32`类型的结果。带着这个基本判断，接下来我来对相关函数设置断点进行调试。

我先将断点打在函数`avf_create_command_fn`上进行单步调试。

```
(gdb) b avf_create_command_fn
Breakpoint 1 at 0xffffb67fac74: file /home/snowball/tasks/avf_plugin_test/vpp/src/plugins/avf/cli.c, line 33.
```

敲入命令`run -c ~/startup_avf.conf`重新启动VPP debug CLI。

```
DBGvpp# create int avf 0000:01:02.0

Thread 1 "vpp_main" hit Breakpoint 1, avf_create_command_fn (vm=0xfffff6fcd000 <vlib_global_main>, input=0xffffb8f90f18,
    cmd=0xffffb7c9ede0) at /home/snowball/tasks/avf_plugin_test/vpp/src/plugins/avf/cli.c:33
33        unformat_input_t _line_input, *line_input = &_line_input;
(gdb) n
37        clib_memset (&args, 0, sizeof (avf_create_if_args_t));
(gdb)
40        if (!unformat_user (input, unformat_line_input, line_input))
(gdb)
43        while (unformat_check_input (line_input) != UNFORMAT_END_OF_INPUT)
(gdb)
45            if (unformat (line_input, "%U", unformat_vlib_pci_addr, &args.addr))
(gdb)
43        while (unformat_check_input (line_input) != UNFORMAT_END_OF_INPUT)
(gdb)
61        unformat_free (line_input);
(gdb)
63        avf_create_if (vm, &args);
```

上述单步调试结果可以看到，在`avf_create_command_fn`函数中，代码做的主要是对我在VPP debug CLI行中输入的命令参数的解析，解析完成后，调用`avf_create_if`函数创建AVF接口。我用`step`或`s`命令进入`avf_create_if`函数，继续单步调试，调试信息如下所示：

```
avf_create_if (vm=0xfffff6fcd000 <vlib_global_main>, args=0xffffb8f90a30)
    at /home/snowball/tasks/avf_plugin_test/vpp/src/plugins/avf/device.c:1324
1324      vnet_main_t *vnm = vnet_get_main ();
(gdb) n
1325      avf_main_t *am = &avf_main;
(gdb)
1328      clib_error_t *error = 0;
(gdb)
1332      args->rxq_size = (args->rxq_size == 0) ? AVF_RXQ_SZ : args->rxq_size;
(gdb)
1333      args->txq_size = (args->txq_size == 0) ? AVF_TXQ_SZ : args->txq_size;
(gdb)
1335      if ((args->rxq_size & (args->rxq_size - 1))
(gdb)
1336          || (args->txq_size & (args->txq_size - 1)))
(gdb)
1344      pool_get (am->devices, ad);
(gdb) n
1345      ad->dev_instance = ad - am->devices;
(gdb)
1346      ad->per_interface_next_index = ~0;
(gdb)
1347      ad->name = vec_dup (args->name);
(gdb)
1349      if (args->enable_elog)
(gdb)
1352      if ((error = vlib_pci_device_open (vm, &args->addr, avf_pci_device_ids,
(gdb)
1362      ad->pci_dev_handle = h;
(gdb)
1363      ad->pci_addr = args->addr;
(gdb)
1364      ad->numa_node = vlib_pci_get_numa_node (vm, h);
(gdb) print ad->numa_node
$1 = 0
(gdb) n
1366      vlib_pci_set_private_data (vm, h, ad->dev_instance);
(gdb) print ad->numa_node
$2 = 4294967295
(gdb) ptype ad->numa_node
type = unsigned int

```

在`avf_create_if`函数里，有一行代码是获取`numa_node`的值，`ad->numa_node = vlib_pci_get_numa_node(vm, h)`。我在执行此行代码之前，先打印出此时的`numa_node`的值为0。代码执行后，`numa_node`的值变为`4294967295`。可以得出结论，问题出在`vlib_pci_get_numa_node`这个函数中。查看函数的定义，如下所示：

```c
u32
vlib_pci_get_numa_node (vlib_main_t * vm, vlib_pci_dev_handle_t h)
{
  linux_pci_device_t *d = linux_pci_get_device (h);
  return d->numa_node;
}
```

在`vlib_pci_get_numa_node`函数设置断点，重新启动VPP debug CLI进行单步调试。

```
Thread 1 "vpp_main" hit Breakpoint 2, vlib_pci_get_numa_node (vm=0xfffff6fcd000 <vlib_global_main>, h=0)
    at /home/snowball/tasks/avf_plugin_test/vpp/src/vlib/linux/pci.c:172
172       linux_pci_device_t *d = linux_pci_get_device (h);
(gdb) print d->numa_node
$3 = 3107993569
(gdb) n
173       return d->numa_node;
(gdb) print d->numa_node
$4 = 4294967295
```

`vlib_pci_get_numa_node`函数中调用了`linux_pci_get_device`函数，我在代码执行这条语句前，打印`numa_node`的值，发现是个随机值。代码执行后，`numa_node`又变成了对应的`4294967295`。重新起VPP debug CLI，我单步调试进入`linux_pci_get_device`探个究竟。`linux_pci_get_device`函数定义如下所示：

```c
static linux_pci_device_t *
linux_pci_get_device (vlib_pci_dev_handle_t h)
{
  linux_pci_main_t *lpm = &linux_pci_main;
  return pool_elt_at_index (lpm->linux_pci_devices, h);
}
```

函数`linux_pci_get_devices`的单步调试结果如下所示：

```
(gdb) s
linux_pci_get_device (h=0) at /home/snowball/tasks/avf_plugin_test/vpp/src/vlib/linux/pci.c:143
143       linux_pci_main_t *lpm = &linux_pci_main;
(gdb) ptype lpm
type = struct {
    vlib_main_t *vlib_main;
    linux_pci_device_t *linux_pci_devices;
} *
(gdb) ptype lpm->linux_pci_devices
type = struct {
    linux_pci_device_type_t type;
    vlib_pci_dev_handle_t handle;
    vlib_pci_addr_t addr;
    u32 numa_node;
    linux_pci_region_t *regions;
    int config_fd;
    u64 config_offset;
    int fd;
    int io_fd;
    u64 io_offset;
    u32 uio_minor;
    linux_pci_irq_t intx_irq;
    linux_pci_irq_t *msix_irqs;
    uword private_data;
    u8 supports_va_dma;
} *
(gdb) print lpm->linux_pci_devices->numa_node
Cannot access memory at address 0x8
(gdb) n
144       return pool_elt_at_index (lpm->linux_pci_devices, h);
(gdb) print lpm->linux_pci_devices->numa_node
$5 = 4294967295
```

在单步调试过程中，我用`ptype`指令查看对应变量的数据类型。可以判断的是，函数在`linux_pci_main_t *lpm = &linux_pci_main;`完成了`numa_node`的赋值。那么，`linux_pci_main`是如何被初始化并完成成员结构体`linux_pci_devices`中`numa_node`的赋值，成为最终问题的聚焦点。

在函数`linux_pci_get_device`中，函数返回的使用的是`pool_elt_at_index`函数，对应的是返回VPP中`pool`中第`h`块的内存地址。顺着这条线索，我应该找到对`linux_pci_devices`在`pool`中分配内存的初始化代码，果然，搜索`pool_get`函数，在`init_device_from_registered`函数中发现了`linux_pci_devices`初始化的代码：

```c
void
init_device_from_registered (vlib_main_t * vm, vlib_pci_device_info_t * di)
{
  vlib_pci_main_t *pm = &pci_main;
  linux_pci_main_t *lpm = &linux_pci_main;
  pci_device_registration_t *r;
  pci_device_id_t *i;
  clib_error_t *err = 0;
  linux_pci_device_t *p;

  pool_get (lpm->linux_pci_devices, p);  // pool_get为device p分配pool中空余的内存块
  p->handle = p - lpm->linux_pci_devices;
  p->intx_irq.fd = -1;

  r = pm->pci_device_registrations;

  while (r)
    {
      for (i = r->supported_devices; i->vendor_id != 0; i++)
	if (i->vendor_id == di->vendor_id && i->device_id == di->device_id)
	  {
	    if (di->iommu_group != -1)
	      err = add_device_vfio (vm, p, di, r);
	    else
	      err = add_device_uio (vm, p, di, r);

	    if (err)
	      clib_error_report (err);
	    else
	      return;
	  }
      r = r->next_registration;
    }

  /* No driver, close the PCI config-space FD */
  clib_memset (p, 0, sizeof (linux_pci_device_t));
  pool_put (lpm->linux_pci_devices, p);
}
```

同样，我对`init_device_from_registered`函数设置断点，重新启动VPP debug CLI。

```
Breakpoint 1, init_device_from_registered (vm=0xfffff6fcd000 <vlib_global_main>, di=0xffffb7c75244)
    at /home/snowball/tasks/avf_plugin_test/vpp/src/vlib/linux/pci.c:1344
1344      vlib_pci_main_t *pm = &pci_main;
(gdb) n
1345      linux_pci_main_t *lpm = &linux_pci_main;
(gdb) n
1348      clib_error_t *err = 0;
(gdb) n
1351      pool_get (lpm->linux_pci_devices, p);
(gdb) n
1352      p->handle = p - lpm->linux_pci_devices;
(gdb) print lpm->linux_pci_devices->numa_node
$1 = 0
(gdb) n
1353      p->intx_irq.fd = -1;
(gdb)
1355      r = pm->pci_device_registrations;
(gdb)
1357      while (r)
(gdb)
1376      clib_memset (p, 0, sizeof (linux_pci_device_t));
(gdb)
1377      pool_put (lpm->linux_pci_devices, p);
(gdb)
1378    }
```

`init_device_from_registered`函数的调试信息说明对应的`pci`设备还没有完成初始化。单步调试完`init_device_from_registered`函数后，函数进入`linux_pci_init`，继续单步调试，发现了问题所在：

```
linux_pci_init (vm=0xfffff6fcd000 <vlib_global_main>) at /home/snowball/tasks/avf_plugin_test/vpp/src/vlib/linux/pci.c:1464
1464              vlib_pci_free_device_info (d);
(gdb) n
1458      vec_foreach (addr, addrs)
(gdb) n
1461          if ((d = vlib_pci_get_device_info (vm, addr, 0)))
(gdb) ptype d
type = struct vlib_pci_device_info {
    u32 flags;
    vlib_pci_addr_t addr;
    int numa_node;
    u16 device_class;
    u16 vendor_id;
    u16 device_id;
    u8 *product_name;
    u8 *vpd_r;
    u8 *vpd_w;
    u8 *driver_name;
    union {
        pci_config_type0_regs_t config0;
        pci_config_type1_regs_t config1;
        u8 config_data[256];
    };
    int iommu_group;
} *
(gdb) print d->numa_node
$2 = 65535
(gdb) n
1463              init_device_from_registered (vm, d);
(gdb) print d->numa_node
$3 = -1
```

`linux_pci_init`函数在执行`d = vlib_pci_get_device_info (vm, addr, 0)`之前，`numa_node`的值为一个随机值`65535`，执行完语句后，`numa_node`的值变为`-1`，而且查看`d`的数据类型，发现`numa_node`作为其结构体成员变量的数据类型为`int`，初步印证了我一开始的假设，bug出现的原因是`int`类型的`-1`隐式转换成`unsigned`类型。将断点打在`vlib_pci_get_device_info`函数，重新调试。

```
(gdb)
260       di->numa_node = -1;
(gdb) print di->numa_node
$4 = 0
(gdb) n
261       vec_reset_length (f);
(gdb) print di->numa_node
$5 = -1
(gdb) n
262       f = format (f, "%v/numa_node%c", dev_dir_name, 0);
(gdb) n
263       err = clib_sysfs_read ((char *) f, "%u", &di->numa_node);
(gdb) print f
$6 = (u8 *) 0xffffb7cc1e40 "/sys/bus/pci/devices/0000:00:00.0/numa_node"
(gdb) n
264       if (err)
(gdb) n
266           di->numa_node = -1;
```

`vlib_pci_get_device_info`函数的单步调试信息如上所示。可以看到，在对`numa_node`赋值为`-1`之前，`numa_node`为0。系统读取`/sys/bus/pci/devices/0000:00:00.0/numa_node`作为`numa_node`的值，查看文件，其值为`-1`，然而代码进入了`err`分支，说明`clib_sysfs_read`函数读取有问题，进入`clib_sysfs_read`函数进行调试。

```
77        result = va_unformat (&input, fmt, &va);
(gdb) print result
$7 = 33
(gdb) n
80        vec_free (s);
(gdb) print result
$8 = 0
(gdb) n
81        close (fd);
(gdb) n
83        if (result == 0)
(gdb)
84          return clib_error_return (0, "unformat error");
```

`result`的值为0， 返回`clib_error_return`，说明`va_unformat`函数解析有问题，进入`va_unformat`函数。

`va_unformat`函数调用`do_percent`函数，最终的错误原因是`clib_sysfs_read`读取的格式是`%u`，最后导致解析的函数是`unformat_integer (input, va, 10, UNFORMAT_INTEGER_UNSIGNED, data_bytes)`，而文件解析得到的值为`-1`，采用`UNFORMAT_INTEGER_UNSIGNED`解析方式就会出错。初步的解决方法是将`err = clib_sysfs_read ((char *) f, "%u", &di->numa_node)`中的`%u`变为`%d`。

## 4. 小结

写的有些凌乱，但大致的调试过程和思路都记录下来。这次调试也给我一个写代码的警示，在大型工程中变量的数据类型一定要统一规范，否则很容易出现错误。

## 5. 参考文献

1. SMP、NUMA、MPP体系结构介绍 https://www.cnblogs.com/yubo/archive/2010/04/23/1718810.html
2. AVF简介 https://www.sdnlab.com/21100.html