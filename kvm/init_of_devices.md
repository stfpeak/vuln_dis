## 设备初始化
1. vm_config_groups是一个数组，数组每一个成员是一个链表表头，这些链表包含了qemu-kvm的各种启动参数，device链表是链表数组中的一个链表。
2. QemuOptsList类型的device链表中的成员是QemuOpts类型，一个QemuOpts变量代表一个qemu-kvm中的“-device参数”，device链表中QemuOpts的数量就等于“-device”的数量，初始化时，会循环为每一个QemuOpts变量调用device_init_func函数。
3. 因为“-device”参数还有子参数（如virtio-scsi-pci.hotplug=on/off），所以QemuOptsList也引导一个链表，链表节点是QemuOpt类型，链表的每一个节点就对应“-device”的一个子参数。
4. device链表中每一个QemuOptsList节点对应一个“-device”参数，QemuOptsList链表中的每一个节点对应该“-device”的一个子参数。这些信息都是从qemu-kvm的启动参数初始化而来的。该函数完成工作就是 在vm_config_groups中找到device链表成员并返回。


## QDEV初始化整体原理：
1. 在main前面，通过type_init->module_init来完成设备模块初始化函数mc146818rtc_register_types的注册
2. 在main中，首先调用module_call_init(MODULE_INIT_QOM)完成各个设备模块的初始化（mc146818rtc_register_types）,并注册设备类的class_init函数
3. 设备的入口点，在machine->init的时候初始化不同设备的init入口函数，参见pc_init1中的xx_init
4. 先建立一个ISA的RTC设备，并调用class_init注册realized函数，其中class_init已经在main函数前就完成了注册
5. 调用对象已注册的realize函数实例化该虚拟设备，设定虚拟设备结构体的虚拟寄存器等

## 设备启动

main函数中调用 module_class->init(current_machine), 可以通过gdb 启动qemu-system-x86_64来调试，查看调用栈
pc_init1分析如下，主要进行CPU、Memory、VGA、NIC、PCI等的初始化
```c
static void pc_init1(MachineState *machine,
                     const char *host_type, const char *pci_type)
{
…

    pc_cpus_init(pcms);  初始化CPU

    if (kvm_enabled() && pcmc->kvmclock_enabled) {
        kvmclock_create();
    }

    if (pcmc->pci_enabled) {
        pci_memory = g_new(MemoryRegion, 1);
        memory_region_init(pci_memory, NULL, "pci", UINT64_MAX);
        rom_memory = pci_memory;
    } else {
        pci_memory = NULL;
        rom_memory = system_memory;
    }

    pc_guest_info_init(pcms);

    if (pcmc->smbios_defaults) {
        MachineClass *mc = MACHINE_GET_CLASS(machine);
        /* These values are guest ABI, do not change */
        smbios_set_defaults("QEMU", "Standard PC (i440FX + PIIX, 1996)",
                            mc->name, pcmc->smbios_legacy_mode,
                            pcmc->smbios_uuid_encoded,
                            SMBIOS_ENTRY_POINT_21);
    }

    /* allocate ram and load rom/bios */
    if (!xen_enabled()) {
        pc_memory_init(pcms, system_memory,
                       rom_memory, &ram_memory);
    } else if (machine->kernel_filename != NULL) {
        /* For xen HVM direct kernel boot, load linux here */
        xen_load_linux(pcms);
    }

    gsi_state = g_malloc0(sizeof(*gsi_state));
    if (kvm_ioapic_in_kernel()) {
        kvm_pc_setup_irq_routing(pcmc->pci_enabled);
        pcms->gsi = qemu_allocate_irqs(kvm_pc_gsi_handler, gsi_state,
                                       GSI_NUM_PINS);
    } else {
        pcms->gsi = qemu_allocate_irqs(gsi_handler, gsi_state, GSI_NUM_PINS);
    }

    if (pcmc->pci_enabled) {
        pci_bus = i440fx_init(host_type,
                              pci_type,
                              &i440fx_state, &piix3_devfn, &isa_bus, pcms->gsi,
                              system_memory, system_io, machine->ram_size,
                              pcms->below_4g_mem_size,
                              pcms->above_4g_mem_size,
                              pci_memory, ram_memory);
        pcms->bus = pci_bus;
    } else {
        pci_bus = NULL;
        i440fx_state = NULL;
        isa_bus = isa_bus_new(NULL, get_system_memory(), system_io,
                              &error_abort);
        no_hpet = 1;
    }
    isa_bus_irqs(isa_bus, pcms->gsi);

…

    pc_register_ferr_irq(pcms->gsi[13]);

    pc_vga_init(isa_bus, pcmc->pci_enabled ? pci_bus : NULL);

    assert(pcms->vmport != ON_OFF_AUTO__MAX);
    if (pcms->vmport == ON_OFF_AUTO_AUTO) {
        pcms->vmport = xen_enabled() ? ON_OFF_AUTO_OFF : ON_OFF_AUTO_ON;
    }

    /* init basic PC hardware */
    pc_basic_device_init(isa_bus, pcms->gsi, &rtc_state, true,
                         (pcms->vmport != ON_OFF_AUTO_ON), pcms->pit, 0x4);

    pc_nic_init(isa_bus, pci_bus);

    ide_drive_get(hd, ARRAY_SIZE(hd));
…

    pc_cmos_init(pcms, idebus[0], idebus[1], rtc_state);

    if (pcmc->pci_enabled && machine_usb(machine)) {
        pci_create_simple(pci_bus, piix3_devfn + 2, "piix3-usb-uhci");
    }

…

    if (pcmc->pci_enabled) {
        pc_pci_device_init(pci_bus);
    }

    if (pcms->acpi_nvdimm_state.is_enabled) {
        nvdimm_init_acpi_state(&pcms->acpi_nvdimm_state, system_io,
                               pcms->fw_cfg, OBJECT(pcms));
    }
}
```
