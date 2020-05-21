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
---
Following are two external links that may provide useful information for debugging:
* Same problem of `Variadic arguments in function pointer.`: https://github.com/the-tcpdump-group/tcpdump/issues/666
* Same problem of `Fatal error: exception (Invalid_argument "mismatched constructor at top of split configuration")`:https://github.com/kframework/c-semantics/issues/512
---
Observation: if we change line 1766-1768 in `tcpdump.c` to the following code:
```c
...
	do {
		// status = pcap_loop(pd, cnt, callback, pcap_userdata);
		printf("Here should be where pcap_loop start to execute.");
		status = -1;
		if (WFileName == NULL) {
...
```
Then the compiled `tcpdump` execution file won't give fatal error, while the execution result is as expected:
```
$ ../tcpdump -S -t -q -n -r ./isup.pcap
reading from file ./isup.pcap, link-type EN10MB (Ethernet)
Here should be where pcap_loop start to execute.tcpdump: pcap_loop:
```
This means `pcap_loop` is likely to be the only single issue that causes kcc's failure.

---
Observation: if we change line 1766-1768 in `tcpdump.c` to the following code:
```c
...
	do {
		// status = pcap_loop(pd, cnt, callback, pcap_userdata);
		printf("Here should be where pcap_loop start to execute.\n");
		printf("Try to get some message:%d\n", pcap_get_selectable_fd(pd));
		status = pcap_dispatch(pd, cnt, callback, pcap_userdata);
		if (WFileName == NULL) {
...
```
Then the compiled `tcpdump` execution file by `gcc` will execute normally, while `kcc` will give same error message:
```
$ ../tcpdump -S -t -q -n -r ./isup.pcap
reading from file ./isup.pcap, link-type EN10MB (Ethernet)
Here should be where pcap_loop start to execute.
Try to get some message:3
Failed to execute native function pcap_dispatch successfully. Reason: Variadic arguments in function pointer.
Fatal error: exception (Invalid_argument "mismatched constructor at top of split configuration")
```

The funtion `pcap_dispatch` has the exactly same function signature with `pcap_loop`:
```c
PCAP_API int	pcap_loop(pcap_t *, int, pcap_handler, u_char *);
PCAP_API int	pcap_dispatch(pcap_t *, int, pcap_handler, u_char *);
```
and here is the code of function `pcap_dispatch`:
```c
int
pcap_dispatch(pcap_t *p, int cnt, pcap_handler callback, u_char *user)
{
	return (p->read_op(p, cnt, callback, user));
}
```
Hence, it is reasonable to locate the problem to the function `p->read_op`, and I guess things like `p->read_op` is exactly the function pointer that `kcc` believe has "variadic arguments".

The function signature of `read_op` is:
```
typedef int	(*read_op_t)(pcap_t *, int cnt, pcap_handler, u_char *);
```

The detailed implementation of `read_op` varies for different types of `pcap_t`. There are nearly 30 different possible `read_op` assignments, and here are two of them:
```c
pcap-airpcap.c:
p->read_op = airpcap_read;

pcap-bof.c:
p->read_op = pcap_read_bpf;

...
```

---
Observation: if we change line 1766-1768 in `tcpdump.c` to the following code:
```c
...
do {
		// status = pcap_loop(pd, cnt, callback, pcap_userdata);
		printf("Here should be where pcap_getnonblock start to execute.\n");
		status = pcap_getnonblock(pd, pcap_userdata);
		status = -1;
		if (WFileName == NULL) {
...
```
The gcc version of code will run normally while the kcc version will give:
```
$ ../tcpdump -S -t -q -n -r ./isup.pcap
reading from file ./isup.pcap, link-type EN10MB (Ethernet)
Here should be where pcap_getnonblock start to execute.
Failed to execute native function pcap_getnonblock successfully. Reason: Variadic arguments in function pointer.
Fatal error: exception (Invalid_argument "mismatched constructor at top of split configuration")
```
This is exactly the same error message as before, except changing `pcap_loop` to `pcap_getnonblock`. So what's in common with `pcap_loop` and `pcap_getnonblock`? Here's the code for `pcap_getnonblock`:
```c
int
pcap_getnonblock(pcap_t *p, char *errbuf)
{
	int ret;

	ret = p->getnonblock_op(p);
	if (ret == -1) {
		/*
		 * The get nonblock operation sets p->errbuf; this
		 * function *shouldn't* have had a separate errbuf
		 * argument, as it didn't need one, but I goofed
		 * when adding it.
		 *
		 * We copy the error message to errbuf, so callers
		 * can find it in either place.
		 */
		pcap_strlcpy(errbuf, p->errbuf, PCAP_ERRBUF_SIZE);
	}
	return (ret);
}
```

What's in common with `pcap_loop` and `pcap_dispatch` is that they all **called one of the function inside struct pcap:**
```c
struct pcap {
	/*
	 * Method to call to read packets on a live capture.
	 */
	read_op_t read_op;
...
     /*
	 * More methods.
	 */
	getnonblock_op_t getnonblock_op;
...
```
where those functions are further assigned in other codes:
```c
pcap-airpcap.c:

p->getnonblock_op = airpcap_getnonblock;
```
Here there are another about 20 cases of different possible `getnonblock_op` assignments.
A possible approach here is to replicate this situation: write a simple testing program with function pointers in struct, and see what will happen if we call them in the way tcpdump did.
