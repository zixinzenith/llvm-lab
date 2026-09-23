# things i still don't get

writing down the open questions so they stop swirling. no answers here, this
is a public todo list of confusion.

- **inlining decisions.** i understand WHAT inlining is. i have no idea how
  the compiler decides. i made a tiny function, called it twice, watched it
  get inlined at -O2, then added a third call site and it WASN'T anymore.
  changed something back, still not inlined. no idea what flipped it. i know
  there's a cost model in there somewhere. currently it's a magic 8-ball.
- **getelementptr.** everyone says it's the hardest instruction and i'm just
  gonna agree preemptively. i read the gep faq for 20 minutes and understood
  maybe 40%. the two-indices-aren't-added thing specifically broke me. will
  revisit. multiple times probably.
- **why phi can't be first in some blocks.** there's a rule that phis must be
  at the top of a block, but also something about not mixing with other values
  and terminator placement. i've hit verifier errors that i "fixed" by moving
  lines around until it compiled. that's not understanding.
- **what a "pipeline" actually is end to end.** i know passes run in an order.
  i found `opt -debug-pass-manager` (p -print-pipeline something? the flag
  names keep changing on me) which dumps the entire thing and it is LONG. like
  hundreds of passes long. i thought i'd read it all. i did not read it all.
- **linking, at a fundamental level.** honestly this predates llvm. what does
  the linker DO. every time something fails to link i cargo-cult flags until
  it works.
- **register allocation.** i know registers exist and there aren't enough of
  them and somehow it's graph coloring?? that's where my knowledge ends.
- **why `opt -O2` and `clang -O2` don't produce identical IR.** clang runs
  some stuff in its own frontend pipeline first i think? still fuzzy.

if any of these turn out to be simple and i look dumb for listing them...
well. that's the learning in public deal i signed up for.
