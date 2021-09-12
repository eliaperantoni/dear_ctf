# CSAW 2019 beleaf

[Link to binary](https://github.com/guyinatuxedo/nightmare/blob/master/modules/03-beginner_re/csaw19_beleaf/beleaf)

---

We're presented by a prompt that asks for input. This time around, it's much more direct: it wants us to enter the flag. Plain and simple!

```
~/Downloads > ./beleaf 
Enter the flag
>>> flag
Incorrect!
```

As suggested by the challenge authors, we're going to use Ghidra this time.

The binary is stripped but finding `main` is not that difficult by searching for a `__libc_start_main` call near the entry point.


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
  long lVar2;
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
    lVar2 = FUN_001007fa(local_98[local_b0]);
    if (lVar2 != *(long *)(&DAT_003014e0 + local_b0 * 8)) {
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

There's an initial check for the length of the input (which is located at `local_98`). Then a for loop seems to be iterating the characters
