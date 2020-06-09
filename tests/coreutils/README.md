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
However for `gcc`, it succeeded in build and is able to pass most of test cases.

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
}
```
While `gcc` succeeded in make and run this program, `kcc` with `x86_64-linux-gcc-glibc-reverse-eval-order` profile failed and reported the following error:
```
test.c[8:0-0] : syntax error
Translation failed (kcc_config dumped). To repeat, run this command in directory test:
kcc -d test.c -o ktest
```
All other profiles report the same error for this test program.

### Previous Version

#### Coreutils-5.96, 5.97, 7.1, 8.1:
`gcc` failed to run `make`, skip those versions.

#### Coreutils-8.24:
`gcc` successed in running `./configure`, `make` and `make check`. However, `kcc` with profile `x86_64-linux-gcc-glibc` and `x86_64-linux-gcc-glibc-reverse-eval-order` failed in the `make` stage with the following error message:
```
lib/isnan.c:147:3: error: Division by 0.

    Undefined behavior (UB-CEMX1):
        see C11 section 6.5.5:5 http://rvdoc.org/C11/6.5.5
        see C11 section J.2:1 item 45 http://rvdoc.org/C11/J.2
        see CERT-C section INT33-C http://rvdoc.org/CERT-C/INT33-C
        see MISRA-C section 8.1:3 http://rvdoc.org/MISRA-C/8.1

lib/isnan.c:147:3: error: Non-constant static initializer.

    Constraint violation (CV-TSE3):
        see C11 section 6.7.9:4 http://rvdoc.org/C11/6.7.9
        see CERT-C section MSC40-C http://rvdoc.org/CERT-C/MSC40-C
        see MISRA-C section 8.1:1 http://rvdoc.org/MISRA-C/8.1

Translation failed (kcc_config dumped). To repeat, run this command in directory coreutils-8.24-kcc:
kcc -d -I. -I./lib -Ilib -I./lib -Isrc -I./src -g -MT lib/isnanf.o -MD -MP -MF lib/.deps/isnanf.Tpo -c -o lib/isnanf.o lib/isnanf.c
```
and here is the code `kcc` is complaining:
```c
...
#  if defined __SUNPRO_C || defined __ICC || defined _MSC_VER \
      || defined __DECC || defined __TINYC__ \
      || (defined __sgi && !defined __GNUC__)
  /* The Sun C 5.0, Intel ICC 10.0, Microsoft Visual C/C++ 9.0, Compaq (ex-DEC)
     6.4, and TinyCC compilers don't recognize the initializers as constant
     expressions.  The Compaq compiler also fails when constant-folding
     0.0 / 0.0 even when constant-folding is not required.  The Microsoft
     Visual C/C++ compiler also fails when constant-folding 1.0 / 0.0 even
     when constant-folding is not required. The SGI MIPSpro C compiler
     complains about "floating-point operation result is out of range".  */
  static DOUBLE zero = L_(0.0);
  memory_double nan;
  DOUBLE plus_inf = L_(1.0) / zero;
  DOUBLE minus_inf = -L_(1.0) / zero;
  nan.value = zero / zero;
#  else
  static memory_double nan = { L_(0.0) / L_(0.0) };       // Line 147
  static DOUBLE plus_inf = L_(1.0) / L_(0.0);
  static DOUBLE minus_inf = -L_(1.0) / L_(0.0);
#  endif
...
```
We can replicate this error with the following simple example C program:
```c
#include <stdio.h>

#define NWORDS \
  ((sizeof (double) + sizeof (unsigned int) - 1) / sizeof (unsigned int))

typedef union { double value; unsigned int word[NWORDS]; } memory_double;

int main () {
    static memory_double nan = { 0.0 / 0.0 };
    return 0;
}
```
While `gcc` succeeded in make and run this program, `kcc` with all profiles failed and reported the following error:
```
test.c:9:5: error: Division by 0.

    Undefined behavior (UB-CEMX1):
        see C11 section 6.5.5:5 http://rvdoc.org/C11/6.5.5
        see C11 section J.2:1 item 45 http://rvdoc.org/C11/J.2
        see CERT-C section INT33-C http://rvdoc.org/CERT-C/INT33-C
        see MISRA-C section 8.1:3 http://rvdoc.org/MISRA-C/8.1

test.c:9:5: error: Non-constant static initializer.

    Constraint violation (CV-TSE3):
        see C11 section 6.7.9:4 http://rvdoc.org/C11/6.7.9
        see CERT-C section MSC40-C http://rvdoc.org/CERT-C/MSC40-C
        see MISRA-C section 8.1:1 http://rvdoc.org/MISRA-C/8.1

Translation failed (kcc_config dumped). To repeat, run this command in directory test:
kcc -d test.c -o ktest
```
We can try to eliminate this error by change the code in `lib/isnan.c` to be:
```c
...
#  if defined __SUNPRO_C || defined __ICC || defined _MSC_VER \
      || defined __DECC || defined __TINYC__ \
      || (defined __sgi && !defined __GNUC__)
  /* The Sun C 5.0, Intel ICC 10.0, Microsoft Visual C/C++ 9.0, Compaq (ex-DEC)
     6.4, and TinyCC compilers don't recognize the initializers as constant
     expressions.  The Compaq compiler also fails when constant-folding
     0.0 / 0.0 even when constant-folding is not required.  The Microsoft
     Visual C/C++ compiler also fails when constant-folding 1.0 / 0.0 even
     when constant-folding is not required. The SGI MIPSpro C compiler
     complains about "floating-point operation result is out of range".  */
  static DOUBLE zero = L_(0.0);
  memory_double nan;
  DOUBLE plus_inf = 1.79769e+308;
  DOUBLE minus_inf = -1.79769e+308;
  nan.value = 1.79769e+308;
#  else
  static memory_double nan = { 1.79769e+308 };
  static DOUBLE plus_inf = 1.79769e+308;
  static DOUBLE minus_inf = -1.79769e+308;
#  endif
...
```
After making this change, `gcc` is still able to build and run `make check`, but is passing fewer cases as expected. However, `kcc` reported a different error:
```
lib/mountlist.c[526:0-26] : syntax error
Translation failed (kcc_config dumped). To repeat, run this command in directory coreutils-8.24-kcc:
kcc -d -I. -I./lib -Ilib -I./lib -Isrc -I./src -g -MT lib/mountlist.o -MD -MP -MF lib/.deps/mountlist.Tpo -c -o lib/mountlist.o lib/mountlist.c
Makefile:8827: recipe for target 'lib/mountlist.o' failed
```
Here is the code where the error appears in `lib/mountlist.c`:
```c
...
            me = xmalloc (sizeof *me);

            me->me_devname = xstrdup (dash + source_s);
            me->me_mountdir = xstrdup (line + target_s);
            me->me_type = xstrdup (dash + type_s);
            me->me_type_malloced = 1;
            me->me_dev = makedev (devmaj, devmin);                       // Line 526
            /* we pass "false" for the "Bind" option as that's only
               significant when the Fs_type is "none" which will not be
               the case when parsing "/proc/self/mountinfo", and only
               applies for static /etc/mtab files.  */
            me->me_dummy = ME_DUMMY (me->me_devname, me->me_type, false);
            me->me_remote = ME_REMOTE (me->me_devname, me->me_type);
...
```
Here `makedev` is a function defined in `sysmacros.h`, which is part of the GNU C Library.
```c
...
#ifdef __SYSMACROS_DEPRECATED_INCLUSION
# define major(dev) __SYSMACROS_DM (major) gnu_dev_major (dev)
# define minor(dev) __SYSMACROS_DM (minor) gnu_dev_minor (dev)
# define makedev(maj, min) __SYSMACROS_DM (makedev) gnu_dev_makedev (maj, min)
#else
# define major(dev) gnu_dev_major (dev)
# define minor(dev) gnu_dev_minor (dev)
# define makedev(maj, min) gnu_dev_makedev (maj, min)
#endif
...
```
while `devmaj` and `devmin` are supposed to be two `unsigned int` variables.

---
Observation: we can build coreutil functions seperately by using the following command:
```
./configure
cd ./lib
make
cd ../src
make version.h
make cat
make ls
```
It succeeded for coreutils-8.19 but may not work in versions after coreutils-8.23. So we choose to run tests on coreutils-8.19 for next stage.

#### Coreutils-8.19
When running `make` in `./lib`, `kcc` would also encounter the problem of division by 0. By using the solution in the last section, `kcc` succeeded in running `make` in `./lib`!

When making all individual programs in `./src`, kcc would reported a great amount of the following errors:
```
...
timespec.h:81:1: error: Multiple external definitions for timespectod.

    Undefined behavior (UB-TIN2):
        see C11 section 6.9:5 http://rvdoc.org/C11/6.9
        see C11 section J.2:1 item 84 http://rvdoc.org/C11/J.2
        see CERT-C section MSC15-C http://rvdoc.org/CERT-C/MSC15-C
        see MISRA-C section 8.5:1 http://rvdoc.org/MISRA-C/8.5
        see MISRA-C section 8.8:6 http://rvdoc.org/MISRA-C/8.8
        see MISRA-C section 8.1:3 http://rvdoc.org/MISRA-C/8.1
...
```
but it would also succeeded in generating the corresponding execution file.

Also, for all commands, runing `--help` would first print out usage normally, then `kcc` would report the following `convert_byte_to_native` error message:
```
Fatal error: exception (Invalid_argument
  "convert_byte_to_native: encodedValue(opaque(#token(\"1\", \"Int\"), ut(`.Set`(.KList), structType(tag(`Identifier`(#token(\"\\\"_IO_FILE\\\"\", \"String\")), #token(\"\\\"/opt/rv-match/c-semantics/profiles/x86_64-linux-gcc-glibc/src/kcc_types.c55777c10-8fd5-11ea-9420-aad0e96ca077\\\"\", \"String\"), `global_C-TYPING-SYNTAX`(.KList))))), #token(\"0\", \"Int\"), #token(\"8\", \"Int\"))")
```
---
### Test Summary
Below is a summarization of projects built. Here italics means also reporting undefined behaviors:
* Commands that are able to compile, could execute and print the result, with almost no error except `convert_byte_to_native` error:
  * nice
* Commands that are able to compile, could execute and print the result, but reported `convert_byte_to_native` error message when ending the program:
  * **cat**, false, pwd, true, whoami, yes , **expr**, logname ,stat, pwd, hostid, printenv, tty
* Commands that are able to compile, but failed to execute and reported `convert_byte_to_native` error message:
  * date, echo, **wc**, uname, **sort** , mkdir ,fold ,echo ,printf ,sum ,uname, factor, fold, groups, id, mkfifo, pathchk,     pr, uniq, users, yes, chcon, chgrp, cksum, comm, 
* Commands that are able to compile, but reported Undefined Behavior when executing:
  * ls, **head**, seq, cp, dd, df, dircolors

Below are detailed report for several commands:

### Building cat

When runing `./cat ../AUTHORS` for `kcc` generated executable file, it reported
```
Modulus operator with a pointer value as an argument:
      > in ptr_align at system.h:496:3
        in main at cat.c:730:11

    Implementation defined behavior (IMPL-CEMX4):
        see C11 section 6.3.2.3:6 http://rvdoc.org/C11/6.3.2.3
        see C11 section J.3.7:1 item 1 http://rvdoc.org/C11/J.3.7
        see MISRA-C section 8.11:4 http://rvdoc.org/MISRA-C/8.11
```
then it succeeded in printing out the contents in the file `../AUTHORS`. However after finish printing the contents and is about to end the program, kcc reports `convert_byte_to_native` error message.

Here is the code for line 496 in `system.h`:

```c
static inline void *
ptr_align (void const *ptr, size_t alignment)
{
  char const *p0 = ptr;
  char const *p1 = p0 + alignment - 1;
  return (void *) (p1 - (size_t) p1 % alignment);         // Line 496
}
```
It seems that `kcc` succeeded in reporting an undefined behavior / nonstandard operation here.

### Building ls

When runing `./ls` for `kcc` generated executable file, it reported
```
An object which has been modified is accessed through an expression based on a restrict-qualified pointer and another lvalue not also based on said pointer:
      > in get_quoting_style at quotearg.c:121:3
        in decode_switches at ls.c:1976:3
        in main at ls.c:1287:3

    Undefined behavior (UB-ECL3):
        see C11 section 6.7.3.1:4 http://rvdoc.org/C11/6.7.3.1
        see C11 section J.2:1 item 68 http://rvdoc.org/C11/J.2
        see CERT-C section EXP43-C http://rvdoc.org/CERT-C/EXP43-C
        see MISRA-C section 8.1:3 http://rvdoc.org/MISRA-C/8.1

An object which has been modified is accessed through an expression based on a restrict-qualified pointer and another lvalue not also based on said pointer:
      > in set_char_quoting at quotearg.c:145:3
        in decode_switches at ls.c:1986:3
        in main at ls.c:1287:3

    Undefined behavior (UB-ECL3):
        see C11 section 6.7.3.1:4 http://rvdoc.org/C11/6.7.3.1
        see C11 section J.2:1 item 68 http://rvdoc.org/C11/J.2
        see CERT-C section EXP43-C http://rvdoc.org/CERT-C/EXP43-C
        see MISRA-C section 8.1:3 http://rvdoc.org/MISRA-C/8.1

Encountered an unknown error. This may be due to encountering undefined behavior, an unsupported language feature, or a bug in this tool:
      > in memset at ls.c:2883:3
        in gobble_file at ls.c:2883:3
        in print_dir at ls.c:2593:15
        in main at ls.c:1444:7

    Unknown error (UNK-1)

Conversion from an integer to non-null pointer:
      > in memset at ls.c:2883:3
        in gobble_file at ls.c:2883:3
        in print_dir at ls.c:2593:15
        in main at ls.c:1444:7

    Implementation defined behavior (IMPL-CCV13):
        see C11 section 6.3.2.3:5 http://rvdoc.org/C11/6.3.2.3
        see CERT section INT36-C http://rvdoc.org/CERT/INT36-C

Fatal error: exception (Invalid_argument "mismatched constructor at top of split configuration")
```

Here is line 2883 in `ls.c`:

```c
...
  f = &cwd_file[cwd_n_used];
  memset (f, '\0', sizeof *f);                // Line 2883
  f->stat.st_ino = inode;
  f->filetype = type;
...
```
Although we can reproduce the same error message of `Unknown error (UNK-1)` by writing a simple c program as following:
```c
#include <stdio.h>
#include <string.h>

struct fileinfo {
    int name;
  };

static struct fileinfo *cwd_file ;

int main () {
    struct fileinfo *f;
    f = &cwd_file[0];
    memset (f, '\0', sizeof *f);
    return 0;
}
```
but `gcc` will also report `Segmentation fault (core dumped)` in the above program. Also, the above program will not report `Invalid_argument` fatal error.

The last error message (`Invalid_argument`) here is same to what we got when testing [tcpdump](https://github.com/WenhaoSu/rv-match-testing-notes/blob/master/tests/tcpdump/README.md).

### Building wc

When runing `./wc touch.c` for `kcc` generated executable file, it reported
```
Type of lvalue (const unsigned int) not compatible with the effective type of the object being accessed (const unsigned int []):
      > in is_basic at ../lib/mbchar.h:310:3
        in wc at wc.c:325:15
        in wc_file at wc.c:515:10
        in main at wc.c:776:9

    Undefined behavior (UB-EIO10):
        see C11 section 6.5:7 http://rvdoc.org/C11/6.5
        see C11 section J.2:1 item 37 http://rvdoc.org/C11/J.2
        see CERT-C section EXP39-C http://rvdoc.org/CERT-C/EXP39-C
        see MISRA-C section 8.1:3 http://rvdoc.org/MISRA-C/8.1


Fatal error: exception (Invalid_argument
  "convert_byte_to_native: encodedValue(opaque(#token(\"1\", \"Int\"), ut(`.Set`(.KList), structType(tag(`Identifier`(#token(\"\\\"_IO_FILE\\\"\", \"String\")), #token(\"\\\"/opt/rv-match/c-semantics/profiles/x86_64-linux-gcc-glibc/src/kcc_types.c55777c10-8fd5-11ea-9420-aad0e96ca077\\\"\", \"String\"), `global_C-TYPING-SYNTAX`(.KList))))), #token(\"0\", \"Int\"), #token(\"8\", \"Int\"))")
```

Line 310 in `/lib/mbchar.h` is:
```c
...
extern const unsigned int is_basic_table[];

static inline bool
is_basic (char c)
{
  return (is_basic_table [(unsigned char) c >> 5] >> ((unsigned char) c & 31))            // Line 310
         & 1;
}
...
```

However, the following program *failed* to provide the same error:
```c
#include <stdio.h>
#include <stdbool.h>

const unsigned int is_basic_table[] = {1, 2, 3, 1, 2, 3, 1, 2, 3, 1, 2, 3};

static inline bool is_basic (char c) {
  return (is_basic_table [(unsigned char) c >> 5] >> ((unsigned char) c & 31)) & 1;
}

int main () {
    char c = 10;
    bool test = is_basic (c);
    return 0;
}
```
Another observation is that if we use the latest release of `kcc` on [runtimeverification/match](https://github.com/runtimeverification/match), this undefined behavior would then not be reported. Hence, I believe it is fine to ignore this undefined behavior for now.

### Building nice

When runing `./nice` followed directly by other commands, `kcc` compiled version succeeded and didn't report any error.
Run `./nice -n 19 whoami` would give:
```
./nice: invalid adjustment ‘19’
```
while `gcc` compiled version would not have this error.

### Building true & false

When no command line arguments are provided, it would not report any error. However, when run `./true ls`, it would report `convert_byte_to_native` error message.
The same holds for `./false`.

### Building sort
 When running './sort' for 'kcc' generated executable file it reported
 ```
 Conversion to signed integer outside the range that can be represented:
      > in inittables at sort.c:1251:7
        in main at sort.c:4183:3

    Implementation defined behavior (IMPL-CCV2):
        see C11 section 6.3.1.3:3 http://rvdoc.org/C11/6.3.1.3
        see C11 section J.3.5:1 item 4 http://rvdoc.org/C11/J.3.5
        see CERT-C section INT31-C http://rvdoc.org/CERT-C/INT31-C

Fatal error: exception (Invalid_argument
  "convert_byte_to_native: encodedValue(opaque(#token(\"0\", \"Int\"), ut(`.Set`(.KList), structType(tag(`Identifier`(#token(\"\\\"_IO_FILE\\\"\", \"String\")), #token(\"\\\"kcc_types.c15e7b5db-8efa-11ea-9073-acb87a5092db\\\"\", \"String\"), `global_C-TYPING-SYNTAX`(.KList))))), #token(\"0\", \"Int\"), #token(\"8\", \"Int\"))")

 ```

Line 1251 in `sort.c` is as following:
```c
...
  for (i = 0; i < UCHAR_LIM; ++i)
    {
      blanks[i] = !! isblank (i);
      nonprinting[i] = ! isprint (i);
      nondictionary[i] = ! isalnum (i) && ! isblank (i);
      fold_toupper[i] = toupper (i);                        //Line 1251
    }
...
```
We can write a simple program to replicate this message:
```c
#include <stdio.h>
#include <limits.h>
#include <ctype.h>

#define UCHAR_LIM ((SCHAR_MAX * 2 + 1) + 1)

int main () {
    static char fold_toupper[UCHAR_LIM];
    size_t i;

    for (i = 0; i < UCHAR_LIM; ++i)
        fold_toupper[i] = toupper (i);

    printf("Reached this point\n");
    return 0;
}
```
`kcc` succeeded in finding possible undefined behaviors for this program and reported it, then it finished the program correctly:
```
Conversion to signed integer outside the range that can be represented:
      > in main at test.c:12:9

    Implementation defined behavior (IMPL-CCV2):
        see C11 section 6.3.1.3:3 http://rvdoc.org/C11/6.3.1.3
        see C11 section J.3.5:1 item 4 http://rvdoc.org/C11/J.3.5
        see CERT-C section INT31-C http://rvdoc.org/CERT-C/INT31-C

Reached this point
```
---
Observation on `convert_byte_to_native` error:
As far as I've discovered, there are mainly two sources that can cause this error. Take `true.c` as example, if we change main function to the following (use funtion `fclose` on `stderr`):
```c
int main (int argc, char **argv) {
  printf ("here\n");
  fclose (stderr);
  printf ("there\n");
}
```
or (use funtion `fputs` on `stdout`):
```c
int main (int argc, char **argv) {
  printf ("here\n");
  fputs (VERSION_OPTION_DESCRIPTION, stdout);
  printf ("there\n");
}
```

and then type `make true; ./true --help`, they would both print
```
here
Fatal error: exception (Invalid_argument
  "convert_byte_to_native: encodedValue(opaque(#token(\"2\", \"Int\"), ut(`.Set`(.KList), structType(tag(`Identifier`(#token(\"\\\"_IO_FILE\\\"\", \"String\")), #token(\"\\\"/opt/rv-match/c-semantics/profiles/x86_64-linux-gcc-glibc/src/kcc_types.c55777c10-8fd5-11ea-9420-aad0e96ca077\\\"\", \"String\"), `global_C-TYPING-SYNTAX`(.KList))))), #token(\"0\", \"Int\"), #token(\"8\", \"Int\"))")
```
However, if we just create the same simple program in a different folder and run it with `kcc test.c -o ktest; ./ktest`, everything goes normal and error message disappeared.

This is probably due to the fact that `true.c` is using `stderr`, `stdout`,`fclose` and `fputs` defined in `lib/stdio.h` inside `coreutils-8.19` instead of system library, and it is also using a quite complicated `Makefile` which could potentially adds up more complexity of the building process. For the same simple program which does not display the error message, it is using the `<stdio.h>` in system library. This may imply that the codeblock which causes this error to be displayed is located in the `lib` files written by coreutils.



### Building paste

When running ./paste after compiling it from kcc we get
```
An object which has been modified is accessed through an expression based on a restrict-qualified pointer and another lvalue not also based on said pointer:
      > in collapse_escapes at paste.c:99:9
        in main at paste.c:503:3

    Undefined behavior (UB-ECL3):
        see C11 section 6.7.3.1:4 http://rvdoc.org/C11/6.7.3.1
        see C11 section J.2:1 item 68 http://rvdoc.org/C11/J.2
        see CERT-C section EXP43-C http://rvdoc.org/CERT-C/EXP43-C
        see MISRA-C section 8.1:3 http://rvdoc.org/MISRA-C/8.1

Fatal error: exception (Invalid_argument
  "convert_byte_to_native: encodedValue(opaque(#token(\"0\", \"Int\"), ut(`.Set`(.KList), structType(tag(`Identifier`(#token(\"\\\"_IO_FILE\\\"\", \"String\")), #token(\"\\\"kcc_types.c15e7b5db-8efa-11ea-9073-acb87a5092db\\\"\", \"String\"), `global_C-TYPING-SYNTAX`(.KList))))), #token(\"0\", \"Int\"), #token(\"8\", \"Int\"))")

```
The specific part showing the undefined behavior is the following:-
```c

static int
collapse_escapes (char const *strptr)
{
  char *strout = xstrdup (strptr);
  bool backslash_at_end = false;

  delims = strout;

  while (*strptr)
    {
      if (*strptr != '\\')	/* Is it an escape character? */
        *strout++ = *strptr++;	/* No, just transfer it. */ (THIS LINE SHOWS ERROR)
        
```
xstrdup is a function that is called from one of the header files. And it is difficult to find the same error through the system files

### Building head

While using './head' after compiling with kcc we get the following undefined behaviour
```
Type of lvalue (const char * const) not compatible with the effective type of the object being accessed (char * [3]):
      > in main at head.c:1063:3

    Undefined behavior (UB-EIO10):
        see C11 section 6.5:7 http://rvdoc.org/C11/6.5
        see C11 section J.2:1 item 37 http://rvdoc.org/C11/J.2
        see CERT-C section EXP39-C http://rvdoc.org/CERT-C/EXP39-C
        see MISRA-C section 8.1:3 http://rvdoc.org/MISRA-C/8.1

```
The part which shows the undefined behaviour in head.c is
```c
 static char const *const default_file_list[] = {"-", NULL};
  char const *const *file_list;
  file_list = (optind < argc
               ? (char const *const *) &argv[optind]
               : default_file_list);
    for (i = 0; file_list[i]; ++i)
    ok &= head_file (file_list[i], n_units, count_lines, elide_from_end);    
```
The following program gives the same error thus helping to indentify the problem here
```c
#include <stdio.h>

int main (int argc, char **argv){
 char const *const *file_list={"-", NULL};
 int i=0;
 for (i = 0; file_list[i]; ++i){printf("%s\n","lol" );}
}
```

Hence kcc may be working correctly and showing us the correct undefined behaviour.

