### 15.3 功耗管理的汇编指令

ARM汇编语言包括一些指令，可以用来将核心置于低功耗状态.  Cortex-A处理器家族中，这些指令的实现方式几乎关闭了核心所有部分的时钟。这意味着核心的功耗显著降低，因此只有静态泄漏电流，没有动态功耗。

WFI指令的作用是暂停执行，直到核心被以下条件之一唤醒:

* IRQ中断, 即使PSTATE.I-bit is set
* FIQ中断, 即使PSTATE.F-bit is set
* asynchronous abort

WFI指令广泛应用于电池供电系统。例如, 在等待你决定按下某个按键的时候, 手机的核心可以在一秒钟之内多次处于待机模式.

WFE与WFI类似. 核心将暂停执行, 直到**事件**发生.  This can be one the event conditions listed or 集群中其它核心发出的事件信号. 其他核心可以执行SEV指令来发出一个信号. SEV 发出的信号能够到达所有核心. 通用定时器也可以被配置为周期触发事件, 唤醒执行WFE的核心.

### 15.4 Power State Coordination Interface

Power State Coordination Interface (PSCI)提供了一种操作系统无关的方法实现电源管理. 其中所有的CPU核心可上电或关闭. PSCI包括:

* 核心空闲管理
* 动态添加和删除核心(热插拔), 和 secondary core boot.
* big.LITTLE 迁移
* 系统关闭和重启

使用PSCI接口发送的消息将被相应的execution level接收. 也就是说, 如果实现了EL2和EL3, 那么运行在EL0和EL1的OS发出的消息必然被EL2 Hypervisor接收. 如果Hypervisor发送消息, 必然被EL3 Secure Firmware接收. 由OS来决定切换时是否保存上下文.