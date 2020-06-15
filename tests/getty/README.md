# getty

The source code of `getty` can be found [here](https://github.com/StarchLinux/getty).

### Summary

The potential issue is that `kcc` does not recognize `#define __attribute_nonstring__`.

The issue page of this problem is [here](https://github.com/runtimeverification/c-semantics/issues/5).

This issue is due to the out-dated `cdefs.h` inside `rv-match`, so it can be fixed by updating the `cdefs.h` file in `rv-match/c-semantics-plugin/src/main/c-semantics/profiles/x86_64-linux-gcc-glibc/include/library/sys/cdefs.h` to the latest version, which can be found [here](https://code.woboq.org/qt5/include/sys/cdefs.h.html).

### Pre-request
```
git clone https://github.com/StarchLinux/getty.git
cd getty/
git checkout a6c24ec43804588a3c35c9ed3325ed3086ddd056
```

### Build
To build with `gcc`, simply run `make`. To build with `kcc`, update `Makefile` to remove `-std=c99`, then type:
```
sed -i "/strip/d" Makefile
CC=kcc LD=kcc make
```

### Test
```
usage: getty 1
       getty vc/1
       getty /dev/tty1
```


### Observation - Build

`kcc` failed in the `make` job.

When running `make` with `gcc`, we get several warning messages on `[-Wunused-result]`, but the compile stage can finish.
When running `make` with `kcc`, we get the following error message:

```
kcc -pipe -Os -fomit-frame-pointer -c getty.c
/usr/include/x86_64-linux-gnu/bits/utmp.h[63:0-0] : syntax error
Translation failed (kcc_config dumped). To repeat, run this command in directory getty:
kcc -d -pipe -Os -fomit-frame-pointer -c getty.c
Makefile:7: recipe for target 'getty.o' failed
make: *** [getty.o] Error 1
```

### Note

This project is pretty small, with only two `.c` files and a 20-line Makefile. Prehaps it's relatively feasible to dig into the project code and find out the potential reason.

### Followup

Observation: If we compile the following code with `kcc`:
```c
#include <stdio.h>
#include <utmp.h>

int main (int argc, char ** argv) {
    return 0;
}
```

We will also get the error message:
```
/usr/include/x86_64-linux-gnu/bits/utmp.h[63:0-0] : syntax error
Translation failed (kcc_config dumped). To repeat, run this command in directory test:
kcc -d test.c -o ktest
```

Below is the code for `/usr/include/x86_64-linux-gnu/bits/utmp.h`:
```c
/* The `struct utmp' type, describing entries in the utmp file.  GNU version.
   Copyright (C) 1993-2018 Free Software Foundation, Inc.
   This file is part of the GNU C Library.

   The GNU C Library is free software; you can redistribute it and/or
   modify it under the terms of the GNU Lesser General Public
   License as published by the Free Software Foundation; either
   version 2.1 of the License, or (at your option) any later version.

   The GNU C Library is distributed in the hope that it will be useful,
   but WITHOUT ANY WARRANTY; without even the implied warranty of
   MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
   Lesser General Public License for more details.

   You should have received a copy of the GNU Lesser General Public
   License along with the GNU C Library; if not, see
   <http://www.gnu.org/licenses/>.  */

#ifndef _UTMP_H
# error "Never include <bits/utmp.h> directly; use <utmp.h> instead."
#endif

#include <paths.h>
#include <sys/time.h>
#include <sys/types.h>
#include <bits/wordsize.h>


#define UT_LINESIZE	32
#define UT_NAMESIZE	32
#define UT_HOSTSIZE	256


/* The structure describing an entry in the database of
   previous logins.  */
struct lastlog
  {
#if __WORDSIZE_TIME64_COMPAT32
    int32_t ll_time;
#else
    __time_t ll_time;
#endif
    char ll_line[UT_LINESIZE];
    char ll_host[UT_HOSTSIZE];
  };


/* The structure describing the status of a terminated process.  This
   type is used in `struct utmp' below.  */
struct exit_status
  {
    short int e_termination;	/* Process termination status.  */
    short int e_exit;		/* Process exit status.  */
  };


/* The structure describing an entry in the user accounting database.  */
struct utmp
{
  short int ut_type;		/* Type of login.  */
  pid_t ut_pid;			/* Process ID of login process.  */
  char ut_line[UT_LINESIZE]
    __attribute_nonstring__;	/* Devicename.  */
  char ut_id[4];		/* Inittab ID.  */
  char ut_user[UT_NAMESIZE]
    __attribute_nonstring__;	/* Username.  */
  char ut_host[UT_HOSTSIZE]
    __attribute_nonstring__;	/* Hostname for remote login.  */
  struct exit_status ut_exit;	/* Exit status of a process marked
				   as DEAD_PROCESS.  */
/* The ut_session and ut_tv fields must be the same size when compiled
   32- and 64-bit.  This allows data files and shared memory to be
   shared between 32- and 64-bit applications.  */
#if __WORDSIZE_TIME64_COMPAT32
  int32_t ut_session;		/* Session ID, used for windowing.  */
  struct
  {
    int32_t tv_sec;		/* Seconds.  */
    int32_t tv_usec;		/* Microseconds.  */
  } ut_tv;			/* Time entry was made.  */
#else
  long int ut_session;		/* Session ID, used for windowing.  */
  struct timeval ut_tv;		/* Time entry was made.  */
#endif

  int32_t ut_addr_v6[4];	/* Internet address of remote host.  */
  char __glibc_reserved[20];		/* Reserved for future use.  */
};

/* Backwards compatibility hacks.  */
#define ut_name		ut_user
#ifndef _NO_UT_TIME
/* We have a problem here: `ut_time' is also used otherwise.  Define
   _NO_UT_TIME if the compiler complains.  */
# define ut_time	ut_tv.tv_sec
#endif
#define ut_xtime	ut_tv.tv_sec
#define ut_addr		ut_addr_v6[0]


/* Values for the `ut_type' field of a `struct utmp'.  */
#define EMPTY		0	/* No valid user accounting information.  */

#define RUN_LVL		1	/* The system's runlevel.  */
#define BOOT_TIME	2	/* Time of system boot.  */
#define NEW_TIME	3	/* Time after system clock changed.  */
#define OLD_TIME	4	/* Time when system clock changed.  */

#define INIT_PROCESS	5	/* Process spawned by the init process.  */
#define LOGIN_PROCESS	6	/* Session leader of a logged in user.  */
#define USER_PROCESS	7	/* Normal process.  */
#define DEAD_PROCESS	8	/* Terminated process.  */

#define ACCOUNTING	9

/* Old Linux name for the EMPTY type.  */
#define UT_UNKNOWN	EMPTY


/* Tell the user that we have a modern system with UT_HOST, UT_PID,
   UT_TYPE, UT_ID and UT_TV fields.  */
#define _HAVE_UT_TYPE	1
#define _HAVE_UT_PID	1
#define _HAVE_UT_ID	1
#define _HAVE_UT_TV	1
#define _HAVE_UT_HOST	1
```
and here is line 62-64:
```c
...
  char ut_line[UT_LINESIZE]
    __attribute_nonstring__;	/* Devicename.  */
  char ut_id[4];		/* Inittab ID.  */
...
```
Below is how the macro `__attribute_nonstring__` is defined in `cdefs.h`:
```c
...
#if __GNUC_PREREQ (8, 0)
/* Describes a char array whose address can safely be passed as the first
   argument to strncpy and strncat, as the char array is not necessarily
   a NUL-terminated string.  */
# define __attribute_nonstring__ __attribute__ ((__nonstring__))
#else
# define __attribute_nonstring__
#endif
...
```
By further simplify the code, we have `kcc` will also give error message when compiling following code:
```c
#include <stdio.h>
#define UT_LINESIZE	32

int main (int argc, char ** argv) {
    char ut_line[UT_LINESIZE]
    __attribute_nonstring__;
    return 0;
}
```
the compile result is:
```
$ kcc test.c -o ktest
test.c[6:0-0] : syntax error
Translation failed (kcc_config dumped). To repeat, run this command in directory test:
kcc -d test.c -o ktest
```
However, `gcc` is able to compile the code above. This means that `kcc` does not recognize the syntax as described above.
By adding `#define __attribute_nonstring__` to the above program, then `kcc` will manage to compile the following code:
```c
#include <stdio.h>
#define UT_LINESIZE	32
#define __attribute_nonstring__

int main (int argc, char ** argv) {
    char ut_line[UT_LINESIZE]
    __attribute_nonstring__;
    return 0;
}
```
This means that unlike `gcc` who supports the `nonstring` attribute, `kcc` does not recognize it.