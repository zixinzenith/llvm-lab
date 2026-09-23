# counting instructions (a pass that does slightly more)

ok printing function names got old. next step: walk the instructions in each
function and count them by opcode. this is the standard "walk a Function" loop
that everything builds on:

```cpp
PreservedAnalyses run(Function &F, FunctionAnalysisManager &AM) {
  StringMap<unsigned> Counts;
  for (BasicBlock &BB : F) {
    for (Instruction &I : BB) {
      Counts[I.getOpcodeName()]++;
    }
  }
  errs() << F.getName() << ":\n";
  for (auto &KV : Counts)
    errs() << "  " << KV.first() << " x" << KV.second() << "\n";
  return PreservedAnalyses::all();
}
```

## the nesting, finally internalized

Function > BasicBlock > Instruction. a BasicBlock is a sequence of instructions
with ONE entry (jump in only at the top) and ONE exit (must end in exactly one
terminator: br, ret, switch, etc). the CFG's nodes are the blocks and the
terminators are the edges. when i looked at the loop in the emit-llvm notes IR again with this in
my head it made a lot more sense.

running it on my sum function before mem2reg gives stuff like:

```
sum:
  alloca x2
  store x4
  load x4
  icmp x1
  br x3
  add x3
  phi x0
  ret x1
```

(eyeballed, didn't paste exactly.) then the same thing after
`-passes=mem2reg`: allocas and loads/stores all gone, phis appeared. it's
kind of a nice little visualization of what that pass does, would recommend as
an exercise.

## opcodes vs everything else

- `I.getOpcodeName()` gives you "add", "br", "alloca" etc as text.
- real code usually switches on `I.getOpcode()` (the enum) or uses
  `dyn_cast<BinaryOperator>(...)` for a specific kind. i got the casting
  syntax wrong twice (`dyn_cast` returns null, it doesn't throw).
- `isBinaryOp()`, `mayReadFromMemory()`, `mayHaveSideEffects()` — a bunch of
  cheap predicates exist, always check for one before handrolling a matcher.
  found `Instruction::isBinaryOp` after writing my own dumb switch over ~20
  opcodes. rip.

next idea: make it print the total across the whole module, then maybe sort
by count. or go look at what an analysis vs a pass actually is, since
FunctionAnalysisManager keeps sitting there in my signature and i'm treating
it like furniture.
