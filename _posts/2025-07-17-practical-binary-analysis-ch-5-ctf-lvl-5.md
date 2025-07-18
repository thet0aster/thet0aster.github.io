---
layout: post
title: 'Practical Binary Analysis Ch. 5 CTF: Lvl 5'
date: 2025-07-17 19:04 -0400
categories: [CTF, Practical Binary Analysis]
tags: [ctf, elf, binary, objdump]
image:
    path: /assets/img/pba.png
    alt: Practical Binary Analysis
---
This is my writeup on Level 5 of the [Practical Binary Analysis][def] CTF challenge. This flag ended up being a lot more involved in order to successfully capture it. 

## <span style="color:red">Fifth Flag: lvl5</span>

If we pass **lvl4**'s flag to the **oracle**, we receive the following hint

![lvl5 flag](/assets/img/lvl5_1.png)

We are greeted with poem of some sort which seems to hint that the flag lies in unused code within the binary and we need to figure out how to redirect to that code, and we should focus our efforts on static analysis rather than dynamic. Seems simple enough. Lets start with a basic ```strings``` analysis on the file

![lvl5 strings](/assets/img/lvl5_2.png)

We see some strings related to a key and a decrypted flag, which we don't see when we run the program. Let's dump ```.rodata``` and see what we can find there

![lvl5 rodata](/assets/img/lvl5_3.png)

It looks like those two strings begin at ```0x400774``` and ```0x400782``` respectively. Lets see where they are accessed in the disassembly

![lvl5 disassembly](/assets/img/lvl5_4.png)

We find two instructions that access the address of these string constants. Now we look at the ```objdump``` output to see where those instructions are

![lvl5 disassembly2](/assets/img/lvl5_5.png)

We see a large chunk of code, which seems like an unused function, that does some interesting things. One thing that sticks out immediately is the 4 ```movabs``` instructions which move four 8-byte immediate magic values into the stack. Maybe this has something to do with our flag? Both of the strings that we saw earlier seem to be within this chunk of code. We just need a way to execute it.

Notice that the starting address of the function appears to be at ```0x400620```, where ```rbx``` is pushed onto the stack. What if we were able to overwrite a pointer somewhere with this address so it jumps here instead? Luckily for us, there is a perfect place for this:

![lvl5 entry](/assets/img/lvl5_6.png)

If we recall, our ELF binaries don't start executing at ```main()```. If we pass our binary to ```readelf``` to examine the executable header, we note that the entry point is at ```0x400520```. This eventually jumps to a PLT stub,  ```__libc_start_main@plt```, which sets up the environment for libc.

Here we have a perfect attack vector! It's kind of up to us the route we want to go at this point to inject our target function address. We could do some GOT hijacking and insert the address into the GOT entry referenced by the PLT stub, or even simpler, we change the value passed to the first argument of ```__libc_start_main```, which is supposed to be a function pointer to the user-defined ```main()```. I decided to go with the latter option

![lvl5 entry](/assets/img/lvl5_7.png)

Running the binary now gets us the final flag:

![lvl5 entry](/assets/img/lvl5_8.png)

It looks like the "key" was simply the hidden function's address. The flag for **lvl5** is <blur>0fa355cbec64a05f7a5d050e836b1a1f</blur>


[def]: https://practicalbinaryanalysis.com
[ELF]: https://refspecs.linuxfoundation.org/elf/gabi4+/ch4.eheader.html