# tmux

The source code of `tmux` can be found [here](https://github.com/tmux/tmux.git).

### Pre-request
```
    sudo apt -y install libevent-dev   
```

### Building with gcc
```
    cd tmux/
    bash autogen.sh
     ./configure CC=kcc CFLAGS="-no-pedantic" LD=kcc |& tee rv_build_0.txt ; results[0]="$?"
      make -j`nproc` |& tee rv_build_1.txt ; results[1]="$?" ; postup 1
```

### Building with kcc
```
    cd tmux/
    bash autogen.sh
    ./configure CC=$compiler |& tee rv_build_0.txt ; results[0]="$?"
     make -j`nproc` |& tee rv_build_1.txt ; results[1]="$?" ; postup 1
```    
### Observation-Build
In the case of 'gcc' the build was successful. In 'kcc' it was successful to some stages but it kept crashing and it was difficult to figure out if it completes the compilation.
