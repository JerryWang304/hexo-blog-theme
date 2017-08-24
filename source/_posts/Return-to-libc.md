---
title: Return-to-libc
date: 2016-07-28 19:09:39
tags:
- 计算机安全
- return-to-libc
---
> 本文简要总结网上的一篇博客 http://www.ibm.com/developerworks/cn/linux/1402_liumei_rilattack/

### DEP与Return-to-libc攻击

缓冲区溢出通常需要将shellcode注入到堆栈中，然后覆盖函数的返回地址，指向shellcode，执行恶意代码。这种攻击方式可以概括为**先写后执行**。数据执行保护策略(Data Execution Prevention，DEP)可以用来防御这种攻击。DEP可以控制程序对内存的访问，被保护的程序只能是**被写或被执行（W⊕X）**，这样就能保证内存不能先被写然后被执行。但是这种方法不能抵御Return-to-libc。Return-to-libc
不需要注入恶意代码，没有同时写和执行的模式。

Return-to-libc利用缓冲区溢出原理，将返回地址覆盖为库函数，并传递设定好的函数，这样就不需要注入可执行代码，即可绕过DEP检测，完成恶意攻击。复杂的Return-to-libc可以调用一系列的库函数来攻击。（库函数可以帮助攻击者完成哪些攻击我现在还不知道，是不是可以完成任意类型的攻击我也不知道，你看到这里你肯定也不知道:)。

### 返回导向编程ROP

Return-to-libc只能调用库函数，攻击能力有限。另一方面，在x86_64平台上，函数参数都是靠寄存器传递，而Return-to-libc靠栈传递参数，所以这种方法会失效。由于这种方法的局限性，ROP（Return-orinted Programming）被提出。ROP不局限于使用库函数，他可以从程序和库函数中选取一组指令，串起来形成shellcode，据说可以实现任意的操作。

### 防御机制
地址空间布局随机化（Address Space Layout Randomization, ASLR）是最好的防御Return-to-libc和ROP的最好的方法。ASLR可以对进程中的堆、栈、代码和共享库里的地址进行随机化。地址被随机化了，那么攻击者就定位不到所需要的函数的地址了。地址混淆（Address Randomization）不仅可以随机化堆、栈、库函数和程序基址，还可以随机化*相对地址*、函数或变量的顺序。
