# pointers in IR are just... values with types

confession: pointers in c never fully clicked for me, so i figured looking at
them in IR might help. spoiler: it kind of did?

```c
void swap(int *a, int *b) {
    int t = *a;
    *a = *b;
    *b = t;
}
```

the IR (after letting clang do its thing, cleaned up):

```llvm
define void @swap(ptr noundef %0, ptr noundef %1) {
entry:
  %t = alloca i32
  %2 = load i32, ptr %0
  store i32 %2, ptr %t
  %3 = load i32, ptr %1
  store i32 %3, ptr %0
  %4 = load i32, ptr %t
  store i32 %4, ptr %1
  ret void
}
```

things that clicked:

- `ptr` is just "pointer". it used to be typed pointers (`i32*`, `i32**`) in
  old llvm, now it's opaque `ptr` everywhere. one more thing old blog posts
  disagree with.
- a pointer isn't a box, it's a VALUE that names a box. `load i32, ptr %0`
  reads the box that %0 points at. `store ... ptr %0` writes into it. once i
  read it as "load from address, store to address" it stopped being scary.
- the local `t` gets an alloca (real stack slot) because i need a place to
  put the temp. the pointer args don't get allocas, they're already just
  values passed in. you only see `&` show up as a `getelementptr` or an
  alloca of the thing being addressed.

also tried `&x` on a normal int:

```c
int f(int x) { int *p = &x; return *p; }
```

and sure enough there's the `alloca i32` for x, because taking its address
forces it to actually live somewhere. same thing i noticed with loops but
now from the other direction. little by little.

honestly i understood swap better after 5 minutes of IR than after reading c
explanations for years. maybe i'm just weird.
