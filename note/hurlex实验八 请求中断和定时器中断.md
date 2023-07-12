# hurlex实验八 请求中断和定时器中断

已知：外设的所有中断由中断控制芯片8259A统一汇集后连接到CPU的INTR引脚。

## 本章内容：探究8259A PIC的初始化和实现定时器的中断处理

### 1、IBM PC/AT 8259A PIC架构

8259A PIC每一片都可以管理8个中断源，显然一般情况下设备数量会超过这个值。 为解决这一问题，IBM的设计方案是使用8259APIC的级联功能，使用两片级联（主片、从片）的方式来管理硬件中断。

其中主片的INT端连接到CPU的INTR引脚，从片的INT连接到主片的IR2引脚。如图所示：

![屏幕截图 2021-12-12 143348](https://raw.githubusercontent.com/exaids66/imgs/main/images/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202021-12-12%20143348.jpg)

PS->  **IRQ：中断请求信号**

已知0~31号中断是CPU使用和保留的，**用户可以使用的中断从32号开始**。故IRQ0对应的中断号就是32号，IRQ1就是33号，以此类推。

### 2、实现代码

#### 1）对8259A PIC的初始化

在init_idt(设置中断描述符表的函数)最前面加上如下代码：

idt/idt.c

```c
//初始化中断描述符表
void init_idt()
{
    //重新映射IRQ表
    //两片级联的Intel 8259A芯片
    //主片端口 0x20 0x21
    //从片端口 0xA0 0xA1
    
    //初始化主片、从片
    //0001 0001
    outb(0x20,0x11);
    outb(0xA0,oxA1);
    
    outb(0x21,0x20)//设置主片IRQ从0x20（32）号中断开始
        
    outb(0xA1,0x28)//设置从片IRQ从0x28（40）号中断开始
        
    outb(0x21,0x04)//设置主片IR2引脚连接从片
        
    outb(0xA1,0x02)//告诉从片输出引脚和主片IR2号相连
        
    outb(0x21,0x01)//设置主片和从片按照8086方式工作
    outb(0xA1,0x01)
        
    outb(0x21,0x0);//设置主从片允许中断
    outb(0xA1,0x0);
    ...
}
```

PS：前置知识科目：微机原理、接口技术

#### 2）添加IRQ处理函数

在idt.h头文件末尾添加如下内容

include/idt.h

```c
// IRQ 处理函数
void irq_handler(pt_regs *regs);

// 定义IRQ
#define  IRQ0     32 	// 电脑系统计时器
#define  IRQ1     33 	// 键盘
#define  IRQ2     34 	// 与 IRQ9 相接，MPU-401 MD 使用
#define  IRQ3     35 	// 串口设备
#define  IRQ4     36 	// 串口设备
#define  IRQ5     37 	// 建议声卡使用
#define  IRQ6     38 	// 软驱传输控制使用
#define  IRQ7     39 	// 打印机传输控制使用
#define  IRQ8     40 	// 即时时钟
#define  IRQ9     41 	// 与 IRQ2 相接，可设定给其他硬件
#define  IRQ10    42 	// 建议网卡使用
#define  IRQ11    43 	// 建议 AGP 显卡使用
#define  IRQ12    44 	// 接 PS/2 鼠标，也可设定给其他硬件
#define  IRQ13    45 	// 协处理器使用
#define  IRQ14    46 	// IDE0 传输控制使用
#define  IRQ15    47 	// IDE1 传输控制使用

// 声明 IRQ 函数
// IRQ:中断请求(Interrupt Request)
void irq0();		// 电脑系统计时器
void irq1(); 		// 键盘
void irq2(); 		// 与 IRQ9 相接，MPU-401 MD 使用
void irq3(); 		// 串口设备
void irq4(); 		// 串口设备
void irq5(); 		// 建议声卡使用
void irq6(); 		// 软驱传输控制使用
void irq7(); 		// 打印机传输控制使用
void irq8(); 		// 即时时钟
void irq9(); 		// 与 IRQ2 相接，可设定给其他硬件
void irq10(); 		// 建议网卡使用
void irq11(); 		// 建议 AGP 显卡使用
void irq12(); 		// 接 PS/2 鼠标，也可设定给其他硬件
void irq13(); 		// 协处理器使用
void irq14(); 		// IDE0 传输控制使用
void irq15(); 		// IDE1 传输控制使用
```

之后再idt_s.s中添加相应的处理过程

gdt/gdt_s.s

```assembly
1 构造中断请求的宏
2 %macro IRQ 2
3 [GLOBAL irq%1]
4 irq%1:
5 cli
6 push byte 0
7 push byte %2
8 jmp irq_common_stub
9 %endmacro
10
11 IRQ 0, 32 ; 电脑系统计时器
12 IRQ 1, 33 ; 键盘
13 IRQ 2, 34 ; 与 IRQ9 相接，MPU−401 MD 使用
14 IRQ 3, 35 ; 串口设备
15 IRQ 4, 36 ; 串口设备
16 IRQ 5, 37 ; 建议声卡使用
17 IRQ 6, 38 ; 软驱传输控制使用
18 IRQ 7, 39 ; 打印机传输控制使用
19 IRQ 8, 40 ; 即时时钟
20 IRQ 9, 41 ; 与 IRQ2 相接，可设定给其他硬件
21 IRQ 10, 42 ; 建议网卡使用
22 IRQ 11, 43 ; 建议 AGP 显卡使用
23 IRQ 12, 44 ; 接 PS/2 鼠标，也可设定给其他硬件
24 IRQ 13, 45 ; 协处理器使用
25 IRQ 14, 46 ; IDE0 传输控制使用
26 IRQ 15, 47 ; IDE1 传输控制使用
27
28 [GLOBAL irq_common_stub]
29 [EXTERN irq_handler]
30 irq_common_stub:
31 pusha ; pushes edi, esi, ebp, esp, ebx, edx, ecx, eax
32
33 mov ax, ds
34 push eax ; 保存数据段描述符
35
36 mov ax, 0x10 ; 加载内核数据段描述符
37 mov ds, ax
38 mov es, ax
39 mov fs, ax
40 mov gs, ax
41 mov ss, ax
42
43 push esp
44 call irq_handler
45 add esp, 4
46
47 pop ebx ; 恢复原来的数据段描述符
48 mov ds, bx
49 mov es, bx
50 mov fs, bx
51 mov gs, bx
52 mov ss, bx
53
54 popa ; Pops edi,esi,ebp...
55 add esp, 8 ; 清理压栈的错误代码和 ISR 编号
56 iret ; 出栈 CS, EIP, EFLAGS, SS, ESP
57 .end:
```

最后是init_idt函数狗仔IRQ的相关描述符和具体的IRQ处理函数

idt/idt.c

```c
// 初始化中断描述符表
2 void init_idt()
3 {
4 ... ...
5 idt_set_gate(31, (uint32_t)isr31, 0x08, 0x8E);
6
7 idt_set_gate(32, (uint32_t)irq0, 0x08, 0x8E);
8 idt_set_gate(33, (uint32_t)irq1, 0x08, 0x8E);
9 idt_set_gate(34, (uint32_t)irq2, 0x08, 0x8E);
10 idt_set_gate(35, (uint32_t)irq3, 0x08, 0x8E);
11 idt_set_gate(36, (uint32_t)irq4, 0x08, 0x8E);
12 idt_set_gate(37, (uint32_t)irq5, 0x08, 0x8E);
13 idt_set_gate(38, (uint32_t)irq6, 0x08, 0x8E);
14 idt_set_gate(39, (uint32_t)irq7, 0x08, 0x8E);
15 idt_set_gate(40, (uint32_t)irq8, 0x08, 0x8E);
16 idt_set_gate(41, (uint32_t)irq9, 0x08, 0x8E);
17 idt_set_gate(42, (uint32_t)irq10, 0x08, 0x8E);
18 idt_set_gate(43, (uint32_t)irq11, 0x08, 0x8E);
19 idt_set_gate(44, (uint32_t)irq12, 0x08, 0x8E);
20 idt_set_gate(45, (uint32_t)irq13, 0x08, 0x8E);
21 idt_set_gate(46, (uint32_t)irq14, 0x08, 0x8E);
22 idt_set_gate(47, (uint32_t)irq15, 0x08, 0x8E);
23
24 // 255 将来用于实现系统调用
25 idt_set_gate(255, (uint32_t)isr255, 0x08, 0x8E);
26
27 ... ...
28 }
29
30 // IRQ 处理函数
31 void irq_handler(pt_regs *regs)
32 {
33 // 发送中断结束信号给 PICs
34 // 按照我们的设置，从 32 号中断起为用户自定义中断
35 // 因为单片的 Intel 8259A 芯片只能处理 8 级中断
36 // 故大于等于 40 的中断号是由从片处理的
37 if (regs−>int_no >= 40) {
38 // 发送重设信号给从片
39 outb(0xA0, 0x20);
40 }
41 // 发送重设信号给主片
42 outb(0x20, 0x20);
43
44 if (interrupt_handlers[regs−>int_no]) {
45 interrupt_handlers[regs−>int_no](regs);
46 }
47 }
```

**其实IRQ和ISR的处理过程很类似：**

ISR：中断服务程序。IRQ：中断请求信号

-  ISR的处理过程是 (isr0 - isr31) -> isr_common_stub -> isr_handler -> 具体的ISR处理函数。
- IRQ的处理过程是 (irq0 - irq15) -> irq_common_stub -> irq_hanlder -> 具体的IRQ处理函数。

#### 3）实现时钟中断的产生和处理

**时钟中断** ：对操作系统内核很重要，它使得CPU无论再执行任何用户或内核的程序时，都能定义的将执行权利交还到CPU手中来。除了**记录时间**外，**时钟中断的处理函数**里通常都是对**进程的调度处理**。 

具体的时钟中断源由8253/8254 Timer产生。若要**按照需要的频率产生中断**，需要先**配置8253、8254 Timer芯片**。代码如下：

drivers/timer.c

```c
1 #include "timer.h"
2 #include "debug.h"
3 #include "common.h"
4 #include "idt.h"
5
6 void timer_callback(pt_regs *regs)
7 {
8 static uint32_t tick = 0;
9 printk_color(rc_black, rc_light_grey, "Tick: %d\n", tick++);
10 }
11
12 void init_timer(uint32_t frequency)
13 {
14 // 注册时间相关的处理函数
15 register_interrupt_handler(IRQ0, timer_callback);
16
17 // Intel 8253/8254 芯片PIT I/端口地址范围是O40h~43h
18 // 输入频率为， 1193180frequency 即每秒中断次数
19 uint32_t divisor = 1193180 / frequency;
20
21 // D7 D6 D5 D4 D3 D2 D1 D0
22 // 0 0 1 1 0 1 1 0
23 // 即就是 36 H
24 // 设置 8253/8254 芯片工作在模式 3 下
25 outb(0x43, 0x36);
26
27 // 拆分低字节和高字节
28 uint8_t low = (uint8_t)(divisor & 0xFF);
29 uint8_t hign = (uint8_t)((divisor >> 8) & 0xFF);
30
31 // 分别写入低字节和高字节
32 outb(0x40, low);
33 outb(0x40, hign);
34 }
```

对应的头文件：

include/timer.h

```c
1 #ifndef INCLUDE_TIMER_H_
2 #define INCLUDE_TIMER_H_
3
4 #include "types.h"
5
6 void init_timer(uint32_t frequency);
7
8 #endif // INCLUDE_TIMER_H_
```

8253/8254 Timer有三种工作模式，我们使用第三种。**init_timer函数的参数是所需的时钟中断的频率**，具体的设置原理不再赘述。

最后修改入口函数进行测试：

init/entry.c

```c
#include "console.h"
#include "debug.h"
#include "gdt.h"
#include "idt.h"
#include "timer.h"

int kern_entry()
{
    init_debug();
    init_gdt();
    init_idt();
    
    console_clear();
    printk_color(rc_black,rc_green,"El Psy Congroo\n");
    
    init_timer(200);
    
    //开启中断
    asm volatile("sti");
    
    return 0;
}
```

