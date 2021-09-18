# CSAW 2019 beleaf

[Link to
binary](https://github.com/guyinatuxedo/nightmare/blob/master/modules/03-beginner_re/csaw19_beleaf/beleaf)

---

We're presented by a prompt that asks for input. This time around, it's much
more direct: it wants us to enter the flag. Plain and simple!

```
~/Downloads > ./beleaf 
Enter the flag
>>> flag
Incorrect!
```

As suggested by the challenge authors, we're going to use Ghidra this time.

The binary is stripped but finding `main` is not that difficult by searching for
a `__libc_start_main` call near the entry point.


```
void entry(undefined8 param_1,undefined8 param_2,undefined8 param_3)

{
  undefined8 in_stack_00000000;
  undefined auStack8 [8];
  
  __libc_start_main(FUN_001008a1,in_stack_00000000,&stack0x00000008,FUN_001009e0,FUN_00100a50,
                    param_3,auStack8);
  do {
                    /* WARNING: Do nothing block with infinite loop */
  } while( true );
}
```

So here's `main`:

```
undefined8 main(void)

{
  size_t sVar1;
  long decoded;
  long in_FS_OFFSET;
  ulong local_b0;
  char local_98 [136];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 40);
  printf("Enter the flag\n>>> ");
  __isoc99_scanf(&DAT_00100a78,local_98);
  sVar1 = strlen(local_98);
  if (sVar1 < 33) {
    puts("Incorrect!");
                    /* WARNING: Subroutine does not return */
    exit(1);
  }
  for (local_b0 = 0; local_b0 < sVar1; local_b0 = local_b0 + 1) {
    decoded = decode((int)local_98[local_b0]);
    if (decoded != want_decoded[local_b0]) {
      puts("Incorrect!");
                    /* WARNING: Subroutine does not return */
      exit(1);
    }
  }
  puts("Correct!");
  if (local_10 != *(long *)(in_FS_OFFSET + 40)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return 0;
}
```

(I've already renamed some functions and locals to some mnemonic names).

There's an initial check for the length of the input (which is located at
`local_98`). Then a `for` loop seems to be iterating the characters

`decode` takes in a character from our string and returns a long in `decoded`.
Whatever value is returned must be equal to some constant found in
`want_decoded` which is an array of 33 longs.

```
long decode(char our_char)

{
  long decoded;
  
  decoded = 0;
  while ((decoded != -1 && ((int)our_char != halt_when[decoded]))) {
    if ((int)our_char < halt_when[decoded]) {
      decoded = decoded * 2 + 1;
    }
    else {
      if (halt_when[decoded] < (int)our_char) {
        decoded = (decoded + 1) * 2;
      }
    }
  }
  return decoded;
}
```

I don't completely understand what this function is doing, but we can make some
simple reasoning:

For each character:
+ Whenever the loop stops, `decoded` must be the correct long (matching the one
  in `want_decoded`) for the input to be accepted.
+ The loop stops when `our_char` (the character from our input we're currently
  decoding) is equal to `halt_when[decoded]`.

So, `decoded` is used to index into `decoding_halt_at` to decide when to stop
the loop. But when that happens, it must also be equal to what is contained in
`want_decoded`.

So let's try working backwards and see if we can guess the first character.
`want_decoded[0]` is 1. This means the result of calling `decode` with our first
character must return 1. This in turn means that the loop stops when `decoded`
(the one local to the `decode` function) is 1 which is when `halt_when[1]` is
equal to our character.

`halt_when[1]` is `0x66` which is the character lowercase f. And that is the
beginning of the flag! We can do this process for all 33 characters and we will
have our flag!

To summarize: for `i` that goes from 0 to 32 (for all characters) it must be true that `our_input[i] == halt_when[want_decoded[i]]`.

Let's write a simple Python script:

```python
from pwn import *

want_decoded = b'\x01\x00\x00\x00\x00\x00\x00\x00...'
want_decoded = [u64(want_decoded[i*8:(i+1)*8]) for i in range(len(want_decoded) // 8)]

halt_when = b'\x77\x00\x00\x00\x66\x00\x00\x00...'
halt_when = [u32(halt_when[i*4:(i+1)*4]) for i in range(len(halt_when) // 4)]

flag = ""

for i in range(33):
    flag += chr(halt_when[want_decoded[i]])

print(flag)
```

```
~/Downloads > python decode_belaf.py 
flag{we_beleaf_in_your_re_future}
~/Downloads > ./beleaf 
Enter the flag
>>> flag{we_beleaf_in_your_re_future}
Correct!
~/Downloads >
```

Nice!
