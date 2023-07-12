-   

## [gdb各种调试命令和技巧](https://www.cnblogs.com/youxin/p/4305227.html)

2015-02-28 13:16  [youxin](https://www.cnblogs.com/youxin/)  阅读(11709)  评论(0)  [编辑](https://i.cnblogs.com/EditPosts.aspx?postid=4305227)  [收藏](javascript:void(0))  [举报](javascript:void(0))

陈皓：**用GDB调试程序**

**GDB概述  
————**

GDB是GNU开源组织发布的一个强大的UNIX下的程序调试工具。或许，各位比较喜欢那种图形界面方式的，像VC、BCB等IDE的调试，但如果你是在UNIX平台下做软件，你会发现GDB这个调试工具有比VC、BCB的图形化调试器更强大的功能。所谓“寸有所长，尺有所短”就是这个道理。

一般来说，GDB主要帮忙你完成下面四个方面的功能：

 1、启动你的程序，可以按照你的自定义的要求随心所欲的运行程序。  
    2、可让被调试的程序在你所指定的调置的断点处停住。（断点可以是条件表达式）  
    3、当程序被停住时，可以检查此时你的程序中所发生的事。  
    4、动态的改变你程序的执行环境。

从上面看来，GDB和一般的调试工具没有什么两样，基本上也是完成这些功能，不过在细节上，你会发现GDB这个调试工具的强大，大家可能比较习惯了图形化的调试工具，但有时候，命令行的调试工具却有着图形化工具所不能完成的功能。让我们一一看来。

**一个调试示例  
——————**

 1 #include <stdio.h>  
     2  
     3 int func(int n)  
     4 {  
     5         int sum=0,i;  
     6         for(i=0; i<n; i++)  
     7         {  
     8                 sum+=i;  
     9         }  
    10         return sum;  
    11 }  
    12  
    13  
    14 main()  
    15 {  
    16         int i;  
    17         long result = 0;  
    18         for(i=1; i<=100; i++)  
    19         {  
    20                 result += i;  
    21         }  
    22  
    23        printf("result[1-100] = %d /n", result );  
    24        printf("result[1-250] = %d /n", func(250) );  
    25 }

编译生成执行文件：（Linux下）  
    hchen/test> cc -g tst.c -o tst

使用GDB调试：

hchen/test> gdb tst  <---------- 启动GDB  
GNU gdb 5.1.1  
Copyright 2002 Free Software Foundation, Inc.  
GDB is free software, covered by the GNU General Public License, and you are  
welcome to change it and/or distribute copies of it under certain conditions.  
Type "show copying" to see the conditions.  
There is absolutely no warranty for GDB.  Type "show warranty" for details.  
This GDB was configured as "i386-suse-linux"...  
(gdb) l     <-------------------- l命令相当于list，从第一行开始例出原码。  
1        #include <stdio.h>  
2  
3        int func(int n)  
4        {  
5                int sum=0,i;  
6                for(i=0; i<n; i++)  
7                {  
8                        sum+=i;  
9                }  
10               return sum;  
(gdb)       <-------------------- 直接回车表示，重复上一次命令  
11       }  
12  
13  
14       main()  
15       {  
16               int i;  
17               long result = 0;  
18               for(i=1; i<=100; i++)  
19               {  
20                       result += i;      
(gdb) break 16    <-------------------- 设置断点，在源程序第16行处。  
Breakpoint 1 at 0x8048496: file tst.c, line 16.  
(gdb) break func  <-------------------- 设置断点，在函数func()入口处。  
Breakpoint 2 at 0x8048456: file tst.c, line 5.  
(gdb) info break  <-------------------- 查看断点信息。  
Num Type           Disp Enb Address    What  
1   breakpoint     keep y   0x08048496 in main at tst.c:16  
2   breakpoint     keep y   0x08048456 in func at tst.c:5  
(gdb) r           <--------------------- 运行程序，run命令简写  
Starting program: /home/hchen/test/tst

Breakpoint 1, main () at tst.c:17    <---------- 在断点处停住。  
17               long result = 0;  
(gdb) n          <--------------------- 单条语句执行，next命令简写。  
18               for(i=1; i<=100; i++)  
(gdb) n  
20                       result += i;  
(gdb) n  
18               for(i=1; i<=100; i++)  
(gdb) n  
20                       result += i;  
(gdb) c          <--------------------- 继续运行程序，continue命令简写。  
Continuing.  
result[1-100] = 5050       <----------程序输出。

Breakpoint 2, func (n=250) at tst.c:5  
5                int sum=0,i;  
(gdb) n  
6                for(i=1; i<=n; i++)  
(gdb) p i        <-------  打印变量i的值，print命令简写。从这里可以看到，n是显示下一条要运行的语句，但还没运行  
$1 = 134513808  
(gdb) n  
8                        sum+=i;  
(gdb) n  
6                for(i=1; i<=n; i++)  
(gdb) p sum  
$2 = 1  
(gdb) n  
8                        sum+=i;  
(gdb) p i  
$3 = 2  
(gdb) n  
6                for(i=1; i<=n; i++)  
(gdb) p sum  
$4 = 3  
**(gdb) bt        <--------------------- 查看函数堆栈。backtrace 打印当前的函数调用栈的所有信息。**  
#0  func (n=250) at tst.c:5  
#1  0x080484e4 in main () at tst.c:24  
#2  0x400409ed in __libc_start_main () from /lib/libc.so.6  
(gdb) finish    <--------------------- 退出函数。  
Run till exit from #0  func (n=250) at tst.c:5  
0x080484e4 in main () at tst.c:24  
24              printf("result[1-250] = %d /n", func(250) );  
Value returned is $6 = 31375  
(gdb) c     <--------------------- 继续运行。  
Continuing.  
result[1-250] = 31375    <----------程序输出。

Program exited with code 027. <--------程序退出，调试结束。  
(gdb) q     <--------------------- 退出gdb。  
hchen/test>

好了，有了以上的感性认识，还是让我们来系统地认识一下gdb吧。


**使用GDB  
————**

一般来说GDB主要调试的是C/C++的程序。要调试C/C++的程序，首先在编译时，我们必须要把调试信息加到可执行文件中。使用编译器（cc/gcc/g++）的 -g 参数可以做到这一点。如：

 > cc -g hello.c -o hello  
    > g++ -g hello.cpp -o hello

如果没有-g，你将看不见程序的函数名、变量名，所代替的全是运行时的内存地址。当你用-g把调试信息加入之后，并成功编译目标代码以后，让我们来看看如何用gdb来调试他。

启动GDB的方法有以下几种：

 1、gdb <program>   
       program也就是你的执行文件，一般在当然目录下。

 2、gdb <program> core  
       用gdb同时调试一个运行程序和core文件，core是程序非法执行后core dump后产生的文件。

 3、gdb <program> <PID>  
       如果你的程序是一个服务程序，那么你可以指定这个服务程序运行时的进程ID。gdb会自动attach上去，并调试他。program应该在PATH环境变量中搜索得到。

GDB启动时，可以加上一些GDB的启动开关，详细的开关可以用gdb -help查看。我在下面只例举一些比较常用的参数：

 -symbols <file>   
    -s <file>   
    从指定文件中读取符号表。

 -se file   
    从指定文件中读取符号表信息，并把他用在可执行文件中。

 -core <file>  
    -c <file>   
    调试时core dump的core文件。

 -directory <directory>  
    -d <directory>  
    加入一个源文件的搜索路径。默认搜索路径是环境变量中PATH所定义的路径。

转自：[http://blog.csdn.net/haoel/article/details/2879](http://blog.csdn.net/haoel/article/details/2879)

 技巧汇总：

输出格式

一般来说，GDB会根据变量的类型输出变量的值。但你也可以自定义GDB的输出的格  
式。例如，你想输出一个整数的十六进制，或是二进制来查看这个整型变量的中的  
位的情况。要做到这样，你可以使用GDB的数据显示格式：

x 按十六进制格式显示变量。  
d 按十进制格式显示变量。  
u 按十六进制格式显示无符号整型。  
o 按八进制格式显示变量。  
t 按二进制格式显示变量。  
a 按十六进制格式显示变量。  
c 按字符格式显示变量。  
f 按浮点数格式显示变量。

(gdb) p i  
$21 = 101

(gdb) p/a i  
$22 = 0x65

(gdb) p/c i  
$23 = 101 'e'

(gdb) p/f i  
$24 = 1.41531145e-43

(gdb) p/x i  
$25 = 0x65

(gdb) p/t i  
$26 = 1100101

程序变量

查看文件中某变量的值：  
file::variable  
function::variable  
可以通过这种形式指定你所想查看的变量，是哪个文件中的或是哪个函数中的。例如，查看文件f2.c中的全局变量x的值：  
gdb) p 'f2.c'::x  

查看数组的值

有时候，你需要查看一段连续的内存空间的值。比如数组的一段，或是动态分配的数据的大小。你可以使用GDB的“@”操

作符，“@”的左边是第一个内存的地址的值，“@”的右边则你你想查看内存的长度。例如，你的程序中有这样的语句：

int *array = (int *) malloc (len * sizeof (int));  
于是，在GDB调试过程中，你可以以如下命令显示出这个动态数组的取值：

p *array@len

二维数组打印

p **array@len

如果是静态数组的话，可以直接用print数组名，就可以显示数组中所有数据的内容了。

[http://www.cppblog.com/chaosuper85/archive/2009/08/04/92123.html](http://www.cppblog.com/chaosuper85/archive/2009/08/04/92123.html)

*启动gdb调试指定程序app： 

$gdb app 

这样就在启动gdb之后直接载入了app可执行程序，需要注意的是，载入的app程序必须在编译的时候有gdb调试选项，例如'gcc -g app app.c',注意，如果修改了程序的源代码，但是没有编译，那么在gdb中显示的会是改动后的源代码，但是运行的是改动前的程序，这样会导致跟踪错乱的。 

*启动程序之后，再用gdb调试： 

$gdb <program> <PID> 

这里，<program>是程序的可执行文件名，<PID>是要调试程序的PID.如果你的程序是一个服务程序，那么你可以指定这个服务程序运行时的进程ID。gdb会自动attach上去，并调试他。program应该在PATH环境变量中搜索得到。 

*启动程序之后，再启动gdb调试： 

$gdb <PID> 

这里，程序是一个服务程序，那么你可以指定这个服务程序运行时的进程ID,<PID>是要调试程序的PID.这样gdb就附加到程序上了，但是现在还没法查看源代码,用file命令指明可执行文件就可以显示源代码了。 

gdb断点设置

[http://sourceware.org/gdb/current/onlinedocs/gdb](http://sourceware.org/gdb/current/onlinedocs/gdb)

二、断点设置

gdb断点分类：

以设置断点的命令分类：

breakpoint

可以根据行号、函数、条件生成断点。

watchpoint

监测变量或者表达式的值发生变化时产生断点。

catchpoint

监测信号的产生。例如c++的throw，或者加载库的时候。

gdb中的变量从1开始标号，不同的断点采用变量标号同一管理，可以 用enable、disable等命令管理，同时支持断点范围的操作，比如有些命令接受断点范围作为参数。

例如：disable 5-8

1、break及break变种详解：

相关命令有break，tbreak，rbreak,hbreak，thbreak，后两种是基于硬件的，先不介绍。

>>break 与 tbreak

> break，tbreak可以根据行号、函数、条件生成断点。tbreak设置方法与break相同，只不过tbreak只在断点停一次，过后会自动将断点删除，break需要手动控制断点的删除和使能。
> 
> break 可带如下参数：
> 
> linenum                 本地行号，即list命令可见的行号
> 
> filename:linenum  制定个文件的行号
> 
> function                函数，可以是自定义函数也可是库函数，如open
> 
> filename:function  制定文件中的函数
> 
> condtion                条件
> 
>  (
> 
> ##### **5.2 多文件设置断点**
> 
> 在进入指定函数时停住:
> 
> C++中可以使用class::function或function(type,type)格式来指定函数名。如果有名称空间，可以使用namespace::class::function或者function(type,type)格式来指定函数名。
> 
> break filename:linenum   
> 在源文件filename的linenum行处停住   
> break filename:function   
> 在源文件filename的function函数的入口处停住
> 
> break **class::functio**n或function(type,type)  （**个人感觉这个比较方便，b 类名::函数名,执行后会提示如：**
> 
> >>b GamePerson::update  
> Breakpoint 1 at 0x46b89e: file GamePerson.cpp, line 14.
> 
> 在类class的function函数的入口处停住
> 
> break namespace::class::function
> 
> 在名称空间为namespace的类class的function函数的入口处停住
> 
> )
> 
> *address      地址，可是函数，变量的地址，此地址可以通过info add命令得到。
> 
> 例如：
> 
> break 10    
> 
> break test.c:10
> 
> break main
> 
> break test.c:main
> 
> break system
> 
> break open
> 
> 如果想在指定的地址设置断点，比如在main函数的地址出设断点。
> 
> 可用info add main 获得main的地址如0x80484624，然后用break *0x80484624.
> 
> 条件断点就是在如上述指定断点的同时指定进入断点的条件。
> 
> 例如：（假如有int 类型变量 index）
> 
> break 10 if index == 3
> 
> tbreak 12 if index == 5
> 
> 8）单步运行　　(gdb) n
> 
> 9）程序继续运行　　(gdb) c
> 
> 　　使程序继续往下运行，直到再次遇到断点或程序结束；
> 
> break + 设置断点的行号　　break n　　　　　　在n行处设置断点
> 
> tbreak + 行号或函数名　　tbreak n/func　　　　设置临时断点，到达后被自动删除
> 
> until   
> 
> **until line-number  继续运行直到到达指定行号，或者函数，地址等。 (这个很有用)**
> 
> until line-number if   condition
> 
> bt:  **显示当前堆栈的追踪，当前所在的函数**。
> 
> (1)如何打印变量的值？(print var) 
> 
> (2)如何打印变量的地址？(print &var) 
> 
> (3)如何打印地址的数据值？(print *address) 
> 
> (4)如何查看当前运行的文件和行？(backtrace) 
> 
> (5)如何查看指定文件的代码？(list file:N) 
> 
> (6)如何立即执行完当前的函数，但是并不是执行完整个应用程序？(finish) 
> 
> (7)如果程序是多文件的，怎样定位到指定文件的指定行或者函数？(list file:N) 
> 
> (8)如果循环次数很多，如何执行完当前的循环？(until) 
> 
> ## **4.调试运行环境相关命令**
> 
> set args　　set args arg1 arg2　　设置运行参数
> 
> show args　　show args　　参看运行参数
> 
> set width + 数目　　set width 70　　设置GDB的行宽
> 
> cd + 工作目录　　cd ../　　切换工作目录
> 
> run　　r/run　　程序开始执行
> 
> step(s)　　s　　进入式（会进入到所调用的子函数中）单步执行，进入函数的前提是，此函数被编译有debug信息
> 
> next(n)　　n　　非进入式（不会进入到所调用的子函数中）单步执行
> 
> finish　　finish　　一直运行到函数返回并打印函数返回时的堆栈地址和返回值及参数值等信息
> 
> until + 行数　 u 3　　运行到函数某一行 
> 
> continue(c)　　c　　执行到下一个断点或程序结束 
> 
> return <返回值>　　return 5　　改变程序流程，直接结束当前函数，并将指定值返回
> 
> call + 函数　　call func　　在当前位置执行所要运行的函数
> 
> 查看堆栈信息  
>   
> info stack  
>   
> **用这条指令你可以看清楚程序的调用层次关系,这个挺有用**。
> 
> 3. 执行一行程序. 若呼叫函数, 则将该包含该函数程序代码视为一行程序 (next 指令可简写为 n)。  
>   
> (gdb) next  
>   
> 4. 执行一行程**序. 若呼叫函数, 则进入函数逐行执行 (step 指令可简写**为 s)。  
> (gdb) step
> 
> 5. 执行一行程序，若此时程序是在 for/while/do loop 循环的最后一行，则一直执行到循环结束后的第一行程序后停止 (until 指令可简写为 u)。  
> (gdb) until
> 
> 6. 执行现行程序到回到上一层程序为止。  
> (gdb) finish
> 
> *执行下一步： 
> 
> (gdb) next 
> 
> 这样，执行一行代码，如果是函数也会跳过函数。这个命令可以简化为n. 
> 
> *执行N次下一步： 
> 
> (gdb) next N 
> 
> *执行上次执行的命令： 
> 
> (gdb) [Enter] 
> 
> 这里，直接输入回车就会执行上次的命令了。 
> 
> *单步进入： 
> 
> (gdb) step 
> 
> 这样**，也会执行一行代码，不过如果遇到函数的话就会进入函数的内部，再一行一行的执行。** 
> 
> *执行完当前函数返回到调用它的函数： 
> 
> (gdb) finish 
> 
> 这里，运行程序，直到当前函数运行完毕返回再停止。例如进入的单步执行如果已经进入了某函数，而想退出该函数返回到它的调用函数中，可使用命令finish. 
> 
> *指定程序直到退出当前循环体： 
> 
> (gdb) until 
> 
> 或(gdb) u 
> 
> 这里，发现需要把光标停止在循环的头部，然后输入u这样就自动执行全部的循环了。
> 
> *列出指定区域(n1到n2之间)的代码： 
> 
> (gdb) list n1 n2 
> 
> 这样,list可以简写为l,将会显示n1行和n2行之间的代码，如果使用-tui启动gdb，将会在相应的位置显示。如果没有n1和n2参数，那么就会默认显示当前行和之后的10行，再执行又下滚10行。另外，list还可以接函数名。 
> 
> 一般来说在list后面可以跟以下这们的参数： 
> 
> <linenum>   行号。 
> 
> <+offset>   当前行号的正偏移量。 
> 
> <-offset>   当前行号的负偏移量。 
> 
> <filename:linenum>  哪个文件的哪一行。 
> 
> <function>  函数名。 
> 
> <filename:function> 哪个文件中的哪个函数。 
> 
> <*address>  程序运行时的语句在内存中的地址
> 
>  **有时候n输不出代码信息，上传缺失的那个文件，然后directory . 表示指定当前目**录。 然后就可以看到了。

>>rbreak

> rbreak 可以跟一个规则表达式。rbreak + 表达式的用法与grep + 表达式相似。即在所有与表达式匹配的函数入口都设置断点。
> 
> rbreak list_* 即在所有以 list_ 为开头字符的函数地方都设置断点。
> 
> rbreak ^list_ 功能与上同。

>>查看断点信息

> info break [break num ]
> 
> info break 可列出所有断点信息，info break 后也可设置要查看的break num如：
> 
> info break 1 列出断点号是1的断点信
> 
> # [Linux编程基础——GDB（设置断点）](http://www.cnblogs.com/TianFang/archive/2013/01/20/2868889.html)

http://www.cnblogs.com/ggjucheng/archive/2011/12/14/2288004.html

[GDB 找不到源代码](http://blog.csdn.net/xzgang/article/details/23437545)

有时候在用gdb调试程序的时候，发现gdb找不到源码。用list命令无效。

记住: gdb的调试信息中并不包含源码，只是包含了怎样去寻找源码，但是因为某种原因，比如你的源码转移了位置或者别的原因。你需要告诉gdb到哪里去寻找源码。这个通过directory命令来实现。

要查看当前gdb寻找源码的路径：

show directories

添加一个新的路径到查找路径：

dir  dirname

添加多个时，个dirname用: 分开。

详细见 ： http://ftp.gnu.org/old-gnu/Manuals/gdb-5.1.1/html_node/gdb_48.html

三、源文件  
GDB时,提示找不到源文件。  
需要做下面的检查:  
编译程序员是否加上了 -g参数 以包含debug信息。  
路径是否设置正确了。  
使用GDB的directory命令来设置源文件的目录。  

下面给一个调试/bin/ls的示例(ubuntu下)  
$ apt-get source coreutils  
$ sudo apt-get install coreutils-dbgsym  
$ gdb /bin/ls  
GNU gdb (GDB) 7.1-ubuntu  
(gdb) list main  
1192    ls.c: No such file or directory.  
in ls.c  
(gdb) directory ~/src/coreutils-7.4/src/  
Source directories searched: /home/hchen/src/coreutils-7.4:$cdir:$cwd  
(gdb) list main  
1192        }  
1193    }  
1194

http://blog.csdn.net/liujiayu2/article/details/50008131

http://blog.chinaunix.net/uid-9525959-id-2001805.html

https://wenku.baidu.com/view/504eaffa02768e9951e738c8.html

gdb打印STL容器

GDB中print方法并不能直接打印STL容器中保存的变量，

(gdb) p vec  
$1 = std::vector of length 1, capacity 1 = {2}

(

其实只要http://www.yolinux.com/TUTORIALS/src/dbinit_stl_views-1.03.txt这个文件保存为~/.**gdbinit** 就可以使用它提供的方法方便调试容器

现在我使用这个脚本提供的plist方法打印

下载 [http://www.yolinux.com/TUTORIALS/src/dbinit_stl_views-1.03.txt](http://www.yolinux.com/TUTORIALS/src/dbinit_stl_views-1.03.txt)   2. #cat dbinit_stl_views-1.03.txt >> ~/.gdbinit   3. 若正处于gdb中，运行命令:    (gdb) source ~/.gdbinit

1.  (gdb) plist lst  
2.  List size = 5   
3.  List type = std::list<int, std::allocator<int> > 
4.  Use plist <variable_name> <element_type> to see the elements in the list. 
5.  (gdb)  

详细查看list中的元素信息

1.  (gdb) plist lst int 
2.  elem[0]: $5 = 7  
3.  elem[1]: $6 = 1  
4.  elem[2]: $7 = 5  
5.  elem[3]: $8 = 9  
6.  elem[4]: $9 = 2  
7.  List size = 5   
8.  (gdb)   

脚本文件如下：把下列内容拷贝到一份txt文件里，然后重命名    .gdbinit（有点号，隐藏文件），然后拷贝到用户主目录下，~/.gdbinit，调用gdb就可以查看容器内容了。

`# include < vector >   
using namespace std ;   

int main( )   
{   
        vector < int > vec;   
        vec. push_back( 2) ;   
        vec. push_back( 3) ;   
        vec. push_back( 4) ;   
        return 0;   
}`

`编译：#g+ + - o bugging - g bugging. cpp`

`Breakpoint 1, main ( ) at bugging. cpp: 6   
6        vector < int > vec;   
( gdb) n   
7        vec. push_back( 2) ;   
( gdb)   
8        vec. push_back( 3) ;   
( gdb) pvector   
Prints std : : vector < T> information.   
Syntax: pvector < vector > < idx1> < idx2>   
Note: idx, idx1 and idx2 must be in acceptable range [ 0. . < vector > . size( ) - 1] .   
Examples:   
pvector v - Prints vector content, size, capacity and T typedef   
pvector v 0 - Prints element[ idx] from vector   
pvector v 1 2 - Prints elements in range [ idx1. . idx2] from vector   
( gdb) pvector vec   
elem[ 0] : $1 = 2   
Vector size = 1   
Vector capacity = 1   
Element type = int *   
( gdb) n   
9        vec. push_back( 4) ;   
( gdb)   
10        return 0;   
( gdb) pvector vec   
elem[ 0] : $2 = 2   
elem[ 1] : $3 = 3   
elem[ 2] : $4 = 4   
Vector size = 3   
Vector capacity = 4   
Element type = int *   
( gdb)`

5. 默认情况下gdb不能用[]查看stl容器的数据元素，提示如下错误：

`( gdb) print vec[ 0]   
One of the arguments you tried to pass to operator [ ] could not be converted to what the function wants.`

Gdb保存断点:

 1. 保存断点

    先用info b 查看一下目前设置的断点，使用save breakpoint命令保存到指定的文件，这里我使用了和进程名字后面加bp后缀，你可以按你的喜好取名字。

    我使用的是save breakpoint fig8.3.bp ，在当前目录下，会生成一个fig8.3.bp的文件，里面存储的就是断点的命令。

![](https://img-blog.csdn.net/20150513104618480?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2Vpd2FuZ2NoYW9f/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

    2. 读取断点

    注意，在gdb已经加载进程的过程中，是不能够读取断点文件的，必须在gdb加载文件的命令中指定断点文件，具体就是使用-x参数。例如，我需要调试fig8.3这个文件，指定刚才保存的断点文件fig8.3.bp。

    我使用的是gdb fig8.3 -x fig8.3.bp

我又仔细看了一下上面的对于“save breakpoints”的注释，在另一**个gdb session中可以使用“source”命令restore（恢复，**还原）它们（断点）。测试一下：先删除所有断点，然后使用source恢复所有保存的断点，如下：

1.  (gdb) D                                                #删除所有断点  
2.  Delete all breakpoints? (y or n) y  
3.  (gdb) info b                                           #查看断点  
4.  No breakpoints or watchpoints.  
5.  (gdb) source gdb.cfg                                   #加载断点  
6.  Breakpoint 9 at 0x40336d: file RecvMain.cpp, line 290.  
7.  Breakpoint 10 at 0x2aaab049c7ef: file CRealCreditEventAb.cpp, line 80.  
8.  ……  

11.  (gdb) info b                                           #查看断点  
12.  Num     Type           Disp Enb Address            What  
13.  9       breakpoint     keep y   0x000000000040336d in main(int, char**) at RecvMain.cpp:290  
14.  10      breakpoint     keep y   0x00002aaab049c7ef in CRealCreditEventAb::LoopEventCreditCtrl(int)  
15.                                                 at CRealCreditEventAb.cpp:80  
16.  ……  

## set命令

![再见](http://static.blog.csdn.net/xheditor/xheditor_emot/default/bye.gif)![吐舌头](http://static.blog.csdn.net/xheditor/xheditor_emot/default/tongue.gif)该命令可以改变一个变量的值。

![再见](http://static.blog.csdn.net/xheditor/xheditor_emot/default/bye.gif)![吐舌头](http://static.blog.csdn.net/xheditor/xheditor_emot/default/tongue.gif)_set variable varname = value_

![再见](http://static.blog.csdn.net/xheditor/xheditor_emot/default/bye.gif)![吐舌头](http://static.blog.csdn.net/xheditor/xheditor_emot/default/tongue.gif)varname是变量名称，value是变量的新值。

![](https://images2017.cnblogs.com/blog/277239/201711/277239-20171120193831180-1537404272.png)

（1）修改变量值：

a. **print**_v=value_:　修改变量值的同时，把修改后的值显示出来

b. **set** [**var**]_v=value_:　修改变量值，需要注意如果变量名与GDB中某个**set**命令中的关键字一样的话，前面加上**var**关键字

**条件断点**

设置一个条件断点，条件由cond指定；在gdb每次执行到此  
断点时,cond都被计算。当cond的值为非零时，程序在断点处停止。

用法：

**break [break-args] if (condition)**

例如：

break main if argc > 1  
break 180 if (string == NULL && i < 0)  
break test.c:34 if (x & y) == 1  
break myfunc if i % (j+3) != 0  
break 44 if strlen(mystring) == 0  
**b 10 if ((int)$gdb_strcmp(a,"chinaunix") == 0)  
b 10 if ((int)aa.find("dd",0) == 0)**

**condition**

可以在我们设置的条件成立时，自动停止当前的程序，先使用break(或者watch也可以）设置断点，  
然后用condition来修改这个断点的停止（就是断）的条件。

用法：  
**condition <break_list> (conditon)**

例如：  
cond 3 i == 3

**condition 2 ((int)strstr($r0,".plist") != 0)**

**ignore**

如果我们不是想根据某一条件表达式来停止，而是想断点自动忽略前面多少次的停止，从某一次开始  
才停止，这时ignore就很有用了。

用法：

**ignore <break_list> count**

上面的命令行表示break_list所指定的断点号将被忽略count次。

例如：

ignore 1 100，表示忽略断点1的前100次停止

  Watchpoints相关命令：（W**atchpoint的作用是让程序在某个表达式的值发生变化的时候停止运行，达到‘监视’该表达式的目**的）

（1）设置watchpoints:

a. **watch** _expr_: 设置写watchpoint，当应用程序写_expr_,修改其值时，程序停止运行

b. **rwatch** _expr_: 设置读watchpoint，当应用程序读表达式_expr_时，程序停止运行

c. **awatch** _expr_: 设置读写watchpoint, 当应用程序读或者写表达式_expr_时，程序都会停止运行

（2）**info watchpoints**:查看当前调试的程序中设置的watchpoints相关信息

（3）watchpoints和breakpoints很相像，都有enable/disabe/delete等操作，使用方法也与breakpoints的类似

**对断点的控制除了建立和删除外，还可以通过使能和禁止来控制，后一种方法更灵活。**  

断点的四种使能操作：  

enable [breakpoints] [range...] 完全使能  
enable                //激活所有断点  
enable 4            //激活4断点  
enable 5-6            //激活5～6断点  
disable [breakpoints] [range...] 禁止  
用法举例：  
**diable                //禁止所有断点**  
disble 2            //禁止第二个断点  
disable 1-5            //禁止第1到第5个断点  

enable once [breakpoints] [range...] 使能一次，触发后禁止  
enable delete [breakpoints] [range...]使能一次，触发后删除

1、程序运行参数。  
set args 可指定运行时参数。（如：set args -f 20 -t 40）  
show args 命令可以查看设置好的运行参数。  

2、运行环境。  
path 可设定程序的运行路径。  
show paths 查看程序的运行路径。  
set environment varname [=value] 设置环境变量。如：set env USER=user  
show environment [varname] 查看环境变量。  

3、工作目录。  
cd 相当于shell的cd命令。  
pwd 显示当前的所在目录。

# 从一个实例来认识GDB与高效调试

http://blog.csdn.net/tonywearme/article/details/41447007

http://blog.csdn.net/lxl584685501/article/details/45575717

GDB的全称是GNU project debugger，是类Unix系统上一个十分强大的调试器。这里通过一个简单的例子（插入算法）来介绍如何使用gdb进行调试，特别是如何通过中断来高效地找出死循环；我们还可以看到，在修正了程序错误并重新编译后，我们仍然可以通过原先的GDB session进行调试（而不需要重开一个GDB），这避免了一些重复的设置工作；同时，在某些受限环境中（比如某些实时或嵌入式系统），往往只有一个Linux字符界面可供调试。这种情况下，可以使用job在代码编辑器、编译器（编译环境）、调试器之间做到无缝切换。这也是高效调试的一个方法。

先来看看这段插入排序算法（a.cpp），里面有一些错误。

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) View Code

码就不分析了，稍微花点时间应该就能明白。你能发现几个错误？

使用gcc编译：

gcc -g -Wall -o insert_sort a.cpp

"-g"告诉gcc在二进制文件中加入调试信息，如符号表信息，这样gdb在调试时就可以把地址和函数、变量名对应起来。在调试的时候你就可以根据变量名查看它的值、在源代码的某一行加一个断点等，这是调试的先决条件。“-Wall”是把所有的警告开关打开，这样编译时如果遇到warning就会打印出来。一般情况下建议打开所有的警告开关。

运行编译后的程序（./insert_sort），才发现程序根本停不下来。上调试器！（有些bug可能一眼就能看出来，这里使用GDB只是为了介绍相关的基本功能）

TUI模式

现在版本的GDB都支持所谓的终端用户接口模式（Terminal User Interface），就是在显示GDB命令行的同时可以显示源代码。好处是你可以随时看到当前执行到哪条语句。之所以叫TUI，应该是从GUI抄过来的。注意，可以通过ctrl + x + a来打开或关闭TUI模式。

gdb -tui ./insert_sort  

  

![](https://img-blog.csdn.net/20141124194010000)

死循环

进入GDB后运行run命令，传入命令行参数，也就是要排序的数组。当然，程序也是停不下来：

![](https://img-blog.csdn.net/20141124170951854)

为了让程序停下来，我们可以发送一个中断信号（ctrl + c），GDB捕捉到该信号后会挂起被调试进程。注意，什么时候发送这个中断有点技巧，完全取决于我们的经验和程序的特点。像这个简单的程序，正常情况下几乎立刻就会执行完毕。**如果感觉到延迟就说明已经发生了死循环（或其他什么），这时候发出中断肯定落在死循环的循环体中**。这样我们才能通过检查上下文来找到有用信息。大型程序如果正常情况下就需要跑个几秒钟甚至几分钟，那么你至少需要等到它超时后再去中断。

![](https://img-blog.csdn.net/20141124200425588)

![](https://img-blog.csdn.net/20141124200436929)

此时，程序暂停在第44行（第44行还未执行），TUI模式下第44行会被高亮显示。我们知道，这一行是某个死循环体中的一部分。

因为暂停的代码有一定的随机性，可以多运行几次，看看每次停留的语句有什么不同。后面执行run命令的时候可以不用再输入命令行参数（“12 5”），GDB会记住。还有，再执行run的时候GDB会问是否重头开始执行程序，当然我们要从头开始执行。

基本确定位置后（如上面的44行），因为这个程序很小，可以单步（step）一条条语句查看。不难发现问题出在第24行，具体的步骤就省略了。

无缝切换

在编码、调试的时候，除非你有集成开发环境，一般你会需要打开三个窗口：代码编辑器（比如很多人用的VIM）、编译器（新开的窗口运行gcc或者make命令、执行程序等）、调试器。集成开发环境当然好，但某些倒闭的场合下你无法使用任何GUI工具，比如一些仅提供字符界面的嵌入式设备——你只有一个Linux命令行可以使用。显然，如果在VIM中修改好代码后需要先关闭VIM才能敲入gcc的编译命令，或者调试过程中发现问题需要先关闭调试器才能重新打开VIM修改代码、编译、再重新打开调试器，那么不言而喻，这个过程太痛苦了！

好在可以通过Linux的作业管理机制，通过ctrl + z把当前任务挂起，返回终端做其他事情。通过jobs命令可以查看当前shell有哪些任务。比如，当我暂停GDB时，jobs显示我的VIM编辑器进程与GDB目前都处于挂起状态。

![](https://img-blog.csdn.net/20141124204748577)

以下是些相关的命令，比较常用

fg %1         // 打开VIM，1是VIM对应的作业号

fg %2         // 打开GDB

bg %1         // 让VIM到后台运行

kill %1 && fg // 彻底杀死VIM进程

**GDB的“在线刷新”**

好了，刚才介绍了无缝切换，那我们可以在不关闭GDB的情况下（注意，ctrl + z不是关闭GDB这个进程，只是挂起）切换到VIM中去修改代码来消除死循环（把第24行的“if (num_y = 0)" 改成"if (num_y == 0)"）。动作序列可以是：

ctrl + z // 挂起GDB

jobs     // 查看VIM对应的作业号，假设为1

fg %1    // 进入VIM，修改代码..

ctrl + z // 修改完后挂起VIM

gcc -g -Wall -o insert_sort a.cpp // 重新编译程序

fg %2    // 进入GDB，假设GDB的作业号为2

现在，我们又返回GDB调试界面了！但在调试前还有一步，如何让GDB识别新的程序（因为程序已经重新编译）？只要再**次运行run就可以了。因为GDB没有关闭，所以之前设置的断点、运行run时传入的命令行参数等还保留着，不需要**重新输入。很好用吧！

gdb Tui窗口:

**TUI模式**

现在版本的GDB都支持所谓的终端用户接口模式（**T**erminal **U**ser **I**nterface），就是在显示GDB命令行的同时可以显示源代码。好处是你可以随时看到当前执行到哪条语句。之所以叫TUI，应该是从GUI抄过来的。注意，可以通过ctrl + x + a来打开或关闭TUI模式。

gdb -tui ./insert_sort

方法1:使用gdb -tui

方法二: 直接使用gdb调试代码，在需要的时候使用切换键 `ctrl+x a`调出gdbtui。

调试代码的时候,只能看到下一行,每次使用list非常烦,不知道当前代码的context 

[http://beej.us/guide/bggdb/#compiling](http://beej.us/guide/bggdb/#compiling)

简单来说就是在以往的gdb开始的时候添加一个-tui选项.有的版本已经有gdbtui这个程序了

在linux自带的终端里是正常显示的,但是在securecrt里面,可能由于编码的问题,边缘会有些乱码,不过不影响使用(如果你的程序有错误输出,会扰乱整个界面,所以在调试的时候,建议添加2>/dev/null,这样的话基本可用) 

启动gdb之后,上面是src窗口,下面是cmd窗口,默认focus在src窗口的,这样的话上下键以及pagedown,pageup都是在移动显示代码,并不显示上下的调试命令.这个时候要切换focus,具体可简单参见

(gdb) info win  查看当前focus
        SRC     (36 lines)  <has focus>
        CMD     (18 lines)
(gdb) fs next  切换focus
Focus set to CMD window.
(gdb) info win 
        SRC     (36 lines)
        CMD     (18 lines)  <has focus>
(gdb) fs SRC  切换指定focus
Focus set to SRC window.
(gdb) 

(Window names are case in-sensitive.)

**To start in neato and highly-recommended GUI mode**, start the debugger with gdb -tui. (For many of the examples, below, I show the output of gdb's dumb terminal mode, but in real life I use TUI mode exclusively.)

And here is a screenshot of what you'll see, approximately:

![](https://images2017.cnblogs.com/blog/277239/201712/277239-20171205220944863-2005191246.png)

gdb设置程序参数：

有三种方法可以指定程序运行的参数，第一种方法是在命令行上直接指定；第二种方法是通过run命令提供程序运行时的参数；第三种方法是通过set args命令指定程序的参数

第一种方法：为程序传递参数5

```
root@guo-virtual-machine:~/debug# gdb --args factorial 5
```

-   1

第二种方法：为程序传递参数5

```
(gdb) run 5

```

-   1
-   2
-   3

第三种方法：为程序传递参数5

```sql
(gdb) set args 5
(gdb) run
Starting program: /root/debug/factorial 5
warning: no loadable sections found in added symbol-file system-supplied DSO at 0x7ffff7ffa000
Factorial of 5 is 120
```

-   1
-   2
-   3
-   4
-   5
-   6

查看指定给程序的参数通过show args

```sql
(gdb) show args
Argument list to give program being debugged when it is started is "5".
(gdb) 
```

显得尤其重要。通过gdb的堆栈跟踪，可以看到所有已调用的函数列表，以及

每个函数在栈中的信息。  
---------------------------------------------------------------------------------  
一，简单实例。  

1.  #include <stdio.h>

3.  int sum(int m,int n)
4.  {
5.      int i = 3;
6.      int j = 4;
7.     return m+n;    
8.  }

10.  int main(void)
11.  {
12.      int m = 10;
13.      int n = 9;
14.      int ret = 0;

16.      ret = sum(m,n);

18.      printf("ret = %d\n",ret);
19.      return 0;    
20.  }

1.  (gdb) bt
2.  #0 sum (m=10, n=9) at sum.c:5
3.  #1 0x08048418 in main () at sum.c:16

每次有函数调用，在栈上就会生成一个栈框（stack frame），也就是一个数据  
单元用来描述该函数，描述函数的地址，参数，还有函数的局部变量的值等信息。  
使用bt命令就可以把这个栈的调用信息全部显示出来。  

由上面的显示结果可以看出，栈上有两个栈框(stack frame),分别用来描述函数

main和函数sum.前面的#0表示sum函数栈框的标号。#1表示main函数栈框的标号。

最新的栈框标号为0.main函数栈框标号最大。  

1.  (gdb) frame 1
2.  #1 0x08048418 in main () at sum.c:16
3.  16 ret = sum(m,n);

frame 1 表示选择栈框1，也就是选择了main函数的栈框，因为我这时候想查看

main函数的信息。  


1.  (gdb) info locals
2.  m = 10
3.  n = 9
4.  ret = 0

这时候可以通过info locals查看main函数栈框里面局部变量的值。  
-----------------------------------------------------------------------------------  
二，使用gdb堆栈跟踪很方面调试递归程序。  


1.  #include <stdio.h>

3.  long long func(int n)
4.  {
5.      int i = 0;

7.      if (n > 20) {
8.          printf("n too large!\n");
9.          return -1;
10.      }
11.      if (n == 0) 
12.          return 1;
13.      else {
14.          i = n * func(n-1);
15.          return i;
16.      }
17.  }

19.  int main(void)
20.  {
21.      long long ret;
22.      ret = func(10);
23.      printf("ret = %lld\n",ret);
24.      return 0;
25.  }

1.  (gdb) bt
2.  #0 func (n=7) at test.c:7
3.  #1 0x0804843f in func (n=8) at test.c:14
4.  #2 0x0804843f in func (n=9) at test.c:14
5.  #3 0x0804843f in func (n=10) at test.c:14
6.  #4 0x08048469 in main () at test.c:22

如上所示，可以很清楚地看到递归深入到了第几层，以及该层局部变量值的情况。  
---------------------------------------------------------------------------------  
三，gdb使用手册上有一块专门说如何查看堆栈，翻译后的文档如下：  
 [![](http://blog.chinaunix.net/blog/image/attachicons/rar.gif) gdb查看堆栈.rar](http://blog.chinaunix.net/attachment/attach/27/03/34/9127033491e445a65ec10a63d0e2ebbd2c83b2ba0b.rar)   

----------------------------------------------------------------------------------

参考： [http://www.ibm.com/developerworks/cn/linux/sdk/gdb/index.html](http://www.ibm.com/developerworks/cn/linux/sdk/gdb/index.html) 

### 打印一个类的成员

**ptype obj/class/struct**

查看obj/class/struct的成员，但是会把基类指针指向的派生类识别为基类

**set print object on**

这个选项可以看到派生对象的真实类名，虽然ptype也可以打印出对象

**set print pretty on**

以树形打印对象的成员，可以清晰展示继承关系，设置为off时对象较大时会显示“一坨”

如调试mysql Item类的派生类对象时会这样显示：

![](https://images2018.cnblogs.com/blog/277239/201804/277239-20180413215040953-601439468.jpg)

**set print vtbl on**

用比较规整的格式来显示虚函数表

推荐设置这两个：

set print object on

set print pretty on

 gdb中查看字符串，地址的操作，数据类型  
比始有一个int型的变量i，相要知道他的相关信息，可以  
(gdb) print i  
打印出变量i的当前值  
(gdb)x &i  
与上面的命令等价。  

如果有x命令看时，需要看一片内存区域，**（如果某个地方的值为0，用x时会自动截断**了）  
(gdb) x/16bx address  
单字节16进制打印address地址处的长度为16的空间的内存，16表示空间长度，不是16进制，x表示16进制，b表示byte单字节  

gdb看变量是哪个数据类型   
(gdb) whatis i      （whatis和ptype类似)  
即可知道i是什么类型的变量  

  

gdb调试中，对于动态创建的数组，比如 int * x;  这个时候不能像静态数组那样通过p x 就可以打印出整个数组。如果想要打印出整个数组，可以通过创建一个人工数组来解决这个问题。其一般形式为：

    [*pointer@number_of_elements](mailto:*pointer@number_of_elements)

gdb还允许在适当的时候使用类型强制转换，比如：

    (gdb) p (int[25]*x

gdb的ptype命令可以很方便的快速浏览类或结构体的结构。

    (gdb) info locals命令得到当前栈中所有局部变量的值列表。

print和display的高级选项

    p /x y 会以十六进制的格式显示变量，而不是十进制的形式。其它常用的格式为c表示字符，s表示字符串，f表示浮点。

可以临时禁用某个显示项。例如

    (gdb) dis disp 1

查看条目好，命令是

    (gdb) info disp

重新启用条目，命令是

    (gdb) enable disp 1

完全删除显示的条目，命令是

    (gdb) undisp 1

在gdb中设置变量

在单步调试程序的中间使用调试器设置变量的值是非常有用的。命令是

    (gdb) set x = 12           ( 前提是x必须在程序中已经声明了）

set var a=3  

可以通过gdb的set args 命令设置程序的命令行参数。

设置“方便变量”，命令是

    (gdb) set $q = p  

用来记录特定节点的历史，这个变量q不会去改变自己的地址。

（

9.GDB环境变量

 你可以在GDB的调试环境中定义自己的变量，用来保存一些调试程序中的运行数据。要定义一个GDB的变量很简单只需。使用GDB的set命令。GDB的环境变量和UNIX一样，也是以$起头。如：  
      
    set $foo = *object_ptr  
      
    使用环境变量时，GDB会在你第一次使用时创建这个变量，而在以后的使用中，则直接对其賦值。环境变量没有类型，你可以给环境变量定义任一的类型。包括结构体和数组。  
      
    show convenience   
        该命令查看当前所设置的所有的环境变量。  
          
    这是一个比较强大的功能，环境变量和程序变量的交互使用，将使得程序调试更为灵活便捷。例如：  
      
        set $i = 0  
        print bar[$i++]->contents  
      
    于是，当**你就不必，print bar[0]->contents, print bar[1]->contents地输入命令了。输入这样的命令后，只用敲回车，重复执行上一条语句，环境变量会自动累加，从而完**成逐个输出的功能。

）

在调用gdb时，可以指定“启动文件”。例如：

    gdb –command=z x

表示要在可执行文件x上运行gdb，首先要从文件z中读取命令。   

    (gdb) jump 34

程序直接跳到34行。

使用strace命令，可以跟踪系统做过的所有系统调用。

进程和线程的主要区别是：与进程一样，虽然每个线程有自己的局部变量，但是多线程环境中父程序的全局变量被所有线程共享，并作为在线程之间通信的主要方法。

可以通过命令 ps axH 来查看系统上当前的所有进程和线程。

## 使用 ptype 检查类型

ptype 命令可能是我最喜爱的命令。它告诉你一个 C 语言表达式的类型。

(gdb) ptype i

type = int

(gdb) ptype &i

type = int *

(gdb) ptype main

type = int (void)

C 语言中的类型可以变得很复杂，但是好在 ptype 允许你交互式地查看他们。

调试已运行的程序

两种方法：   
　　(1)在UNIX**下用ps查看正在运行的程序的PID（进程ID），然后用gdb PID格式挂接正在运**行的程序。   
　　(2)先用gdb 关联上源代码，并进行gdb，在gdb中用attach命令来挂接进程的PID。并用detach来取消挂接的进程。

  detach:

   **当你调试结束之后,可以使用该命令断开进程与gdb的连接(结束gdb对进程的控制),在这个命令执行之后,你所调试的那个进程将继续运**行;

内存查看命令

可以使用examine命令(简写是x)来查看内存地址中的值。x命令的语法如下所示：

x/<n/f/u> <addr>

其中，n是一个正整数，表示需要显示的内存单元的个数。

           f 表示显示的格式：x 按十六进制格式显示变量

>                                  d 按十进制格式显示变量
> 
>                                  u 按十六进制格式显示无符号整型
> 
>                                  o 按八进制格式显示变量
> 
>                                  t 按二进制格式显示变量
> 
>                                  c 按字符格式显示变量
> 
>                                  f 按浮点数格式显示变量
> 
> u 表示从当前地址往后请求的字节数，如果不指定的话，GDB默认是4个bytes。u参数可以用下面的字符来代替，b表示单字节，h表示双字节，w表示四字 节，g表示八字节。当我们指定了字节长度后，GDB会从指内存定的内存地址开始，读写指定字节，并把其当作一个值取出来。
> 
> 命令：x/3uh 0x54320 表示，从内存地址0x54320读取内容，h表示以双字节为一个单位，3表示输出三个单位，u表示按十六进制显示。
> 
>  http://blog.jobbole.com/87482/
> 
>  https://www.cnblogs.com/muhe221/articles/4846680.html
> 
> ## watch
> 
> watch [-l|-location] expr [thread threadnum] [mask maskvalue]
> 
>      -l 与 mask没有仔细研究，thread threadnum 是在多线程的程序中限定只有被线程号是threadnum的线程修改值后进入断点。
> 
> 经常用到的如下命令：
> 
>      watch <expr>
> 
>      为表达式（变量）expr设置一个观察点。变量量表达式值有变化时，马上停住程序。
> 
>      表达式可以是一个变量
> 
>      **例如：watch** value_a
> 
>    表达式可以是一个地址：
> 
>  例如：watch *(int *)0x12345678 可以检测4个字节的内存是否变化。
> 
>      表达式可以是一个复杂的语句表达式：
> 
>      例如：watch a*b + c/d
> 
> watch 在有些操作系统支持硬件观测点，硬件观测点的运行速度比软件观测点的快。如果系统支持硬件观测的话，当设置观测点是会打印如下信息：
> 
>      Hardware watchpoint num: expr
> 
> watch两个变种 rwatch，awatch，这两个命令只支持硬件观测点如果系统不支持硬件观测点会答应出不支持这两个命令的信息
> 
> rwatch <expr>
> 
>     当表达式（变量）expr被读时，停住程序。
> 
>     awatch <expr>
> 
>     当表达式（变量）的值被读或被写时，停住程序。
> 
>     info watchpoints
> 
>     列出当前所设置了的所有观察点。
> 
>      watch 所设置的断点也可以用控制断点的命令来控制。如 disable、enable、delete等。 
> 
> 1：定位某变量/内存地址 何时被修改
> 
> a为待观察的变量
> 
> gdb> watch *(long*)a  
> gdb> watch *(long*)(a+4)  
> gdb> watch *(long*)(a+8)  
> 2：查看数组的值。
> 
> 编程时：array[i]
> 
> 用GDB查看时，用 p array+i即可。
> 
> 3：善于使用$
> 
> http://blog.chinaunix.net/uid-20801390-id-3236840.html
> 
> GDB内存断点(Memory break)的使用举例
> 
> 本文是一篇使用GDB设置内存断点的例子。
> 
> 1. 源程序
> 
> 文件名：testMemBreak.c
> 
> #include <stdio.h>
> 
> #include <string.h>
> 
> int main()
> 
> {
> 
>     int i,j;
> 
>     char buf[256]={0};
> 
>     char* pp = buf;
> 
>     printf("buf addr= 0x%x/r/n",buf);
> 
>     for(i=0;i<16;i++)
> 
>     {
> 
>         printf("addr = 0x%x ~ 0x%x/r/n",pp+i*16,pp+i*16+15);
> 
>         for(j=0;j<16;j++)
> 
>             *(pp+i*16+j)=i*16+j;
> 
>     }
> 
>     printf("ASCII table:/n");
> 
>     for(i=0;i<16;i++)
> 
>     {
> 
>         for(j=0;j<16;j++)
> 
>             printf("%c  ", *(pp+i*16+j));
> 
>         printf("/n");
> 
>     }
> 
>     return 0;
> 
> }
> 
> 即完成ASCII码的初始化并打印出来。
> 
> 要使用的GDB命令，如下。
> 
> (gdb) help watch
> 
> Set a watchpoint for an expression.
> 
> A watchpoint stops execution of your program whenever the value of
> 
> an expression changes.
> 
> (gdb) help rwatch
> 
> Set a read watchpoint for an expression.
> 
> A watchpoint stops execution of your program whenever the value of
> 
> an expression is read.
> 
> (gdb) help awatch
> 
> Set a watchpoint for an expression.
> 
> A watchpoint stops execution of your program whenever the value of
> 
> an expression is either read or written.
> 
> (gdb)
> 
> 2. 调试过程
> 
> 2.1 设置断点并启动程序完成初始化
> 
> 启动程序对其进行初始化，并使用display自动显示buf的地址。
> 
> $ gcc -g -o membreak testMemBreak.c
> 
> $ gdb ./membreak.exe
> 
> GNU gdb 6.3.50_2004-12-28-cvs (cygwin-special)
> 
> Copyright 2004 Free Software Foundation, Inc.
> 
> GDB is free software, covered by the GNU General Public License, and you are
> 
> welcome to change it and/or distribute copies of it under certain conditions.
> 
> Type "show copying" to see the conditions.
> 
> There is absolutely no warranty for GDB.  Type "show warranty" for details.
> 
> This GDB was configured as "i686-pc-cygwin"...
> 
> (gdb) l
> 
> 1       #include <stdio.h>
> 
> 2       #include <string.h>
> 
> 3
> 
> 4       int main()
> 
> 5       {
> 
> 6           int i,j;
> 
> 7           char buf[256]={0};
> 
> 8           char* pp = buf;
> 
> 9
> 
> 10          printf("buf addr= 0x%x/r/n",buf);
> 
> (gdb)
> 
> 11          for(i=0;i<16;i++)
> 
> 12          {
> 
> 13              printf("addr = 0x%x ~ 0x%x/r/n",pp+i*16,pp+i*16+15);
> 
> 14              for(j=0;j<16;j++)
> 
> 15                  *(pp+i*16+j)=i*16+j;
> 
> 16          }
> 
> 17
> 
> 18          printf("ASCII table:/n");
> 
> 19          for(i=0;i<16;i++)
> 
> 20          {
> 
> (gdb)
> 
> 21              for(j=0;j<16;j++)
> 
> 22                  printf("%c  ", *(pp+i*16+j));
> 
> 23              printf("/n");
> 
> 24          }
> 
> 25
> 
> 26          return 0;
> 
> 27      }
> 
> (gdb) b 10
> 
> Breakpoint 1 at 0x4010ae: file testMemBreak.c, line 10.
> 
> (gdb) r
> 
> Starting program: /cygdrive/e/study/programming/linux/2009-12-28 testMemBreak/membreak.exe
> 
> Breakpoint 1, main () at testMemBreak.c:10
> 
> 10          printf("buf addr= 0x%x/r/n",buf);
> 
> (gdb) step
> 
> buf addr= 0x22cb70
> 
> 11          for(i=0;i<16;i++)
> 
> (gdb) p buf
> 
> $1 = '/0' <repeats 255 times>
> 
> (gdb) p &buf
> 
> $2 = (char (*)[256]) 0x22cb70
> 
> (gdb) display &buf
> 
> 1: &buf = (char (*)[256]) 0x22cb70
> 
> (gdb) p/x *0x22cb70@64
> 
> $3 = {0x0 <repeats 64 times>}
> 
> (gdb) x/64w 0x22cb70
> 
> 0x22cb70:       0x00000000      0x00000000      0x00000000      0x00000000
> 
> 0x22cb80:       0x00000000      0x00000000      0x00000000      0x00000000
> 
> 0x22cb90:       0x00000000      0x00000000      0x00000000      0x00000000
> 
> 0x22cba0:       0x00000000      0x00000000      0x00000000      0x00000000
> 
> 0x22cbb0:       0x00000000      0x00000000      0x00000000      0x00000000
> 
> 0x22cbc0:       0x00000000      0x00000000      0x00000000      0x00000000
> 
> 0x22cbd0:       0x00000000      0x00000000      0x00000000      0x00000000
> 
> 0x22cbe0:       0x00000000      0x00000000      0x00000000      0x00000000
> 
> 0x22cbf0:       0x00000000      0x00000000      0x00000000      0x00000000
> 
> 0x22cc00:       0x00000000      0x00000000      0x00000000      0x00000000
> 
> 0x22cc10:       0x00000000      0x00000000      0x00000000      0x00000000
> 
> 0x22cc20:       0x00000000      0x00000000      0x00000000      0x00000000
> 
> 0x22cc30:       0x00000000      0x00000000      0x00000000      0x00000000
> 
> 0x22cc40:       0x00000000      0x00000000      0x00000000      0x00000000
> 
> 0x22cc50:       0x00000000      0x00000000      0x00000000      0x00000000
> 
> 0x22cc60:       0x00000000      0x00000000      0x00000000      0x00000000
> 
> p/x *0x22cb70@64：以16进制方式显示0x22cb70开始的64个双字组成的数组，实际上就是256个字节的数组，只是默认以双字显示。
> 
> 可以看见，buf内存块中的所有数据被初始化为0。
> 
> 2.2 在buf+80处设置内存断点
> 
> 设置断点后，运行程序，使之停在对该内存写的动作上。
> 
> (gdb) watch *(int*)0x22cbc0
> 
> Hardware watchpoint 2: *(int *) 2280384
> 
> (gdb) c
> 
> Continuing.
> 
> addr = 0x22cb70 ~ 0x22cb7f
> 
> addr = 0x22cb80 ~ 0x22cb8f
> 
> addr = 0x22cb90 ~ 0x22cb9f
> 
> addr = 0x22cba0 ~ 0x22cbaf
> 
> addr = 0x22cbb0 ~ 0x22cbbf
> 
> addr = 0x22cbc0 ~ 0x22cbcf
> 
> Hardware watchpoint 2: *(int *) 2280384
> 
> Old value = 0
> 
> New value = 80
> 
> main () at testMemBreak.c:14
> 
> 14              for(j=0;j<16;j++)
> 
> 1: &buf = (char (*)[256]) 0x22cb70
> 
> (gdb) p i
> 
> $4 = 5
> 
> (gdb) p j
> 
> $5 = 0
> 
> (gdb) info breakpoints
> 
> Num Type           Disp Enb Address    What
> 
> 1   breakpoint     keep y   0x004010ae in main at testMemBreak.c:10
> 
>         breakpoint already hit 1 time
> 
> 2   hw watchpoint  keep y              *(int *) 2280384
> 
>         breakpoint already hit 1 time
> 
> (gdb) delete 1
> 
> (gdb) delete 2
> 
> (gdb) b 18
> 
> Breakpoint 3 at 0x40113b: file testMemBreak.c, line 18.
> 
> (gdb) info breakpoints
> 
> Num Type           Disp Enb Address    What
> 
> 3   breakpoint     keep y   0x0040113b in main at testMemBreak.c:18
> 
> (gdb) p/x *0x22cb70@64
> 
> $6 = {0x3020100, 0x7060504, 0xb0a0908, 0xf0e0d0c, 0x13121110, 0x17161514, 0x1b1a1918, 0x1f1e1d1c, 0x23222120,
> 
>   0x27262524, 0x2b2a2928, 0x2f2e2d2c, 0x33323130, 0x37363534, 0x3b3a3938, 0x3f3e3d3c, 0x43424140, 0x47464544,
> 
>   0x4b4a4948, 0x4f4e4d4c, 0x50, 0x0 <repeats 43 times>}
> 
> (gdb) x/64w 0x22cb70
> 
> 0x22cb70:       0x03020100      0x07060504      0x0b0a0908      0x0f0e0d0c
> 
> 0x22cb80:       0x13121110      0x17161514      0x1b1a1918      0x1f1e1d1c
> 
> 0x22cb90:       0x23222120      0x27262524      0x2b2a2928      0x2f2e2d2c
> 
> 0x22cba0:       0x33323130      0x37363534      0x3b3a3938      0x3f3e3d3c
> 
> 0x22cbb0:       0x43424140      0x47464544      0x4b4a4948      0x4f4e4d4c
> 
> 0x22cbc0:       0x00000050      0x00000000      0x00000000      0x00000000
> 
> 0x22cbd0:       0x00000000      0x00000000      0x00000000      0x00000000
> 
> 0x22cbe0:       0x00000000      0x00000000      0x00000000      0x00000000
> 
> 0x22cbf0:       0x00000000      0x00000000      0x00000000      0x00000000
> 
> 0x22cc00:       0x00000000      0x00000000      0x00000000      0x00000000
> 
> 0x22cc10:       0x00000000      0x00000000      0x00000000      0x00000000
> 
> 0x22cc20:       0x00000000      0x00000000      0x00000000      0x00000000
> 
> 0x22cc30:       0x00000000      0x00000000      0x00000000      0x00000000
> 
> 0x22cc40:       0x00000000      0x00000000      0x00000000      0x00000000
> 
> 0x22cc50:       0x00000000      0x00000000      0x00000000      0x00000000
> 
> 0x22cc60:       0x00000000      0x00000000      0x00000000      0x00000000
> 
> 查看buf内存块,可以看见，程序按我们希望的在运行，并在buf+80处停住。此时，程序正试图向该单元即0x22cbc0写入80。
> 
> 2.3 使用continue执行程序直至结束
> 
> (gdb) c
> 
> Continuing.
> 
> addr = 0x22cbd0 ~ 0x22cbdf
> 
> addr = 0x22cbe0 ~ 0x22cbef
> 
> addr = 0x22cbf0 ~ 0x22cbff
> 
> addr = 0x22cc00 ~ 0x22cc0f
> 
> addr = 0x22cc10 ~ 0x22cc1f
> 
> addr = 0x22cc20 ~ 0x22cc2f
> 
> addr = 0x22cc30 ~ 0x22cc3f
> 
> addr = 0x22cc40 ~ 0x22cc4f
> 
> addr = 0x22cc50 ~ 0x22cc5f
> 
> addr = 0x22cc60 ~ 0x22cc6f
> 
> Breakpoint 3, main () at testMemBreak.c:18
> 
> 18          printf("ASCII table:/n");
> 
> 1: &buf = (char (*)[256]) 0x22cb70
> 
> (gdb) p/x *0x22cb70@64
> 
> $7 = {0x3020100, 0x7060504, 0xb0a0908, 0xf0e0d0c, 0x13121110, 0x17161514, 0x1b1a1918, 0x1f1e1d1c, 0x23222120,
> 
>   0x27262524, 0x2b2a2928, 0x2f2e2d2c, 0x33323130, 0x37363534, 0x3b3a3938, 0x3f3e3d3c, 0x43424140, 0x47464544,
> 
>   0x4b4a4948, 0x4f4e4d4c, 0x53525150, 0x57565554, 0x5b5a5958, 0x5f5e5d5c, 0x63626160, 0x67666564, 0x6b6a6968,
> 
>   0x6f6e6d6c, 0x73727170, 0x77767574, 0x7b7a7978, 0x7f7e7d7c, 0x83828180, 0x87868584, 0x8b8a8988, 0x8f8e8d8c,
> 
>   0x93929190, 0x97969594, 0x9b9a9998, 0x9f9e9d9c, 0xa3a2a1a0, 0xa7a6a5a4, 0xabaaa9a8, 0xafaeadac, 0xb3b2b1b0,
> 
>   0xb7b6b5b4, 0xbbbab9b8, 0xbfbebdbc, 0xc3c2c1c0, 0xc7c6c5c4, 0xcbcac9c8, 0xcfcecdcc, 0xd3d2d1d0, 0xd7d6d5d4,
> 
>   0xdbdad9d8, 0xdfdedddc, 0xe3e2e1e0, 0xe7e6e5e4, 0xebeae9e8, 0xefeeedec, 0xf3f2f1f0, 0xf7f6f5f4, 0xfbfaf9f8,
> 
>   0xfffefdfc}
> 
> (gdb) x/64w 0x22cb70
> 
> 0x22cb70:       0x03020100      0x07060504      0x0b0a0908      0x0f0e0d0c
> 
> 0x22cb80:       0x13121110      0x17161514      0x1b1a1918      0x1f1e1d1c
> 
> 0x22cb90:       0x23222120      0x27262524      0x2b2a2928      0x2f2e2d2c
> 
> 0x22cba0:       0x33323130      0x37363534      0x3b3a3938      0x3f3e3d3c
> 
> 0x22cbb0:       0x43424140      0x47464544      0x4b4a4948      0x4f4e4d4c
> 
> 0x22cbc0:       0x53525150      0x57565554      0x5b5a5958      0x5f5e5d5c
> 
> 0x22cbd0:       0x63626160      0x67666564      0x6b6a6968      0x6f6e6d6c
> 
> 0x22cbe0:       0x73727170      0x77767574      0x7b7a7978      0x7f7e7d7c
> 
> 0x22cbf0:       0x83828180      0x87868584      0x8b8a8988      0x8f8e8d8c
> 
> 0x22cc00:       0x93929190      0x97969594      0x9b9a9998      0x9f9e9d9c
> 
> 0x22cc10:       0xa3a2a1a0      0xa7a6a5a4      0xabaaa9a8      0xafaeadac
> 
> 0x22cc20:       0xb3b2b1b0      0xb7b6b5b4      0xbbbab9b8      0xbfbebdbc
> 
> 0x22cc30:       0xc3c2c1c0      0xc7c6c5c4      0xcbcac9c8      0xcfcecdcc
> 
> 0x22cc40:       0xd3d2d1d0      0xd7d6d5d4      0xdbdad9d8      0xdfdedddc
> 
> 0x22cc50:       0xe3e2e1e0      0xe7e6e5e4      0xebeae9e8      0xefeeedec
> 
> 0x22cc60:       0xf3f2f1f0      0xf7f6f5f4      0xfbfaf9f8      0xfffefdfc
> 
> (gdb) c
> 
> Continuing.
> 
> ASCII table:
> 
>    !  "  #  $  %  &  '  (  )  *  +  ,  -  .  /
> 
> 0  1  2  3  4  5  6  7  8  9  :  ;  <  =  >  ?
> 
> @  A  B  C  D  E  F  G  H  I  J  K  L  M  N  O
> 
> P  Q  R  S  T  U  V  W  X  Y  Z  [  /  ]  ^  _
> 
> `  a  b  c  d  e  f  g  h  i  j  k  l  m  n  o
> 
> p  q  r  s  t  u  v  w  x  y  z  {  |  }  ~ 
> 
> €  ? ? ? ? ? ? ? ? ? ? ? ? ? ? ?
> 
> ? ? ? ? ? ? ? ? ? ? ? ? ? ? ? ?
> 
> ? ? ? ? ? ? ? ? ? ? ? ? ? ? ? ?
> 
> ? ? ? ? ? ? ? ? ? ? ? ? ? ? ? ?
> 
> ? ? ? ? ? ? ? ? ? ? ? ? ? ? ? ?
> 
> ? ? ? ? ? ? ? ? ? ? ? ? ? ? ? ?
> 
> ? ? ? ? ? ? ? ? ? ? ? ? ? ? ? ?
> 
> ? ? ? ? ? ? ? ? ? ? ? ? ? ? ? 
> 
> Program exited normally.
> 
> (gdb)
> 
> 至此，我们已经看到watch查看内存的作用了，只要被查看的单元会被修改（读/写），均会断在此处，以方便我们调试程序
> 
> https://blog.csdn.net/livelylittlefish/article/details/5110234
> 
> # gdb 内存断点watch 的使用
> 
>  1.  watch 变量的类型  
>     a. 整形变量： int i; watch i;  
>     b. 指针类型:  char *p; watch p, watch *p;  
>     它们是有区别的.  
>  watch p 是查看 *(&p), 是p 变量本身。  
> 
>       watch (*p) 是 p 所指的内存的内容, 查看地址，一般是我们所需要的。
> 
>       我们就是要看莫地址上的数据是怎样变化的，虽然这个地址具体位置只有编译器知道。
> 
>     c. watch 一个数组或内存区间  
>     char buf[128], watch buf,    
>  是对buf 的128个数据进行了监视. 此时不是采用硬件断点，而是软中断实现的。  
>     软中断方式去检查内存变量是比较耗费cpu资源的。  
>     精确的指明地址是硬件中断。  
>   
> 2. 当你设置的观察点是一个局部变量时。局部变量无效后，观察点无效  
>  Watchpoint 2 deleted because the program has left the block in   
>    which its expression is valid.   
>                  
> 3. 附上一个简单程序方便你利用内存断点观察，调试.  
>   
> 
> 1.  $ cat test.cpp  
> 2.  #include <stdio.h> 
> 3.  #include <string.h> 
> 4.  void initBuf(char *buf); 
> 5.  void prtBuf(char *buf); 
> 6.  char mem[8]; 
> 7.  char buf[128]; 
> 8.  int main() 
> 9.  {  
> 10.      initBuf(buf);  
> 11.      prtBuf(buf);  
> 12.      return 0; 
> 13.  }  
> 
> 15.  void initBuf(char *pBuf) 
> 16.  {  
> 17.      int i, j; 
> 18.      mem[0]='0'; 
> 19.      mem[1]='1'; 
> 20.      mem[2]='2'; 
> 21.      mem[3]='3'; 
> 22.      mem[4]='4'; 
> 23.      mem[5]='5'; 
> 24.      mem[6]='6'; 
> 25.      mem[7]='7'; 
> 26.      //ascii table first 32 is not printable 
> 27.      for(i=2;i<8;i++) 
> 28.      {  
> 29.          for(j=0;j<16;j++) 
> 30.              pBuf[i*16+j]=i*16+j;  
> 31.      }  
> 32.  }  
> 
> 34.  void prtBuf(char *pBuf) 
> 35.  {  
> 36.      int i, j; 
> 37.      for(i=2;i<8;i++) 
> 38.      {  
> 39.          for(j=0;j<16;j++) 
> 40.              printf("%c  ", pBuf[i*16+j]); 
> 41.          printf("\n"); 
> 42.      }  
> 43.  }  
> 
>   
> 玩弄内存调试于股掌之中。  
> (由于效率问题你需要适当控制内存断点设置,当然，对这个小程序无所谓.)  
> ----------------------------------------  
> 看一下mem 数组， 内存数据是怎样被写入的。  
> ----------------------------------------  
> gdb test  
> b main  
> watch mem  
> run  
> Breakpoint 1, main () at test.cpp:9  
> gdb) continue  
>   Continuing.  
>   Hardware watchpoint 2: mem  
>   Old value = "\000\000\000\000\000\000\000"  
>   New value = "0\000\000\000\000\000\000"  
>   initBuf (pBuf=0x6010a0 <buf> "") at test.cpp:18  
> (gdb) continue  
>   Continuing.  
>   Hardware watchpoint 2: mem  
>   Old value = "0\000\000\000\000\000\000"  
>   New value = "01\000\000\000\000\000"  
>   initBuf (pBuf=0x6010a0 <buf> "") at test.cpp:19  
> (gdb) continue  
>   Continuing.  
>   Hardware watchpoint 2: mem  
>   Old value = "01\000\000\000\000\000"  
>   New value = "012\000\000\000\000"  
>   initBuf (pBuf=0x6010a0 <buf> "") at test.cpp:20  
> (gdb)   
> ......  
>   
> (gdb) continue  
>   Continuing.  
>   Hardware watchpoint 2: mem  
>   Old value = "0123456"  
>   New value = "01234567"   
>   initBuf (pBuf=0x6010a0 <buf> "") at test.cpp:26
> 
> # 使用gdb watch调试代码
> 
> 本篇文章将使用一个简单的例子说明如何使用gdb watch调试代码。
> 
>   首先来看以下一段简单的代码
> 
>        ![](https://img-blog.csdn.net/20150925072826402)
> 
>   显然，第7行代码是有问题，那么这个错误的memset会造成什么后果呢？
> 
>   我们运行以下两个指令看下程序的输出：
> 
>    g++ -g main.c -o main.o
> 
>    ./main.o
> 
>   程序的输出是0 0 3，变量a的值因为memset被错误地修改了。
> 
>   因为这是个很短的程序，所以我们能很轻松地看出第7行的代码是有问题的。这里的memset会越界，错误地修改了a的值。在实际情况下，当我们发现某个变量的值不符合预期时，一般的做法是先查下这个变量的引用，找到对该变量有写操作的地方(在本例中，对变量a的写操作只有一处，即第6行)。当我们发现所有的写操作和逻辑都不会产生该非法值时，可以认定程序中有越界的情况。越界的情况在实际项目中是非常令人头痛的。一是问题的根源难以定位：在本例中异常的数据是a，但其根本原因是对b操作不当造成的。二是越界之后程序的行为是未定义的，而除了回档之外也找不到到更好的方法来还原数据。
> 
>   在开发环境下，我们可以使用gdb来辅助定位越界这一问题，这里用的是watch命令。
> 
>   我们将断点设在第7行，并运行程序。可以分别看下a，b，c的值，可以看到这个时候三个变量的值都是正常的(实际这个时候变量c的值也是未定义的，看编译器怎么处理)。我们再分别打印变量a和变量b的地址，可以看到a的地址比b大4 。(回忆下，一个int型变量占4个字节，而栈上的内存是从大到小分配的)。
> 
>       ![](https://img-blog.csdn.net/20150925073753211)
> 
>   这个我们使用watch监控变量a值得变量，一种做法是直接watch a，我的习惯是watch地址，watch地址的方法更加具有普适性一些。
> 
> 具体的指令是 watch *(int *)0x7fff84b67b08。然后我们继续执行程序看会在哪一步停下来，
> 
>       ![](https://img-blog.csdn.net/20150925073952815)
> 
>   程序在执行到第8行时发现watch的一段内存有变化，旧的值是1，新的值是0。这个时候我们回去看程序，就能发现是第7行这个memset错误地清空了a的内存空间。
> 
> watchpoint只能在程序启动后设置，先在main那下个断点，让程序启动后暂停在main函数处。
> 
> [gdb硬件断点-----watch使用方法](http://blog.chinaunix.net/uid-30540544-id-5746456.html)
> 
> 硬件断点使用watch监测，可以监测栈变量和堆变量值的变化，当被监测变量值发生变化时，程序被停住。  
>   
> 1.    栈变量  
> 测试代码(文件名为1.c，比较low，哈哈)：
> 
> 点击(此处)折叠或打开
> 
> 1.  #include <string.h>
> 
> 3.  void test(void)
> 4.  {
> 5.          int iA, iB, iC; 
> 
> 7.          iA = 1;
> 8.          iB = 2;
> 9.          iC = 3;
> 
> 11.          iB += iA + iC; 
> 12.          printf("iB = %d\n", iB);
> 13.          iB = 0;
> 14.          printf("iB = %d\n", iB);
> 15.          iB *= iC; 
> 16.          printf("iB = %d\n", iB);
> 
> 18.          return;
> 19.  }
> 
> 21.  void main(void)
> 22.  {
> 23.          test();
> 
> 25.          return;
> 26.  }
> 
> 测试过程(监测变量iB值在代码中发生变化的位置)：
> 
> 点击(此处)折叠或打开
> 
> 1.  [root@ceph181 test]# vim 1.c
> 2.  [root@ceph181 test]# gcc -g 1.c 
> 3.  [root@ceph181 test]# ls
> 4.  1.c a.out debug.c log.c mmap.c sscanf.c sync_fetch.c time.c unlikely.c
> 5.  [root@ceph181 test]# gdb a.out 
> 6.  GNU gdb (GDB) Red Hat Enterprise Linux (7.2-60.el6_4.1)
> 7.  Copyright (C) 2010 Free Software Foundation, Inc.
> 8.  License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
> 9.  This is free software: you are free to change and redistribute it.
> 10.  There is NO WARRANTY, to the extent permitted by law. Type "show copying"
> 11.  and "show warranty" for details.
> 12.  This GDB was configured as "x86_64-redhat-linux-gnu".
> 13.  For bug reporting instructions, please see:
> 14.  <http://www.gnu.org/software/gdb/bugs/>...
> 15.  Reading symbols from /home/work/test/a.out...done.
> 16.  (gdb) b test 
> 17.  Breakpoint 1 at 0x4004cc: file 1.c, line 8.
> 18.  (gdb) r
> 19.  Starting program: /home/work/test/a.out 
> 
> 21.  Breakpoint 1, test () at 1.c:8
> 22.  8        iA = 1;
> 23.  Missing separate debuginfos, use: debuginfo-install glibc-2.12-1.132.el6.x86_64
> 24.  (gdb) **watch iB**
> 25.  Hardware watchpoint 2: iB
> 26.  (gdb) c
> 27.  Continuing.
> 28.  Hardware watchpoint 2: iB
> 
> 30.  **Old value** **=** **4195296**
> 31.  **New value** **=** **2**
> 32.  test () at 1.c:10
> 33.  10        iC = 3;
> 34.  (gdb) c
> 35.  Continuing.
> 36.  Hardware watchpoint 2: iB
> 
> 38.  **Old value** **=** **2**
> 39.  **New value** **=** **6**
> 40.  test () at 1.c:13
> 41.  13        printf("iB = %d\n", iB);
> 42.  (gdb) c
> 43.  Continuing.
> 44.  iB = 6
> 45.  Hardware watchpoint 2: iB
> 
> 47.  **Old value** **=** **6**
> 48.  **New value** **=** **0**
> 49.  test () at 1.c:15
> 50.  15        printf("iB = %d\n", iB);
> 51.  (gdb) c
> 52.  Continuing.
> 53.  iB = 0
> 54.  iB = 0
> 
> 56.  Watchpoint 2 deleted because the program has left the block in
> 57.  which its expression is valid.
> 58.  main () at 1.c:26
> 59.  26        return;
> 60.  (gdb) c
> 61.  Continuing.
> 
> 63.  Program exited with code 07.
> 64.  (gdb)
> 
> **注意： 栈变量的生命周期有限，因此，使用watch监测其硬件断点时，要注意生命周期**  
>   
> 2.    堆变量  
> int *piA = malloc(sizeof(int));  
> 监测堆变量piA值的变化时，  
>     1）    在gdb中打印出变量piA的地址： print  &piA，记为piB；  
>     2）    watch *piB  
>     3）    continue  
> 如下例所示，要监测指针connection->session的值在什么时候被修改：
> 
> 点击(此处)折叠或打开
> 
> 1.  Breakpoint 1, xio_connection_destroy (connection=0x7ffff0009240) at ../common/xio_connection.c:2364
> 2.  2364        int            retval = 0;
> 3.  Missing separate debuginfos, use: debuginfo-install glibc-2.12-1.132.el6.x86_64 libibverbs-1.1.8mlnx1-OFED.3.1.1.0.0.x86_64 libmlx4-1.0.6mlnx1-OFED.3.1.1.0.0.x86_64 libmlx5-1.0.2mlnx1-OFED.3.1.1.0.3.x86_64 libnl-1.1.4-2.el6.x86_64 librdmacm-1.0.21mlnx-OFED.3.0.1.5.2.x86_64 numactl-2.0.7-8.el6.x86_64
> 4.  (gdb) p connection->session
> 5.  $1 = (struct xio_session *) 0x6120b0
> 6.  (gdb) **p &connection****-****>****session**
> 7.  $2 = (struct xio_session **) 0x7ffff0009298
> 8.  (gdb) **watch** *******(****$****2****)**
> 9.  Hardware watchpoint 2: *($2)
> 10.  (gdb) disable 1
> 11.  (gdb) c
> 12.  Continuing.
> 13.  Hardware watchpoint 2: *($2)
> 
> 15.  **Old value** **=** **(****struct** **xio_session** *******)** **0x6120b0**
> 16.  **New value** **=** **(****struct** **xio_session** *******)** **0x0**
> 17.  0x0000003917805e94 in rdma_get_cm_event () from /usr/lib64/librdmacm.so.1
> 18.  (gdb)
> 
>  [gdb 里设置临时变量](https://blog.csdn.net/robertsong2004/article/details/42917835)
> 
> 使用 set 命令。
> 
> (gdb) set $i="hello"  
> (gdb) ptype $i   
> type = char [6]  
> (gdb) set $i=1  
> (gdb) ptype $i  
> type = int  
> (gdb) set $i=(char)1  
> (gdb) ptype $i  
> type = char  
> (gdb) set $i=(short)1  
> (gdb) ptype $i  
> type = short
> 
> set设置临时变量加$符号。
> 
> （1）修改变量值：
> 
> a. **print**_v=value_:　修改变量值的同时，把修改后的值显示出来
> 
> b. **set** [**var**]_v=value_:　修改变量值，需要注意如果变量名与GDB中某个**set**命令中的关键字一样的话，前面加上**var**关键字
> 
>  **不加$表示修改原有的变量**。
> 
> ## 打印指针内容：
> 
> p *poin
> 
> # gdb 断点设置（二）watch
> 
> 2、watch
> 
>      watch [-l|-location] expr [thread threadnum] [mask maskvalue]
> 
>      -l 与 mask没有仔细研究，thread threadnum 是在多线程的程序中限定只有被线程号是threadnum的线程修改值后进入断点。
> 
>      经常用到的如下命令：
> 
>      watch <expr>
> 
>      为表达式（变量）expr设置一个观察点。变量量表达式值有变化时，马上停住程序。
> 
>      表达式可以是一个变量
> 
>      例如：watch value_a
> 
>      表达式可以是一个地址：
> 
>      例如：watch *(int *)0x12345678 可以检测4个字节的内存是否变化。
> 
>      表达式可以是一个复杂的语句表达式：
> 
>      例如：watch a*b + c/d
> 
>      watch 在有些操作系统支持硬件观测点，硬件观测点的运行速度比软件观测点的快。如果系统支持硬件观测的话，当设置观测点是会打印如下信息：
> 
>      Hardware watchpoint num: expr
> 
>     如果不想用硬件观测点的话可如下设置：
> 
>     set can-use-hw-watchpoints
> 
>     watch两个变种 rwatch，awatch，这两个命令只支持硬件观测点如果系统不支持硬件观测点会答应出不支持这两个命令的信息:，
> 
>     rwatch <expr>
> 
>     当表达式（变量）expr被读时，停住程序。
> 
>     awatch <expr>
> 
>     当表达式（变量）的值被读或被写时，停住程序。
> 
>     info watchpoints
> 
>     列出当前所设置了的所有观察点。
> 
>      watch 所设置的断点也可以用控制断点的命令来控制。如 disable、enable、delete等。 
> 
>      可以为停止点设定运行命令
> 
>      commands [bnum]
> 
>     ... command-list ...
> 
>     end
> 
>     为断点号bnum指写一个命令列表。当程序被该断点停住时，gdb会依次运行命令列表中的命令。
> 
>     例如：
> 
>         break foo if x>0
> 
>         commands
> 
>         printf "x is %d/n",x
> 
>         continue
> 
>         end
> 
>     断点设置在函数foo中，断点条件是x>0，如果程序被断住后，也就是，一旦x的值在foo函数中大于0，GDB会自动打印出x的值，并继续运行程序。 
> 
>    注意：watch 设置也是断点，如果调试的时候设置的断点（任何种类的断点）过多的时候，watch断点会被忽略，有时候没有任何提示，
> 
>             这是我在测试的时候发现的，只有把多余的断点删除后才可用。
> 
> **display +表达式　　display a　　用于显示表达式的值，每当程序运行到断点处都会显示表达式**的值 
> 
> # 1．Segment Fault
> 
> Segment Fault一般是由于访问非法指针或者访问空指针造成的，而且发生了这类错误之后，程序将不能再继续执行，必须重新启动整个系统才可以解决问题，因此此类问题后果十分严重，如何定位这样的问题也一直是一个难题。如果使用debug版本，可以使用gdb巧妙的定位出这样的问题。具体的定位方法如下述：
> 
> 如果使用gdb启动的情况下，一般发生segment fault，程序便停在出错的地方，
> 
> 这是可以使用bt将调用栈打印出来，
> 
> 接着可以使用info local 命令将当前所有的局部变量的值打印出来
> 
> 通过变量的值可以看出xx的指针的值已经不正确，由此可以断定是由于这个非法访问这个指针造成的。如果这个指针是由上层调用栈传入的，可以使用f n ， n代表使用bt打印出的调用栈的第几层，可以在其他的调用层看看哪层开始指针不正确。
> 
> 2. 死循环问题
> 
> 当程序处于死循环状态时， 可以按下ctrl+c， 是程序停下，然后运行命令 thread applly all bt ，这样可以打印出所有线程的当前的调用栈， 再按c， 让程序继续运行，过几秒钟， 再运用thread applly all bt ， 再次打印出所有线程的当前的调用栈， 然后通过比较工具比较这两个调用栈， 如果某个线程的调用栈时刻在改变并且在循环中，那么这个线程可能处于死循环状态。