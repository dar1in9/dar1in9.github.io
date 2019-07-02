---
layout: post
title: 简单的keygenme
category: 逆向
tags: [逆向]
---

## CrackHead

### 过程分析
1.首先输入假码搜集信息，似乎没有反应。

![img](/assets/images/2018-12-19-crackheads/crackhead1.png)

2.载入x32dbg，看看调用，发现这个。

![img](/assets/images/2018-12-19-crackheads/crackhead2.png)

3.来到这里，发现下面有成功弹窗。判断此为关键call，F7跟进。（如要爆破，将jne nop掉即可）

![img](/assets/images/2018-12-19-crackheads/crackhead3.png)

4.发现这里有我们的假码,详见注释。

![img](/assets/images/2018-12-19-crackheads/crackhead4.png)

5.F8单步回到这里，将eax（我们的码），与esi真码对比，若不同则跳转（gg）。

![img](/assets/images/2018-12-19-crackheads/crackhead5.png)

6.那么esi是哪来的。

7.看这个地址，在内存窗口按ctrl+g转到这个地址，右键下内存写入断点，F9观察。

![img](/assets/images/2018-12-19-crackheads/crackhead6.png)

8.发现出现一串字符串。

![img](/assets/images/2018-12-19-crackheads/crackhead7.png)

9.神奇。

![img](/assets/images/2018-12-19-crackheads/crackhead8.png)

10.回到调用，发现这俩。

![img](/assets/images/2018-12-19-crackheads/crackhead9.png)

11.google一下，发现一个是返回磁盘类型（本地磁盘是3），一个是返回卷标。

12.我们在GetVolumeInformation下断，F9来到这，详见注释。

![img](/assets/images/2018-12-19-crackheads/crackhead10.png)

13.下面是我写的keygen。（注意读取的卷标存入ebx中是倒着的！！）

![img](/assets/images/2018-12-19-crackheads/crackhead11.png)

### CrackHead‘s  Keygen

```c
#include <windows.h> 
#include <stdio.h> 

int main(int argc,char **argv) 
{ 
	DWORD   VolumeSerialNumber; 
	int i;
	char VolumeName[256];
	char a[5] = "c:\\";
	char b[5];
	int c; 
	printf("请输入软件所在磁盘:"); 
	scanf("%c",&a[0]);
	GetVolumeInformation(a,VolumeName,12,&VolumeSerialNumber,NULL,NULL,NULL,10);
	
	for(i=3;i>=0;i--)
	{
		b[i] = VolumeName[3-i];
	}//小端顺序 
	
	for(i=0;i<4;i++)
	{
		c*=256;
		c+=b[i];
	}
	c*=6;
	c^=0x797A7553;
	printf("%d",c);
    return 0; 
} 
```
