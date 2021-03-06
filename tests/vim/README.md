# vim

The source code of `vim` can be found [here](https://github.com/vim/vim).

### Pre-request
```
sudo apt -y install ncurses-dev
    sudo apt -y install libncurses5-dev
    sudo apt -y install libncursesw5-dev
    sudo apt -y build-dep vim
```

### Building with gcc
```
cd vim/
    kcc -profile x86_64-linux-gcc-glibc
    ./configure CC=gcc CFLAGS="-std=gnu11" LD=gcc LDFLAGS="-ldl" LINK_AS_NEEDED="yes" |& tee rv_build_0.txt ; results[0]="$?" ; postup 0
    make -j`nproc` |& tee rv_build_1.txt ; results[1]="$?" ; postup 1
```

### Building with kcc
```
cd vim/
    kcc -profile x86_64-linux-gcc-glibc
    ./configure CC=kcc CFLAGS="-std=gnu11" LD=kcc LDFLAGS="-ldl" LINK_AS_NEEDED="yes" |& tee rv_build_0.txt ; results[0]="$?" ; postup 0
    make -j`nproc` |& tee rv_build_1.txt ; results[1]="$?" ; postup 1
```





### Observation - Build

After using the commandd " ./configure CC=$compiler CFLAGS="-std=gnu11" LD=$compiler LDFLAGS="-ldl" LINK_AS_NEEDED="yes" |& tee rv_build_0.txt ;"
Both kcc and gcc compiled successfully.
`gcc` took `30s 24ms` to finish `make`, while `kcc` took `7m 23.569s`.

But after using the command "make -j`nproc` |& tee rv_build_1.txt ; results[1]="$?" ; postup 1"
'gcc' took around the same time as before but kcc took around 15m to compile a part only and after that my laptop crashed. I tried it for 3-4 times every time it stopped somewhere executing one of the obejct files named eval.o
