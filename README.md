# Bruh OS (better name needed. obviously.)

Hobby OS project. Probably very bad. I have no idea what I'm doing.

Inspired by [OSDev](https://wiki.osdev.org/Main_Page), [blog_os](https://os.phil-opp.com/),
and [StreamOS](https://github.com/sphaerophoria/stream-os).

## Goals

- 64-bit, x86_64
- Bootable by grub or similar (BIOS, UEFI is a non-goal for now)
  - Multiboot2
- Run on multiple CPU cores (SMP)
- Support for simple graphics such as text and shapes
- Preemptive, multicore scheduler (main goal, I want to learn about this)
- Userland processes
- Basic network stack (maybe a web server if I am lucky)
- TUI environment with builtin multiplexer
  - Vim bindings EVERYWHERE. Maybe I won't even support mouse at all.
  - If I am able to get GUI, then build the GUI around the terminal.
    Like tmux but you can have splits be GUI windows OR builtin terminals.

## About

I've followed the blog_os series before as well as the OSDev barebones tutorial, but I
am attempting to learn by doing with this project. I will be sometimes indirectly
following tutorials or courses I find, but for the most part
my goal here is to really figure out how OS dev works and how to do it myself.

I am writing it in Rust because I like Rust, but also because most of the available resources use
C so I will not have the temptation to copy-paste. Also, there will be crates available for some
tasks which is a nice improvement from c-land. I am already making use of the
[multiboot2](https://github.com/rust-osdev/multiboot2) and [spin](https://github.com/mvdnes/spin-rs) crates for
parsing multiboot info and sync primitives, respectively.

While I will use crates for some things, I will be implementing as much as possible
of the kernel myself. I really just didn't want to deal with multiboot or writing an elf parser.

## Architecture

Multiboot2 compatible.

Everything can be compiled with a standard nightly Rust toolchain (target = x86_64-unknown-none); no special gcc cross-compiler is needed.

`nasm` and a linker are required as well, to build the assembly files and construct the final executable.

The current build setup uses `grub-mkrescue` to construct a bootable image, but this could be adapted to other
bootloaders / systems as long as multiboot2 is supported.

Bootstrap (/boot.asm, /boot64.asm):

- This is where grub puts us initially
- setup stack and do protected mode (32-bit) init stuff
- setup basic paging so we can enter long mode
- setup basic gdt so CPU lets us into long mode
- setup long mode (64-bit)
- jump to kernel entry point in long mode

TODO: move some of this bootstrap code to 32-bit Rust. I tried to do this before
with *some* success, but found linking back and forth difficult so I went with full asm for now.

Kernel: (/kernel):

- Parse multiboot info (using multiboot2 crate)
- Setup VGA writer
- Setup long mode GDT (WIP)
- Setup IDT and interrupt handlers (WIP)
- Setup and enable PICs and PIT

Working (sorta) but not enabled:

- Jump to userland (WIP)
  - Currently cannot get interrupts to re-enable when entering user mode due to general protection fault

TODO:

- Setup long mode 4-level paging
- Memory allocator so I can use the `alloc` crate and the heap
- Test harness
- Serial driver
- USB driver
- Basic filesystem (FAT32?)
- Scheduler + processes
  - SMP boot / AP trampoline code
- etc...

Runner (/runner):

- Builds and runs the kernel
- I should look at how the bootloader / bootimage crates do things
- Need to make framework for running tests / communicating via serial.
