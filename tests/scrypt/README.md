# scrypt

The source code of `scrypt` can be found [here](https://github.com/Tarsnap/scrypt).

### Pre-request
- You must have automake 1.11.2 or higher, and libtool.
- In order to support the `AX_CFLAGS_WARN_ALL` autoconf directive, you will
  need to install the autoconf archive. On Debian systems, use the
  `autoconf-archive` package; on FreeBSD, use `devel/autoconf-archive`; on Ubuntu, type `sudo apt install autoconf`.
- Ignore this message if it appears:
  `aclocal: warning: couldn't open directory 'm4': No such file or directory`


### Building with gcc
```
git clone https://github.com/Tarsnap/scrypt.git
cd scrypt/
autoreconf -i
./configure
make
make test
```

### Building with kcc
```
git clone https://github.com/Tarsnap/scrypt.git
cd scrypt/
autoreconf -i
./configure CC=kcc LD=kcc
make
make test
```

### Observation

kcc failed the `make` and `make test` job.

When running `make` with `gcc`, following messages are printed:
```
Checking if compiler supports X86 CPUID feature... yes
Checking if compiler supports X86 CPUID_COUNT feature... yes
Checking if compiler supports X86 AESNI feature... yes, via -maes
Checking if compiler supports X86 RDRAND feature... yes, via -mrdrnd
Checking if compiler supports X86 SHANI feature... yes, via -msse2 -msha
Checking if compiler supports X86 SSE2 feature... yes
Checking if compiler supports X86 SSSE3 feature... yes, via -mssse3
ar: `u' modifier ignored since `D' is the default (see `U')
ar: `u' modifier ignored since `D' is the default (see `U')
ar: `u' modifier ignored since `D' is the default (see `U')
ar: `u' modifier ignored since `D' is the default (see `U')
```

When running `make` with `kcc`, following messages are printed:
```
Checking if compiler supports X86 CPUID feature... yes
Checking if compiler supports X86 CPUID_COUNT feature... yes
Checking if compiler supports X86 AESNI feature... no
Checking if compiler supports X86 RDRAND feature... no
Checking if compiler supports X86 SHANI feature... no
Checking if compiler supports X86 SSE2 feature... no
Checking if compiler supports X86 SSSE3 feature... no
ar: `u' modifier ignored since `D' is the default (see `U')
ar: `u' modifier ignored since `D' is the default (see `U')
ar: `u' modifier ignored since `D' is the default (see `U')
ar: `u' modifier ignored since `D' is the default (see `U')
lib/crypto/crypto_scrypt.c:122:2: warning: Conversion from an integer to non-null pointer.

    Implementation defined behavior (IMPL-CCV13):
        see C11 section 6.3.2.3:5 http://rvdoc.org/C11/6.3.2.3
        see CERT section INT36-C http://rvdoc.org/CERT/INT36-C

libcperciva/alg/sha256_shani.h[14:32-32] : syntax error
Translation failed (kcc_config dumped). To repeat, run this command in directory scrypt:
kcc -d -DHAVE_CONFIG_H -I. -I./lib -I./lib/crypto -I./lib/scryptenc -I./lib/util -I./libcperciva/alg -I./libcperciva/cpusupport -I./libcperciva/crypto -I./libcperciva/util -DCPUSUPPORT_CONFIG_FILE="cpusupport-config.h" -D_POSIX_C_SOURCE=200809L -D_XOPEN_SOURCE=700 -g -MT libcperciva/alg/sha256.o -MD -MP -MF libcperciva/alg/.deps/sha256.Tpo -c -o libcperciva/alg/sha256.o libcperciva/alg/sha256.c
make[1]: *** [libcperciva/alg/sha256.o] Error 1
make: *** [all] Error 2
```
---
When running `make test` with `gcc`, following messages are printed:
```
make  all-am
make[1]: Entering directory '/mnt/d/Project_Files/rv-match-playground/gcc/scrypt'
make[1]: Leaving directory '/mnt/d/Project_Files/rv-match-playground/gcc/scrypt'
./tests/test_scrypt.sh .
Running tests
-------------
SUCCESS!
SUCCESS!
SUCCESS!
SUCCESS!
no suitable system scrypt: SKIP!
SUCCESS!
SUCCESS!
SUCCESS!
```

When running `make test` with `kcc`, following messages are printed:
```
make  all-am
make[1]: Entering directory '/mnt/d/Project_Files/rv-match-playground/kcc/scrypt'
depbase=`echo libcperciva/alg/sha256.o | sed 's|[^/]*$|.deps/&|;s|\.o$||'`;\
kcc -DHAVE_CONFIG_H -I.  -I./lib -I./lib/crypto -I./lib/scryptenc -I./lib/util -I./libcperciva/alg -I./libcperciva/cpusupport -I./libcperciva/crypto -I./libcperciva/util -DCPUSUPPORT_CONFIG_FILE=\"cpusupport-config.h\" -D_POSIX_C_SOURCE=200809L -D_XOPEN_SOURCE=700    -g -MT libcperciva/alg/sha256.o -MD -MP -MF $depbase.Tpo -c -o libcperciva/alg/sha256.o libcperciva/alg/sha256.c &&\
mv -f $depbase.Tpo $depbase.Po
Makefile:1077: recipe for target 'libcperciva/alg/sha256.o' failed
make[1]: Leaving directory '/mnt/d/Project_Files/rv-match-playground/kcc/scrypt'
Makefile:659: recipe for target 'all' failed
```

### Note

One potential reason why `kcc` failed maybe that the compiler does not support `X86 AESNI`, `X86 RDRAND`, `X86 SHANI`, `X86 SSE2` and `X86 SSSE3` features.

Nevertheless, `kcc` reported a warning on `lib/crypto/crypto_scrypt.c:122:2`.
