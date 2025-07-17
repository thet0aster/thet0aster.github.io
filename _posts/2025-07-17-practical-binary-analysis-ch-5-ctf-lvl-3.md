---
layout: post
title: 'Practical Binary Analysis Ch. 5 CTF: Lvl 3'
date: 2025-07-17 13:17 -0400
categories: [CTF, Practical Binary Analysis]
tags: [ctf, elf, binary, objdump]
image:
    path: /assets/img/pba.png
    alt: Practical Binary Analysis
---

This post is my writeup detailing the solution to the Level 3 CTF challenge in the book [Practical Binary Analysis][def]. The previous writeup detailing the Level 1 and Level 2 flags can be found [here](/posts/_posts/2025-07-17-practical-binary-analysis-ch-5-ctf-lvl-1.md).

## <span style="color:red">Third Flag: lvl3</span>
At this point, if you are following along, you should have captured the **lvl2** flag from the [previous writeup](/posts/_posts/2025-07-17-practical-binary-analysis-ch-5-ctf-lvl-1.md), which is <blur>034fc4f6a536f2bf74f8d6d3816cdf88</blur>. Throwing this at the oracle gives us a hint as to what to look for

![lvl3 hint](/assets/img/lvl3_1.png)

Next, lets take a look at the **lvl3** binary itself. Giving it execute permissions and attempting to run it seems to fail as it doesn't appear that the format of the ELF is designed to run on this machine. Maybe the file has corrupted fields in the executable header that we need to fix. Let's throw it in ```readelf``` and see what we find

![lvl3 readelf](/assets/img/lvl3_2.png)

From the output it appears that some fields in the executable header are incorrect. Namely, the ABI, the Machine, and the program header table offset. These are three of the four broken things the hint told us to look for. Lets fix them one by one. For a 64-bit program, the executable header used is of type ```Elf64_Ehdr```:
```c
typedef struct {
    unsigned char e_ident[EI_NIDENT];
    uint16_t      e_type;
    uint16_t      e_machine;
    uint32_t      e_version;
    ElfN_Addr     e_entry;
    ElfN_Off      e_phoff;
    ElfN_Off      e_shoff;
    uint32_t      e_flags;
    uint16_t      e_ehsize;
    uint16_t      e_phentsize;
    uint16_t      e_phnum;
    uint16_t      e_shentsize;
    uint16_t      e_shnum;
    uint16_t      e_shstrndx;
} ElfN_Ehdr;
```

We can use the ```hexedit``` tool to modify specific bytes in the file. The first value we need to change is the ```EI_OSABI``` field in the executable header's 16 byte ```e_ident``` array, which is at offset 8 in the file. In order to set it to the standard System V ABI, we need to set this byte to 0. We do something similar for the ```e_machine``` field at offset 18 in the executable header. According to the [ELF specification][ELF], we need the value for **EM_X86_64**, which is 62, or 0x3E in hex. Setting these values now yields the following

![lvl3 readelf](/assets/img/lvl3_3.png)

The ```e_phoff``` field is at offset 32 in the file and it appears the current value is corrupted to ```0xDEADBEEF```.

![lvl3 readelf](/assets/img/lvl3_4.png)

While the [ELF specification][ELF] makes no guarantees about the particular offset the program header table is to be located in the binary, we can often times assume it will appear right after the executable header. So lets try changing that ```0xDEADBEEF``` to ```0x40```, to be right after the 64-byte executable header. And this seems to work! We are able to run the binary and retrieve a flag.

![lvl3 fake flag](/assets/img/lvl3_5.png)

...or so we thought. Unfortunately it seems like a red-herring, and we're going to have to fix one more thing before we get the correct flag. First, lets try to dump the code and see if anything stands out

![lvl3 empty text section](/assets/img/lvl3_6.png)

Strange. It appears there is no code disassembled in the ```.text``` section. Why not? Let's see the section header flags of all the sections to see what's going on

![lvl3 nobits](/assets/img/lvl3_7.png)

It looks like the ```.text``` section has type ```SHT_NOBITS```, which indicates that the section should not contain any bytes in the file. This type is mainly used for uninitialized global data in the ```.bss``` section, which doesn't occupy any space in the ELF file but is created dynamically when the file is loaded into memory. So it appears all we need to do is change ```SHT_NOBITS``` to ```SHT_PROGBITS```, or from 8 to 1. We can see that the ```.text``` section is at offset 14 in the section header table, so we can simply use ```e_shoff``` and ```e_shentsize``` to calculate the byte offset to get to the ```.text``` section header. Then, we can add 4 bytes to get the ```sh_type``` offset into the section header and set it to 1. The final value we need to change is at ```0x1504``` bytes into the file

```c
typedef struct {
    uint32_t   sh_name;
    uint32_t   sh_type;
    uint64_t   sh_flags;
    Elf64_Addr sh_addr;
    Elf64_Off  sh_offset;
    uint64_t   sh_size;
    uint32_t   sh_link;
    uint32_t   sh_info;
    uint64_t   sh_addralign;
    uint64_t   sh_entsize;
} Elf64_Shdr;
```

Once we fix this, we can run the binary again to get the true flag:

![lvl3 true flag](/assets/img/lvl3_8.png)

The final flag for level 3 is <blur>3a5c381e40d2fffd95ba4452a0fb4a40</blur>

[def]: https://practicalbinaryanalysis.com
[ELF]: https://refspecs.linuxfoundation.org/elf/gabi4+/ch4.eheader.html