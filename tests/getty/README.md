# getty

The source code of `getty` can be found [here](https://github.com/StarchLinux/getty).

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
