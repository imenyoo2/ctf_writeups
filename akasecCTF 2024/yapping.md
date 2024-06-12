# Description

![](../media/Screenshot%202024-06-10%20at%2016-05-35%20Akasec%20CTF%202024.png)

## analyzing the binary
decompiling the binary we can figure out what it does, it just read input from user and then print an ascii art

![](../media/Screen%20Shot%202024-06-10%20at%2010.49.03%20PM.png)

### user input
the user input is provided in the form of chunks of 8 bytes, the last iteration is when `local_c > 105` and that will be when `local_c == 112`, so we can overflow the buffer but we only have 4 bytes after the buffer to override.

when we examine the assembly code we find that `local_c` is at the position `RBP - 0x4` and read buffer is at `RBP - 0x70` meaning that `local_c` is lower in the stack so we can override it.

now that we control the loop counter, we can override the return value by calculating the offset between `buffer` and the return value and override `i` accordingly.

### calculating the offset
we can use gdb for this, first we set a break point just before the end of `vuln`, run and provide random input, then we can print the stack and evaluate the offset.

![](../media/Screen%20Shot%202024-06-12%20at%201.20.04%20AM.png)
![](../media/Screen%20Shot%202024-06-12%20at%201.13.55%20AM.png)

```
                   ┌────────────────────┐◄────── RSP                           ┌────────────────────┐◄────── RSP
                   │                    │                                      │                    │           
                   │                    │                                      │                    │           
                   │                    │                                      │                    │           
RBP - 0x70 ───────►│                    │                                      │                    │           
                   │                    │                                      │                    │           
                   │                    │                                      │                    │           
                   │                    │                                      │                    │           
                   │                    │                                      │                    │           
                   │                    │                   RBP - 0x70 ───────►│                    │           
                   │    vuln's stack    │                                      │    vuln's stack    │           
                   │                    │                                      │                    │           
                   │                    │       After poping RBP and           │                    │           
                   │                    │       jumping to gadget              │                    │           
                   │                    │       ───────────────────────►       │                    │           
                   │                    │                                      │                    │           
 RBP - 0x4 ───────►│                    │                                      │                    │           
                   ├────────────────────┤ ◄────── RBP                          ├────────────────────┤           
                   │                    │                                      │                    │           
                   │                    │                                      │                    │           
                   │                    │                                      │                    │           
                   │    main's stack    │                                      │    main's stack    │           
                   │                    │                                      │                    │           
                   │                    │                    RBP - 0x4 ───────►│                    │           
                   └────────────────────┘                                      └────────────────────┘◄────── RBP 
```

