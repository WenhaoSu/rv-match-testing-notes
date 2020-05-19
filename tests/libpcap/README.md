# libpcap

The source code of `libpcap` can be found [here](https://github.com/the-tcpdump-group/libpcap.git).

### Pre-request
```
    sudo apt -y install bison
    sudo apt -y install flex
```

### Building with gcc
```
cd libpcap/
    sed -i '124,125d' tests/Makefile.in
    aclocal; autoreconf
    ./configure CC="gcc -std=gnu11" LD=gcc |& tee rv_build_0.txt ; results[0]="$?" ; postup 0
    make -j`nproc` |& tee rv_build_1.txt ; results[1]="$?" ; postup 1
    sed -i '35,38d' tests/CMakeLists.txt
    sed -i '82d' tests/Makefile
    sed -i -e "s/valgrindtest.c/capturetest.c/g" tests/Makefile
```

### Building with kcc
```
cd libpcap/
    sed -i '124,125d' tests/Makefile.in
    aclocal; autoreconf
    ./configure CC="kcc -std=gnu11" LD=kcc |& tee rv_build_0.txt ; results[0]="$?" ; postup 0
    make -j`nproc` |& tee rv_build_1.txt ; results[1]="$?" ; postup 1
    sed -i '35,38d' tests/CMakeLists.txt
    sed -i '82d' tests/Makefile
    sed -i -e "s/valgrindtest.c/capturetest.c/g" tests/Makefile
```
### Testing
```
cd tests/
    names[2]="make tests" ; make -j`nproc` |& tee rv_build_2.txt ; results[2]="$?" ; postup 2
```


### Observation - Build

Building for both kcc and gcc were successful the only difference were some potential errors caught by kcc.
```
checking if unaligned accesses fail... Encountered an unknown error. This may be due to encountering undefined behavior, an unsupported language feature, or a bug in this tool:
      > in main at conftest.c:10:3

    Unknown error (UNK-1)

Indeterminate value used in an expression:
      > in main at conftest.c:11:3

    Undefined behavior (UB-CEE2):
        see C11 section 6.2.4 http://rvdoc.org/C11/6.2.4
        see C11 section 6.7.9 http://rvdoc.org/C11/6.7.9
        see C11 section 6.8 http://rvdoc.org/C11/6.8
        see C11 section J.2:1 item 11 http://rvdoc.org/C11/J.2
        see CERT-C section EXP33-C http://rvdoc.org/CERT-C/EXP33-C
        see MISRA-C section 8.9:1 http://rvdoc.org/MISRA-C/8.9
        see MISRA-C section 8.1:3 http://rvdoc.org/MISRA-C/8.1

Indeterminate value used in an expression:
      > in main at conftest.c:13:3

    Undefined behavior (UB-CEE2):
        see C11 section 6.2.4 http://rvdoc.org/C11/6.2.4
        see C11 section 6.7.9 http://rvdoc.org/C11/6.7.9
        see C11 section 6.8 http://rvdoc.org/C11/6.8
        see C11 section J.2:1 item 11 http://rvdoc.org/C11/J.2
        see CERT-C section EXP33-C http://rvdoc.org/CERT-C/EXP33-C
        see MISRA-C section 8.9:1 http://rvdoc.org/MISRA-C/8.9
        see MISRA-C section 8.1:3 http://rvdoc.org/MISRA-C/8.1

'printf': Mismatch between the type expected by the conversion specifier %d and the type of the argument:
      > in printf at conftest.c:22:3
        in main at conftest.c:22:3

    Undefined behavior (UB-STDIO1):
        see C11 section 7.21.6.1:9 http://rvdoc.org/C11/7.21.6.1
        see C11 section J.2:1 item 153 http://rvdoc.org/C11/J.2
        see CERT-C section FIO47-C http://rvdoc.org/CERT-C/FIO47-C


```
The duration for 'gcc' was 10.34s whereas it was 1m30.46s for 'kcc'.

### Observation - Build

In case of gcc there was success but in the case of kcc the following "threasdsignaltest" test case failed.

```
kcc -std=gnu11 -O -I. -I.. -I. -I./..  -I/usr/local/include -I/usr/include/dbus-1.0 -I/usr/lib/x86_64-linux-gnu/dbus-1.0/include -DHAVE_CONFIG_H  -g    -I. -L. -o threadsignaltest ./threadsignaltest.c ../libpcap.a -ldbus-1 
Translation failed. To repeat, run this command in directory tests:
kcc -d -std=gnu11 -O -I. -I.. -I. -I./.. -I/usr/local/include -I/usr/include/dbus-1.0 -I/usr/lib/x86_64-linux-gnu/dbus-1.0/include -DHAVE_CONFIG_H -g -I. -L. -o threadsignaltest ./threadsignaltest.c ../libpcap.a -ldbus-1


```
