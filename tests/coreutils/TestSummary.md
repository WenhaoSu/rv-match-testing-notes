## Test Summary

Below is a summarization of projects built. Here italics means also reporting undefined behaviors:
* Commands that are able to compile, could execute and print the result, with almost no error except `convert_byte_to_native` error:
  * nice
* Commands that are able to compile, could execute and print the result, but reported `convert_byte_to_native` error message when ending the program:
  * **cat**, false, pwd, true, whoami, yes , **expr**, logname ,stat, pwd, hostid, printenv, tty, env, getlimits, nproc
* Commands that are able to compile, but failed to execute and reported `convert_byte_to_native` error message:
  * date, echo, **wc**, uname, **sort** , mkdir ,fold ,echo ,printf ,sum ,uname, factor, fold, groups, id, mkfifo, pathchk,     pr, uniq, users, yes, chcon, chgrp, cksum, comm, nl, readlink, stdbuf, sync, tee, touch, tr, truncate, basename
* Commands that are able to compile, but reported Undefined Behavior when executing:
  * ls, **head**, seq, cp, dd, df, dircolors, du, ln, md5sum, realpath, tail

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
