---
layout: post
title: 'Understanding the ELF Binary Format: Part 2'
date: 2025-07-19 21:32 -0400
categories: [Binary Internals, ELF]
tags: [elf, binary, executable, linux, linker]
image:
    path: /assets/img/binary_analysis.jpg
    alt: Executable and Linkable Format
---

This is Part 2 of my two-part series on the ELF binary format. If you haven't read Part 1, you can read it [here]({% post_url 2025-07-19-understanding-the-elf-binary-format %}). In this writeup, we will explore how an ELF file is loaded into memory and executed. What steps does the kernel take in order to create the process, and how does the dynamic linker perform relocations at runtime? 

At this point, you understand how an ELF binary is structured, including its various components: the program headers, the section headers, the sections, and the executable header. You are familiar with the structures and fields stored in each component of the ELF binary, but unsure of what goes on behind the scenes when it's time to run the executable. Let's find out!

## <span style="color:red">The Dynamic Linker</span>
Let's first understand the role of the dynamic linker, or interpreter, on Linux. The dynamic linker is responsible for **loading** an ELF binary into the virtual address space created by the kernel for the new process. Further, it's responsible for performing **relocations** on references to dynamic symbols.

A dynamic symbol, as opposed to a static symbol, is a symbol that is exported by a shared library. These symbols are shared between multiple processes, so each process doesn't need to waste space by including all of ```libc``` within its own static executable code, leading to massive unnecessary bloat. Nearly every program uses ```libc.so``` to some extent, so it is a much more efficient usage of memory to simply allocate a fixed virtual memory space for it and use memory mapping to share that space among all the processes that require access to its ```.text``` section.

The ```PT_INTERP``` segment, which contains the ```.interp``` section, simply holds a string that represents the path of the linux dynamic linker or interpreter. On my machine, this is ```/lib64/ld-linux-x86-64.so.2```. Once control is transfered to the dynamic linker, it is free to perform its functions of loading the binary, performing relocations, and then finally jumping to the binary's entry address as specified in the executable header in ```e_entry```.

## <span style="color:red">Overview of Dynamic Loading</span>
Let's look at an overview of what happens when you run a new executable in a Unix shell. When you run ```./a.out```, the shell calls ```fork()```, which is a syscall that clones the shell process itself. Then, the cloned child process will call the ```execve()``` syscall with the ELF file to transfer control to the kernel to overwrite its segments with the new binary's data.

#### <span style="color:lightcoral">Step 1: Kernel Creates a Process</span>
The kernel gains control, creates a new child process cloning the shell, and begins the initial process of loading the executable into the virtual address space of the child process. First, it reads the executable header to ensure that it is a valid ELF binary by checking the magic bytes ```\7fELF```. It also checks the other metadata in the executable header, such as the endianness, the address size, and the ELF type.

Using the ```e_phoff``` field, it finds the Program Header Table and begins reading the program headers.

> Note: As mentioned in Part 1, the section header table is completely disregarded at this point. Only program headers matter during loading/execution. 
{: .prompt-info}

#### <span style="color:lightcoral">Step 2: Kernel Loads Segments into Memory</span>

The kernel then walks the PHT, searching for ```PT_LOAD``` segments. Once it finds them it will:

1. Map them into memory using ```mmap()```
2. Apply permissions (R, W, X) using ```mprotect()```
3. Performs any initialization. For the ```.bss``` segment, it loads it into memory and zero-initializes the segment

> As we can see, it is the kernel, NOT the interpreter that initially loads the ```PT_LOAD``` segments of the executable into memory. The interpreter only maps the ```PT_LOAD``` segments of shared libraries into the program, not the ```PT_LOAD``` segments of the program itself. 
{: .prompt-tip}

#### <span style="color:lightcoral">Step 3: Kernel Calls the Interpreter</span>

The kernel then searches the PHT for the ```PT_INTERP``` type to find the path to the dynamic linker. If it exists, the kernel will ```mmap()``` the dynamic linker directly as another ELF executable in the process's virtual memory, repeating the above step. Then, it will jump to the entry point of the dynamic linker, **not the original binary's entry point yet!**

#### <span style="color:lightcoral">Step 4: Dynamic Linker Takes Control and Loads Dependencies</span>

Once the kernel jumps to the dynamic linker's entry point, it begins to perform its task of dynamic linking and loading. The first place it will go is to the PHT to find the ```PT_DYNAMIC``` segment.

If you recall from part 1, ```PT_DYNAMIC``` stores the ```.dynamic``` section, which acts as a roadmap for dynamic linking. It will look for
- ```DT_NEEDED```: External dynamic library dependencies this binary needs to load in
- ```DT_STRTAB``` and ```DT_SYMTAB```: Locations of the dynamic string and symbol tables
- Other runtime linking information

Anything that the dynamic linker needs to find in order to perform dynamic linking, it can find within this segment. It first begins by looking for ```DT_NEEDED``` segments to find the shared libraries that this executable has dependencies for, and it will begin the process of ```mmap()```ing their ```PT_LOAD``` segments into the executable's virtual address space using ELF loading rules. It will do this by recursively processing the ```PT_DYNAMIC``` segments of the **shared libraries** for every ```DT_NEEDED``` dependency it can find in the current ELF executable's ```PT_DYNAMIC``` segment.

#### <span style="color:lightcoral">Step 5: Dynamic Linker Performs Initial Relocations</span>

After **all** the dependencies have been loaded, it begins to perform some initial relocations. For this overview, we are going to assume default behavior and assume that **lazy binding is enabled**, thus only data relocations are performed at this step. What is lazy binding? That'll be explained in detail later, but recall from Part 1 that it is a way to defer symbol resolution until runtime using a part of the GOT and special stubs in a segment known as the PLT.

> Note: If eager binding is enabled, with ```LD_BIND_NOW```, all symbols including PLT function calls will be resolved at program startup
{: .prompt-info}

The dynamic linker will perform only **non-PLT** relocations if the default behavior of lazy binding is enabled, which means it will access:

- ```DT_REL```/```DT_RELA```: These point to the main relocation tables for data symbols (```.rel.dyn``` and ```.rela.dyn```) such as those with type ```R_X86_64_GLOB_DAT```
- ```DT_SYMTAB```: It uses the ```r_info``` field of the relocation entry to get a symbol index into the dynamic symbol table (which it can find through ```DT_DYNAMIC```)
- ```DT_STRTAB```: It uses ```st_name``` in ```.dynsym``` to get the symbol name.

When lazy binding is enabled, it will **not** resolve PLT relocations such as those in ```DT_JMPREL```, which is a tag that points to the ```.rel.plt``` or ```.rela.plt``` sections

> A given ELF binary or object will use either REL or RELA relocations, but not both at the same time. This is because different architectures or ABIs standardize on one format or the other
{: .prompt-tip}

After it knows the symbol name, it can perform symbol resolution by performing a lookup for the symbol name across **all loaded shared objects: the main executable and its dependencies**. It does this by recursively looking up the symbol name in each object's ```.dynsym``` table

> This is why it is crucial that this step happens only after all the ```DT_NEEDED``` tags and their associated shared libraries have been mapped into memory. If we attempted to perform relocations after one dependency is loaded, it might fail trying to find symbols in libraries that haven't yet been loaded in
{: .prompt-info}

For the initial data relocations, the symbol name is only used for the this lookup step. After the relocation is applied, the symbol name is no longer needed.

#### <span style="color:lightcoral">Step 6: Dynamic Linker Prepares PLT/GOT for Lazy Binding</span>

Although due to lazy binding, dynamic function symbols will not be resolved until runtime, the dynamic linker still has to set some things up in order for lazy binding to work. Keep in mind, I still haven't explained how lazy binding works in detail, so some of this section may not entirely make sense for now. I'll try to keep it as high level as possible, but it would help to come back and read this overview after I introduce the details of how lazy binding is implemented for a thorough understanding of this step. 

The dynamic linker now sets up a few things in the PLT and the GOT. The first few entries in the GOT point to the **resolver trampoline** inside the dynamic linker, which are used by the default PLT stub, ```PLT0```, to call into the dynamic linker when a dynamic function symbol is first invoked. So,

1. The dynamic linker finds ```DT_PLTGOT``` from the ```.dynamic``` section to get the address of the ```.got.plt```, which store jump slots that the PLT stubs will call into
2. The runtime resolver function ```_dl_runtime_resolve()``` is placed at one of the reserved ```.got.plt``` entries.
3. For each ```.got.plt``` entry that's not the the reserved ones (i.e., the entry associated with a PLT stub), it will set it to point to **the next instruction** in its own PLT entry, which is essentially a per-function trampoline back to the PLT, which will call PLT0 and jump to the **resolver trampolile** entry in the GOT we set up in step 2

After the reserved GOT entries, including the resolver trampoline, along with the per-function trampolines back into the PLT stubs have been successfully set up, the dynamic linker is now ready to perform lazy binding using the PLT/GOT at runtime.

#### <span style="color:lightcoral">Step 7: Transfer Control to Program Entry Point</span>

After the PLT/GOT trampolines have been initialized, the dynamic linker does some final initialization calls and environment setup

1. The linker sets up thread-local storage (TLS) if needed
2. Sets up the ```.init_array``` and ```.init``` for the program and shared libraries
3. Calls the library initializers we set up, and any defined ```.init_array``` constructors
4. Any other relevant setup

Finally, the control is transferred to the user program's entry point, usually ```_start```. Keep in mind that at this point, no PLT relocations have been applied yet due to lazy binding being enabled

#### <span style="color:lightcoral">Step 8: Execute the Program</span>
Finally, the control eventually enters the user's ```main()``` function to begin code execution. At this point, we can start the process of lazy binding. When the user code makes a call to a shared library function, say ```printf()```, it will now jump to the PLT stub for that function, ```printf@plt```, and eventually enter the dynamic linker's resolver, which will patch the GOT entry with the location of ```printf```, after it performs the symbol resolution steps discussed previously in Step 5.

> If a lot of the PLT/GOT jargon is going over your head at this point, come back to this section after I have described the process of lazy binding
{: .prompt-info}

## <span style="color:red">Lazy Binding</span>
Lazy binding is the default behavior of the dynamic linker used by Unix-like operating systems. As mentioned in this part and Part 1, lazy binding is deferring dynamic function resolution to runtime when the function is first called, as opposed to early binding, which means that all symbols including functions are resolved immediately before control is transfered to the user program. 

There are several reasons that this is the default behavior, but the primary one is performance. By deferring function resolution until the point at which they are actually called, it significantly reduces program startup time especially for large programs with many dynamic libraries, and it prevents importing symbols that are unnecessary and unused by the main executable, and only import the ones that are used. 

There are some cases where early binding may be preferred, such as performance critical programs such as servers or real-time applications, the additional runtime overhead may not be desired. 

Furthermore, security critical programs may choose to disable lazy binding to prevent attacks on the ```.got.plt```. These programs can be compiled with full RELRO (Relocation Read-Only), which forces early binding of all dynamic symbols at startup so that the entire GOT (including the ```.got``` and ```.got.plt``` and related tables) are read-only, which prevents GOT hijacking by a malicious actor.

> There is also partial RELRO, which doesn't require early binding. It marks the non-PLT section of the GOT (the ```.got```) as read-only, but leaves the ```.got.plt``` writable for lazy binding to continue to occur
{: .prompt-tip}

So how is lazy binding implemented? And what is the function of the PLT and GOT? Lets answer those questions now

### <span style="color:lightcoral">Procedure Linkage Table</span>
The Procedure Linkage Table (PLT) is a special executable segment of ELF binaries that store stubs to external library functions that need to be resolved. Every time a program makes a call to an external library function, such as ```printf``` it is impossible for it to call that function directly, because it doesn't know the runtime address or location of ```printf``` at compile time. Instead, the compiler will generate a PLT entry or stub, such as ```printf@plt``` which will point to an entry in a table known as the Global Offset Table (GOT), which will eventually store the **real** address of printf after it is dynamically resolved. 

### <span style="color:lightcoral">Global Offset Table</span>
The Global Offset Table (GOT) is a **data** segment. Unlike the PLT, it is nonexecutable. The GOT is split up into two main components, which at link time were denoted by the sections ```.got``` and ```.got.plt```. For lazy binding purposes, we are only concerned about the ```.got.plt```, as this section holds "jump slots" or locations that will be filled in by the dynamic linker resolver with pointers to resolved external function addresses. The other ```.got``` section doesn't hold pointers used by the PLT for lazy binding, it holds global data symbols that need to be resolved. These entries are resolved at startup by the dynamic linker and don't go through the PLT at all.

### <span style="color:lightcoral">Initial Lazy Symbol Resolution</span>
Here is an example of what PLT stubs look like using a disassembler on one of the binaries from the Practical Binary Analysis CTF exercises

```
binary@binary-VirtualBox:~/Desktop/PBA_Official/code/chapter5$ objdump -d --section .plt lvl2

lvl2:     file format elf64-x86-64


Disassembly of section .plt:

0000000000400490 <puts@plt-0x10>:
  400490:	ff 35 72 0b 20 00    	pushq  0x200b72(%rip)        # 601008 <rand@plt+0x200b28>
  400496:	ff 25 74 0b 20 00    	jmpq   *0x200b74(%rip)        # 601010 <rand@plt+0x200b30>
  40049c:	0f 1f 40 00          	nopl   0x0(%rax)

00000000004004a0 <puts@plt>:
  4004a0:	ff 25 72 0b 20 00    	jmpq   *0x200b72(%rip)        # 601018 <rand@plt+0x200b38>
  4004a6:	68 00 00 00 00       	pushq  $0x0
  4004ab:	e9 e0 ff ff ff       	jmpq   400490 <puts@plt-0x10>

00000000004004b0 <__libc_start_main@plt>:
  4004b0:	ff 25 6a 0b 20 00    	jmpq   *0x200b6a(%rip)        # 601020 <rand@plt+0x200b40>
  4004b6:	68 01 00 00 00       	pushq  $0x1
  4004bb:	e9 d0 ff ff ff       	jmpq   400490 <puts@plt-0x10>

00000000004004c0 <srand@plt>:
  4004c0:	ff 25 62 0b 20 00    	jmpq   *0x200b62(%rip)        # 601028 <rand@plt+0x200b48>
  4004c6:	68 02 00 00 00       	pushq  $0x2
  4004cb:	e9 c0 ff ff ff       	jmpq   400490 <puts@plt-0x10>

00000000004004d0 <time@plt>:
  4004d0:	ff 25 5a 0b 20 00    	jmpq   *0x200b5a(%rip)        # 601030 <rand@plt+0x200b50>
  4004d6:	68 03 00 00 00       	pushq  $0x3
  4004db:	e9 b0 ff ff ff       	jmpq   400490 <puts@plt-0x10>

00000000004004e0 <rand@plt>:
  4004e0:	ff 25 52 0b 20 00    	jmpq   *0x200b52(%rip)        # 601038 <rand@plt+0x200b58>
  4004e6:	68 04 00 00 00       	pushq  $0x4
  4004eb:	e9 a0 ff ff ff       	jmpq   400490 <puts@plt-0x10>

```

You can see the various stubs for the external function calls this program makes, which seem to follow a similar format. Lets discuss what this format is. Let's take ```rand@plt```, for instance:

```
00000000004004e0 <rand@plt>:
  4004e0:	ff 25 52 0b 20 00    	jmpq   *0x200b52(%rip)        # 601038 <rand@plt+0x200b58>
  4004e6:	68 04 00 00 00       	pushq  $0x4
  4004eb:	e9 a0 ff ff ff       	jmpq   400490 <puts@plt-0x10>
```

The first ```jmpq``` instruction jumps into the GOT entry (in the ```.got.plt```) for the jump slot associated with this PLT stub. It will do an indirect jump to the associated GOT entry which will resolve to some jump target somewhere Remember, at this point, the dynamic linker has still not resolved the symbol of ```rand()``` into the main executable's ```.got.plt```. If that's the case, what exactly is at the address ```0x601038``` that the program jumps to? Lets find out

```
binary@binary-VirtualBox:~/Desktop/PBA_Official/code/chapter5$ objdump -s --section .got.plt lvl2

lvl2:     file format elf64-x86-64

Contents of section .got.plt:
 601000 280e6000 00000000 00000000 00000000  (.`.............
 601010 00000000 00000000 a6044000 00000000  ..........@.....
 601020 b6044000 00000000 c6044000 00000000  ..@.......@.....
 601030 d6044000 00000000 e6044000 00000000  ..@.......@.....
```

Accounting for little-endian addresses, it looks like the address there is ```0x4004e6```, which just so happens to be the very next instruction in the same PLT stub! So indirect jump essentially just jumps to the next instruction. Why is that? The answer is something I alluded to earlier: the initial GOT entry stores a trampoline back to the PLT, which will call a default stub that patches the GOT entry.

Lets walk through this and see how this happens. We can see that the next instruction is ```push $0x4```, which is an identifier associated with the dynamic symbol ```rand```. The stub then jumps to the location ```puts@plt-0x10```, which is the **default stub** or ```PLT0```. This is the resolver stub that indexes into a special reserved GOT entry that holds the address of the function ```_dl_runtime_resolve()```, the dynamic linker's resolver. If you recall from the previous section, the dynamic linker sets up these entries at startup before we jump to our program code. 

```
0000000000400490 <puts@plt-0x10>:
  400490:	ff 35 72 0b 20 00    	pushq  0x200b72(%rip)        # 601008 <rand@plt+0x200b28>
  400496:	ff 25 74 0b 20 00    	jmpq   *0x200b74(%rip)        # 601010 <rand@plt+0x200b30>
  40049c:	0f 1f 40 00          	nopl   0x0(%rax)
```

From the default stub, we can see that at address ```0x400490```, the stub pushes another reserved GOT entry which identifies the executable itself, then at ```0x400496``` it jumps to the GOT entry that holds the dynamic linker resolver. From that point, we have now pivoted back into the dynamic linker, this time at runtime rather than at load-time. Here, the dynamic linker will:

1. Use the identifiers pushed by both PLT stubs to figure out that it needs to resolve ```rand``` for the current executable.
2. It will perform standard symbol resolution steps to find the location of ```rand``` in memory.
3. It will patch the GOT jump slot with the correct runtime address of ```rand```
4. It will then call ```rand```

Now that the GOT entry has been patched, it no longer resolves to a trampoline back into the PLT stub that called it. The next time the stub is called, it will now directly resolve to the runtime address of ```rand```. 

> Note: The identifiers pushed by the PLT stubs are important, as there can be multiple libraries loaded into the process, each with their own PLT and GOT, so we need a way to tell the dynamic linker to resolve the GOT entry for the main executable
{: .prompt-info} 

This way, only the functions that are actually called will be resolved by the dynamic linker at runtime, avoiding the massive overhead incurred by having to load every single function in a shared library when you only end up using a few of them.