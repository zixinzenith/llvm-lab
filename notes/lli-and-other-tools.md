# lli and other random tools i found

no theme, just poking at tools in the build's bin/ because there are
like a hundred of them and i only know 5.

## lli

runs .ll/.bc files directly with a JIT. no clang, no object files:

```bash
clang -emit-llvm -c hello.c -o hello.bc   # -c, not -S, so you get bitcode
lli hello.bc
```

my hello world ran and printed fine. so the IR really is runnable on its own,
it's not just an intermediate file format for mortals to squint at. kind of
obvious in hindsight but it felt weird the first time — like running a .java
file i guess.

## llvm-dis / llvm-as

already knew these (bitcode <-> text). still my most used pair. pro tip i
found by accident: `llvm-dis` on a .bc that starts with the wrong magic bytes
fails with a garbage error, so always check you actually downloaded the .bc
and not an html error page. don't ask.

## llvm-dwarfdump

dumps debug info. i did `clang -g -emit-llvm -S` and stared at all the
`!dbg`, `!DILocation` metadata at the bottom of the file for a while. every
line of source gets a line number + column + scope that tags along through
optimization, so debuggers can map back. there's WAY more of it than actual
code. respect to whoever debugs the debug info.

## llvm-nm / llvm-objdump

nm lists symbols in an object file, objdump disassembles. the thing i wanted:
compare what clang -O0 vs -O2 actually emit for sum at the ASSEMBLY level:

```bash
clang -O0 -S sum.c -o sum.o0.s
clang -O2 -S sum.c -o sum.o2.s
```

-O0 is a crime scene, stack traffic everywhere. -O2 for the loop version is
like 6 instructions including the closed-form math (imul + sar, because it
strength-reduced the multiply/divide-by-2 into shift. sneaky).

## FileCheck

the thing all llvm tests use. you write CHECK: lines and it pattern-matches
compiler output against them. apparently when people say "llvm's test suite"
a huge chunk is literally .ll files with CHECK lines run through lit. writing
one for my hello pass is on the list.

## verdict

biggest surprise: lli. smallest surprise: that the docs are a .rst file
rendered in 2005 style. tomorrow maybe i finally look at the Kaleidoscope
tutorial everyone keeps mentioning, or take a break. hand hurts from all the
cmake.
