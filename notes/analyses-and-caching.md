# what the analysis manager actually does

so. the second argument to my run() function. turns out it's the interesting
part.

## the idea

analyses compute info about IR (dominator tree, alias analysis, etc) WITHOUT
changing it. passes consume analyses and/or change IR. the manager caches
analysis results, and the whole PreservedAnalyses thing is the invalidation
protocol: when a pass runs and returns "all preserved", the cached dominator
tree etc stays valid. if you say none(), everything gets recomputed lazily next
time someone asks.

you can also be precise: mark everything preserved, then explicitly say one
analysis got invalidated. there's `PreservedAnalyses PA; PA.preserveSet<...>()`
or something, i haven't needed it yet, not going to pretend i know the exact
spelling.

## using one for real

grabbed the dominator tree and printed each function's reverse-post-order
block traversal:

```cpp
auto &DT = AM.getResult<DominatorTreeAnalysis>(F);
errs() << F.getName() << " rpo:\n";
for (auto *BB : post_order(&DT.getBase()))  // note: reverse post order needs llvm::ReversePostOrderTraversal
  errs() << "  " << BB->getName() << "\n";
```

(actually the clean way is `ReversePostOrderTraversal<Function*> RPOT(&F);` —
you don't even need the domtree for rpo, but i wanted to touch
`getResult` at least once and domtree is the classic.)

## what "dominates" means, since i had to re-derive it twice

block A dominates block B if EVERY path from entry to B goes through A. the
"dominator tree" is that relationship compressed into a tree. loops' loop
headers dominate their whole loop body, which is why everyone cares: knowing
something is true at a point inside a loop often reduces to checking the
header, which dominates you.

random useful facts i keep re-deriving:

- the entry block dominates everything (trivially, every path starts there)
- natural loops are basically "back-edge to a block that dominates the source
  of the edge". i checked this against the loop in the emit-llvm notes, the `for.cond` header does
  dominate `for.body`. ok. fine. it holds.

the pass manager apparently reorders passes around these invalidation rules so
hot analyses stay cached as long as possible. i'll take that on faith for now.
