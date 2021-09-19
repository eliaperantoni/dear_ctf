If you're not familiar with PIE and ASLR. Here's a quick rundown of what they
are, how they're related and why they make things difficult for us:

The memory of a running process is made up by contiguous sequences of related
bytes. There's one for the machine code, one for the stack, one for the heap,
one for global data, etc.. They are all located somewhere in the virtual address
space. Sometimes as specified by the executable, other times randomly.

When their position is constant between executions, we can figure out addresses
that are of intereset to us (the address of a variable that needs to be changed
for instance) and use them to prepare our exploit.

But when their position change, we cannot know the address until we execut the
process and thus our exploit must react dynamically, we can't embed a static
address.

ASLR stands for Address Space Layout Randomization and is the process that the
operating system performs when first loading your executable. When enabled, it
places any memory region in a random position along the virtual address space.

But not all executable can handle their memory regions being loaded at any
address! Take the memory region containing the machine code for instance, `JMP`
instructions will have to add an offset to the current instruction instead of
jumping to an absolute address in order to work.

The executables that support this are called PIE: Position Independent
Executables and when you're compiling an executable you can decide whether you
want it to be PIE or not.

ASLR needs an executable to be PIE in order to randomize the position of the
machine code region. If any of those two are disabled, every machine code
instruction will always have the same address! And this can be exploited as we'll see.
