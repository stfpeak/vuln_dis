    set pagination off
    set $dispatch = *(long*)((long)&address_space_memory + 0x40)
    set $sections = *(long*)($dispatch + 0x48)
    printf "sections=0x%lx\n", $sections

    set $count = 0
    while (*(long*)($sections + 0) != 0) && (*(long*)($sections + 8) != 0)
        set $size = *(long*)($sections + 0x20)
        set $addr = *(long*)($sections + 0x30)
        printf "addr=0x%08lx, size=0x%08lx, next=0x%08lx, ", $addr, $size, $addr + $size

        set $mr = *(long*)($sections + 0)
        set $ops = *(long*)($mr + 0x48)
        set $pf_read = *(long*)($ops + 0)
        printf "0x%016lx, ", $pf_read
        x/8xb $ops

        set $count = $count + 1
        set $sections = $sections + 0x40
    end

    printf "total %lu entries.\n", $count

通用版
    set pagination off
    set $dispatch = address_space_memory.dispatch
    set $sections = $dispatch->map->sections
    printf "sections=0x%lx\n", $sections

    set $count = address_space_memory.dispatch->map.sections_nb
    printf "total %lu entries.\n", $count
    while ($count > 0)
    set $size = *(long*)(&$sections->size)
    set $addr = $sections->offset_within_address_space
    printf "addr=0x%08lx, size=0x%08lx, next=0x%08lx, ", $addr, $size, $addr + $size

    set $mr = $sections->mr
    set $ops = $mr->ops
    set $pf_read = $ops
    printf "0x%016lx, ", $pf_read
    x/8xb $ops

    set $count = $count - 1
    set $sections = $sections + 1
    end

    printf "total %lu entries.\n", $count
