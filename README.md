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

# build softbound (optional)
git clone https://github.com/introspection-libc/softboundcets-3.8.0
mkdir softboundcets-3.8.0/llvm-38/build
(cd softboundcets-3.8.0/llvm-38/build && cmake .. && make -j8)
(cd softboundcets-3.8.0/runtime && make)

# build safe-libc runtime component
(cd safec && ./build)
```

How to build and run the vulnerable projects with ASan
------------------------------------------------------

### dnsmasq
```sh
# build
cd $HOME/git/safe-libc-evaluation/cve/asan/dnsmasq-2.77
make

# run
# in one terminal (as root, because port <1024):
src/dnsmasq --no-daemon --dhcp-range=fd00::2,fd00::ff -p 54
# in another terminal:
python2 CVE-2017-14496.py localhost 54
python2 CVE-2017-14493.py ::1 547
# expected output: dnsmasq prints an error message and continues to run
```

### LightFTP
```sh
# build
cd $HOME/git/safe-libc-evaluation/cve/asan/LightFTP-1.1/Source/Other/Release
make

# run
# in one terminal:
./fftp fftp.cfg
# in another terminal:
python -c 'print("USER admin\nPASS me\n" + "A"*499 + "B"*10 + "\x0D\x0A")' | ncat 127.0.0.1 9999
# expected output: LightFTP prints an error message and continues to run
```

### libxml2
```sh
# build
cd $HOME/git/safe-libc-evaluation/cve/asan/libxml2-v2.9.4
./autogen.sh
./configure CC="$HOME/git/llvm-build/bin/clang" CFLAGS="-fsanitize=address -fno-common -g -O3 -include $HOME/git/safec/libc.h" LDFLAGS="-fsanitize=address -fno-common -g -O3 -Wl,-E" LIBS="$HOME/git/safec/libc-asan.o"
make

# run
./xmllint --valid bug1.xml
# expected output: validation error + backtrace to strcat, but no crash
```

### GraphicsMagick
```sh
# build
cd $HOME/git/safe-libc-evaluation/cve/asan/GraphicsMagick-1.3.26
./configure CC="$HOME/git/llvm-build/bin/clang" CXX="$HOME/git/llvm-build/bin/clang++" CFLAGS="-fsanitize=address -fno-common -g -O3 -include $HOME/git/safec/libc.h" CXXFLAGS="-fsanitize=address -fno-common -g -O3 -include $HOME/git/safec/libc.h" LDFLAGS="-fsanitize=address -fno-common -g -O3 -Wl,-E" LIBS="$HOME/git/safec/libc-asan.o"
make

# run
utilities/gm identify -verbose exploit.miff
# expected output: backtrace followed by ASan report / crash
```

How to build and run the vulnerable projects with GCC's Intel-MPX-based Pointer Bounds Check
--------------------------------------------------------------------------------------------

### dnsmasq
```sh
# build
cd $HOME/git/safe-libc-evaluation/cve/mpx/dnsmasq-2.77
make

# run
# in one terminal (as root, because port <1024):
src/dnsmasq --no-daemon --dhcp-range=fd00::2,fd00::ff -p 54
# in another terminal:
python2 CVE-2017-14496.py localhost 54
python2 CVE-2017-14493.py ::1 547
# expected output: dnsmasq prints an error message and continues to run
```

### LightFTP
```sh
# build
cd $HOME/git/safe-libc-evaluation/cve/mpx/LightFTP-1.1/Source/Other/Release
make

# run
# in one terminal:
CHKP_RT_MODE="stop" ./fftp fftp.cfg
# in another terminal:
python -c 'print("USER admin\nPASS me\n" + "A"*499 + "B"*10 + "\x0D\x0A")' | ncat 127.0.0.1 9999
# expected output: LightFTP prints an error message and continues to run
```

### libxml2
```sh
# build
cd $HOME/git/safe-libc-evaluation/cve/mpx/libxml2-v2.9.4
./autogen.sh
./configure CC="gcc" CXX="g++" CFLAGS="-mmpx -fcheck-pointer-bounds -g -O3 -include $HOME/git/safec/libc.h" CXXFLAGS="-mmpx -fcheck-pointer-bounds -g -O3 -include $HOME/git/safec/libc.h" LDFLAGS="-lmpx -lmpxwrappers -O3 -D_FORTIFY_SOURCE=0 -Wl,-E" LIBS="$HOME/git/safec/libc-mpx.o $HOME/git/safec/mpx.o"
make

# run
./xmllint --valid bug1.xml
# expected output: validation error + backtrace to strcat, but no crash
```


### GraphicsMagick
```sh
# build GraphicsMagick
cd $HOME/git/safe-libc-evaluation/cve/mpx/GraphicsMagick-1.3.26
./configure CC="gcc" CXX="g++" CFLAGS="-mmpx -fcheck-pointer-bounds -g -O3 -include $HOME/git/safec/libc.h" CXXFLAGS="-mmpx -fcheck-pointer-bounds -g -O3 -include $HOME/git/safec/libc.h" LDFLAGS="-lmpx -lmpxwrappers -O3 -D_FORTIFY_SOURCE=0 -Wl,-E" LIBS="$HOME/git/safec/libc-mpx.o $HOME/git/safec/mpx.o"
make

# run GraphicsMagick exploit
CHKP_RT_MODE="stop" utilities/gm identify -verbose exploit.miff
# expected output: backtrace followed by MPX report / crash
```

How to build and run the vulnerable projects with SoftBound
-----------------------------------------------------------
The latest stable version of SoftBound failed to build the projects.
However, the extracted fragments can be build and executed (see [here](https://github.com/introspection-libc/safe-libc-evaluation/tree/master/cve-extracted-fragments)).

Modifications to the individual projects
----------------------------------------

- LightFTP: Makefile was modified to use safe-libc, test config was added
- dnsmasq: Makefile was modified to use safe-libc

Repositories
------------

- [libc implementation](https://github.com/introspection-libc/safe-libc)
- [modified ASan](https://github.com/introspection-libc/compiler-rt)
- [evaluation](https://github.com/introspection-libc/safe-libc-evaluation)
