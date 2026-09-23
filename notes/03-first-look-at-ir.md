# day 3 - first look at IR

ok THIS is the fun part. i wrote the world's most boring c program:

```c
int add(int a, int b) {
    return a + b;
}
```

and turned it into IR with:

```bash
clang -S -emit-llvm add.c -o add.ll
```

here's what came out (minus the annoying attribute lines at the end):

```llvm
define i32 @add(i32 noundef %0, i32 noundef %1) {
  %3 = add nsw i32 %0, %1
  ret i32 %3
}
```

things i noticed / learned:

- `i32` = 32-bit integer. not int. the IR doesn't care about your c types,
  only widths.
- unnamed values just get numbers: `%0`, `%1`, `%3`. note there's no `%2`! i
  stared at this for a minute. pretty sure it's because the counter ticks for
  the entry block too or something got numbered then folded. honestly still not
  100% sure, if you know, tell me.
- `nsw` = "no signed wrap". it's a promise that signed overflow won't happen,
  which lets the optimizer be more aggressive. funny that this promise is
  default-on in c. bye bye UB jokes.
- `noundef` on params is another promise: the value isn't undefined.
- `@add` is global (functions/globals get @), locals get %.

then i tried running opts on it to see what changes:

```bash
opt -O2 -S add.ll -o add.opt.ll
```

basically the same for this tiny thing, not surprising. need a bigger function
to actually see passes do stuff. maybe tomorrow i feed it something with a loop.

also `llvm-dis` (bitcode -> text) and `llvm-as` (text -> bitcode) work exactly
like you'd hope. round tripped a file with no complaints.
