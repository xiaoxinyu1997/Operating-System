For this class you'll need the RISC-V versions of a couple different tools: QEMU 5.1, GDB 8.3, GCC, and Binutils.

### Installing on macOS

First, install developer tools:
```
$ xcode-select --install
```
Next, install [Homebrew](https://brew.sh/), a package manager for macOS:
```
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
Next, install the [RISC-V compiler toolchain](https://github.com/riscv/homebrew-riscv):
```
$ brew tap riscv/riscv
$ brew install riscv-tools
```
The brew formula may not link into <tt style="box-sizing: border-box; color: rgb(51, 51, 51); font-size: 14px; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; white-space: normal; background-color: rgb(255, 255, 255); text-decoration-thickness: initial; text-decoration-style: initial; text-decoration-color: initial;">/usr/local</tt>. You will need to update your shell's rc file (e.g. [~/.bashrc](https://www.gnu.org/software/bash/manual/html_node/Bash-Startup-Files.html)) to add the appropriate directory to [$PATH](http://www.linfo.org/path_env_var.html).
```
PATH=$PATH:/usr/local/opt/riscv-gnu-toolchain/bin
```
Finally, install QEMU:
```
$ brew install qemu
```
### Installing via APT (Debian/Ubuntu)
Make sure you are running either "bullseye" or "sid" for your debian version (on ubuntu this can be checked by running cat /etc/debian_version), then run:
```
$ sudo apt-get install git build-essential gdb-multiarch qemu-system-misc gcc-riscv64-linux-gnu binutils-riscv64-linux-gnu 
```
(The version of QEMU on "buster" is too old, so you'd have to get that separately.)
#### qemu-system-misc fix
At this moment in time, it seems that the package qemu-system-misc has received an update that breaks its compatibility with our kernel. If you run make qemu and the script appears to hang after
qemu-system-riscv64 -machine virt -bios none -kernel kernel/kernel -m 128M -smp 3 -nographic -drive file=fs.img,if=none,format=raw,id=x0 -device virtio-blk-device,drive=x0,bus=virtio-mmio-bus.0
you'll need to uninstall that package and install an older version:
```
  $ sudo apt-get remove qemu-system-misc
  $ sudo apt-get install qemu-system-misc=1:4.2-3ubuntu6
```

### Installing on Arch
```
sudo pacman -S riscv64-linux-gnu-binutils riscv64-linux-gnu-gcc riscv64-linux-gnu-gdb qemu-arch-extra
```

### Other Linux distributions (i.e. compiling your own toolchain)
We assume that you are installing the toolchain into /usr/local on a modern Ubuntu installation. You will need a fair amount of disk space to compile the tools (around 9GiB). If you don't have that much space, consider using an MIT Athena machine.

First, clone the repository for the RISC-V GNU Compiler Toolchain:
```
$ git clone  https://gitee.com/mirrors/riscv-gnu-toolchain
$ cd riscv-gnu-toolchain
# remove the empty directories
$ rm -rf riscv-*
$ git clone -b upstream git@gitee.com:mirrors/riscv-newlib.git
$ git clone -b riscv-glibc-2.29 git@gitee.com:mirrors/riscv-glibc.git
$ git clone -b riscv-gcc-9.2.0-rvv git@gitee.com:mirrors/riscv-gcc.git
$ git clone git@gitee.com:mirrors/riscv-dejagnu.git
$ git clone -b rvv-0.8.x git@gitee.com:mirrors/riscv-binutils-gdb.git riscv-binutils
$ git clone -b fsf-gdb-8.3-with-sim git@gitee.com:mirrors/riscv-binutils-gdb.git riscv-gdb


```
Next, make sure you have the packages needed to compile the toolchain:
```
$ sudo apt-get install autoconf automake autotools-dev curl libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev
```
Configure and build the toolchain:
```
$ cd riscv-gnu-toolchain
$ ./configure --prefix=/usr/local
$ sudo make
$ cd ..
```
Next, retrieve and extract the source for QEMU 5.1.0:
```
$ wget https://download.qemu.org/qemu-5.1.0.tar.xz
$ tar xf qemu-5.1.0.tar.xz
```
Build QEMU for riscv64-softmmu:
```
$ cd qemu-5.1.0
$ ./configure --disable-kvm --disable-werror --prefix=/usr/local --target-list="riscv64-softmmu"
$ make
$ sudo make install
$ cd ..
```
### Windows (i.e. running a Linux VM)
You need get everything via the Windows Subsystem for Linux or otherwise compiling the tools yourself.

However, an easier option would probably be to run a virtual machine with one of the other operating systems listed above. With platform virtualization, Linux can cohabitate with your normal computing environment. Installing a Linux virtual machine is a two step process. First, you download the virtualization platform.

VirtualBox (free for Mac, Linux, Windows) — [Download page](https://www.virtualbox.org/wiki/Downloads)

Once the virtualization platform is installed, download a boot disk image for the Linux distribution of your choice.

[Ubuntu Desktop](http://www.ubuntu.com/download/desktop) is one option.
This will download a file named something like ubuntu-18.04.3-desktop-amd64.iso. Start up your virtualization platform and create a new (64-bit) virtual machine. Use the downloaded Ubuntu image as a boot disk; the procedure differs among VMs but is pretty simple.

### Testing your Installation

To test your installation, you should be able to check the following:
```
$ riscv64-unknown-elf-gcc --version
riscv64-unknown-elf-gcc (GCC) 10.1.0
...

$ qemu-system-riscv64 --version
QEMU emulator version 5.1.0
```
You should also be able to compile and run xv6:
```
# in the xv6 directory
$ make qemu
# ... lots of output ...
init: starting sh
$
To quit qemu type: Ctrl-a x.
```


## Boot xv6 (easy)
Fetch the xv6 source for the lab and check out the util branch:
```
$ git clone git://g.csail.mit.edu/xv6-labs-2020
Cloning into 'xv6-labs-2020'...
...
$ cd xv6-labs-2020
$ git checkout util
Branch 'util' set up to track remote branch 'util' from 'origin'.
Switched to a new branch 'util'
```
Build and run xv6:
```
$ make qemu
riscv64-unknown-elf-gcc    -c -o kernel/entry.o kernel/entry.S
riscv64-unknown-elf-gcc -Wall -Werror -O -fno-omit-frame-pointer -ggdb -DSOL_UTIL -MD -mcmodel=medany -ffreestanding -fno-common -nostdlib -mno-relax -I. -fno-stack-protector -fno-pie -no-pie   -c -o kernel/start.o kernel/start.c
...  
riscv64-unknown-elf-ld -z max-page-size=4096 -N -e main -Ttext 0 -o user/_zombie user/zombie.o user/ulib.o user/usys.o user/printf.o user/umalloc.o
riscv64-unknown-elf-objdump -S user/_zombie > user/zombie.asm
riscv64-unknown-elf-objdump -t user/_zombie | sed '1,/SYMBOL TABLE/d; s/ .* / /; /^$/d' > user/zombie.sym
mkfs/mkfs fs.img README  user/xargstest.sh user/_cat user/_echo user/_forktest user/_grep user/_init user/_kill user/_ln user/_ls user/_mkdir user/_rm user/_sh user/_stressfs user/_usertests user/_grind user/_wc user/_zombie 
nmeta 46 (boot, super, log blocks 30 inode blocks 13, bitmap blocks 1) blocks 954 total 1000
balloc: first 591 blocks have been allocated
balloc: write bitmap block at sector 45
qemu-system-riscv64 -machine virt -bios none -kernel kernel/kernel -m 128M -smp 3 -nographic -drive file=fs.img,if=none,format=raw,id=x0 -device virtio-blk-device,drive=x0,bus=virtio-mmio-bus.0

xv6 kernel is booting

hart 2 starting
hart 1 starting
init: starting sh
$ 
```
If you type ls at the prompt, you should see output similar to the following:
```
$ ls
.              1 1 1024
..             1 1 1024
README         2 2 2059
xargstest.sh   2 3 93
cat            2 4 24256
echo           2 5 23080
forktest       2 6 13272
grep           2 7 27560
init           2 8 23816
kill           2 9 23024
ln             2 10 22880
ls             2 11 26448
mkdir          2 12 23176
rm             2 13 23160
sh             2 14 41976
stressfs       2 15 24016
usertests      2 16 148456
grind          2 17 38144
wc             2 18 25344
zombie         2 19 22408
console        3 20 0
```
These are the files that mkfs includes in the initial file system; most are programs you can run. You just ran one of them: ls.
xv6 has no ps command, but, if you type Ctrl-p, the kernel will print information about each process. If you try it now, you'll see two lines: one for init, and one for sh.

To quit qemu type: Ctrl-a x.

## sleep (easy)
Implement the UNIX program sleep for xv6; your sleep should pause for a user-specified number of ticks. A tick is a notion of time defined by the xv6 kernel, namely the time between two interrupts from the timer chip. Your solution should be in the file user/sleep.c.

Some hints:

- Before you start coding, read Chapter 1 of the xv6 book.
- Look at some of the other programs in user/ (e.g., user/echo.c, user/grep.c, and user/rm.c) to see how you can obtain the command-line arguments passed to a program.
- If the user forgets to pass an argument, sleep should print an error message.
- The command-line argument is passed as a string; you can convert it to an integer using atoi (see user/ulib.c).
- Use the system call sleep.
- See kernel/sysproc.c for the xv6 kernel code that implements the sleep system call (look for sys_sleep), user/user.h for the C definition of sleep callable from a user program, and user/usys.S for the assembler code that jumps from user code into the kernel for sleep.
- Make sure main calls exit() in order to exit your program.
- Add your sleep program to UPROGS in Makefile; once you've done that, make qemu will compile your program and you'll be able to run it from the xv6 shell.
- Look at Kernighan and Ritchie's book The C programming language (second edition) (K&R) to learn about C.

Run the program from the xv6 shell:
```
 $ make qemu
...
init: starting sh
$ sleep 10
(nothing happens for a little while)
$
```
Your solution is correct if your program pauses when run as shown above. Run make grade to see if you indeed pass the sleep tests.

Note that make grade runs all tests, including the ones for the assignments below. If you want to run the grade tests for one assignment, type:
```
$ ./grade-lab-util sleep
```
This will run the grade tests that match "sleep". Or, you can type:
```
$ make GRADEFLAGS=sleep grade
```
which does the same.
