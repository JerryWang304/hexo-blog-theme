---
title: shellcode开发
date: 2016-07-27 07:57:04
tags:
- 计算机安全
---

shellcode开发是一件非常麻烦的事情，programmer必须要掌握汇编语言。网上搜集到的shellcode都是机器码，要想在本机调试运行，我们可以使用下面简单的代码来装载：
```C
char shellcode[]= "...." //十六进制的机器码
// 如果shellcode调用了其它的库，这里得include进来
void main() {
    __asm {
        lea eax, shellcode ;eax寄存器里是shellcode代码的地址
        push eax
        ret ;将地址赋值给IP寄存器
    }
}
```
