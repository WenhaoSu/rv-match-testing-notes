# remotery

The source code of `remotery` can be found [here](git clone https://github.com/Celtoys/Remotery.git).

### Pre-request
```
  None    
```

### Building with gcc
```
    cd Remotery/ ; results[0]="$?" ;
    kcc -profile x86_64-linux-gcc-glibc-gnuc
    gcc -std=gnu11 lib/Remotery.c sample/sample.c -I lib -pthread -lm |& tee rv_build_1.txt ; results[1]="$?" ; postup 1
    kcc-profile x86_64-linux-gcc-glibc
    
````    


### Building with kcc
```
    cd Remotery/ ; results[0]="$?" ;
   `kcc -profile x86_64-linux-gcc-glibc-gnuc
    kcc -std=gnu11 lib/Remotery.c sample/sample.c -I lib -pthread -lm |& tee rv_build_1.txt ; results[1]="$?" ; postup 1
    kcc -profile x86_64-linux-gcc-glibc
```

### Observation-Build

Build was successful in gcc case. In kcc it found some errors which may have resulted in failure of producing an executable files.
```
/usr/include/x86_64-linux-gnu/bits/floatn.h[75:9-17] : Warning: Encountered _Complex type.  These are not yet supported, and are currently ignored.
lib/Remotery.c:461:9: error: Trying to look up identifier __sync_bool_compare_and_swap, but no such identifier is in scope.

    Syntax error (SE-CID1):
        see C11 section 6.5.1:2 http://rvdoc.org/C11/6.5.1
        see CERT-C section DCL31-C http://rvdoc.org/CERT-C/DCL31-C

lib/Remotery.c:475:9: error: Trying to look up identifier __sync_bool_compare_and_swap, but no such identifier is in scope.

    Syntax error (SE-CID1):
        see C11 section 6.5.1:2 http://rvdoc.org/C11/6.5.1
        see CERT-C section DCL31-C http://rvdoc.org/CERT-C/DCL31-C

lib/Remotery.c:490:9: error: Trying to look up identifier __sync_fetch_and_add, but no such identifier is in scope.

    Syntax error (SE-CID1):
        see C11 section 6.5.1:2 http://rvdoc.org/C11/6.5.1
        see CERT-C section DCL31-C http://rvdoc.org/CERT-C/DCL31-C

lib/Remotery.c:901:5: warning: Conversion from an integer to non-null pointer.

    Implementation defined behavior (IMPL-CCV13):
        see C11 section 6.3.2.3:5 http://rvdoc.org/C11/6.3.2.3
        see CERT section INT36-C http://rvdoc.org/CERT/INT36-C

/usr/include/x86_64-linux-gnu/bits/floatn.h[75:9-17] : Warning: Encountered _Complex type.  These are not yet supported, and are currently ignored.

```
### Observation- Testing

The gcc produced an executable file but it didn't print anything probably requiring input whereas the kcc falied to produce and executable.
