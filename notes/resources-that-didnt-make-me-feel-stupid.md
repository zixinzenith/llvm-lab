# resources that didn't make me feel stupid

most llvm docs assume you already know a lot. these didn't, bookmarking them
here so i stop losing them in my browser tabs (currently at 73. it's fine.)

- **the LLVM Language Reference** (llvm.org/docs/LangRef.html). yes it's
  giant. no you don't read it. you ctrl-F it when you meet an instruction you
  don't know. it's a dictionary, not a novel.
- **the Kaleidoscope tutorial.** making a tiny language with a JIT. only
  started it, but so far it's the first official llvm thing that reads like
  it WANTS you to finish. note the c++ version needs small updates on modern
  llvm (opaque pointers mainly), the kind folk on forums already posted the
  fixes.
- **"LLVM for Grad Students"** by Jay Lee. short, friendly, exactly the right
  level of hand-holding for where i am. found it through a forum thread and
  it instantly made the pass stuff less mysterious.
- **the map of llvm blog posts** — there's a famous one (brson's "map of
  llvm", iirc) that lists like 80 articles by topic. i keep it open and hop
  around. half the links are old pass manager stuff but the concepts carry
  over ok.
- **godbolt.** covered already but it belongs on any list.
- **the llvm discord / mailing lists.** haven't asked anything yet (imposter
  syndrome), but reading other beginners' questions and the patient answers
  has been weirdly encouraging. someone asked almost my exact phi question
  last month. i am not alone in the confusion mines.

anti-recommendation: trying to read clang's source "to get a feel for it".
maybe someday. not today. not soon.
