# helithumper re

[Link to binary](https://github.com/guyinatuxedo/nightmare/blob/master/modules/03-beginner_re/helithumper_re/rev)

---

We are given a `rev` executable.

```
~/Downloads > file rev
rev: ELF 64-bit LSB pie executable, [...], not stripped
~/Downloads > checksec --file=rev
RELRO           STACK CANARY      NX            PIE         [...]
Full RELRO      Canary found      NX enabled    PIE enabled [...]
~/Downloads >
```

Running it presents us with a prompt that asks for input. After that, a message is shown and the program exits.

```
~/Downloads > ./rev 
Welcome to the Salty Spitoon™, How tough are ya?
very
Yeah right. Back to Weenie Hut Jr™ with ya
~/Downloads >
```

Disassembling the `main` functions reveals a call to `validate` followed by a conditional jump.

```
0x00000000000011bb <+70>:	call   0x11ea <validate>
0x00000000000011c0 <+75>:	test   eax,eax
0x00000000000011c2 <+77>:	je     0x11d7 <main+98>
```

The return value from `validate` is put in the `eax` register. By doing `test eax, eax` the zero flag is set if and only if the result was 0.

If we break just before `je` is execute and flip the zero flag, we take the opposite branch.

```
gef➤  b *main+77
Breakpoint 1 at 0x11c2
gef➤  r
Starting program: /home/elia/Downloads/rev

gef➤  p $eflags
$3 = [ PF ZF IF ]
gef➤  flags ~zero
[zero carry PARITY [...]]
gef➤  p $eflags
$4 = [ PF IF ]

gef➤  c
Continuing.
Right this way...
[Inferior 1 (process 3468) exited normally]
```

So we used the debugger to make the process happy but there's no flag in sight.

That probably means that we have to figure out which input is the correct one. The flag is encoded in `validate` function.

Let's take a look at the disassembly:

```
0x00005555555551ea <+0>:	push   rbp
0x00005555555551eb <+1>:	mov    rbp,rsp
0x00005555555551ee <+4>:	sub    rsp,0x60
0x00005555555551f2 <+8>:	mov    QWORD PTR [rbp-0x58],rdi
0x00005555555551f6 <+12>:	mov    rax,QWORD PTR fs:0x28
0x00005555555551ff <+21>:	mov    QWORD PTR [rbp-0x8],rax
0x0000555555555203 <+25>:	xor    eax,eax
0x0000555555555205 <+27>:	mov    DWORD PTR [rbp-0x40],0x66
0x000055555555520c <+34>:	mov    DWORD PTR [rbp-0x3c],0x6c
0x0000555555555213 <+41>:	mov    DWORD PTR [rbp-0x38],0x61
0x000055555555521a <+48>:	mov    DWORD PTR [rbp-0x34],0x67
0x0000555555555221 <+55>:	mov    DWORD PTR [rbp-0x30],0x7b
0x0000555555555228 <+62>:	mov    DWORD PTR [rbp-0x2c],0x48
0x000055555555522f <+69>:	mov    DWORD PTR [rbp-0x28],0x75
0x0000555555555236 <+76>:	mov    DWORD PTR [rbp-0x24],0x43
0x000055555555523d <+83>:	mov    DWORD PTR [rbp-0x20],0x66
0x0000555555555244 <+90>:	mov    DWORD PTR [rbp-0x1c],0x5f
0x000055555555524b <+97>:	mov    DWORD PTR [rbp-0x18],0x6c
0x0000555555555252 <+104>:	mov    DWORD PTR [rbp-0x14],0x41
0x0000555555555259 <+111>:	mov    DWORD PTR [rbp-0x10],0x62
0x0000555555555260 <+118>:	mov    DWORD PTR [rbp-0xc], 0x7d
0x0000555555555267 <+125>:	mov    rax,QWORD PTR [rbp-0x58]
0x000055555555526b <+129>:	mov    rdi,rax
0x000055555555526e <+132>:	call   0x555555555040 <strlen@plt>
0x0000555555555273 <+137>:	mov    DWORD PTR [rbp-0x44],eax
0x0000555555555276 <+140>:	mov    DWORD PTR [rbp-0x48],0x0
0x000055555555527d <+147>:	jmp    0x5555555552aa <validate+192>
0x000055555555527f <+149>:	mov    eax,DWORD PTR [rbp-0x48]
0x0000555555555282 <+152>:	movsxd rdx,eax
0x0000555555555285 <+155>:	mov    rax,QWORD PTR [rbp-0x58]
0x0000555555555289 <+159>:	add    rax,rdx
0x000055555555528c <+162>:	movzx  eax,BYTE PTR [rax]
0x000055555555528f <+165>:	movsx  edx,al
0x0000555555555292 <+168>:	mov    eax,DWORD PTR [rbp-0x48]
0x0000555555555295 <+171>:	cdqe   
0x0000555555555297 <+173>:	mov    eax,DWORD PTR [rbp+rax*4-0x40]
0x000055555555529b <+177>:	cmp    edx,eax
0x000055555555529d <+179>:	je     0x5555555552a6 <validate+188>
0x000055555555529f <+181>:	mov    eax,0x0
0x00005555555552a4 <+186>:	jmp    0x5555555552b7 <validate+205>
0x00005555555552a6 <+188>:	add    DWORD PTR [rbp-0x48],0x1
0x00005555555552aa <+192>:	mov    eax,DWORD PTR [rbp-0x48]
0x00005555555552ad <+195>:	cmp    eax,DWORD PTR [rbp-0x44]
0x00005555555552b0 <+198>:	jl     0x55555555527f <validate+149>
0x00005555555552b2 <+200>:	mov    eax,0x1
0x00005555555552b7 <+205>:	mov    rcx,QWORD PTR [rbp-0x8]
0x00005555555552bb <+209>:	xor    rcx,QWORD PTR fs:0x28
0x00005555555552c4 <+218>:	je     0x5555555552cb <validate+225>
0x00005555555552c6 <+220>:	call   0x555555555050 <__stack_chk_fail@plt>
0x00005555555552cb <+225>:	leave  
0x00005555555552cc <+226>:	ret
```

It looks like lines from +27 to +118 are initializing a local variable on the stack that contains the wanted string. Then, a loop cycles the input a character at a time and compares it with that. You can see the cursor being moved forward at +188.

Let's try decoding those bytes:

```python
input = [
    0x66,
    0x6c,
    0x61,
    0x67,
    0x7b,
    0x48,
    0x75,
    0x43,
    0x66,
    0x5f,
    0x6c,
    0x41,
    0x62,
    0x7d,
]

for byte in input:
    print(chr(byte), end="")
```

```
~/Downloads > python hex 
flag{HuCf_lAb}
```

So there's our flag!

```
~/Downloads > ./rev 
Welcome to the Salty Spitoon™, How tough are ya?
flag{HuCf_lAb}
Right this way...
```
