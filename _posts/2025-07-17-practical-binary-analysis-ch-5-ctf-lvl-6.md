---
layout: post
title: 'Practical Binary Analysis Ch. 5 CTF: Lvl 6'
date: 2025-07-17 20:03 -0400
categories: [CTF, Practical Binary Analysis]
tags: [ctf, elf, binary, objdump]
image:
    path: /assets/img/pba.png
    alt: Practical Binary Analysis
---

This is my solution to capture the Level 6 flag in the [Practical Binary Analysis][def] CTF challenge. I had some trouble with this flag due to some technical issues with discrepancies between the code on the VM and the code archive on the website, which I will explain later.

## <span style="color:red">Sixth Flag: lvl6</span>

Running the **lvl6** binary prints out a series of of prime numbers from 2 to 97

![lvl6 init](/assets/img/lvl6_1.png)

Strange, doesn't appear to do anything else. Lets see if ```ltrace``` gives us something useful

![lvl6 init](/assets/img/lvl6_2.png)

It seems these printf calls are merely calls for the sequence of numbers we see printed out on the screen, and there are no other interesting library calls here. Disappointing. Maybe if we check the strings within the binary we'll find something interesting

![lvl6 init](/assets/img/lvl6_3.png)

We see two strings that are interesting. One appears to reference a command line argument and also ```get_data_addr```. We've seen this exact layout before in the initial **lvl1** flag, so why not try passing ```get_data_addr``` as a command line argument and seeing if anything changes?

![lvl6 init](/assets/img/lvl6_4.png)

Looks like something did change. We now see that there was a ```strcmp``` that checks for the value of ```get_data_addr```. Perhaps we would have noticed this if we passed in garbage data for the command line arg initially, but I guess we were lucky and skipped that step. We see that an environment variable ```DATA_ADDR``` is set to ```0x4006c1```. This definitely seems like its hinting to check a program address, so lets see where that takes us.

![lvl6 init](/assets/img/lvl6_5.png)

This appears to take us to an address within our executable ```.text```, but if you analyze the instructions, they appear to be pretty obscure. The only instruction that seems to be recognizable is the ```lea``` instruction. Coincidentally, there are exactly 16 bytes, or 32 nybbles, between the start of the ```DATA_ADDR``` address and that ```lea``` instruction, which is the exact length we need for our flag. It seems the hint was suggesting that there was data mixed in with instructions in the ```.text``` section. Lets concatenate the bytes in order and pass them to the oracle and see if we have captured the correct flag.

![lvl6 init](/assets/img/lvl6_6.png)

Huh? I'm sure I used the correct sequence, <blur>2e29c64a0f03a6ee2a307fecc8c3ff42</blur>, why didn't it work? Well, I ended up spending way too much time on trying to figure out what the correct flag is when eventually I realized I had it correct all along, the **oracle** program was just broken

It appears the **oracle** binary pre-installed on the VM under /home/binary/code/chapter5 had a bug in it somewhere that doesn't accept the coprrect flag. Perhaps it is an earlier version of the binary that they forgot to update. In any case, I downloaded the code archive from the website and ran the same command on the **oracle** packaged within and it accepted the flag:

![lvl6 init](/assets/img/lvl6_7.png)

I submitted an email to the author requesting that this be added to the **errata** on the [website][def] and am still awaiting a response. In any case, the correct flag should be <blur>2e29c64a0f03a6ee2a307fecc8c3ff42</blur>


[def]: https://practicalbinaryanalysis.com
[ELF]: https://refspecs.linuxfoundation.org/elf/gabi4+/ch4.eheader.html