# day 2 - building the thing (pain)

so i tried building llvm from source. it went about as well as the internet
promised.

## what i did

got the source from github (llvm/llvm-project), and no i did not clone the
whole history, `--depth 1` exists for a reason:

```bash
git clone --depth 1 https://github.com/llvm/llvm-project.git
cd llvm-project && mkdir build && cd build
cmake -G Ninja ../llvm \
  -DCMAKE_BUILD_TYPE=Release \
  -DLLVM_ENABLE_PROJECTS="clang" \
  -DCMAKE_INSTALL_PREFIX=$HOME/llvm-install
ninja
```

## how it went

- first attempt: forgot ninja wasn't installed. cool. `sudo apt install ninja-build`.
- second attempt: i had 4GB of ram free and no swap on this box. linking clang
  ate my machine alive and the OOM killer got ld. had to add
  `-DLLVM_USE_LINKER=lld` and also just... add swap. if you're on a small vps
  do NOT try this with default settings.
- third attempt: ~2.5 hours of ninja on 8 cores. it's a lot of code. i went and
  made dinner in the middle.

## stuff i'd tell past me

1. Release build. a Debug build is huge AND slow, don't.
2. you probably don't need clang enabled, i just wanted it. less projects =
  faster build.
3. `ninja llvm-as llvm-dis opt llc clang` builds just the tools you need way
  faster than everything. found this out after. classic.
4. check `llvm-project/llvm/docs/GettingStarted.rst` when something breaks, it's
  actually decent.

tomorrow: actually look at some IR instead of watching cmake percentages.
