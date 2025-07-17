---
layout: post
title: 'Practical Binary Analysis Ch. 5 CTF: Lvl 4'
date: 2025-07-17 14:59 -0400
categories: [CTF, Practical Binary Analysis]
tags: [ctf, elf, binary, ltrace]
image:
    path: /assets/img/pba.png
    alt: Practical Binary Analysis
---
This is my writeup for Level 4 of the [Practical Binary Analysis][def] CTF challenge. Unlike the others, this flag was incredibly obvious and easy to retrieve, so I won't waste any time going into unecessary details and just get straight to the point

## <span style="color:red">Fourth Flag: lvl4</span>

Passing the **lvl3** flag <blur>3a5c381e40d2fffd95ba4452a0fb4a40</blur> to the **oracle** program gives us this hint.

![lvl4 hint](/assets/img/lvl4_1.png)

Clearly it is telling us to inspect what the program seems to do while running. This means examining any syscalls or external library calls it makes. Let's first start with ```strace```:

![lvl4 strace](/assets/img/lvl4_2.png)

Nothing particularly sticks out to me here. Maybe ```ltrace``` will have some more useful info?

![lvl4 ltrace](/assets/img/lvl4_3.png)

Well that seems quite obvious... it seems that the flag is simply set by the program in an environment variable. Let's pass it to the oracle to see what happens. And sure enough...

![lvl4 flag](/assets/img/lvl4_4.png)

It works. Definitely thought it was going to be another red-herring, but it seems it was just that simple. The final flag was <blur>656cf8aecb76113a4dece1688c61d0e7</blur>. Now onto Level 5!


[def]: https://practicalbinaryanalysis.com
[ELF]: https://refspecs.linuxfoundation.org/elf/gabi4+/ch4.eheader.html