## Guest和Host之间的切换
X86虚拟化技术Intel VT-x，提供了两种工作环境，VMCS实现两种环境之间的切换。VM Entry是虚拟机进入guest模式，VM Exit使虚拟机退出guest模式。

VMM调度guest执行时，qemu通过ioctl系统调用进入内核模式，在KVM Driver中获得当前物理CPU的引用。之后将guest状态从VMCS中读出，并装入物理CPU中。执行 VMLAUCH 指令使得物理处理器进入非根操作环境，运行guest OS代码。

当 guest OS 执行一些特权指令或者外部事件时， 比如I/O访问，对控制寄存器的操作，MSR的读写等， 都会导致物理CPU发生 VMExit， 停止运行 Guest OS，将 Guest OS保存到VMCS中， Host 状态装入物理处理器中， 处理器进入根操作环境，KVM取得控制权，通过读取 VMCS 中 VM_EXIT_REASON 字段得到引起 VM Exit 的原因。 从而调用kvm_exit_handler 处理函数。 如果由于 I/O 获得信号到达，则退出到userspace模式的 Qemu 处理。处理完毕后，重新进入guest模式运行虚拟 CPU。


