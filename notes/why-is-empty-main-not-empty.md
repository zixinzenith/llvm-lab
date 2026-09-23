# why is an empty main not empty

so i wrote this:

```c
int main(void) { return 0; }
```

and dumped the IR, expecting basically nothing. nope. clang -O0 gives me a
whole function with a stack frame, allocas for nothing, stores of nothing.
there's an `alloca i32` for... the return value?? that i immediately store 0
into and load back and return. it's doing a bunch of ceremony around a number
it already knows.

my first reaction was "clang is dumb". my second reaction, after running
clang -O2, was "oh". at -O2 main is literally:

```llvm
define i32 @main() {
  ret i32 0
}
```

so the ceremony at -O0 is deliberate. as far as i can tell, -O0 generates the
most naive, direct, "every c variable lives in memory" translation possible,
so that (a) debuggers can find every variable, (b) compilation is fast, and
(c) the optimizer has a completely predictable starting point. then the
optimizers strip all the ceremony later if you ask for it.

that actually explains a lot. i used to think -O0 output was "what the
compiler really thinks" and -O2 was some magic layer on top. it's more like
-O0 is a boring but faithful transcription, and the IR is designed so boring
faithful transcriptions can be cleaned up mechanically.

still funny to look at though. my one-line main is 10 instructions. the
computer is doing paperwork.
