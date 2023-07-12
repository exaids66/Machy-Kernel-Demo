# hurlex实验七：添加中断描述符表

## 7.1 中断的引入

作用：提高计算机工作效率、增强计算机功能。简单说是一种**通知机制**。

当某个中断发生时，典型的处理方式就是CPU会打断当前的任务，保留执行现场后再转移到该中断事先安排好的**中断处理函数**（又称中断服务例程）去执行。当中断处理函数执行结束后再恢复中断之前的执行现场，去执行之前的任务。

中断有编号：中断在物理学上是一种电信号，一般由硬件设备生成并送入**中断控制器**统一协调（协调机构是必须的，否则所有设备会不分轻重缓急的和CPU发送中断信号）。中断控制器作用是将汇集的多路中断管线，采用复用技术只通过一条中断线和CPU相连接。因为中断控制器只使用一条线与CPU连接，那为了区分各个设备，中断自然有了编号。

PS：CPU的中断管脚其实有两根：NMI和INTR。因为从严重性来看，中断分为两类：由NMI管脚触发的中断是需要无条件立即处理的，这种类型的中断不会被阻塞和屏蔽，叫做非屏蔽中断（No Maskable Interrupt, NMI）。一旦产生NMI中断，说明CPU遇到了不可挽回的错误，一般不会进行处理，只是给出一个错误信息。我们之前所说的中断控制器连接的管脚叫INTR，其特点为：数量多、可屏蔽。我们主要关注INTR中断。

在X86PC中，我们熟知的中断控制芯片是8259A PIC,其就是我们所说的中断控制器。Intel处理器允许256个中断，中断号范围为0~255。8295A PIC芯片负责15个，但不固定中断号，允许通过IO端口设置以避免冲突。其全称是**可编程中断控制器**（Programmable Interrupt Controller，PIC）。

中断的概念：简单说即**硬件**发生了某个事件后告知**中断控制器**，中断控制器汇报给**CPU**，CPU从中断控制器处得知**中断号**，根据该中断号找到对应的**中断处理程序**并转移执行，执行完毕后**重新回到之前的执行流程**中。

上述中断均为**硬件中断**。除了硬件中断外还有**软件中断**，即软件系统可利用中断机制来完成一些任务，如有些OS的系统调用就实现了采用中断的方式。

## 7.2 中断的实现

关注重点：**保护模式下的中断处理**。因为中断处理程序运行在**ring0层**，这表示中断处理程序拥有系统的全部权限，仿照内存段描述符的思路，Intel设置了**中断描述符表（**IDT，Interrupt Descriptor Table），和段描述符表一样放置在主存中，同时也设置**中断描述符表寄存器**（IDTR）记录这个表的起始地址。那么下文的重点就是该中**断描述符的结构和设置方法**。

IDT：记录了0~255的中断号和调用函数之间的关系。

PS：**ring0层**：Intel的CPU运行级别。总共有ring0、ring1、ring2、ring3。ring0只能给操作系统用，ring3谁都能用。ring0是最高级别，ring3是最低级别。

PS2：当应用程序要访问磁盘、写文件时，通过执行系统调用（函数），执行系统调用时，CPU的运行级别会发生从ring3到ring0的切换，并跳转到系统调用对应的内核代码位置执行，这样内核就为我们完成了设备访问，完成后再从ring0返回ring3.该过程也称为**用户态和内核态的切换**。

下图是中断描述符表的结构：

![](https://raw.githubusercontent.com/exaids66/imgs/main/images/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202021-12-01%20084848.jpg)

据此给出相关C语言结构体定义：

1、include/idt.h

```c
#ifndef INCLUDE_IDT_H_
#define INCLUDE_IDT_H_

#include "types.h"

//初始化中断描述符表
void init_idt();

//中断描述符
typedef
    struct idt_entry_t{
        uint16_t base_lo;//中断处理函数地址15~0位
        uint16_t sel;    //目标代码段描述符选择子
        uint8_t  always0;//置0段
        uint8_t  flags;  //一些标志，文档有解释
        uint16_t base_hi;//中断处理函数地址31~16位
    }__attribute__((packed)) idt_entry_t;

//IDTR
typedef
    struct idt_ptr_t{
        uint16_t limit;  //限长
        uint32_t base;   //基址
    }__attribute__((packed)) idt_ptr_t;

#endif  //INCLUDE_IDT_H_
```

## 细化CPU处理中断的过程

### 起始过程

起始过程：从CPU发现中断事件后，打断当前程序或任务的执行，根据某种机制跳转到中断处理函数去执行的过程。

1. CPU在执行完当前程序的每一条指令后，都会**确认**在执行刚才的指令过程中是否发送中断请求过来，如果有那么CPU就会在相应的时钟脉冲到来时从总线上读取中断请求对应的中断向量。然后根据得到的中断向量为索引到IDT中找到该向量对应的中断描述符，中断描述符里保存着中断处理函数的段选择子。
2. CPU使用IDT查到的中断处理函数段选择子从GDT中取得相应的段描述符，段描述符中保存了中断处理函数的段基址和属性信息。此时CPU要进行**特权检验**，涉及到CPL、RPL和DPL的数值检验以及判断是否发生用户态到内核态的切换。如果发生了切换，还要涉及到TSS段和用户栈和内核栈的切换。（这部分看不明白，之后看保护环的时候细说）
3. 确认无误后CPU开始保存当前被打断的程序的现场（即此时一些寄存器的信息），以便于将来恢复被打断的程序继续执行。这需要利用内核栈来保存相关现场信息，即依次压入当前被打断程序使用的eflags、cs、eip以及错误代码号（如果当前中断有错误代码的话）。
4. 最后，CPU会从中断描述符中取出中断处理函数的起始地址并跳转执行。

以上是起始过程，中断处理函数执行完成后要通过iret或iretd指令恢复被打断的程序的执行。此时CPU首先会从内存栈中弹出先前保存的被打断的程序的现场信息，即之前的eflags，cs，eip重新开始被打断前的任务。（PS：如果之前存在特权级转换，即从内核态转换到用户态，则还需要从内核栈中弹出用户态栈的ss和esp，这样也意味着栈也被切换回原先使用的用户态的栈了。）

PS：需要注意的是，若此次处理的是带有错误码的中断，CPU在恢复之前程序的现场时，并不会弹出错误代码。这一步需要通过软件来完成，即要求相关的中断处理函数在使用iret指令返回之前添加出栈代码主动弹出错误代码。

下图描述了CPU自动保护和恢复的寄存器的栈结构：

![屏幕截图 2021-12-01 100341](https://raw.githubusercontent.com/exaids66/imgs/main/images/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202021-12-01%20100341.jpg)

### 操作系统在发生中断时的行动

首先**实现中断处理函数**。按照Intel的规定，0~19号中断属于CPU所有（这些由CPU自身产生的中断也被称为**异常**），且第20~31号中断也被Intel保留，故从32~255号才属于用户自定义中断。虽说是“用户自定义”，其实在x86上有些中断按习惯还是给予了固定的设备。如32号中断为timer中断，33号中断是键盘中断等等。

如何实现中断处理函数？不能直接进行相关的逻辑处理，因为虽然CPU在中断产生时自动保存了部分的执行现场，但依旧有很多寄存器需要我们自己去保护和恢复，以下**是CPU保护的寄存器和剩余需要保护的寄存器一起定义的结构体**。

2、include/idt.h

```c
//寄存器类型
typrdef
    struct pt_regs_t{
        uint32_t ds;    //用于保护用户的数据段描述符
        uint32_t edi;   //从edi到eax由pusha指令压入
        uint32_t esi;
        uint32_t ebp;
        uint32_t esp;
        uint32_t ebx;
        uint32_t edx;
        uint32_t ecx;
        uint32_t eax;
        uint32_t int_no; //中断号
        uint32_t err_code;//错误代码有终端错误代码的中断会由（压入CPU）
        uint32_t eip;    //以下由处理器自动压入
        uint32_t cs;
        uint32_t eflags;
        uint32_t useresp;
        uint32_t ss;
    }pt_regs;

//定义中断处理函数指针
typedef void (*interrupt_handler_t)(pt_regs *);

//注册一个中断处理函数
void register_interrupt_handler(uint8_t n,interrupt_handler_t h);

//调用中断处理函数
void isr_handler(pt_regs *regs);

//声明中断处理函数0~19属于CPU的异常中断
//ISR中断服务程序：（interrupt service routine）
void isr0();
void isr1();
void isr2();
void isr3();
void isr4();
void isr5();
void isr6();
void isr8();
void isr9();
void isr10();
void isr11();
void isr12();
void isr13();
void isr14();
void isr15();
void isr16();
void isr17();
void isr18();
void isr19();

//20~31 Intel保留
void isr20();
void isr21();
void isr22();
void isr23();
void isr24();
void isr25();
void isr26();
void isr27();
void isr28();
void isr29();
void isr30();
void isr31();

//32~255用户自定义异常
void isr255();
```

一个很现实的问题：所有的中断处理函数中，除了CPU本身保护的现场外，其他寄存器的保护和恢复过程都是一样的。故，如果在每个中断处理函数中都实现一次显然冗余且易错。

所以我们把原本的中断处理函数逻辑上拆解为三部分，第一部分为一致的现场保护操作；第二部分为每个中断特有的处理逻辑；第三部分为一致的现场恢复。

### 中断处理函数的具体实现

实际上我们把每个中断处理含糊拆解为4段，在四个函数里实现。具体的实现如下：

1、idt/idt_s.s

```assembly
;定义两个构造中断处理函数的宏有的中断有错误代码，有的没有（）
;用于没有错误代码的中断
%macro ISR_NOERRCODE 1
[GLOBAL isr%1]
isr%1:
		cli			;首先关闭中断
		push 0		;push无效的中断错误代码，作为占位
		push %1		;push无效的中断错误代码
		jmp isr_common_stub
%endmacro

;用于有错误代码的中断
%macro ISR_ERRCODE 1
[GLOBAL isr%1]
isr%1:
		cli          ;关闭中断
		push %1      ;push中断号
		jmp isr_common_stub
%endmacro

;定义中断处理函数
ISR_NOERRCODE 0			;0 #DE 除0异常
ISR_NOERRCODE 1			;1 #DB 调试异常
ISR_NOERRCODE 2			;2 NMI
ISR_NOERRCODE 3			;3 BP 断点异常
ISR_NOERRCODE 4			;4 #OF 溢出
ISR_NOERRCODE 5			;5 #BR 对数组的引用超出边界
ISR_NOERRCODE 6			;6 #UD 无效或未定义的操作码
ISR_NOERRCODE 7			;7 #NM 设备不可用无数学协处理器（）
ISR_ERRCODE   8			;8 #DF 双重故障有错误代码（）
ISR_NOERRCODE 9			;9 协处理器跨段操作
ISR_ERRCODE   10		;10 #TS 无效TSS有错误代码（）
ISR_ERRCODE   11		;11 #NP 段不存在有错误代码（）
ISR_ERRCODE   12		;12 #SS 栈错误有错误代码（）
ISR_ERRCODE   13		;13 #GP 常规保护有错误代码（）
ISR_ERRCODE   14		;14 #PF 页故障有错误代码（）
ISR_NOERRCODE 15		;15 CPU 保留
ISR_NOERRCODE 16		;16 #MF 浮点处理单元错误
ISR_ERRCODE   17		;17 #AC 对齐检查
ISR_NOERRCODE 18		;18 #MC 机器检查
ISR_NOERRCODE 19		;19 #XM SIMD单指令多数据（）浮点异常

;20~31 Intel保留
ISR_NOERRCODE 20
ISR_NOERRCODE 21
ISR_NOERRCODE 22
ISR_NOERRCODE 23
ISR_NOERRCODE 24
ISR_NOERRCODE 25
ISR_NOERRCODE 26
ISR_NOERRCODE 27
ISR_NOERRCODE 28
ISR_NOERRCODE 29
ISR_NOERRCODE 30
ISR_NOERRCODE 31
;32~255用户自定义
ISR_NOERRCODE 255
```

函数执行的开始位置我们自行向栈压入中断号，这是为了识别出中断号码。上面的代码用到了**NASM宏汇编**，因为指令结构是相同的，故我们可用宏汇编来生成代码。

PS：因为有的中断处理函数会自动压入错误号，有的不会，这样给我们清栈带来麻烦。所以我们在不会压入错误号的中断处理函数中**手动压入0作为占位**，这样方便我们在清理的时候不用分类处理。

以上是**中断处理函数的最前一部分**，接下来是**所有中断处理函数共有的保护现场操作**：

2、idt/idt_s.s

```assembly
[GLOBAL isr_common_stub]
[EXTERN isr_handler]
;中断服务程序
isr_common_stub:
	pusha			;Pushes edi,esi,ebp,esp,ebx,edx,ecx,eax
	mov ax, ds		
	push eax		;保存数据段描述符
	
	mov ax, 0x10	;加载内核数据段描述符表
    mov ds, ax
    mov es, ax
    mov fs, ax
    mov gs, ax
    mov ss, ax
    
    push esp		;此时的esp寄存器的值等价于 pt_regs结构体的指针
    call isr_handler ;在C语言代码里
    add esp, 4		;清除压入的参数
    
    pop ebx			;恢复原来的数据段描述符
    mov ds, bx
    mov es, bx
    mov fs, bx
    mov gs, bx
    mov ss, bx
    
    popa			;Pops edi, esi, ebp, ebx, edx, ecx, eax
    add esp, 8		;清理栈里的error code和ISR
    iret
.end:
```

上述代码中声明了一个外部函数isr_handler，其实现如下：

3、idt/idt.c

```c
//调用中断处理函数
void isr_handler(pt_regs *regs)
{
    if(interrupt_handlers[regs->int_no]){
        interrupt_handlers[regs->int_no](regs);
    }
    else{
        printk_color(rc_black,rc_bblue,"Unhandled interrupt: %d\n",regs->int_no);
    }
}
```

在上述代码中我们可以看到，事实上具体的中断处理函数原型都是void(pt_regs *)，它们被统一组织放置在了**全局的函数指针数组interrupt_handlers**里面。isr_handlers函数如判断这个中断函数是否注册，如果注册了会执行该函数，否则打印出Unhandled interrupt（未处理的中断）和中断号码。

PS：**pt_regs**:Linux中的一种结构体，定义于include/asm-i386/ptrace.h

最后是中断描述符表的创建和加载函数：

4、idt/idt.c

```c
#include "common.h"
#include "string.h"
#include "debug.h"
#include "idt.h"

// 中断描述符表
idt_entry_t idt_entries[256];

// IDTR
idt_ptr_t idt_ptr;

// 中断处理函数的指针数组
interrupt_handler_t interrupt_handlers[256];

// 设置中断描述符
static void idt_set_gate(uint8_t num, uint32_t base, uint16_t sel, uint8_t flags);

// 声明加载 IDTR 的函数
extern void idt_flush(uint32_t);

// 初始化中断描述符表
void init_idt()
{	
	bzero((uint8_t *)&interrupt_handlers, sizeof(interrupt_handler_t) * 256);
	
	idt_ptr.limit = sizeof(idt_entry_t) * 256 - 1;
	idt_ptr.base  = (uint32_t)&idt_entries;
	
	bzero((uint8_t *)&idt_entries, sizeof(idt_entry_t) * 256);

	// 0-32:  用于 CPU 的中断处理
	idt_set_gate( 0, (uint32_t)isr0,  0x08, 0x8E);
	idt_set_gate( 1, (uint32_t)isr1,  0x08, 0x8E);
	idt_set_gate( 2, (uint32_t)isr2,  0x08, 0x8E);
	idt_set_gate( 3, (uint32_t)isr3,  0x08, 0x8E);
	idt_set_gate( 4, (uint32_t)isr4,  0x08, 0x8E);
	idt_set_gate( 5, (uint32_t)isr5,  0x08, 0x8E);
	idt_set_gate( 6, (uint32_t)isr6,  0x08, 0x8E);
	idt_set_gate( 7, (uint32_t)isr7,  0x08, 0x8E);
	idt_set_gate( 8, (uint32_t)isr8,  0x08, 0x8E);
	idt_set_gate( 9, (uint32_t)isr9,  0x08, 0x8E);
	idt_set_gate(10, (uint32_t)isr10, 0x08, 0x8E);
	idt_set_gate(11, (uint32_t)isr11, 0x08, 0x8E);
	idt_set_gate(12, (uint32_t)isr12, 0x08, 0x8E);
	idt_set_gate(13, (uint32_t)isr13, 0x08, 0x8E);
	idt_set_gate(14, (uint32_t)isr14, 0x08, 0x8E);
	idt_set_gate(15, (uint32_t)isr15, 0x08, 0x8E);
	idt_set_gate(16, (uint32_t)isr16, 0x08, 0x8E);
	idt_set_gate(17, (uint32_t)isr17, 0x08, 0x8E);
	idt_set_gate(18, (uint32_t)isr18, 0x08, 0x8E);
	idt_set_gate(19, (uint32_t)isr19, 0x08, 0x8E);
	idt_set_gate(20, (uint32_t)isr20, 0x08, 0x8E);
	idt_set_gate(21, (uint32_t)isr21, 0x08, 0x8E);
	idt_set_gate(22, (uint32_t)isr22, 0x08, 0x8E);
	idt_set_gate(23, (uint32_t)isr23, 0x08, 0x8E);
	idt_set_gate(24, (uint32_t)isr24, 0x08, 0x8E);
	idt_set_gate(25, (uint32_t)isr25, 0x08, 0x8E);
	idt_set_gate(26, (uint32_t)isr26, 0x08, 0x8E);
	idt_set_gate(27, (uint32_t)isr27, 0x08, 0x8E);
	idt_set_gate(28, (uint32_t)isr28, 0x08, 0x8E);
	idt_set_gate(29, (uint32_t)isr29, 0x08, 0x8E);
	idt_set_gate(30, (uint32_t)isr30, 0x08, 0x8E);
	idt_set_gate(31, (uint32_t)isr31, 0x08, 0x8E);

	// 255 将来用于实现系统调用
	idt_set_gate(255, (uint32_t)isr255, 0x08, 0x8E);

	// 更新设置中断描述符表
	idt_flush((uint32_t)&idt_ptr);
}

// 设置中断描述符
static void idt_set_gate(uint8_t num, uint32_t base, uint16_t sel, uint8_t flags)
{
	idt_entries[num].base_lo = base & 0xFFFF;
	idt_entries[num].base_hi = (base >> 16) & 0xFFFF;

	idt_entries[num].sel     = sel;
	idt_entries[num].always0 = 0;

	// 先留下 0x60 这个魔数，以后实现用户态时候
	// 这个与运算可以设置中断门的特权级别为 3
	idt_entries[num].flags = flags;  // | 0x60
}

// 调用中断处理函数
void isr_handler(pt_regs *regs)
{
	if (interrupt_handlers[regs->int_no]) {
	      interrupt_handlers[regs->int_no](regs);
	} else {
		printk_color(rc_black, rc_blue, "Unhandled interrupt: %d\n", regs->int_no);
	}
}

// 注册一个中断处理函数
void register_interrupt_handler(uint8_t n, interrupt_handler_t h)
{
	interrupt_handlers[n] = h;
}
ude "string.h"
#include "debug.h"
#include "idt.h"
//中断描述符表
idt_entry_t idt_entries[256];

//IDTR
idt_ptr_t idt_ptr;

//中断处理函数的指针数组
interrupt_handler_t interrupt_handlers[256];

//设置中断描述符
static void idt_set_gate(uint8_t num, uint32_t base,uint16_t sel,uint8_t flags);

//声明加载 IDTR的函数
extern void idt_flush(uint32_t);

//初始化中断描述符表
void init_idt()
{
    bzero((uint8_t *)&interrupt_handlers, sizeof(interrupt_handler_t) * 256);
    idt_ptr.limit = sizeof(idt_entry_t) * 256 -1;
    idt_ptr.base = (uint32_t)&idt_enteries;
    
    bzero((uint8_t *)&idt_entries, sizeof(idt_entry_t) * 256);

    //0-32：用于CPU的中断处理
    idt_set_gate(0,(uint32_t)isr0, 0x08,0x8E);
    
}
```

加载中断描述符表的函数：

idt/idt_s.s

```assembly
[GLOBAL idt_flush]
idt_flush:
	mov eax,[esp+4]		;参数存入eax寄存器
	lidt [eax]			;加载到IDTR
	ret
.end:
```

将上述所有代码整理完毕后，修改入口函数测试中断：

init/entry.c

```c
#include "console.h"
#include "debug.h"
#include "gdt.h"
#include "idt.h"

int kern_entry()
{
    init_debug();//初始化debug信息
    init_gdt();//初始化全局描述符表
    init_idt();//初始化中断描述符表
    
    console_clear();
    printk_color(rc_black, rc_green, "el psy congroo\n");
    asm volatile ("int $0x3");
    asm volatile ("int $0x4");
    
    return 0;
}
```

#### PS1:关于volatile关键字。

volatile的本意是“易变化的”，因为访问寄存器要比访问内存单元快的多，所以编译器一般都会作减少存取内存的优化，但有可能会读脏数据。当要求使用volatile声明变量值时，系统总是重新从它所在的内存读取数据，即使它前面的指令刚刚从该处读取过数据。精确的说就是，遇到这个关键字声明的变量，编译器对访问该变量的代码就不再进行优化，从而可以提供对特殊地址的稳定访问；如果不使用valatile，则编译器将对所声明的语句进行优化。

简单来说就是，volatile关键词影响编译器的编译结果，用volatile声明的变量表示该变量随时可能发生变化，与该变量有关的运算，不要进行编译优化，以免出错。

#### PS2:关于extern关键字。

##### 一、extern是关于声明的关键字

已知变量的声明有两种情况：

1. 需要建立存储空间的。如int a 在声明时已经建立了存储空间。
2. 不需要建立存储空间的，通过使用extern关键字声明变量名而不定义它。如：extern int a 其中变量a可以在别的文件中定义。

总结：除非有extern关键字，否则都是变量的定义。

```c
extern int i; //声明，不是定义
int i;		 //声明，也是定义
```

##### 二、extern的用法详解

**C语言中，修饰符extern主要用在变量或函数的声明前，用来说明“此变量/函数是在别处定义的，要在此处引用”。**

1. extern修饰变量的声明

   **如果文件a.c 需要引用b.c 中变量int v，就可以在a.c 中声明extern int v，然后就可以引用变量v**。能够被其他模块以entern修饰符引用到的变量**通常是全局变量**。

   PS：**extern int v 可以放在a.c中的任何地方，如可在a.c中的函数fun定义的开头处声明extern int v，然后就可以引用到变量v了**，不过这样只能在函数fun作用域中应用v，这是变量作用域的问题。

2. extern修饰函数声明

   从**本质来说，变量和函数没有区别。**函数名是指向函数二进制块开头处的指针。若a.c需要引用b.c中的函数，如在b.c中原型是int fun(int mu)，那么就可以在a.c中声明extern int fun(int mu)，然后就能用fun来做任何事情。**如同变量的声明，extern int fun(int mu)可以放在a.c中的任何地方，而不一定非要放在a.c的文件作用域的范围中。**对其他模块中函数的引用，**最常用的方法是包含这些函数声明的头文件**。

   相比于包含头文件，extern的引用方式要简洁的多，能够加速编译，节省时间。

##### 三、文件操作举例

我们希望在头文件中定义一个全局变量，然后包含到两个不同的c文件中，希望这个全局变量能在两个文件中共用。

解决方案：

使用extern关键字来声明为外部变量。具体说就是在其中一个c文件**定义**一个全局变量key，然后再另一个要使用key这个变量的c文件中使用extern关键字**声明**一次，说明该变量为外部变量，是在其他的c文件中定义的全局变量。（关键字：定义和声明）如在main.c中定义变量key，在common.c中声明key变量为外部变量，这样这两个文件中就能共享这个变量key了。

