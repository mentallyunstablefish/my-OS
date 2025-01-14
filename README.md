MyOS is a simple operating system that demonstrates how a bootloader and kernel work together to boot into protected mode. Here's how I built and ran MyOS if you want to test it out yourself:

Before getting started, make sure you have NASM, QEMU, and a cross-compiler and cross-linker configured for the x86 architecture (https://github.com/mell-o-tron/MellOs/blob/main/A_Setup/setup-gcc-debian.sh). Also, make sure the cross-compiler is accessible by exporting it path:

`export PATH=$PATH:/usr/local/i386elfgcc/bin`

First, I assembled the bootloader (boot.asm) using NASM. This generated a flat binary file called boot.bin:

`nasm "boot.asm" -f bin -o "Binaries/boot.bin"`

Next, I assembled the kernel entry point (kernel_entry.asm) into an ELF object file. This basically serves as the bridge between the bootloader and the main kernel code:

`nasm "kernel_entry.asm" -f elf -o "Binaries/kernel_entry.o"`

For the kernel itself, I used the cross-compiler to compile the C++ file (kernel.cpp) into an ELF object file:

`i386-elf-gcc -ffreestanding -m32 -g -c "kernel.cpp" -o "Binaries/kernel.o"`

To ensure the final OS image had the correct size for QEMU, I also created a padding file (zeroes.bin) filled with zeros:

`nasm "zeroes.asm" -f bin -o "Binaries/zeroes.bin"`

I then linked the kernel entry point and kernel together using the cross-linker. This step combined the files into a single flat binary (full_kernel.bin) and ensured it would load correctly at memory address 0x1000:

`i386-elf-ld -o "Binaries/full_kernel.bin" -Ttext 0x1000 "Binaries/kernel_entry.o" "Binaries/kernel.o" --oformat binary`

Finally, I concatenated the bootloader, kernel binary, and padding into a single bootable OS image (OS.bin):

`cat "Binaries/boot.bin" "Binaries/full_kernel.bin" "Binaries/zeroes.bin" > "Binaries/OS.bin"`

This command loads the OS binary as a raw floppy disk image and emulates the system with 128 MB of RAM:

`qemu-system-x86_64 -drive format=raw,file="Binaries/OS.bin",index=0,if=floppy -m 128M`
