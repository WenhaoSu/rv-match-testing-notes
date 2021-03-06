# tcpdump

The source code of `tcpdump` can be found [here](https://github.com/the-tcpdump-group/tcpdump).


### Summary

The error message `kcc` gives is
```
Failed to execute native function pcap_getnonblock successfully. Reason: Variadic arguments in function pointer.
Fatal error: exception (Invalid_argument "mismatched constructor at top of split configuration")
```
but there are no variadic function pointers related to `pcap_getnonblock` in the codebase.

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
Then the compiled `tcpdump` execution file by `gcc` will execute normally, while `kcc` will give same error message (it failed whrn running `pcap_dispatch`):
```
$ ../tcpdump -S -t -q -n -r ./isup.pcap
reading from file ./isup.pcap, link-type EN10MB (Ethernet)
Here should be where pcap_loop start to execute.
Try to get some message:3
Failed to execute native function pcap_dispatch successfully. Reason: Variadic arguments in function pointer.
Fatal error: exception (Invalid_argument "mismatched constructor at top of split configuration")
```
This means that kcc can run `pcap_get_selectable_fd`, whose code is:
```c
int
pcap_get_selectable_fd(pcap_t *p)
{
	return (p->selectable_fd);
}
```
```c
struct pcap {
...
	int selectable_fd;	/* FD on which select()/poll()/epoll_wait()/kevent()/etc. can be done */
...
```
This means that we can safely access the non-function members, such as `int` in struct `pcap`.

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

---
Here is the complete code for struct `pcap`:
```c
struct pcap {
	/*
	 * Method to call to read packets on a live capture.
	 */
	read_op_t read_op;

	/*
	 * Method to call to read the next packet from a savefile.
	 */
	next_packet_op_t next_packet_op;

#ifdef _WIN32
	HANDLE handle;
#else
	int fd;
	int selectable_fd;
#endif /* _WIN32 */

	/*
	 * Read buffer.
	 */
	u_int bufsize;
	void *buffer;
	u_char *bp;
	int cc;

	sig_atomic_t break_loop; /* flag set to force break from packet-reading loop */

	void *priv;		/* private data for methods */

#ifdef ENABLE_REMOTE
	struct pcap_samp rmt_samp;	/* parameters related to the sampling process. */
#endif

	int swapped;
	FILE *rfile;		/* null if live capture, non-null if savefile */
	u_int fddipad;
	struct pcap *next;	/* list of open pcaps that need stuff cleared on close */

	/*
	 * File version number; meaningful only for a savefile, but we
	 * keep it here so that apps that (mistakenly) ask for the
	 * version numbers will get the same zero values that they
	 * always did.
	 */
	int version_major;
	int version_minor;

	int snapshot;
	int linktype;		/* Network linktype */
	int linktype_ext;       /* Extended information stored in the linktype field of a file */
	int tzoff;		/* timezone offset */
	int offset;		/* offset for proper alignment */
	int activated;		/* true if the capture is really started */
	int oldstyle;		/* if we're opening with pcap_open_live() */

	struct pcap_opt opt;

	/*
	 * Place holder for pcap_next().
	 */
	u_char *pkt;

#ifdef _WIN32
	struct pcap_stat stat;		/* used for pcap_stats_ex() */
#endif

	/* We're accepting only packets in this direction/these directions. */
	pcap_direction_t direction;

	/*
	 * Flags to affect BPF code generation.
	 */
	int bpf_codegen_flags;

	/*
	 * Placeholder for filter code if bpf not in kernel.
	 */
	struct bpf_program fcode;

	char errbuf[PCAP_ERRBUF_SIZE + 1];
	int dlt_count;
	u_int *dlt_list;
	int tstamp_type_count;
	u_int *tstamp_type_list;
	int tstamp_precision_count;
	u_int *tstamp_precision_list;

	struct pcap_pkthdr pcap_header;	/* This is needed for the pcap_next_ex() to work */

	/*
	 * More methods.
	 */
	activate_op_t activate_op;
	can_set_rfmon_op_t can_set_rfmon_op;
	inject_op_t inject_op;
	save_current_filter_op_t save_current_filter_op;
	setfilter_op_t setfilter_op;
	setdirection_op_t setdirection_op;
	set_datalink_op_t set_datalink_op;
	getnonblock_op_t getnonblock_op;
	setnonblock_op_t setnonblock_op;
	stats_op_t stats_op;

	/*
	 * Routine to use as callback for pcap_next()/pcap_next_ex().
	 */
	pcap_handler oneshot_callback;

#ifdef _WIN32
	/*
	 * These are, at least currently, specific to the Win32 NPF
	 * driver.
	 */
	stats_ex_op_t stats_ex_op;
	setbuff_op_t setbuff_op;
	setmode_op_t setmode_op;
	setmintocopy_op_t setmintocopy_op;
	getevent_op_t getevent_op;
	oid_get_request_op_t oid_get_request_op;
	oid_set_request_op_t oid_set_request_op;
	sendqueue_transmit_op_t sendqueue_transmit_op;
	setuserbuffer_op_t setuserbuffer_op;
	live_dump_op_t live_dump_op;
	live_dump_ended_op_t live_dump_ended_op;
	get_airpcap_handle_op_t get_airpcap_handle_op;
#endif
	cleanup_op_t cleanup_op;
};
```

while all methods in `pcap` are defined as
```c
typedef int	(*activate_op_t)(pcap_t *);
typedef int	(*can_set_rfmon_op_t)(pcap_t *);
typedef int	(*read_op_t)(pcap_t *, int cnt, pcap_handler, u_char *);
typedef int	(*next_packet_op_t)(pcap_t *, struct pcap_pkthdr *, u_char **);
typedef int	(*inject_op_t)(pcap_t *, const void *, size_t);
typedef void	(*save_current_filter_op_t)(pcap_t *, const char *);
typedef int	(*setfilter_op_t)(pcap_t *, struct bpf_program *);
typedef int	(*setdirection_op_t)(pcap_t *, pcap_direction_t);
typedef int	(*set_datalink_op_t)(pcap_t *, int);
typedef int	(*getnonblock_op_t)(pcap_t *);
typedef int	(*setnonblock_op_t)(pcap_t *, int);
typedef int	(*stats_op_t)(pcap_t *, struct pcap_stat *);
```

What is strange here is that there seems to be no variadic functions. Maybe kcc is treating some of them as variadic functions, so it throw that error message?
