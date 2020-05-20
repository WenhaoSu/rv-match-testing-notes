# libuv

The source code of `libuv` can be found [here](https://github.com/libuv/libuv).

### Pre-request
```
git clone https://github.com/libuv/libuv.git
```


### Build
To run with kcc, change `./configure` to be `./configure CC=kcc LD=kcc`.
```
bash autogen.sh
./configure
make
```

### Test
```
make check
```

### Observation - Build

`kcc` succeeded in the `make` job.

When running `make` with `kcc`, following warning will be printed:
```
./include/uv.h:1194:40: warning: C++ style comments are not allowed in ISO C90
 # define UV_PRIORITY_LOW 39            // RUNPTY(99)
```
while `gcc` would not raise this warning.

`gcc` took `3m45.496s` to finish `make`, while `kcc` took `16m12.070s`.

### Observation - Testing

`kcc` failed `make check` in the stage `CCLD     test/run-tests`, with the error message `Translation failed. To repeat, run this command in directory libuv:...`. However, in the prior stages, `kcc` managed to print out several undefined behavior warning and errors. Here several interesting errors are listed:
```
  CC       test/test_run_tests-test-spawn.o
test/test-spawn.c:161:3: error: Dereferencing a pointer past the end of an array.

    Undefined behavior (UB-CER4):
        see C11 section 6.5.6:8 http://rvdoc.org/C11/6.5.6
        see C11 section J.2:1 items 47 and 49 http://rvdoc.org/C11/J.2
        see CERT-C section ARR30-C http://rvdoc.org/CERT-C/ARR30-C
        see CERT-C section ARR37-C http://rvdoc.org/CERT-C/ARR37-C
        see CERT-C section STR31-C http://rvdoc.org/CERT-C/STR31-C
        see MISRA-C section 8.18:1 http://rvdoc.org/MISRA-C/8.18
        see MISRA-C section 8.1:3 http://rvdoc.org/MISRA-C/8.1

test/test-spawn.c:1057:8: warning: Conversion from an integer to non-null pointer.

    Implementation defined behavior (IMPL-CCV13):
        see C11 section 6.3.2.3:5 http://rvdoc.org/C11/6.3.2.3
        see CERT section INT36-C http://rvdoc.org/CERT/INT36-C

test/test-spawn.c:1070:8: warning: Conversion from an integer to non-null pointer.

    Implementation defined behavior (IMPL-CCV13):
        see C11 section 6.3.2.3:5 http://rvdoc.org/C11/6.3.2.3
        see CERT section INT36-C http://rvdoc.org/CERT/INT36-C
```
```
  CC       test/test_run_tests-test-stdio-over-pipes.o
test/test-stdio-over-pipes.c:66:3: error: Dereferencing a pointer past the end of an array.

    Undefined behavior (UB-CER4):
        see C11 section 6.5.6:8 http://rvdoc.org/C11/6.5.6
        see C11 section J.2:1 items 47 and 49 http://rvdoc.org/C11/J.2
        see CERT-C section ARR30-C http://rvdoc.org/CERT-C/ARR30-C
        see CERT-C section ARR37-C http://rvdoc.org/CERT-C/ARR37-C
        see CERT-C section STR31-C http://rvdoc.org/CERT-C/STR31-C
        see MISRA-C section 8.18:1 http://rvdoc.org/MISRA-C/8.18
        see MISRA-C section 8.1:3 http://rvdoc.org/MISRA-C/8.1
```
Here although `gcc` can run `make check` and proceed to the `CCLD     test/run-tests` stage, it still failed several tests, such as:
```
...
ok 356 - udp_no_autobind
ok 357 - udp_open
not ok 358 - udp_open_bound
# exit code 134
# Output from process `udp_open_bound`:
# Assertion failed in test/test-udp-open.c on line 240: r == 0
not ok 359 - udp_open_connect
# exit code 134
# Output from process `udp_open_connect`:
# Assertion failed in test/test-udp-open.c on line 272: r == 0
ok 360 - udp_open_twice
...
```
and result in:
```
FAIL: test/run-tests
======================================================
1 of 1 test failed
Please report to https://github.com/libuv/libuv/issues
======================================================
Makefile:5086: recipe for target 'check-TESTS' failed
make[1]: *** [check-TESTS] Error 1
make[1]: Leaving directory '/mnt/d/Project_Files/rv-match-playground/libuv/gcc/libuv'
Makefile:5342: recipe for target 'check-am' failed
make: *** [check-am] Error 2
```