---
layout: post
title: 简单的keygenme
category: 逆向
tags: [逆向]
---

## CrackHead

### 过程分析
1.首先输入假码搜集信息，似乎没有反应。

![img](/assets/images/crackhead1.png)

2.载入x32dbg，看看调用，发现这个。

![img](/assets/images/crackhead2.png)

3.来到这里，发现下面有成功弹窗。判断此为关键call，F7跟进。

![img](/assets/images/crackhead3.png)

4.发现这里有我们的假码,详见注释。

![img](/assets/images/crackhead4.png)

5.F8单步回到这里，将eax（我们的码），与esi真码对比，若不同则跳转（gg）。

![img](/assets/images/crackhead5.png)



### CrackHead‘s  Keygen

```c
#include <windows.h> 
#include <stdio.h> 
#include <stdlib.h>

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
	GetVolumeInformation(a,VolumeName,12,&VolumeSerialNumber,NULL,NULL,NULL,10); //VolumeName卷标
	
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


