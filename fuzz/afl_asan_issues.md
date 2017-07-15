## AFL + ASAN(UBSAN)编译过程遇到的问题与解决方法
    在用afl编译程序或者库时，将CC=afl-gcc CXX=afl-g++

    CFLAG 加上   -fsanitize=address -fsanitize=undefined -fno-omit-frame-pointer （不能加-fsanitize-coverage，gcc暂时还不支持）


    如果编译器是clang， 可以加上 -fsanitize-coverage=trace-pc-guard,trace-cmp,trace-gep,trace-div

    编程libxxx.so  或者 libxxx.a（最后编译成libxxx.a）

    在编译成so的情况下， 程序动态链接如：afl-gcc file.c -L. -lipsi_sftposal -lipsi_sftp -lasan -lubsan -g -o get_file

    -L. 表示动态链接库so 在当前文件夹

    -lasan -lubsan动态链接libasan.so 与 libubsan.so  

    将动态链接库（当前目录）加到环境变量中： export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:.

    生成可执行文件getfile

    执行可执行文件： ./getfile
    报错：
    ASan runtime does not come first in initial library list; you should either link runtime to your application or manually preload it with LD_PRELOAD.

    ldd getfile
    linux-vdso.so.1 =>  (0x00007ffd983e5000)
        /usr/lib/x86_64-linux-gnu/libasan.so.2 (0x00007fd3a562c000)
        /usr/lib/x86_64-linux-gnu/libubsan.so.0 (0x00007fd3a491f000)
        libipsi_sftp.so (0x00007fd3a3810000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fd3a343c000)
        libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007fd3a321f000)
        libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007fd3a301a000)
        libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007fd3a2d11000)
        libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007fd3a2afb000)
        libstdc++.so.6 => /usr/lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007fd3a2778000)
        libipsi_sftposal.so (0x00007fd3a254b000)
        /lib64/ld-linux-x86-64.so.2 (0x00005565be773000)
    可以获得程序动态链接库的路径
    将文件路径加入LD_PRELOAD：export LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libasan.so.2:/usr/lib/x86_64-linux-gnu/libubsan.so.0

    然后开始afl-fuzz
