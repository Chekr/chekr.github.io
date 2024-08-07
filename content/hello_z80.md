# SG-1000 Tutorial


The two three things you're going to want to have on hand are:

1. the [Z80 Manual](https://www.zilog.com/docs/z80/um0080.pdf">Z80 Manual)[mirror](z80_manual.pdf)
2. the [VASM Manual](http://sun.hasenbraten.de/vasm/release/vasm.pdf)[online](http://sun.hasenbraten.de/vasm/release/vasm.html)[mirror](vasm_manual.pdf)

There are numerous helpful cheat-sheets and guides out there, but in the end the manuals don't leave anything out.

Second, you are going to want an emulator OR some sort of programmable cartridge WITH a Sega SG-1000, SC-3000, Master System/Mk3, or Genesis/Mega Drive (or clone system).

Emulators have the benefit of often being free, and commonly offering debug options, but are fairly likely to have bugs/implementation issues compared to real hardware. Clone consoles are prone to the same problems. FPGA-based clone systems are less prone to such issues, but are still often imperfect. 

To start off, I recommend the [MEKA](https://www.smspower.org/meka/) emulator [windows binary](https://www.smspower.org/meka/download.shtml)[build source for linux](https://github.com/ocornut/meka) which offers robust debugging options.

If VASM is not already set up, please reference the [VASM Setup Process](vasm.html)

Commands largely fall into two categories: 
1. instructions - represent native processor commands, native to the processor type
2. directives - provide directions to the assembler, can vary between assemblers

I will provide a list of new commands with their type and general usage as they are added to each program. Referencing the manuals is still recommended. 

Let's get started with our first Z80 program. In the file "main.asm" in the "src" directory, enter the following text:



[main.asm]
```
    org $0000
; first 128 bytes are RSTs and interrupts, fill with RET's ($C9)
    jr ProgramStart ;$0000 - RST0
    ;note: "ds" is blk/block fill of memory
    ds 6,$C9    ;$0002 - RST0, Reset
    ds 8,$C9    ;$0008 - RST1
    ds 8,$C9    ;$0010 - RST2
    ds 8,$C9    ;$0018 - RST3
    ds 8,$C9    ;$0020 - RST4
    ds 8,$C9    ;$0028 - RST5
    ds 8,$C9    ;$0030 - RST6
    ds 8,$C9    ;$0038 - RST7, IM1 (interrupt mode 1, usualy VBLANK)
                ;        rom mapped on SMS
    ;jp InterruptHandler
    ds 38,$C9   ;$0066 - NMI, hardware interrupt, Pause button
    ds 26,$C9   ;$0080



; start address, $0080
ProgramStart:
    di              ; disable interrupts
    im 1            ; interrupt mode 1
    ld sp, $dff0     ; default stack pointer


    ;otir    ; Out (c),(h1).. inc HL... decB, djnz


MainLoop:
; initialize cursor position

    jp MainLoop
    di ;disable interrupts
    halt
```

As a reminder, for purposes of this tutorial your file structure should look somehting like:
<p>
<pre>
&lt;project&gt;
-&lt;out&gt;
-&lt;src&gt;
--main.asm
-build.bat
-build.sh
</p>
</pre>
Where the "build.bat" and "build.sh" files should contain scripts for executing the assembly process for windows and linux respectively

build.bat (windows):
```
c:\vasm\vasmz80_oldstyle_Win64\vasmz80_oldstyle.exe -chklabels -nocase -Dvasm=1 -Fbin -o .\out\rom.bin .\src\main.asm
```
build.sh (linux):
```
#!/bin/bash
vasmz80_oldstyle -chklabels -nocase -Dvasm=1 -Fbin -o ./out/rom.bin ./src/main.asm
```
You can also view this example in the git repo <a href="">here</a>

Try executing the scripts from the command line to build the "main.asm" into a proper binary files, stored in "./out/rom.bin"

Windows:
```
&gt; ./build.bat
```
Linux:
```
&gt; ./build.sh
```
If the file builds correctly, you should see output something like:
```
vasm 1.8e (c) in 2002-2018 Volker Barthelmann                                 
vasm 8080/gbz80/z80/z180/rcmX000 cpu backend 0.3 (c) 2007,2009 Dominic Morris 
vasm oldstyle syntax module 0.13f (c) 2002-2018 Frank Wille                   
vasm binary output module 1.8a (c) 2002-2009,2013,2015,2017 Volker Barthelmann
                                                                              
seg0(acrwx1):            139 bytes                                            
seg7ff0(acrwx1):                  16 bytes                                    
```
Before executing the binary file, let's discuss a bit about the commands we used in this example. 


### DIRECTIVES

`ORG [#]` - "ORG" followed by a number sets the address the following code will be stored at. In this case, at address zero
`DS <exp>[,<fill>]` - will fill <exp> bytes of the <fill> value (or zero) into the code

### INSTRUCTIONS
`RET` ($C9) - returns to the original caller
`DI` ($F3) - disables maskable interrupt
`IM 1` ($ED $56) - sets processor to Interrupt Mode 1, processor responds to an interrupt by executing a restart at address `$0038`
`LD SP, nn` ($31)- load "nn" into the stack pointer (SP)
`JP nn</code> ($C3) - jump to location
`JR e` ($18) - jumps to the program counter offset defined by "e". It differs from 'JP' in that 'JR'''s destination must be withing 128 bytes
`HALT` ($76) - suspends CPU operation until interrupt or reset

!!! Please note that hex values can be identified by a preceeding "&" or "$" (ex: `$80`). Binary values are preceeded by a "%" (ex: `%10000000`). 

The first line we have is:
```
org $0000
```
Which is an assembler directing, telling the assembler "the following code will start at this location", `$0000` being the very first line, counting up from "0".

The next command:
```
    jr ProgramStart
```
... represents `JR`, a jump command. Functionally, JR will jump to a program location within 128 bytes, in our case the start of our program. We are intentionally skipping executing the next few lines of code execution, which are going to be reserved. 

The next few locations represent restart commands on the Z80 processer, which can be called with the `RST` command. The beginning memory locaitons represent the callable locations of this command. 

The availible addresses are:
`$0000, $0008, $0010, $0018, $0020, $0028, $0030, $0038`

The `DS` command will fill a certain number of blocks with a value. In this case, I am using the `$C9` hex value, which corresponds to the `RET` command. `RET` will return to the previously called location. This serves as a good placeholder value for the reset locations until callbacks are added as necessary. 

So...
```
    ds 6,$C9
```
... will insert six instances of `$C9` into the code, representing `RET` each time. And accidental `RST` calls to these locations will simply return back to the original caller. 

We will also map out `RET` blocks through `$0066` and `$0080` for later use.

Our main program code wills start at the:
```
ProgramStart:
```

...label, at location `$0080`

Getting started, we first want to disable all interrupts, then set the interrupt mode to "1". Interrupt Mode 1 means "processor responds to an interrupt by executing a restart at address `$0038`.

```
    di              ; disable interrupts    
    im 1            ; interrupt mode 1      
```

Next we load `LD` a default stack pointer into the stack pointer register(`SP`). The default value will be `dff0`

```
    ld sp, $dff0     ; default stack pointer
```



