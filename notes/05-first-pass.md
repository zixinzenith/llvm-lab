# day 5 - writing my first pass (cmake hell as promised)

ok so today was the "HelloWorld" pass from the WritingAnLLVMPass doc, except
the doc version i found first was the OLD pass manager (registerPass,
legacy::PassManager) and half the internet still shows that. the new one is the
"new pass manager" (NPM) and it's completely different plumbing. lost like an
hour to that.

## what actually worked

out-of-tree pass, in its own folder, not inside llvm source. project layout:

```
hello-pass/
├── CMakeLists.txt
├── HelloWorld.cpp
```

HelloWorld.cpp (the useful part):

```cpp
#include "llvm/IR/PassManager.h"
#include "llvm/Passes/PassBuilder.h"
#include "llvm/Passes/PassPlugin.h"

using namespace llvm;

struct HelloWorldPass : PassInfoMixin<HelloWorldPass> {
  PreservedAnalyses run(Function &F, FunctionAnalysisManager &AM) {
    errs() << "hello from function: " << F.getName() << "\n";
    return PreservedAnalyses::all();
  }
};

extern "C" LLVM_ATTRIBUTE_WEAK PassPluginLibraryInfo llvmGetPassPluginInfo() {
  return {LLVM_PLUGIN_API_VERSION, "hello", "v0.1",
          [](PassBuilder &PB) {
            PB.registerPipelineParsingCallback(
                [](StringRef Name, FunctionPassManager &FPM,
                   ArrayRef<PassBuilder::PipelineElement>) {
                  if (Name == "hello-pass") {
                    FPM.addPass(HelloWorldPass());
                    return true;
                  }
                  return false;
                });
          }};
}
```

CMakeLists.txt:

```cmake
cmake_minimum_required(VERSION 3.20)
project(hello-pass)
find_package(LLVM REQUIRED CONFIG)
add_llvm_pass_plugin(hello hello.cpp)
```

the magic cmake bit is `add_llvm_pass_plugin` — you get it by adding
`${LLVM_CMAKE_DIR}` to the module path, which find_package sets up. then:

```bash
cmake -B build -DLLVM_DIR=$HOME/llvm-install/lib/cmake/llvm
cmake --build build
```

and run it (opt loads the .so at runtime, no rebuilding llvm needed):

```bash
opt -load-pass-plugin=./build/hello.so \
    -passes=hello-pass -disable-output sum.ll
```

## output

```
hello from function: sum
```

i know. incredible. ten out of ten.

## notes to self

- `PreservedAnalyses::all()` means "i changed nothing, cache everything".
  if you modify the IR you return `PreservedAnalyses::none()`. getting this
  wrong probably makes later passes see stale analyses. will absolutely make
  this mistake later.
- the plugin api version matters. if you build the .so against llvm 19 and run
  opt from llvm 17 it just refuses to load with a version error. fine, but
  confusing the first time.
- `llvmGetPassPluginInfo` is the ONLY symbol opt looks for. everything else
  hangs off the callback you register there.

took way longer than the ~30 lines suggest but it does feel like a real
checkpoint. i can now technically write a compiler pass. it prints function
names. don't ask me to do more yet.
