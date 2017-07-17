## 虚拟机启动过程

1. 获取到kvm句柄
* kvmfd = open("/dev/kvm", O_RDWR);

2. 创建虚拟机，获取到虚拟机句柄。
* vmfd = ioctl(kvmfd, KVM_CREATE_VM, 0);

3. 为虚拟机映射内存，还有其他的PCI，信号处理的初始化。
* ioctl(kvmfd, KVM_SET_USER_MEMORY_REGION, &mem);

4. 将虚拟机镜像映射到内存，相当于物理机的boot过程，把镜像映射到内存。

5. 创建vCPU，并为vCPU分配内存空间。
* ioctl(kvmfd, KVM_CREATE_VCPU, vcpuid);
* vcpu->kvm_run_mmap_size = ioctl(kvm->dev_fd, KVM_GET_VCPU_MMAP_SIZE, 0);
6. 创建vCPU个数的线程并运行虚拟机。
* ioctl(kvm->vcpus->vcpu_fd, KVM_RUN, 0);

7. 线程进入循环，并捕获虚拟机退出原因，做相应的处理。

这里的退出并不一定是虚拟机关机，虚拟机如果遇到IO操作，访问硬件设备，缺页中断等都会退出执行，退出执行可以理解为将CPU执行上下文返回到QEMU。
