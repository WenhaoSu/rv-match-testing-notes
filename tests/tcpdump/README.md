# tcpdump

The source code of `tcpdump` can be found [here](https://github.com/the-tcpdump-group/tcpdump).

### Pre-request
```
git clone https://github.com/the-tcpdump-group/tcpdump.git
cd tcpdump/
git checkout af974494da71f2dae8eeac40e1611db5d6a82668
sudo apt install libpcap-dev
sudo apt install autoconf
```

### Building with gcc
```
aclocal; autoreconf
./configure CC="gcc -std=gnu11" LD=gcc
make
```

### Building with kcc
```
aclocal; autoreconf
./configure CC="kcc -std=gnu11" LD=kcc
make
```


### Test
```
cd tests/
bash TESTrun.sh
```


### Observation - Build

`kcc` succeeded in the `make` job.

When running `make` with `gcc` and compiling `print-esp.c`, following undefined behaviors are printed:
```
kcc -std=gnu11 -O -DHAVE_CONFIG_H   -D_U_="__attribute__((unused))" -I. -I/usr/include  -I/usr/local/include -g -c ./print-esp.c
./print-esp.c:192:2: error: Incompatible pointer types in initializer or function call arguments.

    Constraint violation (CV-TSE2):
        see C11 section 6.7.9:11 http://rvdoc.org/C11/6.7.9
        see C11 section 6.5.16.1:1 http://rvdoc.org/C11/6.5.16.1
        see CERT-C section MSC40-C http://rvdoc.org/CERT-C/MSC40-C
        see MISRA-C section 8.1:1 http://rvdoc.org/MISRA-C/8.1

./print-esp.c:710:4: error: Incompatible pointer types in initializer or function call arguments.

    Constraint violation (CV-TSE2):
        see C11 section 6.7.9:11 http://rvdoc.org/C11/6.7.9
        see C11 section 6.5.16.1:1 http://rvdoc.org/C11/6.5.16.1
        see CERT-C section MSC40-C http://rvdoc.org/CERT-C/MSC40-C
        see MISRA-C section 8.1:1 http://rvdoc.org/MISRA-C/8.1
```

`gcc` took `1m12.882s` to finish `make`, while `kcc` took `24m37.459s`.

### Observation - Testing

`kcc` failed in running all test files in `tests/`. Running the `kcc` compiled file on any of the testcase would generate the following error:
```
Failed to execute native function pcap_loop successfully. Reason: Variadic arguments in function pointer.
Fatal error: exception (Invalid_argument "mismatched constructor at top of split configuration")
```
However, compiled file by running `gcc` is able to run test files.

### Note

One potential reason why `kcc` failed maybe that it shows differnt behavior with `gcc` when compiling `print-hncp.c`.
Here `gcc` gives the following output:
```
gcc -std=gnu11 -ffloat-store -DHAVE_CONFIG_H   -D_U_="__attribute__((unused))" -I. -I/usr/include  -g -O2 -c ./print-hncp.c
./print-hncp.c: In function ‘format_256’:
./print-hncp.c:154:26: warning: ‘%016lx’ directive output truncated writing 16 bytes into a region of size 12 [-Wformat-truncation=]
     snprintf(buf[i], 28, "%016" PRIx64 "%016" PRIx64 "%016" PRIx64 "%016" PRIx64,
                          ^~~~~~
./print-hncp.c:154:41: note: format string is defined here
     snprintf(buf[i], 28, "%016" PRIx64 "%016" PRIx64 "%016" PRIx64 "%016" PRIx64,
./print-hncp.c:154:26: note: using the range [0, 18446744073709551615] for directive argument
     snprintf(buf[i], 28, "%016" PRIx64 "%016" PRIx64 "%016" PRIx64 "%016" PRIx64,
                          ^~~~~~
./print-hncp.c:154:26: note: using the range [0, 18446744073709551615] for directive argument
./print-hncp.c:154:26: note: using the range [0, 18446744073709551615] for directive argument
In file included from /usr/include/stdio.h:862:0,
                 from /usr/include/pcap/pcap.h:54,
                 from /usr/include/pcap.h:43,
                 from ./netdissect.h:75,
                 from ./print-hncp.c:38:
/usr/include/x86_64-linux-gnu/bits/stdio2.h:64:10: note: ‘__builtin___snprintf_chk’ output 65 bytes into a destination of size 28
   return __builtin___snprintf_chk (__s, __n, __USE_FORTIFY_LEVEL - 1,
          ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
        __bos (__s), __fmt, __va_arg_pack ());
        ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

```

while `kcc` gives the following output:

```
kcc -O -std=gnu11 -DHAVE_CONFIG_H   -D_U_="__attribute__((unused))" -I. -I/usr/include  -I/usr/local/include -std=gnu11 -c ./print-hncp.c
./print-hncp.c: In function ‘format_256’:
./print-hncp.c:154:26: warning: ‘%016lx’ directive output truncated writing 16 bytes into a region of size 12 [-Wformat-truncation=]
     snprintf(buf[i], 28, "%016" PRIx64 "%016" PRIx64 "%016" PRIx64 "%016" PRIx64,
                          ^~~~~~
./print-hncp.c:154:41: note: format string is defined here
     snprintf(buf[i], 28, "%016" PRIx64 "%016" PRIx64 "%016" PRIx64 "%016" PRIx64,
./print-hncp.c:154:5: note: ‘snprintf’ output 65 bytes into a destination of size 28
     snprintf(buf[i], 28, "%016" PRIx64 "%016" PRIx64 "%016" PRIx64 "%016" PRIx64,
     ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
          EXTRACT_64BITS(data),
          ~~~~~~~~~~~~~~~~~~~~~
          EXTRACT_64BITS(data + 8),
          ~~~~~~~~~~~~~~~~~~~~~~~~~
          EXTRACT_64BITS(data + 16),
          ~~~~~~~~~~~~~~~~~~~~~~~~~~
          EXTRACT_64BITS(data + 24)
          ~~~~~~~~~~~~~~~~~~~~~~~~~
     );
     ~
```

### Followup

Here is the code for function `pcap_loop`:

```c
pcap.h:

...
PCAP_API int	pcap_loop(pcap_t *, int, pcap_handler, u_char *);
...
```

```c
pcap.c:

...
int
pcap_loop(pcap_t *p, int cnt, pcap_handler callback, u_char *user)
{
	register int n;

	for (;;) {
		if (p->rfile != NULL) {
			/*
			 * 0 means EOF, so don't loop if we get 0.
			 */
			n = pcap_offline_read(p, cnt, callback, user);
		} else {
			/*
			 * XXX keep reading until we get something
			 * (or an error occurs)
			 */
			do {
				n = p->read_op(p, cnt, callback, user);
			} while (n == 0);
		}
		if (n <= 0)
			return (n);
		if (!PACKET_COUNT_IS_UNLIMITED(cnt)) {
			cnt -= n;
			if (cnt <= 0)
				return (0);
		}
	}
}
...
```
