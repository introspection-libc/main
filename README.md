Safe libc
=========

How to build
------------

```sh
# llvm toolchain
[ ! -d $HOME/git ] && mkdir $HOME/git
cd $HOME/git
git clone https://github.com/introspection-libc/safe-libc-evaluation
git clone https://github.com/introspection-libc/safe-libc safec
git clone https://github.com/introspection-libc/llvm
(cd llvm/tools && git clone https://github.com/introspection-libc/clang)
(cd llvm/projects && git clone https://github.com/introspection-libc/compiler-rt)
mkdir llvm-build
(cd llvm-build && CC=clang CXX=clang++ cmake -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_ASSERTIONS=ON -DLLVM_TARGETS_TO_BUILD=X86 ~/git/llvm && make)

# build safe-libc runtime component
(cd safec && ./build)

# build softbound (optional)
git clone https://github.com/introspection-libc/softboundcets-3.8.0
mkdir softboundcets-3.8.0/llvm-38/build
(cd softboundcets-3.8.0/llvm-38/build && cmake .. && make -j8)
(cd softboundcets-3.8.0/runtime && make)
```

How to build the vulnerable projects
------------------------------------

```sh
# dnsmasq
cd $HOME/git/safe-libc-evaluation/cve/dnsmasq-2.77
make

# LightFTP
cd $HOME/git/safe-libc-evaluation/cve/LightFTP-1.1/Source/Other/Release
make

# libxml2
cd $HOME/git/safe-libc-evaluation/cve/libxml2-v2.9.4
./autogen.sh
./configure CC="$HOME/git/llvm-build/bin/clang" CFLAGS="-fsanitize=address -fno-common -g -O0 -include $HOME/git/safec/libc.h" LDFLAGS="-fsanitize=address -fno-common -g -O0 -Wl,-E" LIBS="$HOME/git/safec/libc.o"
make

# GraphicsMagick
cd $HOME/git/safe-libc-evaluation/cve/GraphicsMagick-1.3.26
./configure CC="$HOME/git/llvm-build/bin/clang" CXX="$HOME/git/llvm-build/bin/clang++" CFLAGS="-fsanitize=address -fno-common -g -O3 -include $HOME/git/safec/libc.h" CXXFLAGS="-fsanitize=address -fno-common -g -O3 -include $HOME/git/safec/libc.h" LDFLAGS="-fsanitize=address -fno-common -g -O3 -Wl,-E" LIBS="$HOME/git/safec/libc.o"
make

Repositories
------------

- [libc implementation](https://github.com/introspection-libc/safe-libc)
- [modified ASan](https://github.com/introspection-libc/compiler-rt)
- [evaluation](https://github.com/introspection-libc/safe-libc-evaluation)
