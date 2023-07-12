# hurlex实验六：添加全局段描述符表 笔记

## 6.1 保护模式的引入

开始涉及x86保护模式编程的细节问题。

80386处理器：CPU有四种运行模式,即**实模式、保护模式、虚拟8086模式和SMM模式**。

在8086汇编中，实模式机制：1MB的线性地址空间、内存寻址方法、寄存器、端口读写以及中断处理方法等。

80386引入了保护模式这一CPU运行机制。新的特色：内存保护、分页系统、硬件支持的虚拟内存。

大部分现今基于x86的操作系统都在保护模式下运行。

虚拟8086：在保护模式下运行原来实模式下的16位程序

SMM：不对程序员开放

## 6.2 保护模式下的内存分段

已知对CPU而言，系统中所有存储器中的储存单元都处于一个统一的**逻辑存储器**中，其容量受CPU寻址能力限制（逻辑存储器在此指的是**线性地址空间**）。

8086：20位地址线，有1MB的线性地址空间。

80386：32位地址线，有4GB线性地址空间。

但80386依然保留了8086采用的地址分段方式，只是增加了一个折中方案：只分一个段，段基地址0x00000000，段长0xFFFFFFFF（4GB）。这样整个线性地址空间可看作一个段，即所谓“**平坦模型”（flat mode）。**

### 平坦模型和分段模型

在内存分段环境下，8086（实模式）CPU的16位寄存器最多只能在一个内存段（内存分段）的64kb空间寻址，要是超过64kb，它只能先变换段基址CS来达到长距离取指目的。一个程序肯定包括多个段（指**程序分段**），这样它就要花很多功夫去寻址、访存。

平坦模型：相对于多段模型，它可理解为始终只有一个段，能直接访问内存空间，不用再进行段基址的变换。 保护模式下，32位环境下用一个段就能访问4GB的内存位置，故不用再分段，不用来回切换要访问的段。

PS：**程序分段和CPU内存分段是不同的概念**，现代操作系统一般在平坦模型下工作（整个4GB空间为一段），编译器也按照平坦模型为程序布局，程序中的代码和数据都在同一段中整齐排列。

一般高级语言不允许程序员把代码分成各种各样的段，因为编译器是针对某个操作系统编写的。编译器会将程序按内容分成代码段、数据段等部分，而后操作系统会将各个段分配到不同的物理内存上，之后的事情就不用程序员操心了

关于多段模型：当使用汇编语言编写程序时，我们需要自己定义数据段、代码段、栈段之类的东西，这就定义了多个段。当我们使用他们时（类似于使用栈，都要指定栈基址，再根据需要给sp（栈顶指针寄存器）赋予相应的值，才能对数据进行正确的操作）。多个段可能在不同的内存分段中存在，大小不一故需要进行相应的跳转访存。

**程序分段的好处：**

1. 程序分段能赋予程序段不同属性，按此执行不同的安全策略（如代码段只读等）
2. 能提高cpu缓存命中，按照局部性原理，在分段的基础下对不同的段采取不同策略
3. 能节省内存。如在一个程序的多个副本一同执行时，没必要在内存中有多个相同的代码块。把代码段共享即可。

**平坦模型总结：**平坦模型在程序分段的基础上将多个段统筹管理，在上面做了很多努力，让我们不用花很多精力在代码分段上，解放了生产力。

### 保护模式下内存如何分段管理

#### 内存分段中每一个段的描述

最初cpu只有实模式，其对内存段没有访问控制，任意的程序可以修改任意地址的变量。保护模式需要对内存段的性质和允许的操作给出定义，实现对特定内存段的访问检测和数据保护。

考虑各种属性和需要设置的操作，32位保护模式下对一个内存段的描述需要**8个字节**，称之为**段描述符**。段描述符分为**数据段描述符、指令段描述符、系统段描述符**三种。

#### 描述符表相关

由于寄存器不足以存放很多个内存段的描述符的集合，故这些描述符的集合（称之为**描述符表**）被放置在内存中了。在这些描述符表中，最重要的就是**全局描述符表**（Global Descriptor Table，GDT），它为整个软硬件系统服务。

把描述符表放在内存中，带来了其他问题：

1. 这些描述符表放在内存哪里？：没有固定位置，可以由程序员安排在任意合适的位置。
2. 既然没有固定位置，CPU如何知道全局描述符表在哪？：Intel设置了一个**48位**的专用的**全局描述符表寄存器（GDTR）**来保存全局描述符表的信息。
3. 这48位是怎么分配的呢？：如下图，0-15位表示GDT的边界位置（数值为表的长度-1，因为从0开始），16-47位这32位存放的就是**GDT的基地址（类似于数组的首地址**）

![屏幕截图 2021-11-25 155908](https://raw.githubusercontent.com/exaids66/imgs/main/images/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202021-11-25%20155908.jpg)

已知用16位来表示表的长度，则2^16就是65536字节，除以每一个描述符的8字节，则最多能创建8192个描述符。

#### CPU的工作方式

80386CPU加电的时候自动进入实模式。那如何进入保护模式？：80386CPU内部有5个32位的控制寄存器（control Register,CR）,分别是CR0~CR3 以及CR8。这些用来表示CPU的一些状态，其中CR0寄存器的**PE位**（protection Enable，**保护模式允许位**）、也称0号位，就表示了CPU的运行状态，0为实模式，1为保护模式。通过修改这个位就可以立即改变CPU的工作模式。

![屏幕截图 2021-11-25 161952](https://raw.githubusercontent.com/exaids66/imgs/main/images/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202021-11-25%20161952.jpg)

PS：一旦CR0寄存器的PE位被修改，CPU便立即按照保护模式寻址，这要求我们在进入保护模式前就要在内存中放置号GDT（全局描述符表），然后设置好GDTR寄存器（全局描述符表寄存器）。已知实模式下寻址空间只有1MB，故GDT只能在这里放置。在进入保护模式后，我们可以在4GB的空间内设置并修改原来的GDTR了。

现在有了描述符的数组（即全局描述符表），也有了”数组指针“（即GDTR，全局描述符寄存器），如何表示我们要访问哪个段呢？：使用段选择器/段选择子。这是8086时代的段寄存器。此时的CS等寄存器不再保存段基址了，而是保存其指向段的索引信息，CPU会根据这些信息在内存中获取到段信息。

地址合成过程如下图所示：

![屏幕截图 2021-11-25 164903](https://raw.githubusercontent.com/exaids66/imgs/main/images/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202021-11-25%20164903.jpg)

## 6.3 具体采用的分段策略

以下是具体的设计方案。已知现代操作系统几乎不再使用分段而是绕过分段技术直接使用分页。分段和分页本没有什么必然联系。只是8086开始，其制造的CPU就以段地址+偏移地址的方式来访问内存。后来要兼容以前的CPU，Intel不得不保留该传统。

分段是Intel的CPU一直保持着的一种机制，而分页仅是保护模式下的一种内存管理策略。想开启分页机制，CPU就必须工作在保护模式，而工作在保护模式时可以不开启分页。所以事实上分段是必须的，而分页是可选的。

**绕过分段机制**的方法：**利用平坦模式**。即当整个虚拟地址空间是一个**起始地址为0、限长为4G的”段“**时，我们给出的偏移地址就在数值上等于是段机制处理后的地址了。不过我们不是简单的对所有的段使用同样的描述符，而是给代码段和数据段分配不同的描述符。如图所示上述抽象：

![屏幕截图 2021-11-25 170958](https://raw.githubusercontent.com/exaids66/imgs/main/images/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202021-11-25%20170958.jpg)

在之前提到过GRUB在载入内核前的一些状态，包括以下两条：

1. CS寄存器指向基地址为0x00000000，限长为4G-1的代码段描述符。
2. DS，SS，ES，FS和GS指向基地址为0x00000000，限长 为4G-1的数据段描述符。

以下代码为实现GRUB上述两条状态：

1、include/gdt.h

```c
#ifndef INCLUDE_GDT_H_
#define INCLUDE_GDT_H_

#include "types.h"

// 全局描述符类型
typedef
struct gdt_entry_t {
	uint16_t limit_low;     // 段界限   15～0
	uint16_t base_low;      // 段基地址 15～0
	uint8_t  base_middle;   // 段基地址 23～16
	uint8_t  access;        // 段存在位、描述符特权级、描述符类型、描述符子类别
	uint8_t  granularity; 	// 其他标志、段界限 19～16
	uint8_t  base_high;     // 段基地址 31～24
} __attribute__((packed)) gdt_entry_t;

// GDTR
typedef
struct gdt_ptr_t {
	uint16_t limit; 	// 全局描述符表限长
	uint32_t base; 		// 全局描述符表 32位 基地址
} __attribute__((packed)) gdt_ptr_t;

// 初始化全局描述符表
void init_gdt();

// GDT 加载到 GDTR 的函数[汇编实现]
extern void gdt_flush(uint32_t);

#endif 	// INCLUDE_GDT_H_
```

以上为两个结构体的定义。具体函数实现如下

2、gdt/gdt.c

```c
#include "gdt.h"
#include "string.h"

// 全局描述符表长度
#define GDT_LENGTH 5

// 全局描述符表定义
gdt_entry_t gdt_entries[GDT_LENGTH];

// GDTR
gdt_ptr_t gdt_ptr;

// 全局描述符表构造函数，根据下标构造
static void gdt_set_gate(int32_t num, uint32_t base, uint32_t limit, uint8_t access, uint8_t gran);

// 声明内核栈地址
extern uint32_t stack;

// 初始化全局描述符表
void init_gdt()
{
	// 全局描述符表界限 e.g. 从 0 开始，所以总长要 - 1
	gdt_ptr.limit = sizeof(gdt_entry_t) * GDT_LENGTH - 1;
	gdt_ptr.base = (uint32_t)&gdt_entries;

	// 采用 Intel 平坦模型
	gdt_set_gate(0, 0, 0, 0, 0);             	// 按照 Intel 文档要求，第一个描述符必须全 0
	gdt_set_gate(1, 0, 0xFFFFFFFF, 0x9A, 0xCF); 	// 指令段
	gdt_set_gate(2, 0, 0xFFFFFFFF, 0x92, 0xCF); 	// 数据段
	gdt_set_gate(3, 0, 0xFFFFFFFF, 0xFA, 0xCF); 	// 用户模式代码段
	gdt_set_gate(4, 0, 0xFFFFFFFF, 0xF2, 0xCF); 	// 用户模式数据段

	// 加载全局描述符表地址到 GPTR 寄存器
	gdt_flush((uint32_t)&gdt_ptr);
}

// 全局描述符表构造函数，根据下标构造
// 参数分别是 数组下标、基地址、限长、访问标志，其它访问标志
/* 结构体定义如下：
typedef struct
{
	uint16_t limit_low;     // 段界限   15～0
	uint16_t base_low;      // 段基地址 15～0
	uint8_t  base_middle;   // 段基地址 23～16
	uint8_t  access;        // 段存在位、描述符特权级、描述符类型、描述符子类别
	uint8_t  granularity; 	// 其他标志、段界限 19～16
	uint8_t  base_high;     // 段基地址 31～24
} __attribute__((packed)) gdt_entry_t;
*/
static void gdt_set_gate(int32_t num, uint32_t base, uint32_t limit, uint8_t access, uint8_t gran)
{
	gdt_entries[num].base_low     = (base & 0xFFFF);
	gdt_entries[num].base_middle  = (base >> 16) & 0xFF;
	gdt_entries[num].base_high    = (base >> 24) & 0xFF;

	gdt_entries[num].limit_low    = (limit & 0xFFFF);
	gdt_entries[num].granularity  = (limit >> 16) & 0x0F;

	gdt_entries[num].granularity |= gran & 0xF0;
	gdt_entries[num].access       = access;
}
```

该.c文件的主要工作量在于：对照Intel文档的说明，为每一个段描述符计算权限位的数值。

以下汇编语言（**Assembly Language**）实现了加载全局描述符的操作，即**：将GDT地址载入GDTR。**

gdt_s.s

```assembly
[GLOBAL gdt_flush]

gdt_flush:
	mov eax, [esp+4]  ; 参数存入 eax 寄存器
	lgdt [eax]        ; 加载到 GDTR [修改原先GRUB设置]

	mov ax, 0x10      ; 加载我们的数据段描述符
	mov ds, ax        ; 更新所有可以更新的段寄存器
	mov es, ax
	mov fs, ax
	mov gs, ax
	mov ss, ax
	jmp 0x08:.flush   ; 远跳转，0x08是我们的代码段描述符
			  ; 远跳目的是清空流水线并串行化处理器
.flush:
	ret
```

对于jmp跳转那一条语句的解释：0x08是跳转目标段的段选择子，其对应段描述符第二项。后面的跳转目标标号即为下一行的代码.flush。为什么是.flush呢？

1. Intel不允许直接修改段寄存器CS的值，我们只好这样通过这种方式更新CS段寄存器；
2. x86以后CPU所增加的指令流水线和高速缓存可能会在新的全局描述符表加载后依然保持之前的缓存，那么修改GDTR之后最安全的做法就是立即**清空流水线和更新高速缓存**。

即：仅需一句jmp跳转就可以强迫CPU自动更新。

最后再次修改入口函数entry.c如下：


```c
#include "gdt.h"
#include "console.h"
#include "debug.h"

int kern_entry()
{
	init_debug();
	init_gdt();

	console_clear();
	printk_color(rc_black, rc_green, "Hello, MACHY!\n");

	return 0;
}
```

