    afl-gcc.c

    /*
    The wrapper needs to know the path to afl-as (renamed to 'as'). The default is /usr/local/lib/afl/. A convenient way to specify alternative directories
    would be to set AFL_PATH.
    */

    find_as()
    首先寻找afl-as的路径，默认在/usr/local/lib/afl/， 也可以通过设置AFL_PATH

    edit_params()
    通过对CFLAGS的循环比对以及从环境变量中获取配置选项，将编译选项放在cc_params[]数组中

    main()
    cc_params[0]   确定编译器

    if (!strcmp(name, "afl-clang++")) {
          u8* alt_cxx = getenv("AFL_CXX");
          cc_params[0] = alt_cxx ? alt_cxx : (u8*)"clang++";
        } else {
          u8* alt_cc = getenv("AFL_CC");
          cc_params[0] = alt_cc ? alt_cc : (u8*)"clang";
        }

      if (!strcmp(name, "afl-g++")) {
          u8* alt_cxx = getenv("AFL_CXX");
          cc_params[0] = alt_cxx ? alt_cxx : (u8*)"g++";
        } else if (!strcmp(name, "afl-gcj")) {
          u8* alt_cc = getenv("AFL_GCJ");
          cc_params[0] = alt_cc ? alt_cc : (u8*)"gcj";
        } else {
          u8* alt_cc = getenv("AFL_CC");
          cc_params[0] = alt_cc ? alt_cc : (u8*)"gcc";
        }

    然后执行  execvp(cc_params[0], (char**)cc_params);
    编译程序



    afl-as.c

    /*
          The sole purpose of this wrapper is to preprocess assembly files generated
       by GCC / clang and inject the instrumentation bits included from afl-as.h. It
       is automatically invoked by the toolchain when compiling programs using
       afl-gcc / afl-clang.

       Note that it's an explicit non-goal to instrument hand-written assembly,
       be it in separate .s files or in __asm__ blocks. The only aspiration this
       utility has right now is to be able to skip them gracefully and allow the
       compilation process to continue.

       That said, see experimental/clang_asm_normalize/ for a solution that may
       allow clang users to make things work even with hand-crafted assembly. Just
       note that there is no equivalent for GCC.
    */



    /* Process input file, generate modified_file. Insert instrumentation in all
       the appropriate places. 
    */
    function add_instrumentation()

    通过分析preprocess后的文件，一般是.i文件，读取输入文件，然后对输入文件逐行语法分析，通过判断满足条件后对outf 插入插桩代码(插桩代码位于afl-as.h文件)，插入后继续分析文件，遇到下一个满足条件的行时，再次插入插桩代码。
    afl-as.h中共有4类插桩代码

    然后将插桩后的代码传递给真正的as来汇编，完成剩余的编译链接工作。


    afl-fuzz.c

    /* 
          This is the real deal: the program takes an instrumented binary and
       attempts a variety of basic fuzzing tricks, paying close attention to
       how they affect the execution path.
     */

    检查各种配置，不停的循环变异，如果程序执行发现新的路径，记录此用例，一直循环，配合Address Sanitizer发现问题
    Fuzz流程：
	1. 读取输入的初始testcase, 将其放入到queue中
	2. 从queue中读取内容作为程序输入
	3. 尝试在不影响流程的情况下精简输入
	4. 对输入进行自动突变
	5. 如果突变后的输入能够有新的状态转移，将修改后的输入放入queue中
	6. 回到2


    假设有#1和#2执行流，当先执行#1再执行#2后，#2流会被认为是新路
    径，因为出现了新的tuples (CA, AE):
    #1: A -> B -> C -> D -> E
    #2: A -> B -> C -> A -> E
    在执行完#2，下边的执⾏流会不会被认为是新的路径，尽管总的来说看
    起来和上边的都不同:
    #3: A -> B -> C -> A -> B -> C -> A -> B -> C -> D -> E

    AFL-fuzz 与 目标程序通信
    ** function  EXP_ST void setup_shm()
    在afl-fuzz对目标程序进行execve之前，首先先申请一段内存，随后将这段内存的fd通过setenv写入环境变量中。这段内存将作为fuzzer和被fuzz进程的共享       内存。
    使用持久化模式
    function static void __afl_map_shm(void)
    随后被fuzz的程序在被execve后首先从环境变量中提取fd，从而获得共享内存的地址。
    然后在这段内存的第一个字节写入1，从而确保fuzzer不会主动停止本次fuzz。
    捕获新的路径

    AFL主要通过对每个基本块进行插装来计算覆盖率。回调函数每次向共享内存的地址中写入数据， 表示当前分支的状态。
    function  __sanitizer_cov_trace_pc


    每当fuzzer检测到共享内存的hash有所改变，就会调用has_new_bits函数对路径状态进行更新并写回覆盖率文
    件里，讲本次的变化后的输放入queue当中。 afl/output/fuzz_bitmap

    使用符号执行，可以更好的寻找路径

    
