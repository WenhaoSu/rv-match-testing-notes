# dpkg

The source code of `dpkg` can be found [here]( https://salsa.debian.org/dpkg-team/dpkg.git) and tests can be downloaded from
( https://salsa.debian.org/dpkg-team/dpkg-tests.git).

### Pre-request
```
    sudo apt -y purge gettext
    sudo apt-get -y install gettext
    gettext --version
    apt-cache policy gettext
    sudo apt update
    sudo apt -y upgrade    
```

### Building with gcc
```
    cd dpkg/
    autoscan
    aclocal
    autoheader
    autoreconf
    automake --add-missing
    autoreconf -vif
    ./configure CC=gcc |& tee rv_build_0.txt ; results[0]="$?"
    make -j`nproc` |& tee rv_build_1.txt ; results[1]="$?" ; postup 1
    names[2]="make check"
    make -j`nproc` check |& tee rv_build_2.txt ; results[2]="$?" ; postup 2
````    


### Building with kcc
```
    cd dpkg/
    autoscan
    aclocal
    autoheader
    autoreconf
    automake --add-missing
    autoreconf -vif
    ./configure CC=kcc LD=kcc LDFLAGS="-lz" |& tee rv_build_0.txt ; results[0]="$?"
    make -j`nproc` |& tee rv_build_1.txt ; results[1]="$?" ; postup 1
    names[2]="make check"
    make -j`nproc` check |& tee rv_build_2.txt ; results[2]="$?" ; postup 2
```
### Testing
```
cd dpkg-tests/ ; ./db-regen

```
### Observation - Build
After the command ./configure both the programs compiled successfully. The 'gcc' compiler took 23.47s whereas the 'kcc' compiler took 6m4.18s to do this task.

After the second command, the gcc compiler build the program successfully whereas the kcc one crashed after 15m.
The kcc compiler caught some potential errors.
```
triglib.c:506:2: error: Potential negative zero produced via bitwise operations, undefined under sign and magnitude or one's complement arithmetic, implementation-defined otherwise.

    Implementation-dependent undefined behavior (IMPLUB-CEB12):
        see C11 section 6.2.6.2:4 http://rvdoc.org/C11/6.2.6.2
        see C11 section J.2:1 item 14 http://rvdoc.org/C11/J.2
        see CERT-C section INT13-C http://rvdoc.org/CERT-C/INT13-C
        see CERT-C section MSC15-C http://rvdoc.org/CERT-C/MSC15-C
        see MISRA-C section 8.1:3 http://rvdoc.org/MISRA-C/8.1

triglib.c:690:2: error: Potential negative zero produced via bitwise operations, undefined under sign and magnitude or one's complement arithmetic, implementation-defined otherwise.

    Implementation-dependent undefined behavior (IMPLUB-CEB12):
        see C11 section 6.2.6.2:4 http://rvdoc.org/C11/6.2.6.2
        see C11 section J.2:1 item 14 http://rvdoc.org/C11/J.2
        see CERT-C section INT13-C http://rvdoc.org/CERT-C/INT13-C
        see CERT-C section MSC15-C http://rvdoc.org/CERT-C/MSC15-C
        see MISRA-C section 8.1:3 http://rvdoc.org/MISRA-C/8.1


```

### Observation - Test
The gcc compiler passed 217 tests successfully whereas testing was not possible for kcc compiler.
