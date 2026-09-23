# clang -emit-llvm experiments

how i poked at how different c constructs show up in the IR. this is mostly
me annotating dumps for my own memory.

## a loop

```c
int sum(int n) {
    int s = 0;
    for (int i = 0; i < n; i++)
        s = s + i;
    return s;
}
```

IR (clang -S -emit-llvm, cleaned up):

```llvm
define i32 @sum(i32 noundef %0) {
entry:
  br label %for.cond

for.cond:
  %s.0 = phi i32 [ 0, %entry ], [ %add, %for.body ]
  %i.0 = phi i32 [ 0, %entry ], [ %inc, %for.body ]
  %cmp = icmp slt i32 %i.0, %0
  br i1 %cmp, label %for.body, label %for.end

for.body:
  %add = add nsw i32 %s.0, %i.0
  %inc = add nsw i32 %i.0, 1
  br label %for.cond

for.end:
  ret i32 %s.0
}
```

- no "variables" in the IR really. just SSA values. each name is assigned
  exactly once.
- the weird one is `phi`. when control flow merges, the value of `s` "coming
  in" depends on where you came from. phi is how SSA expresses that: "if you
  jumped from entry, i'm 0; if from for.body, i'm %add". i've read this
  explained 5 times and it only clicked when i saw a loop.
- alloca (stack slots) shows up for real c locals when it can't just keep
  everything in registers, e.g. when you take the address of something. i'll
  dig into that another day.

## mem2reg, the famous one

people kept telling me about this pass so i ran it manually:

```bash
opt -passes=mem2reg -S test.ll -o test.m2r.ll
```

it promotes all the alloca/load/store noise into straight SSA values with phis.
basically it makes the IR look like the loop above instead of a pile of stack
accesses. apparently clang runs this basically always and it's step one of
making IR optimizable. makes sense, hard to reason about values hiding behind
pointers.

## o2 vs o0 sanity check

same sum function at -O2: the loop is GONE. it turned it into the closed form
(n * (n-1) / 2 style, with a sdiv and everything). i'm sure every textbook
mentions this but seeing it on a function i typed myself was different.
compilers are scary.

next up: i want to write my own (useless) pass with the new pass manager, the
ones that print function names in the tutorials. that involves linking against
llvm as a library and cmake hell, so, wish me luck.
