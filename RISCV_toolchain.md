# RISCV environment for RV32I assembly testing
# 1.05.2022
> In all repositories `git pull was made`. Commit dates were dumped below.
## ENV
```bash
mkdir /home/alexander/riscv
mkdir /home/alexander/riscv/riscv_multilib
export RISCV=/home/alexander/riscv/riscv_multilib
export PATH=$RISCV:$PATH
```

## [riscv-gnu-toolchain](https://github.com/riscv-collab/riscv-gnu-toolchain)
**firstly, download all additional listed on the github**
```bash
# in riscv-gnu-toolchain
./configure --prefix=$RISCV --enable-multilib
make
```
commit hash:
```
commit 9249802bfaf023fa14441754b0ffb488c7c1977d
Merge: ad52c0d a939758
Author: Kito Cheng <kito.cheng@sifive.com>
Date:   Fri Apr 22 11:20:14 2022 +0800

    Merge pull request #1062 from palmer-dabbelt/libsanitizer
    
    Add an "--enable-libsanitizer" configure-time argument
```

check targets
```bash
/home/alexander/riscv/riscv_multilib/bin/riscv64-unknown-elf-gcc --print-multi-lib
```
```bash
# output
rv32i/ilp32;@march=rv32i@mabi=ilp32
rv32im/ilp32;@march=rv32im@mabi=ilp32
rv32iac/ilp32;@march=rv32iac@mabi=ilp32
rv32imac/ilp32;@march=rv32imac@mabi=ilp32
rv32imafc/ilp32f;@march=rv32imafc@mabi=ilp32f
rv64imac/lp64;@march=rv64imac@mabi=lp64
```

## [spike](https://github.com/riscv-software-src/riscv-isa-sim/) for RV32I
```bash
# in riscv-isa-sim
mkdir build && cd build
../configure --prefix=$RISCV --with-isa=RV32I
make
make install
```

commit hash
```
commit 1df65613df9970dc7f5c2f3d1bf343dbb0497828
Author: Andrew Waterman <andrew@sifive.com>
Date:   Sat Apr 30 16:59:07 2022 -0700

    Add missing description of --dtb in --help message

commit 1cfffeda1e8323729e584c904b2ce78681ba0283
Author: Yan <phantom@zju.edu.cn>
Date:   Sat Apr 23 13:46:07 2022 +0800
```

After that `RV32I` will be default `march` for spike

## [pk](https://github.com/riscv-software-src/riscv-pk) (includes boot loader bbl)
```bash
# in riscv-pk
mkdir build && cd build
../configure --prefix=$RISCV --host=riscv64-unknown-elf --with-arch=rv32i
make
make install
```

commit hash
```
commit 2efabd3e6604b8a9e8f70baf52f57696680c7855
Author: MaxXing <x@MaxXSoft.net>
Date:   Sun May 1 08:17:33 2022 +0800

    Fix a file leak in function `at_kfd` (#276)

commit c7e75bf944957e58f16531eb6b8d118a92069346
Author: Lucheng Zhang <geekLucian@outlook.com>
Date:   Wed Apr 20 16:37:44 2022 +0800
```

# install tree
```bash
alexander@alexander-Inspiron-3180:~/riscv$ ls riscv_multilib/risc*/bin riscv_multilib/bin
riscv_multilib/bin:
elf2hex                        riscv64-unknown-elf-g++         riscv64-unknown-elf-gcov-tool      riscv64-unknown-elf-objcopy  spike
riscv64-unknown-elf-addr2line  riscv64-unknown-elf-gcc         riscv64-unknown-elf-gdb            riscv64-unknown-elf-objdump  spike-dasm
riscv64-unknown-elf-ar         riscv64-unknown-elf-gcc-11.1.0  riscv64-unknown-elf-gdb-add-index  riscv64-unknown-elf-ranlib   spike-log-parser
riscv64-unknown-elf-as         riscv64-unknown-elf-gcc-ar      riscv64-unknown-elf-gprof          riscv64-unknown-elf-readelf  termios-xspike
riscv64-unknown-elf-c++        riscv64-unknown-elf-gcc-nm      riscv64-unknown-elf-ld             riscv64-unknown-elf-run      xspike
riscv64-unknown-elf-c++filt    riscv64-unknown-elf-gcc-ranlib  riscv64-unknown-elf-ld.bfd         riscv64-unknown-elf-size
riscv64-unknown-elf-cpp        riscv64-unknown-elf-gcov        riscv64-unknown-elf-lto-dump       riscv64-unknown-elf-strings
riscv64-unknown-elf-elfedit    riscv64-unknown-elf-gcov-dump   riscv64-unknown-elf-nm             riscv64-unknown-elf-strip

riscv_multilib/riscv32-unknown-elf/bin:
bbl  dummy_payload  pk

riscv_multilib/riscv64-unknown-elf/bin:
ar  as  ld  ld.bfd  nm  objcopy  objdump  ranlib  readelf  strip
alexander@alexander-Inspiron-3180:~/riscv$ 
```

## Objdump
> see `test1.s` below for src
```bash
riscv64-unknown-elf-objdump -D -M no-aliases a.out |less
```
```
a.out:     file format elf32-littleriscv


Disassembly of section .text:

00010074 <_start>:
   10074:       00500293                addi    t0,zero,5
   10078:       00512023                sw      t0,0(sp)
   1007c:       00800293                addi    t0,zero,8
   10080:       00037313                andi    t1,t1,0

00010084 <.loop>:
   10084:       00130313                addi    t1,t1,1
   10088:       fe531ee3                bne     t1,t0,10084 <.loop>
   1008c:       00012303                lw      t1,0(sp)
   10090:       00600533                add     a0,zero,t1
   10094:       05d00893                addi    a7,zero,93
   10098:       00000073                ecall

Disassembly of section .riscv.attributes:

00000000 <.riscv.attributes>:
   0:   1b41                    .2byte  0x1b41
   2:   0000                    .2byte  0x0
   4:   7200                    .2byte  0x7200
   6:   7369                    .2byte  0x7369
   8:   01007663                bgeu    zero,a6,14 <_start-0x10060>
   c:   0011                    .2byte  0x11
   e:   0000                    .2byte  0x0
  10:   1004                    .2byte  0x1004
  12:   7205                    .2byte  0x7205
  14:   3376                    .2byte  0x3376
  16:   6932                    .2byte  0x6932
  18:   7032                    .2byte  0x7032
  1a:   0030                    .2byte  0x30
(END)
```

## run spike with logging
> see `test1.s` below for src

```bash
spike -l --log log.txt /home/alexander/riscv/riscv_multilib/riscv32-unknown-elf/bin/pk a.out
vim log.txt
```

log.txt
```bash
...
core   0: 0x00010074 (0x00500293) li      t0, 5
core   0: 0x00010078 (0x00512023) sw      t0, 0(sp)
core   0: 0x0001007c (0x00800293) li      t0, 8
core   0: 0x00010080 (0x00037313) andi    t1, t1, 0
core   0: 0x00010084 (0x00130313) addi    t1, t1, 1
core   0: 0x00010088 (0xfe531ee3) bne     t1, t0, pc - 4
core   0: 0x00010084 (0x00130313) addi    t1, t1, 1
core   0: 0x00010088 (0xfe531ee3) bne     t1, t0, pc - 4
core   0: 0x00010084 (0x00130313) addi    t1, t1, 1
core   0: 0x00010088 (0xfe531ee3) bne     t1, t0, pc - 4
core   0: 0x00010084 (0x00130313) addi    t1, t1, 1
core   0: 0x00010088 (0xfe531ee3) bne     t1, t0, pc - 4
core   0: 0x00010084 (0x00130313) addi    t1, t1, 1
core   0: 0x00010088 (0xfe531ee3) bne     t1, t0, pc - 4
core   0: 0x00010084 (0x00130313) addi    t1, t1, 1
core   0: 0x00010088 (0xfe531ee3) bne     t1, t0, pc - 4
core   0: 0x00010084 (0x00130313) addi    t1, t1, 1
core   0: 0x00010088 (0xfe531ee3) bne     t1, t0, pc - 4
core   0: 0x00010084 (0x00130313) addi    t1, t1, 1
core   0: 0x00010088 (0xfe531ee3) bne     t1, t0, pc - 4
core   0: 0x0001008c (0x00012303) lw      t1, 0(sp)
core   0: 0x00010090 (0x00600533) add     a0, zero, t1
core   0: 0x00010094 (0x05d00893) li      a7, 93
core   0: 0x00010098 (0x00000073) ecall
core   0: exception trap_user_ecall, epc 0x00010098
...
```

## Testing
### test 0
create this simple `test0.s`
```asm
.option nopic
.attribute arch, "rv32i2p0"
.attribute unaligned_access, 0
.attribute stack_align, 16

.text
.align	2
.globl	_start
.type	_start, @function

######################################################
_start:

	addi	t0, zero, 777
    
    addi    a0, x0, 0   # Return code 0
    addi    a7, x0, 93  # Syscall 93 terminates
	ecall

######################################################

.size	_start, .-_start
```

compile it
```bash
riscv64-unknown-elf-gcc -march=rv32i -mabi=ilp32 -nostdlib -static ../tests/test0.s
```

and run
```bash
spike -l --log=log.txt /home/alexander/riscv/riscv_multilib/riscv32-unknown-elf/bin/pk a.out
```

output
```
bin/pk a.out 
bbl loader
alexander@alexander-Inspiron-3180:~/GitHub/RiscVSimulator/trash$ 
```

## test 1
`test1.s`
```
.option nopic
.attribute arch, "rv32i2p0"
.attribute unaligned_access, 0
.attribute stack_align, 16

.text
.align	2
.globl	_start
.type	_start, @function

######################################################
_start:

	addi	t0, zero, 5
    sw      t0, 0(sp)
   
    addi    t0, zero, 8
    andi t1, t1, 0
.loop: 
    addi t1, t1, 1
    bne t1, t0, .loop

    lw      t1, 0(sp)

    add    a0, zero, t1   # Return code 0
    addi    a7, zero, 93  # Syscall 93 terminates
	ecall

######################################################

.size	_start, .-_start
```
```bash
riscv64-unknown-elf-gcc -march=rv32i -mabi=ilp32 -nostdlib -static ../tests/test1.s
alexander@alexander-Inspiron-3180:~/GitHub/RiscVSimulator/trash$ spike /home/alexander/riscv/riscv_multilib/riscv32-unknown-elf/bin/pk a.out 
bbl loader
alexander@alexander-Inspiron-3180:~/GitHub/RiscVSimulator/trash$ echo $?
5
alexander@alexander-Inspiron-3180:~/GitHub/RiscVSimulator/trash$
```
### test 3
```C++
// hello.c
#include "stdio.h"
int main() {
    printf("hello world\n");
}
```
```bash
alexander@alexander-Inspiron-3180:~/GitHub/RiscVSimulator/trash$ riscv64-unknown-elf-gcc -march=rv32i -mabi=ilp32 hello.c 
alexander@alexander-Inspiron-3180:~/GitHub/RiscVSimulator/trash$ spike /home/alexander/riscv/riscv_multilib/riscv32-unknown-elf/bin/pk a.out 
bbl loader
hello world
alexander@alexander-Inspiron-3180:~/GitHub/RiscVSimulator/trash$ 
```


# DOCUENTATION & RISC-V Assembly
## About ISA extentions (RV32I and others)
unprivileged spec: [here](https://riscv.org/technical/specifications/)
## About ABI terms (ilp32, register aliases as `sp`, `t0`, `a0` and others)
can be downloaded [here](https://github.com/riscv-non-isa/riscv-elf-psabi-doc/releases)
## About system registers, MMU and other
privileged spec: [here](https://riscv.org/technical/specifications/)
## Assembly Examples
look [here](https://shakti.org.in/docs/risc-v-asm-manual.pdf)

