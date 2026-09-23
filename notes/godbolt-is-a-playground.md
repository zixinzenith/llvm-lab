# godbolt is a playground and i'm late to it

everyone's been saying "just use godbolt" and i kept not doing it because i
had my own clang build and stubbornness. finally tried it. ok. fine. you were
all right.

for anyone else stubborn: [godbolt.org](https://godbolt.org) lets you type c
code in one pane and see the assembly / IR update live as you type. no
terminal, no files, no build. you can even pick the llvm IR view (add a
compiler, then in the output menu pick "LLVM IR") and diff compilers side by
side.

things i did within like 20 minutes of opening it:

1. watched `volatile` turn a load into ... the same load, every time, and
   felt the difference between "compiler may cache it" vs "must not" click.
2. compared gcc and clang on the same loop. they optimize DIFFERENTLY. i
   somehow assumed there was one optimal answer and both would converge to
   it. nope, clang unrolled, gcc did something else entirely. humbling.
3. made the classic mistake of benchmarking an empty loop, watching it
   disappear entirely, and having to add `volatile` / a print to keep my
   "experiment" alive. hello to every person who's done this before me.
4. found out you can drag a slider to go through optimization LEVELS. sliding
   O0 -> O1 -> O2 -> O3 on a small function is honestly the best free
   compilers education there is. watching 40 instructions become 8 is a vibe.

the one thing godbolt can't do is run opt with my dumb little hello pass,
so local builds aren't dead. but for "what does the compiler do to this
snippet" questions it's now my first stop instead of `clang -S` in a scratch
folder. which i will still do, because muscle memory.
