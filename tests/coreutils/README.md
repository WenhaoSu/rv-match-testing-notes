# coreutils

The source code of `coreutils` can be found [here](https://github.com/coreutils/coreutils).

### Pre-request
```shell
wget https://ftp.gnu.org/gnu/coreutils/coreutils-8.19.tar.xz
tar -xf coreutils-8.19.tar.xz
rm coreutils-8.19.tar.xz
cd coreutils-8.19/
```

### Build
To build with `gcc`, type:
```shell
./configure --disable-gcc-warnings
make
```
To build with `kcc`, type:
```shell
./configure CC=kcc LD=kcc --disable-gcc-warnings
make
```
Check following sections on how to build and test a specific command.

### Note

To run `./configure` and `make` really take time for both `gcc` and `kcc`. To test each command individually could save time.

In order to build each commands individually, we use **Coreutils-8.19** version, which can be found [here](https://ftp.gnu.org/gnu/coreutils/). [Here](https://unix.stackexchange.com/questions/50484/install-only-a-few-gnu-coreutils/277272) is the instructions for building different commands individually.

#### Coreutils-8.19:
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
After getting rid of the `division by 0` error, we can build coreutil commands seperately by using the following command:
```shell
./configure CC=kcc LD=kcc --disable-gcc-warnings
cd ./lib
make
cd ../src
make version.h
make cat
make ls
```

When making all individual programs in `./src`, kcc would reported a great amount of the following `Multiple external definitions` errors:
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

Visit [here](https://github.com/WenhaoSu/rv-match-testing-notes/blob/master/tests/coreutils/TestSummary.md) for detailed testing results on different coreutil commands.

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

---
Observation on `convert_byte_to_native` error:
If we comment the line `atexit (close_stdout);` in `true.c` and the line `#include "unlocked-io.h"` in `system.h`, then by typing `make true`, we will get an executive file which can execute normally and will not report the `convert_byte_to_native` error.

Following commands are verified that can be executed correctly after commenting the `atexit (close_stdout);` and  `#include "unlocked-io.h"` line:

* `basename`, `cat`, `true`, `false`, `date`, `whoami`, `wc`, `mkdir`, `dirname`, `cp`, `env`, `pwd`, `uname`, `users` , `tty`, `touch` , `timeout` , `test`, `echo` , `expr`, `head`, `tail`, `printf`, `hostid`, `hostname`, `id` ,`logname`,`dircolors` ,`expand` ,`factor` ,`fold` ,`groups` , `ln` , `chmod`, `chown`, `comm` , `nproc` , `pathchk` , `stat` , `sleep` , `rmdir`, `split`, `who`, `yes`, `pinky`, `chgrp`, `nice`, `chcon`, `base64`, `chroot`, `printenv`, `csplit`, `link` , `unlink` , `readlink` , `truncate` , `sync` , `runcon` , `nohup` , `mkfifo` , `mv`

* `rm`: succeed in removing files, but will report quite a lot undefined behaviors

Following commands still failed:

* `ls`: reports `Fatal error: exception (Invalid_argument "mismatched constructor at top of split configuration")`

* `shuf`,`sort`, `cut`, `cksum`: succeeded in running `--help`, but still reports `convert_byte_to_native` error while running
* `unexpand` , `mktemp` , `join` , `seq`: succeeded in running and printing/executing the right result but still reports convert_byte_to_native` error while running

Untested commands:

* `dd` `df` `dir` `du` `fmt` `install` `join` `kill` `link` `mkfifo` `mknod` `mktemp` `mv` `nl` `nohup` `od` `paste` `pr` `ptx` `readlink` `realpath` `runcon` `seq` `shred` `stdbuf` `stty` `sum` `sync` `tac` `tee` `tr` `truncate` `tsort` `unexpand` `uniq` `unlink` `uptime` `vdir`
