# GCC编译器选项及优化提示

大多数程序和库在编译时默认的优化级别是"2"(使用gcc选项:"-O2")并且在Intel/AMD平台上默认按照i386处理器来编译。

如果你只想让编译出来的程序运行在特定的平台上，就需要执行更高级的编译器优化选项，以产生只能运行于特定平台的代码。

一种方法是修改每个源码包中的Makefile文件，在其中寻找CFLAGS和CXXFLAGS变量(C和C++编译器的编译选项)并修改它的值。
一些源码包比如binutils, gcc, glibc等等，在每个子文件夹中都有Makefile文件，这样修改起来就太累了！

另一种简易做法是设置CFLAGS和CXXFLAGS环境变量。大多数configure脚本会使用这两个环境变量代替Makefile文件中的值。
但是少数configure脚本并不这样做，他们必须需要手动编辑才行。

为了设置CFLAGS和CXXFLAGS环境变量，你可以在bash中执行如下命令(也可以写进.bashrc以成为默认值)：
export CFLAGS="-O3 -march=<cpu类型>" && CXXFLAGS=$CFLAGS
这是一个确保能够在几乎所有平台上都能正常工作的最小设置。

"-march"选项表示为特定的cpu类型编译二进制代码(不能在更低级别的cpu上运行)，
Intel通常是：pentium2, pentium3, pentium3m, pentium4, pentium4m, pentium-m, prescott, nocona
说明：pentium3m/pentium4m是笔记本用的移动P3/P4；pentium-m是迅驰I/II代笔记本的cpu；
prescott是带SSE3的P4(以滚烫到可以煎鸡蛋而闻名)；nocona则是最新的带有EMT64(64位)的P4(同样可以煎鸡蛋)
AMD通常是：k6, k6-2, k6-3, athlon, athlon-tbird, athlon-xp, athlon-mp, opteron, athlon64, athlon-fx
用AMD的一般都是DIYer，就不必解释了吧。

如果编译时没有抱怨"segmentation fault, core dumped"，那么你设定的"-O"优化参数一般就没什么问题。
否则请降低优化级别("-O3" -> "-O2" -> "-O1" -> 取消)。
个人意见：服务器使用"-O2"就可以了，它是最安全的优化参数(集合)；桌面可以使用"-O3" ；
不鼓励使用过多的自定义优化选项，其实他们之间没什么明显的速度差异(有时"-O3"反而更慢)。

编译器对硬件非常敏感，特别是在使用较高的优化级别的时候，一丁点的内存错误都可能导致致命的失败。
所以在编译时请千万不要超频你的电脑(我编译关键程序时总是先降频然的)。

注意：选项的顺序很重要，如果有两个选项互相冲突，则以后一个为准。
比如"-O3"将打开-finline-functions选项，但是可以用"-O3 -fno-inline-functions"既使用-O3的功能又关闭函数内嵌功能。

声明，"-O2"已经启用绝大多数安全的优化选项了，所以其实你不必对那一堆选项发愁。
先说说"-O3"在"-O2"基础上增加的几项，你可以按需添加(还算比较安全)：
[gcc-3.4.4]
-finline-functions 允许编译器选择某些简单的函数在其被调用处展开
-fweb 为每个web结构体分配一个伪寄存器
-frename-registers 试图驱除代码中的假依赖关系，这个选项对具有大量寄存器的机器很有效。
[gcc-4.0.2]
-finline-functions 说明如上
-funswitch-loops 将循环体中不改变值的变量移动到循环体之外
-fgcse-after-reload **不太明白它的含义**[哪位大峡知道给小弟讲解一下，先行谢过  ]

说完"-O3"再说说在嵌入式系统上常用的"-Os"选项，这个选项其实也很重要，它的含义是对生成的二进制代码进行尺寸上的优化，它打开了所有"- O2"打开的选项，因此通常认为的"-Os"生成的二进制代码执行效率低的潜在意识是错误的！当然该选项与"-O2"的不同之处在于它在"-O2"的基础 上禁止了所有为了对齐而插入的空间，也就是将所有"-falign-*"系列的选项禁用了。这种禁用究竟是否一定降低了代码的执行效率，依据程序的不同而 不同，据说某些情况下"-Os"的效率比"-O3"还要高14%！请兄弟们在实践中自己摸索吧...

---------------------------------------------

下面选择我认为比较重要的几项简单介绍一下[gcc-3.4.4]
[注意]这里列出的都是非默认的选项，你只需要添加你所需要的选项即可

-w 禁止输出警告消息

-Werror 将所有警告转换为错误

-Wall 显示所有的警告消息

-v 显示编译程序的当前版本号

-V<version> 指定gcc将要运行的版本。只有在安装了多个版本gcc的机器上才有效。

-ansi 按照ANSI标准编译程序，但并不限制与标准并不冲突的GNU扩展(一般不用该选项)

-pedantic 如果要限制代码必须严格符合ISO标准，就在"-ansi"的基础上同时启用这个选项(很少使用)

-std=<name> 指定C语言的标准(c89,c99,gnu89)，该选项禁止了GNU C的扩展关键字asm,typeof,inline (一般不用该选项)

-static 连接器将忽略动态连接库，同时通过将静态目标文件直接包含到结果目标文件完成对所有引用的解析。

-shared 连接器将生成共享目标代码，该共享库可在运行时动态连接到程序形成完整的可执行体。
如果使用gcc命令创建共享库作为其输出，该选项可以防止连接器将缺失main()方法视为错误。
为了可以正确的工作，应该一致的使用选项"-fpic"以及目标平台选项编译构成同一个库的所有共享目标模块。

-shared-libgcc 该选项指定使用共享版本的libgcc，在没有共享版本的libgcc的机器上该选项无效。

-specs=<filename> gcc驱动程序读取该文件以确定哪些选项应该传递给那些子进程。
该选项可以通过指定配置文件来覆盖默认配置，指定的文件将在默认配置文件读取后进行处理以修改默认配置。

-pipe 使用管道而不是临时文件一个阶段到另一个阶段交换输出的方式，可以加快编译速度。建议使用。

-o <filename> 指定输出文件，对各种输出皆有效。由于只能指定一个文件，所以在产生多个输出文件的情况下不要使用该选项。

--help 显示gcc的命令行选项列表；与"-v"一起使用时还将显示gcc调用的各个进程所接受的选项。

--target-help 显示目标机器相关的命令行选项列表

-b<machine> 指示需要编译程序的目标机器；默认为编译程序所运行的目标机编译代码。
目标机通过指定包含编译程序的目录来确定，通常为/usr/local/lib/gcc-lib/<machine>/<version>

-B<lib-prefix> 指定库文件的位置，包括编译程序的文件、执行程序和数据文件，如果需要运行子程序(如cpp,as,ld)就会用该前缀来定位。
这个前缀可以是用冒号分割的多个路径，环境变量GCC_EXEC_PREFIX和这个选项有相同的效果。

-I<dir> 指定搜索系统头文件的目录，可以重复使用多个该选项指定多个目录。

-dumpmachine 显示该程序的目标机名字，不做其他任何动作

-dumpspecs 显示构件编译程序的规范信息，包括用来编译、汇编和连接gcc编译程序自身用到的所有选项，不做其他任何动作。

-dumpversion 显示编译程序自身的版本号，不做其他任何动作

-falign-functions=N 将所有函数的起始地址在N(N=1,2,4,8,16...)的边界上对齐，默认为机器自身的默认值，指定为1表示禁止对齐。

-falign-jumps=N 将分支目标在N(N=1,2,4,8,16...)的边界上对齐，默认为机器自身的默认值，指定为1表示禁止对齐。
-fno-align-labels 建议使用它，以保证不和-falign-jumps("-O2"默认启用的选项)冲突

-fno-align-loops 建议使用它，以确保不会在分支目标前插入多余的空指令。

-fbranch-probabilities 在使用"-fprofile-arcs"选项编译程序并执行它来创建包含每个代码块执行次数的文件之后，程序可以利用这一选项再次编译，
文件中所产生的信息将被用来优化那些经常发生的分支代码。如果没有这些信息，gcc将猜测那一分支可能经常发生并进行优化。
这类优化信息将会存放在一个以源文件为名字的并以".da"为后缀的文件中。

-fno-guess-branch-probability 默认情况下gcc将使用随机模型进行猜测哪个分支更可能被经常执行，并以此来优化代码，该选项关闭它。

-fprofile-arcs 在使用这一选项编译程序并运行它以创建包含每个代码块的执行次数的文件后，程序可以再次使用"-fbranch-probabilities"编译，
文件中的信息可以用来优化那些经常选取的分支。如果没有这些信息，gcc将猜测哪个分支将被经常运行以进行优化。
这类优化信息将会存放在一个以源文件为名字的并以".da"为后缀的文件中。

-fforce-addr 必须将地址复制到寄存器中才能对他们进行运算。由于所需地址通常在前面已经加载到寄存器中了，所以这个选项可以改进代码。
-fforce-mem 必须将数值复制到寄存器中才能对他们进行运算。由于所需数值通常在前面已经加载到寄存器中了，所以这个选项可以改进代码。

-ffreestanding 所编译的程序能够在独立的环境中运行，该环境可以没有标准库，而且可以不从main()函数开始运行。
该选项将设置"-fno-builtin"，且等同于"-fno-hosted"。
-fhosted 所编译的程序需要运行在宿主环境中，其中需要有完整的标准库，而且main()函数具有int型的返回值。
-fno-builtin 除非利用"__builtin_"进行引用，否则不识别所有内建函数。

-fmerge-all-constants 试图将跨编译单元的所有常量值和数组合并在一个副本中。但是标准C/C++要求每个变量都必须有不同的存储位置。

-fmove-all-movables 将所有不变的表达式移动到循环体之外，这种做法的好坏取决于源代码中的循环结构。

-fnon-call-exceptions 产生的代码可供陷阱指令(如非法浮点运算和非法内存寻址)抛出异常，需要相关平台的运行时支持，并不普遍有效。

-fomit-frame-pointer 对于不需要栈指针的函数就不在寄存器中保存指针，因此可以忽略存储和检索地址的代码，并将寄存器用于普通用途。
所有"-O"级别都打开着一选项，但仅在调试器可以不依靠栈指针运行时才有效。建议不需要调试的情况下显式的设置它。

-fno-optional-diags 禁止输出诊断消息，C++标准并不需要这些消息。
-fpermissive 将代码中与标准不符合的诊断消息作为警告而不是错误输出。

-fpic 生成可用于共享库的位置独立代码(PIC)，所有的内存寻址均通过全局偏移表(GOT)完成。该选项并非在所有的机器上都有效。
要确定一个地址，需要将代码自身的内存位置作为表中的一项插入。该选项可以产生在共享库中存放并从中加载的目标模块。

-fprefetch-loop-arrays 生成数组预读取指令，对于使用巨大数组的程序可以加快代码执行速度，适合数据库相关的大型软件等。

-freg-struct-return 生成用寄存器返回短结构的代码，如果寄存器无法荣纳将使用内存。

-fstack-check 为防止程序栈溢出而进行必要的检测，在多线程环境中运行时才可能需要它。

-ftime-report 编译完成后显示编译耗时的统计信息

-funroll-loops 如果在编译时可以确定迭代的次数非常少而且循环中的指令也非常少，可以使用该选项进行循环展开，以驱除循环和复制指令。

-finline-limit=<size> 对伪指令数超过<size>的函数，编译程序将不进行展开，默认为600

--param <name>=<value> gcc内部存在一些优化代码程度的限制，调整这些限制就是调整整个优化全局。下面列出了参数的名字和对应的解释：
名字 解释
max-delay-slot-insn-search 较大的数目可以生成更优化的代码，但是会降低编译速度，默认为100
max-delay-slot-live-search 较大的数目可以生成更优化的代码，但是会降低编译速度，默认为333
max-gcse-memory 执行GCSE优化使用的最大内存量，太小将使该优化无法进行，默认为50M
max-gcse-passes 执行GCSE优化的最大迭代次数，默认为1

*******************************************************************
说完了命令行选项，下面来说说与硬件体系结构(主要是cpu)相关的设置[仅针对i386/x86_64]
最大名鼎鼎的"-march"上面已经说过了，下面讲讲别的(仅挑些实用的)

-mfpmath=sse P3和athlon-tbird以上级别的cpu支持

-masm=<dialect> 使用指定的dialect输出汇编语言指令，可以使用"intel"或"att"；默认为"att"

-mieee-fp 指定编译器使用IEEE浮点比较，这样将会正确的处理比较结果为无序的情况。

-malign-double 将double, long double, long long对齐于双字节边界上；有助于生成更高速的代码，但是程序的尺寸会变大。

-m128bit-long-double 指定long double为128位，pentium以上的cpu更喜欢这种标准。

-mregparm=N 指定用于传递整数参数的寄存器数目(默认不使用寄存器)。0<=N<=3 ；注意：当N>0时你必须使用同一参数重新构建所有的模块，包括所有的库。

-mmmx
-mno-mmx
-msse
-mno-sse
-msse2
-mno-sse2
-msse3
-mno-sse3
-m3dnow
-mno-3dnow
上面的这些不用解释了，一看就明白，根据自己的CPU决定吧

-maccumulate-outgoing-args 指定在函数引导段中计算输出参数所需最大空间，这在大部分现代cpu中是较快的方法；缺点是会增加代码尺寸。

-mthreads 支持Mingw32的线程安全异常处理。对于依赖于线程安全异常处理的程序，必须启用这个选项。
使用这个选项时会定义"-D_MT"，它将包含使用选项"-lmingwthrd"连接的一个特殊的线程辅助库，用于为每个线程清理异常处理数据。

-minline-all-stringops 嵌入所有的字符串操作。可以提高字符串操作的性能，但是会增加代码尺寸。

-momit-leaf-frame-pointer 不为叶子函数在寄存器中保存栈指针，这样可以节省寄存器，但是将会是调试变的困难。参见"-fomit-frame-pointer"。

下面这几个仅用于x86_64环境：

-m64 生成专门运行于64位环境的代码，不能运行于32位环境

-mcmodel=small [默认值]程序和它的符号必须位于2GB以下的地址空间。指针仍然是64位。程序可以静态连接也可以动态连接。
-mcmodel=kernel 内核运行于2GB地址空间之外。在编译linux内核时必须使用该选项！
-mcmodel=medium 程序必须位于2GB以下的地址空间，但是它的符号可以位于任何地址空间。程序可以静态连接也可以动态连接。
注意：共享库不能使用这个选项编译！
-mcmodel=large 对地址空间没有任何限制，这个选项的功能目前尚未实现。

＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝
既然已经讲了这么多了索性再讲讲gcc使用的一些环境变量
除了大名鼎鼎的CFLAGS和CXXFLAGS以外(其实是Autoconf的环境变量)，再挑几个说说：
所有的PATH类环境变量(除LD_RUN_PATH外)都是用冒号分割的目录列表。

C_INCLUDE_PATH 编译C程序时使用的环境变量，用于查找头文件。

CPLUS_INCLUDE_PATH 编译C++程序时使用的环境变量，用于查找头文件。

OBJC_INCLUDE_PATH 编译Obj-C程序时使用的环境变量，用于查找头文件。

CPATH 编译C/C++/Obj-C程序时使用的环境变量，用于查找头文件。

COMPILER_PATH 如果没有用GCC_EXEC_PREFIX定位子程序，编译程序将会在此查找它的子程序。

LIBRARY_PATH 连接程序将在这些目录中寻找特殊的连接程序文件。

LD_LIBRARY_PATH 该环境变量不影响编译程序，但是程序运行的时候会有影响：程序会查找该目录列表以寻找共享库。
当不能够在编译程序的目录中找到共享库的时候，执行程序必须设置该环境变量。

LD_RUN_PATH 该环境变量不影响编译程序，但是程序运行的时候会有影响：它在运行时指出了文件的名字，运行的程序可以由此得到它的符号名字和地址。
由于地址不会重新载入，因而可能符号应用其他文件中的绝对地址。这个和ld工具使用的"-R"选项完全一样。

GCC_EXEC_PREFIX 编译程序执行所有子程序的名字的前缀，默认值是"<prefix>/lib/gcc-lib/"，
其中的<prefix>是安装时configure脚本指定的前缀。

LANG 指定编译程序使用的字符集，可用于创建宽字符文件、串文字、注释；默认为英文。[目前只支持日文"C-JIS,C-SJIS,C-EUCJP"，不支持中文]

LC_ALL 指定多字节字符的字符分类，主要用于确定字符串的字符边界以及编译程序使用何种语言发出诊断消息；默认设置与LANG相同。
中文相关的几项："zh_CN.GB2312 , zh_CN.GB18030 , zh_CN.GBK , zh_CN.UTF-8 , zh_TW.BIG5"

TMPDIR 编译程序存放临时工作文件的临时目录，这些临时文件通常在编译结束时被删除。
