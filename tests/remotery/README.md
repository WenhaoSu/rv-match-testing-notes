# remotery

The source code of `remotery` can be found [here](https://github.com/Celtoys/Remotery).

### Summary

The potential issue is that `x86_64-linux-gcc-glibc` profile have trouble supporting `math.h`, while the code block in `math.h` that checks the verison of `gcc` throwed an error message and caused `kcc_config dumped`.

The issue page of this problem is [here](https://github.com/runtimeverification/c-semantics/issues/6).

This issue is closed since it would no longer report `Translation failed`, but only report `Warning: Encountered _Complex type.` on the latest version of kcc.

### Pre-request
```
git clone https://github.com/Celtoys/Remotery.git
cd Remotery/
```

### Building with gcc
```
gcc -std=gnu11 lib/Remotery.c sample/sample.c -I lib -pthread -lm
````    


### Building with kcc
```
kcc -profile x86_64-linux-gcc-glibc-gnuc
kcc -std=gnu11 lib/Remotery.c sample/sample.c -I lib -pthread -lm
kcc -profile x86_64-linux-gcc-glibc
```

### Observation-Build

Build was successful in gcc case. In kcc it found some errors which may have resulted in failure of producing an executable files.
```
lib/Remotery.c:454:9: error: Trying to look up identifier __sync_bool_compare_and_swap, but no such identifier is in scope.

    Syntax error (SE-CID1):
        see C11 section 6.5.1:2 http://rvdoc.org/C11/6.5.1
        see CERT-C section DCL31-C http://rvdoc.org/CERT-C/DCL31-C

lib/Remotery.c:468:9: error: Trying to look up identifier __sync_bool_compare_and_swap, but no such identifier is in scope.

    Syntax error (SE-CID1):
        see C11 section 6.5.1:2 http://rvdoc.org/C11/6.5.1
        see CERT-C section DCL31-C http://rvdoc.org/CERT-C/DCL31-C

lib/Remotery.c:483:9: error: Trying to look up identifier __sync_fetch_and_add, but no such identifier is in scope.

    Syntax error (SE-CID1):
        see C11 section 6.5.1:2 http://rvdoc.org/C11/6.5.1
        see CERT-C section DCL31-C http://rvdoc.org/CERT-C/DCL31-C

lib/Remotery.c:957:5: warning: Conversion from an integer to non-null pointer.

    Implementation defined behavior (IMPL-CCV13):
        see C11 section 6.3.2.3:5 http://rvdoc.org/C11/6.3.2.3
        see CERT section INT36-C http://rvdoc.org/CERT/INT36-C

lib/Remotery.c:3494:1: error: Enum initializer outside the range representable by int.

    Constraint violation (CV-CDE2):
        see C11 section 6.7.2.2:2 http://rvdoc.org/C11/6.7.2.2
        see CERT-C section MSC40-C http://rvdoc.org/CERT-C/MSC40-C
        see MISRA-C section 8.1:1 http://rvdoc.org/MISRA-C/8.1

lib/Remotery.c:3494:1: warning: Conversion to signed integer outside the range that can be represented.

    Implementation defined behavior (IMPL-CCV2):
        see C11 section 6.3.1.3:3 http://rvdoc.org/C11/6.3.1.3
        see C11 section J.3.5:1 item 4 http://rvdoc.org/C11/J.3.5
        see CERT-C section INT31-C http://rvdoc.org/CERT-C/INT31-C

lib/Remotery.c:3500:25: warning: ISO C restricts enumerator values to range of ‘int’ [-Wpedantic]
     MsgID_Force32Bits = 0xFFFFFFFF,
                         ^~~~~~~~~~
In file included from sample/sample.c:2:0:
/usr/include/math.h:542:5: error: #error "Non-typedef _FloatN but no _Generic."
 #   error "Non-typedef _FloatN but no _Generic."
     ^~~~~
Translation failed (kcc_config dumped). To repeat, run this command in directory Remotery:
kcc -d -std=gnu11 lib/Remotery.c sample/sample.c -I lib -pthread -lm
```

### Observation- Testing

We can test the executable file (`a.out`) produced by `gcc` by typing `./a.out` and open the `index.html` in `Remotery/vis`. The result would be a frontend displaying the realtime CPU/GPU profile.

However, `kcc` falied to produce the executable file.

### Followup

If we use other profiles for testing, then `kcc` will fail on `syntax error`. After testing, we found that only `x86_64-linux-gcc-glibc-gnuc` and `x86_64-linux-gcc-glibc-gnuc-reverse-eval-order` profiles can provide the error shown above, while others fail to parse the synatx at the beginning.

---
Observation: the error message seems to come from line 542 of `math.h`:
```c
...
#  if __HAVE_FLOATN_NOT_TYPEDEF
#   error "Non-typedef _FloatN but no _Generic."
#  endif
...
```
where `__HAVE_FLOATN_NOT_TYPEDEF` comes from line 62-66 in `floatn-common.h`:
```c
...
#if __GNUC_PREREQ (7, 0) && !defined __cplusplus
# define __HAVE_FLOATN_NOT_TYPEDEF 1
#else
# define __HAVE_FLOATN_NOT_TYPEDEF 0
#endif
...
```
here `__GNUC_PREREQ` is a macro defined as following in `features.h`:
```c
...
/* Convenience macro to test the version of gcc.
   Use like this:
   #if __GNUC_PREREQ (2,8)
   ... code requiring gcc 2.8 or later ...
   #endif
   Note: only works for GCC 2.0 and later, because __GNUC_MINOR__ was
   added in 2.0.  */
#if defined __GNUC__ && defined __GNUC_MINOR__
# define __GNUC_PREREQ(maj, min) \
	((__GNUC__ << 16) + __GNUC_MINOR__ >= ((maj) << 16) + (min))
#else
# define __GNUC_PREREQ(maj, min) 0
#endif
...
```

---
Observation: I wrote a simple c program and tried to compile it using `kcc`:
```c
#include <math.h>

int main (int argc, char ** argv) {
    return 0;
}
```
If we are using `x86_64-linux-gcc-glibc` profile, it would success, but if we are using `x86_64-linux-gcc-glibc-gnuc` profile, it would also give the following error which is exactly the same as remotery:
```
In file included from test2.c:1:0:
/usr/include/math.h:542:5: error: #error "Non-typedef _FloatN but no _Generic."
 #   error "Non-typedef _FloatN but no _Generic."
     ^~~~~
Translation failed (kcc_config dumped). To repeat, run this command in directory test:
kcc -d test2.c -o test2
```

This may potentially imply that `x86_64-linux-gcc-glibc-gnuc` may not support `math.h`?
