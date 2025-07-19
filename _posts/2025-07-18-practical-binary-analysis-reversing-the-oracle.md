---
layout: post
title: 'Practical Binary Analysis: Reversing the Oracle'
date: 2025-07-18 22:03 -0400
categories: [CTF, Practical Binary Analysis]
tags: [ctf, elf, binary, ida]
image:
    path: /assets/img/pba.png
    alt: Practical Binary Analysis
---

I've decided to take a break and approach Flag 8 another time, as it appears to be far more of a challenge than I originally anticipated, involving getting into the weeds with steganography and file representation. Instead, I wanted to make a post about something related but not directly in the scope of this CTF. Namely, I wanted to reverse engineer the **oracle** program to understand the details of how it generates each level and how it verifies each level's flag. 

## <span style="color:red">Decompilation and Static Analysis </span>

Lets start our efforts by checking to see if there are any interesting strings in the binary

```
kishoreg@kishoreg-G3-3779:~/Desktop/pba/oracle_extract$ strings oracle
/lib64/ld-linux-x86-64.so.2
libcrypto.so.1.0.0
_ITM_deregisterTMCloneTable
__gmon_start__
_Jv_RegisterClasses
_ITM_registerTMCloneTable
BIO_ctrl
BIO_write
BIO_push
BIO_set_flags
BIO_read
BIO_free_all
BIO_new
BIO_s_mem
BIO_new_mem_buf
BIO_f_base64
libcrypt.so.1
crypt
libc.so.6
__printf_chk
fopen
puts
__stack_chk_fail
fgets
strlen
__fprintf_chk
memcpy
fclose
stderr
fwrite
strcmp
__libc_start_main
OPENSSL_1.0.0
GLIBC_2.14
GLIBC_2.4
GLIBC_2.2.5
GLIBC_2.3.4
UH-@!`
AWAVI
AUATLc
[]A\A]A^A_
AWAV
AUAT
[]A\A]A^A_
AWAVAUATA
[]A\A]A^A_
AWAVA
AUATL
[]A\A]A^A_
$1$pba
levels.db
Failed to open %s
Base64 decode failed
Level exceeds maximum size
Failed to write %s
Usage: %s <flag> [-h]
Invalid flag: %s
$1$pba$cC.J56kTHt0f2BeCmPY0S0
$1$pba$ThmQr44j/SzgLnGGr3z0t.
$1$pba$bVbBZVMYF.yq/D95S14hi/
$1$pba$eZVsO8xkHTfWXQJlVj3vF.
$1$pba$9dVFs3IW334QFtvvh1ZkF0
$1$pba$/DR3oJXK6fqvBgD4db3EJ0
$1$pba$KjeCu9tJUVtd24nC4g75L/
$1$pba$mMAAETuTP0ixunRdwG9PT0
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
| Level %d completed, unlocked %-12s |
Run oracle with -h to show a hint
Use the flag obtained in the previous level to unlock the next
-h: Show a hint for the level unlocked by the given <flag>
Failed to load levels, missing/corrupted level file?
Failed to unlock level, corrupted level file?
;*3$"
GCC: (Ubuntu 5.4.0-6ubuntu1~16.04.10) 5.4.0 20160609
.shstrtab
.interp
.note.ABI-tag
.note.gnu.build-id
.gnu.hash
.dynsym
.dynstr
.gnu.version
.gnu.version_r
.rela.dyn
.rela.plt
.init
.plt.got
.text
.fini
.rodata
.eh_frame_hdr
.eh_frame
.init_array
.fini_array
.jcr
.dynamic
.got.plt
.data
.bss
.comment
```

Some interesting things that pop out include what appear to be MD5-based Unix ```crypt()``` password hashes, with a salt. 

First, lets throw our **oracle** binary into IDA and see if we can figure out what's going on. 

![stage1 strings](/assets/img/ida.png)

I renamed some function calls after determining what their intended purpose is. The main thing to see is that the flag user input is passed to a function I called ```MakeLowercase``` which merely makes any uppercase ASCII characters in the flag into lowercase. Next, it is passed to a function I determined was performing MD5 hash matching, so I named it ```MD5Match```. Lets look a little deeper at this function

![ida2](/assets/img/ida2.png)

As we suspected earlier, it appears to use ```crypt()``` with the salt ```$1$pba``` to indicate an MD5-crypt hash calculation. This is compared to an array **s2** which likely holds the hardcoded hashes we saw earlier to see if there are any matches. The index ```v1``` is there to ensure we don't go past 8, which is the total number of levels we have. On failure, this function return -1. Lets take a look at the ```.data``` section, which is where **s2** resides.

![ida3](/assets/img/ida3.png)

Here we can see all 8 hashes that are used for the comparison. Unfortunately, the only option to get a flag from one of these hashes and the salt alone is by bruteforce, and given that each password is 32 characters of randomly generated hex, we can't feasibly use a dictionary attack either. 