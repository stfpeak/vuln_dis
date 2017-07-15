#### AFL是什么
American fuzzy lop 号称是当前最高级的 Fuzzing 测试工具之一，由谷歌的 Michal Zalewski 所开发。通过对源码进行重新编译时进行插桩（简称编译时插桩）的方式自动产生测试用例来探索二进制程序内部新的执行路径。与其他基于插桩技术的 fuzzers 相比， afl-fuzz 具有较低的性能消耗，有各种高效的 fuzzing 策略和 tricks 最小化技巧，不需要先行复杂的配置，能无缝处理复杂的现实中的程序。当然 AFL 也支持直接对没有源码的二进制程序进行测试，但需要 QEMU 的支持，将在本文后面做详细介绍。

#### 安装 
从官网 http://lcamtuf.coredump.cx/afl/ 下载最新版的源码（ latest version），解压后进入所在目录。执行以下命令进行编译和安装：

1. make
2. sudo make install

#### 介绍
#####AFL文件说明：
* afl-gcc 和 afl-g++ 分别对应的是 gcc 和 g++ 的封装
* afl-clang 和 afl-clang++ 分别对应 clang 的 c 和 c++ 编译器封装➀。
* afl-fuzz 是 AFL 的主体，用于对目标程序进行 fuzz。
* afl-analyze 可以对用例进行分析，通过分析给定的用例，看能否发现用例中有意义的字段。
* afl-qemu-trace 用于 qemu-mode，默认不安装，需要手工执行 qemu-mode 的编译脚本进行编译，后面会介绍。
* afl-plot 生成测试任务的状态图
* afl-tmin 和 afl-cmin 对用例进行简化
* afl-whatsup 用于查看 fuzz 任务的状态
* afl-gotcpu 用于查看当前 CPU 状态
* afl-showmap 用于对单个用例进行执行路径跟踪

#### 使用
AFL 设计之初是用于文件格式 fuzz 的，也就是说被测对象的行为是解析从从文件中读取的数据。为了用 AFL 对网络程序 fuzzing，有两种途径➂：
1. 如果手头有被测程序的源码，可以修改源码，对网络接收报文的函数进行封装，改为从文件文件读取数据；
2. 如果没有源码，或者不想修改代码这么麻烦的话，可以使用LD_PRELOAD 加载一些 hook 库，替换掉recv、 recv_from 等网路 I/O 函数。 preeny 是一个很好的选择。

#### 例子
1. 修改CC = /usr/local/bin/afl-gcc 或者 
2. make
3. mkdir afl_in afl_out 
4. 将种子文件放置在afl_in文件夹中
5. afl-fuzz -i afl_in -o afl_out ./xxx @@ (@@表示从文件读取)
> 如果要保证文件名不变，加-f 参数：（afl-fuzz -i afl_in -o afl_out -f aa.txt ./xxx）
> 最终分析afl_out中的 crash/hang 中分析导致crash的文件

#### 参考链接
* [AFL使用](http://www.cnblogs.com/alert123/p/4916481.html)
* [模糊测试工具American Fuzzy Lop](http://www.jianshu.com/p/015c471f5a9d)
