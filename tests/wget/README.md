# wget

The source code of `wget` can be found [here](https://github.com/mirror/wget).
**Note**: `wget` is extremely large, it may take a long time to clone the repo and build.

### Pre-request
The following packages are needed to build the software:

* autotools (autoconf, autogen, automake, autopoint, libtool)
* python (recommended for faster bootstrap)
* rsync
* tar
* makeinfo (part of texinfo)
* pkg-config >= 0.28 (recommended)
* doxygen (for creating the documentation)
* pandoc (for creating the wget2 man page)
* gettext >= 0.18.2
* libiconv (needed for IRI and IDN support)
* libz >= 1.2.3 (the distribution may call the package zlib*, eg. * zlib1g on Debian)
* liblzma >= 5.1.1alpha (optional, if you want HTTP lzma * decompression)
* libbz2 >= 1.0.6 (optional, if you want HTTP bzip2 decompression)
* libbrotlidec/libbrotli >= 1.0.0 (optional, if you want HTTP brotli * decompression)
* libzstd >= 1.3.0 (optional, if you want HTTP zstd decompression)
* libgnutls (3.3, 3.5 or 3.6)
* libidn2 >= 0.14 (libidn >= 1.25 if you don't have libidn2)
* flex >= 2.5.35
* libpsl >= 0.5.0
* libnghttp2 >= 1.3.0 (optional, if you want HTTP/2 support)
* libmicrohttpd >= 0.9.51 (optional, if you want to run the test * suite)
* lzip (optional, if you want to build distribution tarballs)
* lcov (optional, for coverage reports)
* libgpgme >= 0.4.2 (optional, for automatic signature verification)
* libpcre | libpcre2 (optional, for filtering by PCRE|PCRE2 regex)
* libhsts (optional, to support HSTS preload lists)
* libwolfssl (optional, to support WolfSSL instead of GnuTLS)


```
sudo apt -y install autopoint
sudo apt -y install texinfo
sudo apt -y install gperf
sudo apt -y install libgnutls-dev

git clone https://github.com/mirror/wget.git
cd wget/
git checkout 6aa6b669efbef62f60471c18e7fdef0206f92337
./bootstrap
```


### Building with gcc
```
./bootstrap
./configure --disable-threads
make
```

### Building with kcc
```
./bootstrap
./configure CC=kcc LD=kcc --disable-threads
make
```

### Observation

Both `gcc` and `kcc` failed to run due to the error message `configure.ac:186: error: required file './ABOUT-NLS' not found`. This maybe due to several libraries missing or git version out of date. An alternative solution is to try [wget2](https://github.com/rockdaboot/wget2), which is a more active repo.
