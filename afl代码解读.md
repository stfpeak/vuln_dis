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
