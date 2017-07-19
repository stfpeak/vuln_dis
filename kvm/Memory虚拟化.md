## Memory虚拟化

OS对于物理内存主要有两点认识：1.物理地址从0开始；2.内存地址是连续的。VMM接管了所有内存，但guest OS的对内存的使用就存在这两点冲突了，除此之外，一个guest对内存的操作很有可能影响到另外一个guest乃至host的运行。VMM的内存虚拟化就要解决这些问题。

在OS代码中，应用也是占用所有的逻辑地址，同时不影响其他应用的关键点在于有线性地址这个中间层；解决方法则是添加了一个中间层：guest物理地址空间；guest看到是从0开始的guest物理地址空间（类比从0开始的线性地址），而且是连续的，虽然有些地址没有映射；同时guest物理地址映射到不同的host逻辑地址，如此保证了VM之间的安全性要求。

这样MEM虚拟化就是GVA->GPA->HPA的寻址过程，传统软件方法有影子页表，硬件虚拟化提供了**EPT**支持。
可能GVA->GPA->HVA->HPA更全面一点。
* GVA: Guest Virtual Address
* GPA: Guest Physical Address
* HVA: Host Virtual Address
* HPA: Host Physical Address
