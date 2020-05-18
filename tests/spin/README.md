# spin

The source code of `spin` can be found [here](http://spinroot.com/spin/Src/spin647.tar.gz).

### Pre-request
```
wget http://spinroot.com/spin/Src/spin647.tar.gz
tar -xvzf spin647.tar.gz
rm spin647.tar.gz
```


### Build
To run with kcc, change `CC=kcc` in  `makefile`.
```
cd Spin/Src6.4.7
make
```

### Test
Run `./spin` on `pml` files in `Spin/Examples/`.
```
./spin ../Examples/hello.pml
```


### Observation - Build

`kcc` succeeded in the `make` job.

When running `make` with `gcc` and compiling `pangen6.c`, following undefined behaviors are printed:
```
kcc -O2 -DNXT	   -c -o pangen6.o pangen6.c
pangen6.c:2174:6: error: Potential negative zero produced via bitwise operations, undefined under sign and magnitude or one's complement arithmetic, implementation-defined otherwise.

    Implementation-dependent undefined behavior (IMPLUB-CEB12):
        see C11 section 6.2.6.2:4 http://rvdoc.org/C11/6.2.6.2
        see C11 section J.2:1 item 14 http://rvdoc.org/C11/J.2
        see CERT-C section INT13-C http://rvdoc.org/CERT-C/INT13-C
        see CERT-C section MSC15-C http://rvdoc.org/CERT-C/MSC15-C
        see MISRA-C section 8.1:3 http://rvdoc.org/MISRA-C/8.1

pangen6.c:2203:3: error: Potential negative zero produced via bitwise operations, undefined under sign and magnitude or one's complement arithmetic, implementation-defined otherwise.

    Implementation-dependent undefined behavior (IMPLUB-CEB12):
        see C11 section 6.2.6.2:4 http://rvdoc.org/C11/6.2.6.2
        see C11 section J.2:1 item 14 http://rvdoc.org/C11/J.2
        see CERT-C section INT13-C http://rvdoc.org/CERT-C/INT13-C
        see CERT-C section MSC15-C http://rvdoc.org/CERT-C/MSC15-C
        see MISRA-C section 8.1:3 http://rvdoc.org/MISRA-C/8.1
```
and also, unlike `gcc` 7.5.0, `kcc` gives warning when compiling `flow.c`:
```
kcc -O2 -DNXT	   -c -o flow.o flow.c
flow.c: In function ‘match_struct’:
flow.c:929:27: warning: format ‘%p’ expects argument of type ‘void *’, but argument 3 has type ‘struct Symbol *’ [-Wformat=]
  { printf("index type %s %p ==\n", s->Snm->name, s->Snm);
                          ~^                      ~~~~~~
flow.c:930:26: warning: format ‘%p’ expects argument of type ‘void *’, but argument 3 has type ‘struct Symbol *’ [-Wformat=]
   printf("chan type  %s %p --\n\n", t->ini->rgt->sym->name, t->ini->rgt->sym);
                         ~^                                  ~~~~~~~~~~~~~~~~
```

`gcc` took `0m8.118s` to finish `make`, while `kcc` took `3m8.318s`.

### Observation - Testing

`kcc` succeeded in running test files in `Spin/Examples/` and can provide undefined behavior warning. Running the `kcc` compiled file on almost all testcases would generate the following undefined behavior:
```
Indeterminate value used in an expression:
      > in yyparse at y.tab.c:2125:3
        in main at main.c:1116:2

    Undefined behavior (UB-CEE2):
        see C11 section 6.2.4 http://rvdoc.org/C11/6.2.4
        see C11 section 6.7.9 http://rvdoc.org/C11/6.7.9
        see C11 section 6.8 http://rvdoc.org/C11/6.8
        see C11 section J.2:1 item 11 http://rvdoc.org/C11/J.2
        see CERT-C section EXP33-C http://rvdoc.org/CERT-C/EXP33-C
        see MISRA-C section 8.9:1 http://rvdoc.org/MISRA-C/8.9
        see MISRA-C section 8.1:3 http://rvdoc.org/MISRA-C/8.1

```
Below is the running time difference between `kcc` and `gcc` on several testcases. Since the testcases themselves are multi-threaded, here the total number of processes created is also recorded. We only record result for several testcases since some of them would give endless loop for both `gcc` and `kcc`.

| Testcase name | Processes created | Time for gcc | Time for kcc | Time diff |
| ----------- | ----------- | ----------- | ----------- | ----------- |
| calculator.pml | 10 | 0m0.099s | 1m50.772s | 1118.9x |
| dtp.pml | 4 | 0m0.111s | 14m13.053s | 7685.2x |
| eratosthenes.pml | 10 | 0m0.119s | 4m47.253s | 2413.9x |
| for_example.pml | 1 | 0m0.064 | 0m59.943s | 936.61x |
| for_select_example.pml | 1 | 0m0.116s | 3m22.199s | 1743.1x |
| hello.pml | 1 | 0m0.136s | 0m11.399s | 83.816x |
| leader_trace.pml | 6 | 0m0.079s | 3m59.130s | 3027.0x |
| leader0.pml | 6 | 0m0.109s | 3m24.400s | 1875.2x |
| pathfinder.pml | 6 | 0m0.117s | 1m24.117s | 718.95x |
| rtos1.pml | 3 | 0m0.793s | 0m54.138s | 68.270x |
| sat.pml | 1 | 0m0.107s | 0m51.633s | 482.55x |
| snoopy.pml | 7 | 0m0.121s | 12m15.353s | 6077.3x |
| sort.pml | 9 | 0m0.081s | 3m34.904s | 2653.1x |
| welfare.pml | 1 | 0m0.132s | 1m38.119s | 743.32x |
| werkplaats.pml | 1 | 0m0.105s | 3m32.260s | 2021.5x |
