# day 1 - what even is LLVM

ok so before this week my entire mental model of compilers was:

1. you write c
2. ??? 
3. executable

turns out the ??? is actually a lot. i kept hearing people say "LLVM" like it was a
compiler and also a backend and also a library?? so here's my attempt to untangle it
before i start actually using it.

## the big picture (as i understand it today, could be wrong)

LLVM is not a compiler. it's a bunch of libraries + tools + an intermediate
representation that compilers are built OUT of. clang is the c/c++ frontend, it
eats your .c file and spits out LLVM IR, and then LLVM's backend lowers that IR
to actual machine code for x86/arm/whatever.

the IR is the whole point really. frontends produce it, backends consume it.
rust does this (rustc emits LLVM IR), swift does it, julia does it. so if i learn
the IR i'm kind of learning the common language of a bunch of compilers.

also "LLVM" apparently doesn't stand for anything anymore. it was "Low Level
Virtual Machine" back in the day but they officially killed the meaning. lol.

## random things that confused me immediately

- LLVM vs clang vs llc vs opt. as far as i can tell:
  - clang: frontend, c/c++ -> ir (or all the way to binary)
  - opt: runs IR -> IR optimizations (passes, whatever those are)
  - llc: IR -> assembly for a target
- people say "pass" a lot. i think it's just a transform that walks the IR and
  changes it. will find out.
- the docs say `.ll` is human readable IR and `.bc` is bitcode (binary form of
  the same thing). apparently you can go back and forth.

## plan

build llvm from source tomorrow because apparently that's a rite of passage and
also i want opt and the other tools. everyone warns it takes forever. bracing.
