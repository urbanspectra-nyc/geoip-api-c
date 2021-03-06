# GeoIP Legacy C Library #

## Important Note ##

This library is for the GeoIP Legacy format (dat). To read the MaxMind DB
format (mmdb) used by GeoIP2, please see
[libmaxminddb](https://github.com/maxmind/libmaxminddb).

## Description ##

The GeoIP Legacy C library enables the user to find geographical and network
information of an IP address. To use this library, you may download our free
GeoLite Legacy Country or City databases. These are updated at the beginning
of every month. The latest versions are available at:

http://dev.maxmind.com/geoip/legacy/geolite

We also offer commercial GeoIP Legacy databases with greater accuracy and
additional network information. For more details, see:

https://www.maxmind.com/en/geolocation_landing

If you use GeoIP Legacy to block access from high risk countries, you may wish
to use our proxy detection service to block access from known proxy servers to
reduce fraud and abuse. For more details, see:

https://www.maxmind.com/en/proxy


## Installation ##

### On Ubuntu Using PPA ###

MaxMind provides a PPA for recent version of Ubuntu. To add the PPA to your
APT sources, run:

    $ sudo add-apt-repository ppa:maxmind/ppa

Then install the packages by running:

    $ sudo aptitude update
    $ sudo aptitude install libgeoip1 libgeoip-dev geoip-bin

### From Source ###

To install, run:

```
./configure
make
make check
make install
```

If you are using a GitHub checkout, please run the `bootstrap` script first
to set up the build environment.

The GeoIP Legacy C library relies on GNU make, not on BSD make

## Memory Caching and Other Options ##

There are five options available:

* `GEOIP_STANDARD` - Read database from file system. This uses the least
  memory.
* `GEOIP_MEMORY_CACHE` - Load database into memory. Provides faster
  performance but uses more memory.
* `GEOIP_CHECK_CACHE` - Check for updated database. If database has been
  updated, reload file handle and/or memory cache.
* `GEOIP_INDEX_CACHE` - Cache only the the most frequently accessed index
  portion of the database, resulting in faster lookups than `GEOIP_STANDARD`,
  but less memory usage than `GEOIP_MEMORY_CACHE`. This is useful for larger
  databases such as GeoIP Legacy Organization and GeoIP Legacy City. Note: for
  GeoIP Legacy Country, Region and Netspeed databases, `GEOIP_INDEX_CACHE` is
  equivalent to `GEOIP_MEMORY_CACHE`.
* `GEOIP_MMAP_CACHE` - Load database into mmap shared memory. MMAP is not
  available for 32bit Windows.
* `GEOIP_SILENCE` - Disables diagnostic messages to stderr in the library;
  error conditions are detectable via the return values.

These options can be combined using bit operators. For example you can
use both `GEOIP_MEMORY_CACHE` and `GEOIP_CHECK_CACHE by calling`:

```c
GeoIP_open("/path/to/GeoIP.dat", GEOIP_MEMORY_CACHE | GEOIP_CHECK_CACHE);
```

By default, the city name is returned in iso-8859-1 charset. To obtain the
city name in utf8 instead, run:

```c
GeoIP_set_charset(gi, GEOIP_CHARSET_UTF8);
```

To get the netmask of the netblock of the last lookup, use
`GeoIP_last_netblock(gi)`.

## Examples ##

See the following files for examples of how to use the API:

```
test/
     test-geoip.c
     test-geoip-region.c
     test-geoip-city.c
     test-geoip-isp.c
     test-geoip-org.c
     test-geoip-netspeed.c
```

The test-geoip.c program works with both the GeoLite Legacy and GeoIP Legacy
Country databases. The test-geoip-city.c program works with both the GeoLite
Legacy and GeoIP Legacy City databases. The other example programs require the
paid databases available (https://www.maxmind.com/en/geolocation_landing).


## Automatic Updates ##

MaxMind offers a service where you can have your database updated
automically each week. For more details see:

http://www.maxmind.com/en/license_key

## Resources ##

### Performance Patches ###

Patrick McManus provide a patch to enhance the lookup speed in `MEMORY_CACHE`
mode. If you feel, that the current `MEMORY_CACHE` mode is to slow try the
patch:

http://sourceforge.net/mailarchive/forum.php?forum_name=geoip-c-discuss&max_rows=25&style=nested&viewmonth=200803

### Development Version ###

Please find the latest version of the C API on GitHub:

https://github.com/maxmind/geoip-api-c

## Troubleshooting ##

### Autotool Issues ###
In case of trouble building from source with libtool or autotools, update
the generated configuration files by running:

```
./bootstrap
```
or

```
autoreconf -vfi
```
or
```
aclocal && autoconf && automake --add-missing
```

### Threads

Lookups are thread safe, but libGeoIP does not guard the functions. Do not
drop or change the GeoIP database if other threads might be using the
database.

The recommended pattern is:

1. open database with GEOIP_MEMORY_CACHE
2. create_threads(your_thread_entry_function)
3. use the database
4. join_threads
5. close database

### Thread Safety on Windows ###

The Windows build is not thread-safe in STANDARD mode because the `pread` is
not thread-safe.

### Other Build Issues ###

If you run into trouble building your application with GeoIP Legacy support,
try adding `-fms-extensions` to your `CFLAGS`. If you use Solaris and the
default C compiler, use `-features=extensions` instead. These options enable
unnamed union support to fix problems like: `improper member use: dma_code` or
`'GeoIPRecord' has no member named 'dma_code'`.

Note that it is recommended that you use GNU make. Also, if you are using
OpenBSD, this library requires OpenBSD 3.1 or greater.

If you get a "cannot load shared object file: No such file or directory"
error, add the directory `libGeoIP.so` was installed to the `/etc/ld.so.conf`
file and run `ldconfig`.

On Solaris, if you get a `ld: fatal: relocations remain against allocatable
but non-writable sections`, try runnign:

```
make clean
./configure --disable-shared
make
```

If you get a `ar : command not found` error, make sure that `ar` is in your
path. On Solaris, `ar` is typically found in `/usr/ccs/bin`

If you get a `bad interpreter: No such file or directory` error when running
`./configure`, make sure that there are no DOS returns in the configure
script. To remove DOS returns, run `perl -pi -e 's/\r//g' configure`.

If gcc fails while consuming a large amount of memory, try compiling with
`CFLAGS=-O1` (or `-O0`) instead of the default `-O2`. It seems that some
versions of gcc have a bug and consume 1 GB of memory when optimizing certain
source files. It has been reported on gcc 3.3.1 and with gcc 4.2(.0). Thanks
to Kai Schaetzl for the report.

If `GEOIP_MMAP_CACHE` doesn't work on a 64bit machine, try adding the flag
`MAP_32BIT` to the mmap call.

If you get a `passing argument 3 of 'gethostbyname_r' from incompatible
pointer type` error on AIX, untar a fresh copy of thie library and delete the
following two lines from `./configure`:

```
#define HAVE_GETHOSTBYNAME_R 1

#define GETHOSTBYNAME_R_RETURNS_INT 1
```

then save the configure script and build it as usual:

```
./configure
make
sudo make install
```

## Bug Tracker ##

Please report all issues with this code using the [GitHub issue tracker]
(https://github.com/maxmind/geoip-api-c/issues).

If you are having an issue with a MaxMind database that is not specific to
this API, please [contact MaxMind support]
(http://www.maxmind.com/en/support).

## Contributing ##

To contribute, please submit a pull request on
[GitHub](https://github.com/maxmind/geoip-api-c/).
