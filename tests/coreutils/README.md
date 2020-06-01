# coreutils

The source code of `coreutils` can be found [here](https://github.com/coreutils/coreutils).

### Pre-request
```
git clone https://github.com/coreutils/coreutils.git
cd coreutils/
```

### Build
To build with `gcc`, type:
```
./bootstrap
./configure --disable-gcc-warnings
make
```
Here if we don't include the flag `--disable-gcc-warnings`, then once `./configure` find a warning, it would stop. To build with `kcc`, type:
```
./bootstrap
./configure CC=kcc LD=kcc --disable-gcc-warnings
make
```

### Test
```
make check
```


### Observation - Build

`kcc` failed in the `make` job.

When running `make` with `kcc`, we get the following error message when compiling `src/make-prime-list`:

```
make src/make-prime-list
make[1]: Entering directory '/mnt/c/Users/suwen/Desktop/intern/coreutils-test/kcc/coreutils'
  CC       src/make-prime-list.o
In file included from src/make-prime-list.c:22:0:
./lib/limits.h:41:3: warning: #include_next is a GCC extension
 # include_next <limits.h>
   ^~~~~~~~~~~~
In file included from src/make-prime-list.c:24:0:
./lib/inttypes.h:41:4: warning: #include_next is a GCC extension
 #  include_next <inttypes.h>
    ^~~~~~~~~~~~
In file included from src/make-prime-list.c:25:0:
./lib/stdio.h:43:2: warning: #include_next is a GCC extension
 #include_next <stdio.h>
  ^~~~~~~~~~~~
In file included from ./lib/stdio.h:58:0,
                 from src/make-prime-list.c:25:
./lib/sys/types.h:39:2: warning: #include_next is a GCC extension
 #include_next <sys/types.h>
  ^~~~~~~~~~~~
In file included from src/make-prime-list.c:26:0:
./lib/string.h:41:2: warning: #include_next is a GCC extension
 #include_next <string.h>
  ^~~~~~~~~~~~
In file included from src/make-prime-list.c:27:0:
./lib/stdlib.h:36:2: warning: #include_next is a GCC extension
 #include_next <stdlib.h>
  ^~~~~~~~~~~~
In file included from src/make-prime-list.c:22:0:
./lib/limits.h:41:3: warning: #include_next is a GCC extension
 # include_next <limits.h>
   ^~~~~~~~~~~~
In file included from src/make-prime-list.c:24:0:
./lib/inttypes.h:41:4: warning: #include_next is a GCC extension
 #  include_next <inttypes.h>
    ^~~~~~~~~~~~
In file included from src/make-prime-list.c:25:0:
./lib/stdio.h:43:2: warning: #include_next is a GCC extension
 #include_next <stdio.h>
  ^~~~~~~~~~~~
In file included from ./lib/stdio.h:58:0,
                 from src/make-prime-list.c:25:
./lib/sys/types.h:39:2: warning: #include_next is a GCC extension
 #include_next <sys/types.h>
  ^~~~~~~~~~~~
In file included from src/make-prime-list.c:26:0:
./lib/string.h:41:2: warning: #include_next is a GCC extension
 #include_next <string.h>
  ^~~~~~~~~~~~
In file included from src/make-prime-list.c:27:0:
./lib/stdlib.h:36:2: warning: #include_next is a GCC extension
 #include_next <stdlib.h>
  ^~~~~~~~~~~~
src/make-prime-list.c:43:18: warning: ISO C does not support '__int128' types [-Wpedantic]
 typedef unsigned __int128 wide_uint;
                  ^~~~~~~~
  CCLD     src/make-prime-list
make[1]: Leaving directory '/mnt/c/Users/suwen/Desktop/intern/coreutils-test/kcc/coreutils'
  GEN      src/primes.h
Referring to an object outside of its lifetime:
      > in main at src/make-prime-list.c:223:3

    Undefined behavior (UB-CEE4):
        see C11 section 6.2.4:2 http://rvdoc.org/C11/6.2.4
        see C11 section J.2:1 item 9 http://rvdoc.org/C11/J.2
        see CERT-C section DCL21-C http://rvdoc.org/CERT-C/DCL21-C
        see CERT-C section DCL30-C http://rvdoc.org/CERT-C/DCL30-C
        see CERT-C section MEM30-C http://rvdoc.org/CERT-C/MEM30-C
        see MISRA-C section 8.18:6 http://rvdoc.org/MISRA-C/8.18
        see MISRA-C section 8.22:6 http://rvdoc.org/MISRA-C/8.22
        see MISRA-C section 8.1:3 http://rvdoc.org/MISRA-C/8.1

Dereferencing a null pointer:
      > in ferror at src/make-prime-list.c:223:3
        in main at src/make-prime-list.c:223:3

    Undefined behavior (UB-CER3):
        see C11 section 6.5.3.2:4 http://rvdoc.org/C11/6.5.3.2
        see C11 section J.2:1 item 43 http://rvdoc.org/C11/J.2
        see CERT-C section EXP34-C http://rvdoc.org/CERT-C/EXP34-C
        see MISRA-C section 8.1:3 http://rvdoc.org/MISRA-C/8.1

Execution failed (configuration dumped)
Makefile:15417: recipe for target 'src/primes.h' failed
make: *** [src/primes.h] Error 139
```
However for `gcc`, it succeeded in build and is able to pass most of test cases:
```
============================================================================
Testsuite summary for GNU coreutils 8.32.19-ae79d4
============================================================================
# TOTAL: 622
# PASS:  404
# SKIP:  170
# XFAIL: 0
# FAIL:  44
# XPASS: 0
# ERROR: 4
============================================================================
See ./tests/test-suite.log
Please report to bug-coreutils@gnu.org
============================================================================
```

### Note

To run `./configure` and `make` really take time for both `gcc` and `kcc`, and it also takes time to run all test cases by typing `make check`. To test each cases individually could save time.

### Followup

Line 223 of `make-prime-list.c` is as following:
```c
...
if (ferror (stdout) + fclose (stdout))      // Line 223
    {
      fprintf (stderr, "write error: %s\n", strerror (errno));
      return EXIT_FAILURE;
    }
...
```

Hence we can try to write a simple programming only includes this part of logic:
```c
#include <stdio.h>

int main () {
    if (ferror (stdout) + fclose (stdout)) {
      fprintf (stderr, "inside if\n");
      return -1;
    }
    fprintf (stderr, "outside if\n");
    return 0;
}
```
We can build this file normally with both `gcc` and `kcc`. If we run the compiled file with `gcc`, it will print out `outside if`, which means that it does not enter the if block. However if we run the compiled file with `kcc`, it will then report those error:
```
Referring to an object outside of its lifetime:
      > in main at test.c:4:5

    Undefined behavior (UB-CEE4):
        see C11 section 6.2.4:2 http://rvdoc.org/C11/6.2.4
        see C11 section J.2:1 item 9 http://rvdoc.org/C11/J.2
        see CERT-C section DCL21-C http://rvdoc.org/CERT-C/DCL21-C
        see CERT-C section DCL30-C http://rvdoc.org/CERT-C/DCL30-C
        see CERT-C section MEM30-C http://rvdoc.org/CERT-C/MEM30-C
        see MISRA-C section 8.18:6 http://rvdoc.org/MISRA-C/8.18
        see MISRA-C section 8.22:6 http://rvdoc.org/MISRA-C/8.22
        see MISRA-C section 8.1:3 http://rvdoc.org/MISRA-C/8.1

Dereferencing a null pointer:
      > in ferror at test.c:4:5
        in main at test.c:4:5

    Undefined behavior (UB-CER3):
        see C11 section 6.5.3.2:4 http://rvdoc.org/C11/6.5.3.2
        see C11 section J.2:1 item 43 http://rvdoc.org/C11/J.2
        see CERT-C section EXP34-C http://rvdoc.org/CERT-C/EXP34-C
        see MISRA-C section 8.1:3 http://rvdoc.org/MISRA-C/8.1

Execution failed (configuration dumped)
```
Which is exactly the error message we get when compiling `coreutils`. For both `x86_64-linux-gcc-glibc-gnuc` and `x86_64-linux-gcc-glibc` profiles, they give the same error.

---
Observation: if we use `x86_64-linux-gcc-glibc-gnuc-reverse-eval-order` or `x86_64-linux-gcc-glibc-reverse-eval-order` to build the above simple program, then it would success. Since `x86_64-linux-gcc-glibc-gnuc-reverse-eval-order` will crash on the `./configure` stage, it seems that we need to use this profile: `x86_64-linux-gcc-glibc-reverse-eval-order` in order to build `coreutils`.

However, although `kcc` succeeded in making `make-prime-list.c`, it still met an error in a later stage during make:
```
lib/fts.c[789:0-0] : syntax error
Translation failed (kcc_config dumped). To repeat, run this command in directory coreutils:
kcc -d -I. -I./lib -Ilib -I./lib -Isrc -I./src -g -MT lib/fts.o -MD -MP -MF lib/.deps/fts.Tpo -c -o lib/fts.o lib/fts.c
Makefile:9782: recipe for target 'lib/fts.o' failed
```

Line 789 of `fts.c` is as following:
```c
...
static enum leaf_optimization
leaf_optimization (FTSENT const *p, int dir_fd)
{
  switch (filesystem_type (p, dir_fd))
    {
    case 0:
      /* Leaf optimization is unsafe if the file system type is unknown.  */
      FALLTHROUGH;                                     // Line 789
    case S_MAGIC_AFS:
      /* Although AFS mount points are not counted in st_nlink, they
         act like directories.  See <https://bugs.debian.org/143111>.  */
      FALLTHROUGH;
...
}
```
where `FALLTHROUGH` is defined in `attribute.h`:
```c
...
/* Do not warn if control flow falls through to the immediately
   following 'case' or 'default' label.  */
/* Applies to: Empty statement (;), inside a 'switch' statement.  */
#define FALLTHROUGH _GL_ATTRIBUTE_FALLTHROUGH
...
```
and we can also traceback to `_GL_ATTRIBUTE_FALLTHROUGH` in `config.h`:
```c
...
/* FALLTHROUGH is special, because it always expands to something.  */
#if 201710L < __STDC_VERSION__
# define _GL_ATTRIBUTE_FALLTHROUGH [[__fallthrough__]]
#elif _GL_HAS_ATTRIBUTE (fallthrough)
# define _GL_ATTRIBUTE_FALLTHROUGH __attribute__ ((__fallthrough__))
#else
# define _GL_ATTRIBUTE_FALLTHROUGH ((void) 0)
#endif
...
```
We can replicate this error with the following simple example C program:
```c
#include <stdio.h>

# define _GL_ATTRIBUTE_FALLTHROUGH __attribute__ ((__fallthrough__))

int main () {
    switch (0) {
    case 0:
      _GL_ATTRIBUTE_FALLTHROUGH;
    default:
      return 0;
    }
    return 0;
}s
```
While `gcc` succeeded in make and run this program, `kcc` with `x86_64-linux-gcc-glibc-reverse-eval-order` profile failed and reported the following error:
```
test.c[8:0-0] : syntax error
Translation failed (kcc_config dumped). To repeat, run this command in directory test:
kcc -d test.c -o ktest
```
All other profiles report the same error for this test program.
