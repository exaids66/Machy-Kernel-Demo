# Machy-Kernel-Demo
>  @Brief: This is a project repository about a kernel demo based on the X86 architecture
>
> @Author: [machy]([exaids66 (machy) (github.com)](https://github.com/exaids66))

****

## Introduction

The project's main objective was to learn to build a simple operating system kernel on an IBM PC and its compatible computers based on the Intel x86 architecture. The following documentation was referenced: [hurlex](http://wiki.0xffffff.org/)

****

PS: Some of the notes were lost due to improper preservation in the early days, and now only the note report after experiment 6 remains, with the hope that it can be restored in the future.

****

总结一下，搭建一个基于GRUB引导程序的toy OS kernel需要哪些知识

- Linux下的存储维护相关的命令
  用于制作GRUB引导盘

- 汇编语言，NASM或者AT&T。在x86架构下进行汇编的能力
  NASM和Intel汇编语法一样，更简单，AT&T也有学的必要

- C语言的高级用法
  需要看一遍《C语言专家编程》，掌握一些C语言的高级用法

- GCC的使用
  起码知道做一个内核要用GCC的哪些参数

- 链接器ld的使用
  要做到会写ld脚本的水平

- Makefile脚本
  会写Makefile脚本才是一个合格的程序员

- QEMU的使用
  需要学会QEMU和GDB的联合调试

- 内核调试工具的使用

- multiboot规范
  使用GRUB引导，必须熟知Multiboot规范，并且必须熟知使用GRUB引导后机器处于什么状态

- 熟知x86架构下的各种CPU细节、规范
  可以下一份Intel的开源文档来看

- 熟知操作系统原理
  看那本操作系统概念
