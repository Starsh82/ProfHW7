# ProfHW7
Домашнее задание №7. Сборка DEB-пакета и создание репозитория.

---
Устанавливаем пакеты для сборки
```
root@UbuntuTestVirt:~# apt update
root@UbuntuTestVirt:~# apt install -y dpkg-dev build-essential zlib1g-dev libpcre3 libpcre3-dev unzip
```
---
Включаем доступ к исходникам в sources.list
```
root@UbuntuTestVirt:~# nano /etc/apt/sources.list.d/ubuntu.sources

Types: deb
URIs: http://ru.archive.ubuntu.com/ubuntu/
Suites: noble noble-updates noble-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

#Репозиторий для исходников
Types: deb-src
URIs: http://archive.ubuntu.com/ubuntu/
Suites: noble
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

Types: deb
URIs: http://security.ubuntu.com/ubuntu/
Suites: noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
```
---
Создаём директорию для сборки и скачиваем исходники nginx
```
root@UbuntuTestVirt:~# mkdir ~/custom-nginx && cd ~/custom-nginx
root@UbuntuTestVirt:~# apt update
root@UbuntuTestVirt:~# apt source nginx
```
```
root@UbuntuTestVirt:~/custom-nginx# apt source nginx
Чтение списков пакетов… Готово
ВНИМАНИЕ: работа над пакетом «nginx» ведётся в системе управления версиями «Git»:
https://salsa.debian.org/nginx-team/nginx.git
Используйте:
git clone https://salsa.debian.org/nginx-team/nginx.git
для получения последних (возможно, невыпущенных) обновлений пакета.
Необходимо скачать 1 198 kB архивов исходного кода.
Пол:1 http://archive.ubuntu.com/ubuntu noble/main nginx 1.24.0-2ubuntu7 (dsc) [3 646 B]
Пол:2 http://archive.ubuntu.com/ubuntu noble/main nginx 1.24.0-2ubuntu7 (tar) [1 112 kB]
Пол:3 http://archive.ubuntu.com/ubuntu noble/main nginx 1.24.0-2ubuntu7 (diff) [81,8 kB]
Получено 1 198 kB за 1с (1 611 kB/s)
dpkg-source: info: extracting nginx in nginx-1.24.0
dpkg-source: info: unpacking nginx_1.24.0.orig.tar.gz
dpkg-source: info: unpacking nginx_1.24.0-2ubuntu7.debian.tar.xz
dpkg-source: info: using patch list from debian/patches/series
dpkg-source: info: applying 0003-define_gnu_source-on-other-glibc-based-platforms.patch
dpkg-source: info: applying nginx-fix-pidfile.patch
dpkg-source: info: applying nginx-ssl_cert_cb_yield.patch
dpkg-source: info: applying CVE-2023-44487.patch
dpkg-source: info: applying ubuntu-branding.patch
W: Загрузка выполняется от лица суперпользователя без ограничений песочницы, так как файл «nginx_1.24.0-2ubuntu7.dsc» недоступен для пользователя «_apt». - pkgAcquire::Run (13: Permission denied)
```
---
Скачиваем модуль ngx_brotli
```
root@UbuntuTestVirt:~/custom-nginx# cd ~/custom-nginx/nginx-1.24.0/
root@UbuntuTestVirt:~/custom-nginx/nginx-1.24.0# git clone --recursive https://github.com/google/ngx_brotli.git
Cloning into 'ngx_brotli'...
remote: Enumerating objects: 237, done.
remote: Counting objects: 100% (37/37), done.
remote: Compressing objects: 100% (16/16), done.
remote: Total 237 (delta 24), reused 21 (delta 21), pack-reused 200 (from 1)
Receiving objects: 100% (237/237), 79.51 KiB | 631.00 KiB/s, done.
Resolving deltas: 100% (114/114), done.
Submodule 'deps/brotli' (https://github.com/google/brotli.git) registered for path 'deps/brotli'
Cloning into '/root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli'...
remote: Enumerating objects: 8047, done.
remote: Counting objects: 100% (91/91), done.
remote: Compressing objects: 100% (53/53), done.
remote: Total 8047 (delta 56), reused 40 (delta 38), pack-reused 7956 (from 3)
Receiving objects: 100% (8047/8047), 40.96 MiB | 2.73 MiB/s, done.
Resolving deltas: 100% (5195/5195), done.
Submodule path 'deps/brotli': checked out 'ed738e842d2fbdf2d6459e39267a633c4a9b2f5d'
```
---
Доустанавлмваем cmake
```
root@UbuntuTestVirt:~/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli# apt install cmake
```
---
Создаём папку out и запускаем сборку brotli
```
root@UbuntuTestVirt:~/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/out# cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF -DCMAKE_C_FLAGS="-Ofast -m64 -march=native -mtune=native -flto -funroll-loops -ffunction-sections -fdata-sections -Wl,--gc-sections" -DCMAKE_CXX_FLAGS="-Ofast -m64 -march=native -mtune=native -flto -funroll-loops -ffunction-sections -fdata-sections -Wl,--gc-sections" -DCMAKE_INSTALL_PREFIX=./installed ..
-- The C compiler identification is GNU 13.3.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Build type is 'Release'
-- Performing Test BROTLI_EMSCRIPTEN
-- Performing Test BROTLI_EMSCRIPTEN - Failed
-- Compiler is not EMSCRIPTEN
-- Looking for log2
-- Looking for log2 - not found
-- Looking for log2
-- Looking for log2 - found
-- Configuring done (1.2s)
-- Generating done (0.0s)
CMake Warning:
  Manually-specified variables were not used by the project:

    CMAKE_CXX_FLAGS


-- Build files have been written to: /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/out
root@UbuntuTestVirt:~/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/out# cmake --build . --config Release --target brotlienc
[  3%] Building C object CMakeFiles/brotlicommon.dir/c/common/constants.c.o
[  6%] Building C object CMakeFiles/brotlicommon.dir/c/common/context.c.o
[ 10%] Building C object CMakeFiles/brotlicommon.dir/c/common/dictionary.c.o
[ 13%] Building C object CMakeFiles/brotlicommon.dir/c/common/platform.c.o
[ 17%] Building C object CMakeFiles/brotlicommon.dir/c/common/shared_dictionary.c.o
[ 20%] Building C object CMakeFiles/brotlicommon.dir/c/common/transform.c.o
[ 24%] Linking C static library libbrotlicommon.a
[ 24%] Built target brotlicommon
[ 27%] Building C object CMakeFiles/brotlienc.dir/c/enc/backward_references.c.o
[ 31%] Building C object CMakeFiles/brotlienc.dir/c/enc/backward_references_hq.c.o
[ 34%] Building C object CMakeFiles/brotlienc.dir/c/enc/bit_cost.c.o
[ 37%] Building C object CMakeFiles/brotlienc.dir/c/enc/block_splitter.c.o
[ 41%] Building C object CMakeFiles/brotlienc.dir/c/enc/brotli_bit_stream.c.o
[ 44%] Building C object CMakeFiles/brotlienc.dir/c/enc/cluster.c.o
[ 48%] Building C object CMakeFiles/brotlienc.dir/c/enc/command.c.o
[ 51%] Building C object CMakeFiles/brotlienc.dir/c/enc/compound_dictionary.c.o
[ 55%] Building C object CMakeFiles/brotlienc.dir/c/enc/compress_fragment.c.o
[ 58%] Building C object CMakeFiles/brotlienc.dir/c/enc/compress_fragment_two_pass.c.o
[ 62%] Building C object CMakeFiles/brotlienc.dir/c/enc/dictionary_hash.c.o
[ 65%] Building C object CMakeFiles/brotlienc.dir/c/enc/encode.c.o
[ 68%] Building C object CMakeFiles/brotlienc.dir/c/enc/encoder_dict.c.o
[ 72%] Building C object CMakeFiles/brotlienc.dir/c/enc/entropy_encode.c.o
[ 75%] Building C object CMakeFiles/brotlienc.dir/c/enc/fast_log.c.o
[ 79%] Building C object CMakeFiles/brotlienc.dir/c/enc/histogram.c.o
[ 82%] Building C object CMakeFiles/brotlienc.dir/c/enc/literal_cost.c.o
[ 86%] Building C object CMakeFiles/brotlienc.dir/c/enc/memory.c.o
[ 89%] Building C object CMakeFiles/brotlienc.dir/c/enc/metablock.c.o
[ 93%] Building C object CMakeFiles/brotlienc.dir/c/enc/static_dict.c.o
[ 96%] Building C object CMakeFiles/brotlienc.dir/c/enc/utf8_util.c.o
[100%] Linking C static library libbrotlienc.a
[100%] Built target brotlienc
```
---
Переходим в директорию с исходниками nginx, прописываем модуль ngx_brotli в конфиге сборки пакета
```
root@UbuntuTestVirt:~/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/out# cd ../../../..
root@UbuntuTestVirt:~/custom-nginx/nginx-1.24.0# pwd
/root/custom-nginx/nginx-1.24.0
root@UbuntuTestVirt:~/custom-nginx/nginx-1.24.0# nano debian/rules
```
```
common_configure_flags := \
                        --add-module=$(CURDIR)/ngx_brotli \
                        --with-cc-opt="$(debian_cflags)" \
                        --with-ld-opt="$(debian_ldflags)" \
                        $(basic_configure_flags)
```
---
Меняем версию
```
root@UbuntuTestVirt:~/custom-nginx/nginx-1.24.0# nano debian/changelog
```
```
nginx (1.24.0-2ubuntu7-custom) noble; urgency=medium
```
---
<details>
  <summary>Собираем пакет</summary>
    <code>root@UbuntuTestVirt:~/custom-nginx/nginx-1.24.0# dpkg-buildpackage -b
dpkg-buildpackage: info: source package nginx
dpkg-buildpackage: info: source version 1.24.0-2ubuntu7-custom
dpkg-buildpackage: info: source distribution noble
dpkg-buildpackage: info: source changed by Steve Langasek <steve.langasek@ubuntu.com>
dpkg-buildpackage: info: host architecture amd64
 dpkg-source --before-build .
 debian/rules clean
dh clean --without autoreconf
   debian/rules override_dh_clean
make[1]: Entering directory '/root/custom-nginx/nginx-1.24.0'
rm -rf /root/custom-nginx/nginx-1.24.0/debian/build-bin
rm -rf /root/custom-nginx/nginx-1.24.0/debian/build-src
dh_clean
        rm -f debian/debhelper-build-stamp
        rm -rf debian/.debhelper/
        rm -f -- debian/nginx.substvars debian/nginx-doc.substvars debian/nginx-common.substvars debian/nginx-dev.substvars
debian/nginx-core.substvars debian/nginx-full.substvars debian/nginx-light.substvars debian/nginx-extras.substvars debian/libnginx-mod-http-geoip.substvars debian/libnginx-mod-http-image-filter.substvars debian/libnginx-mod-http-xslt-filter.substvars debian/libnginx-mod-mail.substvars debian/libnginx-mod-stream.substvars debian/libnginx-mod-stream-geoip.substvars debian/libnginx-mod-http-perl.substvars debian/files
        rm -fr -- debian/nginx/ debian/tmp/ debian/nginx-doc/ debian/nginx-common/ debian/nginx-dev/ debian/nginx-core/ debian/nginx-full/ debian/nginx-light/ debian/nginx-extras/ debian/libnginx-mod-http-geoip/ debian/libnginx-mod-http-image-filter/ debian/libnginx-mod-http-xslt-filter/ debian/libnginx-mod-mail/ debian/libnginx-mod-stream/ debian/libnginx-mod-stream-geoip/ debian/libnginx-mod-http-perl/
        find .  \( \( \
                \( -path .\*/.git -o -path .\*/.svn -o -path .\*/.bzr -o -path .\*/.hg -o -path .\*/CVS -o -path .\*/.pc -o
-path .\*/_darcs \) -prune -o -type f -a \
                \( -name '#*#' -o -name '.*~' -o -name '*~' -o -name DEADJOE \
                 -o -name '*.orig' -o -name '*.rej' -o -name '*.bak' \
                 -o -name '.*.orig' -o -name .*.rej -o -name '.SUMS' \
                 -o -name TAGS -o \( -path '*/.deps/*' -a -name '*.P' \) \
                \) -exec rm -f {} + \) -o \
                \( -type d -a \( -name autom4te.cache -o -name __pycache__ \) -prune -exec rm -rf {} + \) \)
make[1]: Leaving directory '/root/custom-nginx/nginx-1.24.0'
 debian/rules binary
dh binary --without autoreconf
   dh_update_autotools_config
   debian/rules override_dh_auto_configure
make[1]: Entering directory '/root/custom-nginx/nginx-1.24.0'
dh_testdir
dh_testdir
mkdir -p /root/custom-nginx/nginx-1.24.0/debian/build-src
mkdir -p /root/custom-nginx/nginx-1.24.0/debian/build-bin
cp -Pa /root/custom-nginx/nginx-1.24.0/auto /root/custom-nginx/nginx-1.24.0/debian/build-bin/
cp -Pa /root/custom-nginx/nginx-1.24.0/conf /root/custom-nginx/nginx-1.24.0/debian/build-bin/
cp -Pa /root/custom-nginx/nginx-1.24.0/configure /root/custom-nginx/nginx-1.24.0/debian/build-bin/
cp -Pa /root/custom-nginx/nginx-1.24.0/contrib /root/custom-nginx/nginx-1.24.0/debian/build-bin/
cp -Pa /root/custom-nginx/nginx-1.24.0/src /root/custom-nginx/nginx-1.24.0/debian/build-bin/
cp -Pa /root/custom-nginx/nginx-1.24.0/man /root/custom-nginx/nginx-1.24.0/debian/build-bin/
cd /root/custom-nginx/nginx-1.24.0/debian/build-bin && ./configure --add-module=/root/custom-nginx/nginx-1.24.0/ngx_brotli --with-cc-opt="-g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3" --with-ld-opt="-Wl,-Bsymbolic-functions -flto=auto -ffat-lto-objects -Wl,-z,relro -Wl,-z,now -fPIC" --prefix=/usr/share/nginx --conf-path=/etc/nginx/nginx.conf --http-log-path=/var/log/nginx/access.log --error-log-path=stderr --lock-path=/var/lock/nginx.lock --pid-path=/run/nginx.pid --modules-path=/usr/lib/nginx/modules --http-client-body-temp-path=/var/lib/nginx/body --http-fastcgi-temp-path=/var/lib/nginx/fastcgi --http-proxy-temp-path=/var/lib/nginx/proxy --http-scgi-temp-path=/var/lib/nginx/scgi --http-uwsgi-temp-path=/var/lib/nginx/uwsgi --with-compat --with-debug --with-pcre-jit --with-http_ssl_module --with-http_stub_status_module --with-http_realip_module --with-http_auth_request_module --with-http_v2_module --with-http_dav_module --with-http_slice_module --with-threads --with-http_addition_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_secure_link_module --with-http_sub_module --with-mail_ssl_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-stream_realip_module --with-http_geoip_module=dynamic --with-http_image_filter_module=dynamic --with-http_perl_module=dynamic --with-http_xslt_module=dynamic --with-mail=dynamic --with-stream=dynamic --with-stream_geoip_module=dynamic
checking for OS
 + Linux 6.8.0-62-generic x86_64
checking for C compiler ... found
 + using GNU C compiler
checking for --with-ld-opt="-Wl,-Bsymbolic-functions -flto=auto -ffat-lto-objects -Wl,-z,relro -Wl,-z,now -fPIC" ... found
checking for -Wl,-E switch ... found
checking for gcc builtin atomic operations ... found
checking for C99 variadic macros ... found
checking for gcc variadic macros ... found
checking for gcc builtin 64 bit byteswap ... found
checking for unistd.h ... found
checking for inttypes.h ... found
checking for limits.h ... found
checking for sys/filio.h ... not found
checking for sys/param.h ... found
checking for sys/mount.h ... found
checking for sys/statvfs.h ... found
checking for crypt.h ... found
checking for Linux specific features
checking for epoll ... found
checking for EPOLLRDHUP ... found
checking for EPOLLEXCLUSIVE ... found
checking for eventfd() ... found
checking for O_PATH ... found
checking for sendfile() ... found
checking for sendfile64() ... found
checking for sys/prctl.h ... found
checking for prctl(PR_SET_DUMPABLE) ... found
checking for prctl(PR_SET_KEEPCAPS) ... found
checking for capabilities ... found
checking for crypt_r() ... found
checking for sys/vfs.h ... found
checking for UDP_SEGMENT ... found
checking for nobody group ... not found
checking for nogroup group ... found
checking for poll() ... found
checking for /dev/poll ... not found
checking for kqueue ... not found
checking for crypt() ... not found
checking for crypt() in libcrypt ... found
checking for F_READAHEAD ... not found
checking for posix_fadvise() ... found
checking for O_DIRECT ... found
checking for F_NOCACHE ... not found
checking for directio() ... not found
checking for statfs() ... found
checking for statvfs() ... found
checking for dlopen() ... found
checking for sched_yield() ... found
checking for sched_setaffinity() ... found
checking for SO_SETFIB ... not found
checking for SO_REUSEPORT ... found
checking for SO_ACCEPTFILTER ... not found
checking for SO_BINDANY ... not found
checking for IP_TRANSPARENT ... found
checking for IP_BINDANY ... not found
checking for IP_BIND_ADDRESS_NO_PORT ... found
checking for IP_RECVDSTADDR ... not found
checking for IP_SENDSRCADDR ... not found
checking for IP_PKTINFO ... found
checking for IPV6_RECVPKTINFO ... found
checking for TCP_DEFER_ACCEPT ... found
checking for TCP_KEEPIDLE ... found
checking for TCP_FASTOPEN ... found
checking for TCP_INFO ... found
checking for accept4() ... found
checking for int size ... 4 bytes
checking for long size ... 8 bytes
checking for long long size ... 8 bytes
checking for void * size ... 8 bytes
checking for uint32_t ... found
checking for uint64_t ... found
checking for sig_atomic_t ... found
checking for sig_atomic_t size ... 4 bytes
checking for socklen_t ... found
checking for in_addr_t ... found
checking for in_port_t ... found
checking for rlim_t ... found
checking for uintptr_t ... uintptr_t found
checking for system byte ordering ... little endian
checking for size_t size ... 8 bytes
checking for off_t size ... 8 bytes
checking for time_t size ... 8 bytes
checking for AF_INET6 ... found
checking for setproctitle() ... not found
checking for pread() ... found
checking for pwrite() ... found
checking for pwritev() ... found
checking for strerrordesc_np() ... found
checking for localtime_r() ... found
checking for clock_gettime(CLOCK_MONOTONIC) ... found
checking for posix_memalign() ... found
checking for memalign() ... found
checking for mmap(MAP_ANON|MAP_SHARED) ... found
checking for mmap("/dev/zero", MAP_SHARED) ... found
checking for System V shared memory ... found
checking for POSIX semaphores ... found
checking for struct msghdr.msg_control ... found
checking for ioctl(FIONBIO) ... found
checking for ioctl(FIONREAD) ... found
checking for struct tm.tm_gmtoff ... found
checking for struct dirent.d_namlen ... not found
checking for struct dirent.d_type ... found
checking for sysconf(_SC_NPROCESSORS_ONLN) ... found
checking for sysconf(_SC_LEVEL1_DCACHE_LINESIZE) ... found
checking for openat(), fstatat() ... found
checking for getaddrinfo() ... found
configuring additional modules
adding module in /root/custom-nginx/nginx-1.24.0/ngx_brotli
 + ngx_brotli was configured
checking for PCRE2 library ... found
checking for OpenSSL library ... found
checking for zlib library ... found
checking for libxslt ... found
checking for libexslt ... found
checking for GD library ... found
checking for GD WebP support ... found
checking for perl
 + perl version: This is perl 5, version 38, subversion 2 (v5.38.2) built for x86_64-linux-gnu-thread-multi
 + perl interpreter multiplicity found
checking for GeoIP library ... found
checking for GeoIP IPv6 support ... found
creating objs/Makefile

Configuration summary
  + using threads
  + using system PCRE2 library
  + using system OpenSSL library
  + using system zlib library

  nginx path prefix: "/usr/share/nginx"
  nginx binary file: "/usr/share/nginx/sbin/nginx"
  nginx modules path: "/usr/lib/nginx/modules"
  nginx configuration prefix: "/etc/nginx"
  nginx configuration file: "/etc/nginx/nginx.conf"
  nginx pid file: "/run/nginx.pid"
  nginx logs errors to stderr
  nginx http access log file: "/var/log/nginx/access.log"
  nginx http client request body temporary files: "/var/lib/nginx/body"
  nginx http proxy temporary files: "/var/lib/nginx/proxy"
  nginx http fastcgi temporary files: "/var/lib/nginx/fastcgi"
  nginx http uwsgi temporary files: "/var/lib/nginx/uwsgi"
  nginx http scgi temporary files: "/var/lib/nginx/scgi"

dh override_dh_auto_configure --without autoreconf
make[1]: Leaving directory '/root/custom-nginx/nginx-1.24.0'
   debian/rules override_dh_auto_build
make[1]: Entering directory '/root/custom-nginx/nginx-1.24.0'
/usr/bin/make -C /root/custom-nginx/nginx-1.24.0/debian/build-bin build
make[2]: Entering directory '/root/custom-nginx/nginx-1.24.0/debian/build-bin'
cp -Pa /root/custom-nginx/nginx-1.24.0/auto /root/custom-nginx/nginx-1.24.0/debian/build-src/
/usr/bin/make -f objs/Makefile
sed -i '/^# create Makefile/,/^END$/d' /root/custom-nginx/nginx-1.24.0/debian/build-src/auto/make /root/custom-nginx/nginx-1.24.0/debian/build-src/auto/init /root/custom-nginx/nginx-1.24.0/debian/build-src/auto/install
find /root/custom-nginx/nginx-1.24.0/src -type f -name '*.h' -printf 'src/%P\0' | tar -C /root/custom-nginx/nginx-1.24.0 --null --files-from - -c | tar -C /root/custom-nginx/nginx-1.24.0/debian/build-src/ -x
make[3]: Entering directory '/root/custom-nginx/nginx-1.24.0/debian/build-bin'
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/nginx.o \
        src/core/nginx.c
if [ -e /root/custom-nginx/nginx-1.24.0/configure ]; then cp /root/custom-nginx/nginx-1.24.0/configure /root/custom-nginx/nginx-1.24.0/debian/build-src/; fi
echo "NGX_CONF_FLAGS=(" --prefix=/usr/share/nginx --conf-path=/etc/nginx/nginx.conf --http-log-path=/var/log/nginx/access.log --error-log-path=stderr --lock-path=/var/lock/nginx.lock --pid-path=/run/nginx.pid --modules-path=/usr/lib/nginx/modules --http-client-body-temp-path=/var/lib/nginx/body --http-fastcgi-temp-path=/var/lib/nginx/fastcgi --http-proxy-temp-path=/var/lib/nginx/proxy --http-scgi-temp-path=/var/lib/nginx/scgi --http-uwsgi-temp-path=/var/lib/nginx/uwsgi --with-compat --with-debug --with-pcre-jit --with-http_ssl_module --with-http_stub_status_module --with-http_realip_module --with-http_auth_request_module --with-http_v2_module --with-http_dav_module --with-http_slice_module --with-threads ")" > /root/custom-nginx/nginx-1.24.0/debian/build-src/conf_flags
pod2man debian/debhelper/dh_nginx > /root/custom-nginx/nginx-1.24.0/debian/build-src/dh_nginx.1
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_log.o \
        src/core/ngx_log.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_palloc.o \
        src/core/ngx_palloc.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_array.o \
        src/core/ngx_array.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_list.o \
        src/core/ngx_list.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_hash.o \
        src/core/ngx_hash.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_buf.o \
        src/core/ngx_buf.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_queue.o \
        src/core/ngx_queue.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_output_chain.o \
        src/core/ngx_output_chain.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_string.o \
        src/core/ngx_string.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_parse.o \
        src/core/ngx_parse.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_parse_time.o \
        src/core/ngx_parse_time.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_inet.o \
        src/core/ngx_inet.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_file.o \
        src/core/ngx_file.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_crc32.o \
        src/core/ngx_crc32.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_murmurhash.o \
        src/core/ngx_murmurhash.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_md5.o \
        src/core/ngx_md5.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_sha1.o \
        src/core/ngx_sha1.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_rbtree.o \
        src/core/ngx_rbtree.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_radix_tree.o \
        src/core/ngx_radix_tree.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_slab.o \
        src/core/ngx_slab.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_times.o \
        src/core/ngx_times.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_shmtx.o \
        src/core/ngx_shmtx.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_connection.o \
        src/core/ngx_connection.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_cycle.o \
        src/core/ngx_cycle.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_spinlock.o \
        src/core/ngx_spinlock.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_rwlock.o \
        src/core/ngx_rwlock.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_cpuinfo.o \
        src/core/ngx_cpuinfo.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_conf_file.o \
        src/core/ngx_conf_file.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_module.o \
        src/core/ngx_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_resolver.o \
        src/core/ngx_resolver.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_open_file_cache.o \
        src/core/ngx_open_file_cache.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_crypt.o \
        src/core/ngx_crypt.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_proxy_protocol.o \
        src/core/ngx_proxy_protocol.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_syslog.o \
        src/core/ngx_syslog.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/event/ngx_event.o \
        src/event/ngx_event.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/event/ngx_event_timer.o \
        src/event/ngx_event_timer.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/event/ngx_event_posted.o \
        src/event/ngx_event_posted.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/event/ngx_event_accept.o \
        src/event/ngx_event_accept.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/event/ngx_event_udp.o \
        src/event/ngx_event_udp.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/event/ngx_event_connect.o \
        src/event/ngx_event_connect.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/event/ngx_event_pipe.o \
        src/event/ngx_event_pipe.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/os/unix/ngx_time.o \
        src/os/unix/ngx_time.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/os/unix/ngx_errno.o \
        src/os/unix/ngx_errno.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/os/unix/ngx_alloc.o \
        src/os/unix/ngx_alloc.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/os/unix/ngx_files.o \
        src/os/unix/ngx_files.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/os/unix/ngx_socket.o \
        src/os/unix/ngx_socket.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/os/unix/ngx_recv.o \
        src/os/unix/ngx_recv.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/os/unix/ngx_readv_chain.o \
        src/os/unix/ngx_readv_chain.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/os/unix/ngx_udp_recv.o \
        src/os/unix/ngx_udp_recv.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/os/unix/ngx_send.o \
        src/os/unix/ngx_send.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/os/unix/ngx_writev_chain.o \
        src/os/unix/ngx_writev_chain.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/os/unix/ngx_udp_send.o \
        src/os/unix/ngx_udp_send.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/os/unix/ngx_udp_sendmsg_chain.o \
        src/os/unix/ngx_udp_sendmsg_chain.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/os/unix/ngx_channel.o \
        src/os/unix/ngx_channel.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/os/unix/ngx_shmem.o \
        src/os/unix/ngx_shmem.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/os/unix/ngx_process.o \
        src/os/unix/ngx_process.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/os/unix/ngx_daemon.o \
        src/os/unix/ngx_daemon.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/os/unix/ngx_setaffinity.o \
        src/os/unix/ngx_setaffinity.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/os/unix/ngx_setproctitle.o \
        src/os/unix/ngx_setproctitle.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/os/unix/ngx_posix_init.o \
        src/os/unix/ngx_posix_init.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/os/unix/ngx_user.o \
        src/os/unix/ngx_user.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/os/unix/ngx_dlopen.o \
        src/os/unix/ngx_dlopen.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/os/unix/ngx_process_cycle.o \
        src/os/unix/ngx_process_cycle.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/os/unix/ngx_linux_init.o \
        src/os/unix/ngx_linux_init.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/event/modules/ngx_epoll_module.o \
        src/event/modules/ngx_epoll_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/os/unix/ngx_linux_sendfile_chain.o \
        src/os/unix/ngx_linux_sendfile_chain.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_thread_pool.o \
        src/core/ngx_thread_pool.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/os/unix/ngx_thread_cond.o \
        src/os/unix/ngx_thread_cond.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/os/unix/ngx_thread_mutex.o \
        src/os/unix/ngx_thread_mutex.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/os/unix/ngx_thread_id.o \
        src/os/unix/ngx_thread_id.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/event/ngx_event_openssl.o \
        src/event/ngx_event_openssl.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/event/ngx_event_openssl_stapling.o \
        src/event/ngx_event_openssl_stapling.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/src/core/ngx_regex.o \
        src/core/ngx_regex.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/ngx_http.o \
        src/http/ngx_http.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/ngx_http_core_module.o \
        src/http/ngx_http_core_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/ngx_http_special_response.o \
        src/http/ngx_http_special_response.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/ngx_http_request.o \
        src/http/ngx_http_request.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/ngx_http_parse.o \
        src/http/ngx_http_parse.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_log_module.o \
        src/http/modules/ngx_http_log_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/ngx_http_request_body.o \
        src/http/ngx_http_request_body.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/ngx_http_variables.o \
        src/http/ngx_http_variables.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/ngx_http_script.o \
        src/http/ngx_http_script.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/ngx_http_upstream.o \
        src/http/ngx_http_upstream.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/ngx_http_upstream_round_robin.o \
        src/http/ngx_http_upstream_round_robin.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/ngx_http_file_cache.o \
        src/http/ngx_http_file_cache.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/ngx_http_huff_decode.o \
        src/http/ngx_http_huff_decode.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/ngx_http_huff_encode.o \
        src/http/ngx_http_huff_encode.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/ngx_http_write_filter_module.o \
        src/http/ngx_http_write_filter_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/ngx_http_header_filter_module.o \
        src/http/ngx_http_header_filter_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_chunked_filter_module.o \
        src/http/modules/ngx_http_chunked_filter_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/v2/ngx_http_v2_filter_module.o \
        src/http/v2/ngx_http_v2_filter_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_range_filter_module.o \
        src/http/modules/ngx_http_range_filter_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_gzip_filter_module.o \
        src/http/modules/ngx_http_gzip_filter_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/ngx_http_postpone_filter_module.o \
        src/http/ngx_http_postpone_filter_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_ssi_filter_module.o \
        src/http/modules/ngx_http_ssi_filter_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_charset_filter_module.o \
        src/http/modules/ngx_http_charset_filter_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_sub_filter_module.o \
        src/http/modules/ngx_http_sub_filter_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_addition_filter_module.o \
        src/http/modules/ngx_http_addition_filter_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_gunzip_filter_module.o \
        src/http/modules/ngx_http_gunzip_filter_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_userid_filter_module.o \
        src/http/modules/ngx_http_userid_filter_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_headers_filter_module.o \
        src/http/modules/ngx_http_headers_filter_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/ngx_http_copy_filter_module.o \
        src/http/ngx_http_copy_filter_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_not_modified_filter_module.o \
        src/http/modules/ngx_http_not_modified_filter_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_slice_filter_module.o \
        src/http/modules/ngx_http_slice_filter_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/v2/ngx_http_v2.o \
        src/http/v2/ngx_http_v2.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/v2/ngx_http_v2_table.o \
        src/http/v2/ngx_http_v2_table.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/v2/ngx_http_v2_encode.o \
        src/http/v2/ngx_http_v2_encode.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/v2/ngx_http_v2_module.o \
        src/http/v2/ngx_http_v2_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_static_module.o \
        src/http/modules/ngx_http_static_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_gzip_static_module.o \
        src/http/modules/ngx_http_gzip_static_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_dav_module.o \
        src/http/modules/ngx_http_dav_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_autoindex_module.o \
        src/http/modules/ngx_http_autoindex_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_index_module.o \
        src/http/modules/ngx_http_index_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_random_index_module.o \
        src/http/modules/ngx_http_random_index_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_mirror_module.o \
        src/http/modules/ngx_http_mirror_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_try_files_module.o \
        src/http/modules/ngx_http_try_files_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_auth_request_module.o \
        src/http/modules/ngx_http_auth_request_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_auth_basic_module.o \
        src/http/modules/ngx_http_auth_basic_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_access_module.o \
        src/http/modules/ngx_http_access_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_limit_conn_module.o \
        src/http/modules/ngx_http_limit_conn_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_limit_req_module.o \
        src/http/modules/ngx_http_limit_req_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_realip_module.o \
        src/http/modules/ngx_http_realip_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_geo_module.o \
        src/http/modules/ngx_http_geo_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_map_module.o \
        src/http/modules/ngx_http_map_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_split_clients_module.o \
        src/http/modules/ngx_http_split_clients_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_referer_module.o \
        src/http/modules/ngx_http_referer_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_rewrite_module.o \
        src/http/modules/ngx_http_rewrite_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_ssl_module.o \
        src/http/modules/ngx_http_ssl_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_proxy_module.o \
        src/http/modules/ngx_http_proxy_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_fastcgi_module.o \
        src/http/modules/ngx_http_fastcgi_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_uwsgi_module.o \
        src/http/modules/ngx_http_uwsgi_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_scgi_module.o \
        src/http/modules/ngx_http_scgi_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_grpc_module.o \
        src/http/modules/ngx_http_grpc_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_memcached_module.o \
        src/http/modules/ngx_http_memcached_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_empty_gif_module.o \
        src/http/modules/ngx_http_empty_gif_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_browser_module.o \
        src/http/modules/ngx_http_browser_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_secure_link_module.o \
        src/http/modules/ngx_http_secure_link_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_flv_module.o \
        src/http/modules/ngx_http_flv_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_mp4_module.o \
        src/http/modules/ngx_http_mp4_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_upstream_hash_module.o \
        src/http/modules/ngx_http_upstream_hash_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_upstream_ip_hash_module.o \
        src/http/modules/ngx_http_upstream_ip_hash_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_upstream_least_conn_module.o \
        src/http/modules/ngx_http_upstream_least_conn_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_upstream_random_module.o \
        src/http/modules/ngx_http_upstream_random_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_upstream_keepalive_module.o \
        src/http/modules/ngx_http_upstream_keepalive_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_upstream_zone_module.o \
        src/http/modules/ngx_http_upstream_zone_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include \
        -o objs/src/http/modules/ngx_http_stub_status_module.o \
        src/http/modules/ngx_http_stub_status_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations  -I src/core
-I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/addon/filter/ngx_http_brotli_filter_module.o \
        /root/custom-nginx/nginx-1.24.0/ngx_brotli/filter/ngx_http_brotli_filter_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations  -I src/core
-I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/addon/static/ngx_http_brotli_static_module.o \
        /root/custom-nginx/nginx-1.24.0/ngx_brotli/static/ngx_http_brotli_static_module.c
cc -c -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs \
        -o objs/ngx_modules.o \
        objs/ngx_modules.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/http/modules/ngx_http_xslt_filter_module.o \
        src/http/modules/ngx_http_xslt_filter_module.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/ngx_http_xslt_filter_module_modules.o \
        objs/ngx_http_xslt_filter_module_modules.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/http/modules/ngx_http_image_filter_module.o \
        src/http/modules/ngx_http_image_filter_module.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/ngx_http_image_filter_module_modules.o \
        objs/ngx_http_image_filter_module_modules.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/http/modules/ngx_http_geoip_module.o \
        src/http/modules/ngx_http_geoip_module.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/ngx_http_geoip_module_modules.o \
        objs/ngx_http_geoip_module_modules.c
grep 'define NGINX_VERSION' src/core/nginx.h \
        | sed -e 's/^.*"\(.*\)".*/\1/' > \
        objs/src/http/modules/perl/version
sed "s/%%VERSION%%/`cat objs/src/http/modules/perl/version`/" \
        src/http/modules/perl/nginx.pm > \
        objs/src/http/modules/perl/nginx.pm
cp -p src/http/modules/perl/nginx.xs objs/src/http/modules/perl/
cp -p src/http/modules/perl/typemap objs/src/http/modules/perl/
cp -p src/http/modules/perl/Makefile.PL objs/src/http/modules/perl/
cd objs/src/http/modules/perl \
        && NGX_PM_CFLAGS="-D_REENTRANT -D_GNU_SOURCE -DDEBIAN -fwrapv -fno-strict-aliasing -pipe -I/usr/local/include -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64 -I/usr/lib/x86_64-linux-gnu/perl/5.38/CORE -g -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3" \
                NGX_PM_LDFLAGS="-Wl,-Bsymbolic-functions -flto=auto -ffat-lto-objects -Wl,-z,relro -Wl,-z,now -fPIC -shared
-L/usr/local/lib -fstack-protector-strong" \
                NGX_INCS="src/core src/event src/event/modules src/os/unix src/http/modules/perl   /usr/include/libxml2   objs  src/http src/http/modules src/http/v2 /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include" \
                NGX_DEPS="src/core/nginx.h src/core/ngx_config.h src/core/ngx_core.h src/core/ngx_log.h src/core/ngx_palloc.h src/core/ngx_array.h src/core/ngx_list.h src/core/ngx_hash.h src/core/ngx_buf.h src/core/ngx_queue.h src/core/ngx_string.h src/core/ngx_parse.h src/core/ngx_parse_time.h src/core/ngx_inet.h src/core/ngx_file.h src/core/ngx_crc.h src/core/ngx_crc32.h src/core/ngx_murmurhash.h src/core/ngx_md5.h src/core/ngx_sha1.h src/core/ngx_rbtree.h src/core/ngx_radix_tree.h src/core/ngx_rwlock.h src/core/ngx_slab.h src/core/ngx_times.h src/core/ngx_shmtx.h src/core/ngx_connection.h src/core/ngx_cycle.h
src/core/ngx_conf_file.h src/core/ngx_module.h src/core/ngx_resolver.h src/core/ngx_open_file_cache.h src/core/ngx_crypt.h src/core/ngx_proxy_protocol.h src/core/ngx_syslog.h src/event/ngx_event.h src/event/ngx_event_timer.h src/event/ngx_event_posted.h src/event/ngx_event_connect.h src/event/ngx_event_pipe.h src/event/ngx_event_udp.h src/os/unix/ngx_time.h src/os/unix/ngx_errno.h src/os/unix/ngx_alloc.h src/os/unix/ngx_files.h src/os/unix/ngx_channel.h src/os/unix/ngx_shmem.h src/os/unix/ngx_process.h src/os/unix/ngx_setaffinity.h src/os/unix/ngx_setproctitle.h src/os/unix/ngx_atomic.h src/os/unix/ngx_gcc_atomic_x86.h src/os/unix/ngx_thread.h src/os/unix/ngx_socket.h src/os/unix/ngx_os.h src/os/unix/ngx_user.h src/os/unix/ngx_dlopen.h src/os/unix/ngx_process_cycle.h src/os/unix/ngx_linux_config.h src/os/unix/ngx_linux.h src/core/ngx_thread_pool.h src/event/ngx_event_openssl.h src/core/ngx_regex.h objs/ngx_auto_config.h src/http/ngx_http.h src/http/ngx_http_request.h src/http/ngx_http_config.h src/http/ngx_http_core_module.h src/http/ngx_http_cache.h src/http/ngx_http_variables.h src/http/ngx_http_script.h src/http/ngx_http_upstream.h src/http/ngx_http_upstream_round_robin.h src/http/modules/ngx_http_ssi_filter_module.h
src/http/v2/ngx_http_v2.h src/http/v2/ngx_http_v2_module.h src/http/modules/ngx_http_ssl_module.h" \
        perl Makefile.PL \
                LIB= \
                INSTALLSITEMAN3DIR=
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/ngx_http_perl_module_modules.o \
        objs/ngx_http_perl_module_modules.c
Generating a Unix-style Makefile
Writing Makefile for nginx
Writing MYMETA.yml and MYMETA.json
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/stream/ngx_stream_geoip_module.o \
        src/stream/ngx_stream_geoip_module.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/ngx_stream_geoip_module_modules.o \
        objs/ngx_stream_geoip_module_modules.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/mail/ngx_mail.o \
        src/mail/ngx_mail.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/mail/ngx_mail_core_module.o \
        src/mail/ngx_mail_core_module.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/mail/ngx_mail_handler.o \
        src/mail/ngx_mail_handler.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/mail/ngx_mail_parse.o \
        src/mail/ngx_mail_parse.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/mail/ngx_mail_ssl_module.o \
        src/mail/ngx_mail_ssl_module.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/mail/ngx_mail_pop3_module.o \
        src/mail/ngx_mail_pop3_module.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/mail/ngx_mail_pop3_handler.o \
        src/mail/ngx_mail_pop3_handler.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/mail/ngx_mail_imap_module.o \
        src/mail/ngx_mail_imap_module.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/mail/ngx_mail_imap_handler.o \
        src/mail/ngx_mail_imap_handler.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/mail/ngx_mail_smtp_module.o \
        src/mail/ngx_mail_smtp_module.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/mail/ngx_mail_smtp_handler.o \
        src/mail/ngx_mail_smtp_handler.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/mail/ngx_mail_auth_http_module.o \
        src/mail/ngx_mail_auth_http_module.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/mail/ngx_mail_proxy_module.o \
        src/mail/ngx_mail_proxy_module.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/mail/ngx_mail_realip_module.o \
        src/mail/ngx_mail_realip_module.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/ngx_mail_module_modules.o \
        objs/ngx_mail_module_modules.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/stream/ngx_stream.o \
        src/stream/ngx_stream.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/stream/ngx_stream_variables.o \
        src/stream/ngx_stream_variables.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/stream/ngx_stream_script.o \
        src/stream/ngx_stream_script.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/stream/ngx_stream_handler.o \
        src/stream/ngx_stream_handler.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/stream/ngx_stream_core_module.o \
        src/stream/ngx_stream_core_module.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/stream/ngx_stream_log_module.o \
        src/stream/ngx_stream_log_module.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/stream/ngx_stream_proxy_module.o \
        src/stream/ngx_stream_proxy_module.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/stream/ngx_stream_upstream.o \
        src/stream/ngx_stream_upstream.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/stream/ngx_stream_upstream_round_robin.o \
        src/stream/ngx_stream_upstream_round_robin.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/stream/ngx_stream_write_filter_module.o \
        src/stream/ngx_stream_write_filter_module.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/stream/ngx_stream_ssl_module.o \
        src/stream/ngx_stream_ssl_module.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/stream/ngx_stream_realip_module.o \
        src/stream/ngx_stream_realip_module.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/stream/ngx_stream_limit_conn_module.o \
        src/stream/ngx_stream_limit_conn_module.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/stream/ngx_stream_access_module.o \
        src/stream/ngx_stream_access_module.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/stream/ngx_stream_geo_module.o \
        src/stream/ngx_stream_geo_module.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/stream/ngx_stream_map_module.o \
        src/stream/ngx_stream_map_module.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/stream/ngx_stream_split_clients_module.o \
        src/stream/ngx_stream_split_clients_module.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/stream/ngx_stream_return_module.o \
        src/stream/ngx_stream_return_module.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/stream/ngx_stream_set_module.o \
        src/stream/ngx_stream_set_module.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/stream/ngx_stream_upstream_hash_module.o \
        src/stream/ngx_stream_upstream_hash_module.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/stream/ngx_stream_upstream_least_conn_module.o \
        src/stream/ngx_stream_upstream_least_conn_module.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/stream/ngx_stream_upstream_random_module.o \
        src/stream/ngx_stream_upstream_random_module.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/stream/ngx_stream_upstream_zone_module.o \
        src/stream/ngx_stream_upstream_zone_module.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/stream/ngx_stream_ssl_preread_module.o \
        src/stream/ngx_stream_ssl_preread_module.c
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/ngx_stream_module_modules.o \
        objs/ngx_stream_module_modules.c
sed -e "s|%%PREFIX%%|/usr/share/nginx|" \
        -e "s|%%PID_PATH%%|/run/nginx.pid|" \
        -e "s|%%CONF_PATH%%|/etc/nginx/nginx.conf|" \
        -e "s|%%ERROR_LOG_PATH%%|stderr|" \
        < man/nginx.8 > objs/nginx.8
cc -o objs/nginx \
objs/src/core/nginx.o \
objs/src/core/ngx_log.o \
objs/src/core/ngx_palloc.o \
objs/src/core/ngx_array.o \
objs/src/core/ngx_list.o \
objs/src/core/ngx_hash.o \
objs/src/core/ngx_buf.o \
objs/src/core/ngx_queue.o \
objs/src/core/ngx_output_chain.o \
objs/src/core/ngx_string.o \
objs/src/core/ngx_parse.o \
objs/src/core/ngx_parse_time.o \
objs/src/core/ngx_inet.o \
objs/src/core/ngx_file.o \
objs/src/core/ngx_crc32.o \
objs/src/core/ngx_murmurhash.o \
objs/src/core/ngx_md5.o \
objs/src/core/ngx_sha1.o \
objs/src/core/ngx_rbtree.o \
objs/src/core/ngx_radix_tree.o \
objs/src/core/ngx_slab.o \
objs/src/core/ngx_times.o \
objs/src/core/ngx_shmtx.o \
objs/src/core/ngx_connection.o \
objs/src/core/ngx_cycle.o \
objs/src/core/ngx_spinlock.o \
objs/src/core/ngx_rwlock.o \
objs/src/core/ngx_cpuinfo.o \
objs/src/core/ngx_conf_file.o \
objs/src/core/ngx_module.o \
objs/src/core/ngx_resolver.o \
objs/src/core/ngx_open_file_cache.o \
objs/src/core/ngx_crypt.o \
objs/src/core/ngx_proxy_protocol.o \
objs/src/core/ngx_syslog.o \
objs/src/event/ngx_event.o \
objs/src/event/ngx_event_timer.o \
objs/src/event/ngx_event_posted.o \
objs/src/event/ngx_event_accept.o \
objs/src/event/ngx_event_udp.o \
objs/src/event/ngx_event_connect.o \
objs/src/event/ngx_event_pipe.o \
objs/src/os/unix/ngx_time.o \
objs/src/os/unix/ngx_errno.o \
objs/src/os/unix/ngx_alloc.o \
objs/src/os/unix/ngx_files.o \
objs/src/os/unix/ngx_socket.o \
objs/src/os/unix/ngx_recv.o \
objs/src/os/unix/ngx_readv_chain.o \
objs/src/os/unix/ngx_udp_recv.o \
objs/src/os/unix/ngx_send.o \
objs/src/os/unix/ngx_writev_chain.o \
objs/src/os/unix/ngx_udp_send.o \
objs/src/os/unix/ngx_udp_sendmsg_chain.o \
objs/src/os/unix/ngx_channel.o \
objs/src/os/unix/ngx_shmem.o \
objs/src/os/unix/ngx_process.o \
objs/src/os/unix/ngx_daemon.o \
objs/src/os/unix/ngx_setaffinity.o \
objs/src/os/unix/ngx_setproctitle.o \
objs/src/os/unix/ngx_posix_init.o \
objs/src/os/unix/ngx_user.o \
objs/src/os/unix/ngx_dlopen.o \
objs/src/os/unix/ngx_process_cycle.o \
objs/src/os/unix/ngx_linux_init.o \
objs/src/event/modules/ngx_epoll_module.o \
objs/src/os/unix/ngx_linux_sendfile_chain.o \
objs/src/core/ngx_thread_pool.o \
objs/src/os/unix/ngx_thread_cond.o \
objs/src/os/unix/ngx_thread_mutex.o \
objs/src/os/unix/ngx_thread_id.o \
objs/src/event/ngx_event_openssl.o \
objs/src/event/ngx_event_openssl_stapling.o \
objs/src/core/ngx_regex.o \
objs/src/http/ngx_http.o \
objs/src/http/ngx_http_core_module.o \
objs/src/http/ngx_http_special_response.o \
objs/src/http/ngx_http_request.o \
objs/src/http/ngx_http_parse.o \
objs/src/http/modules/ngx_http_log_module.o \
objs/src/http/ngx_http_request_body.o \
objs/src/http/ngx_http_variables.o \
objs/src/http/ngx_http_script.o \
objs/src/http/ngx_http_upstream.o \
objs/src/http/ngx_http_upstream_round_robin.o \
objs/src/http/ngx_http_file_cache.o \
objs/src/http/ngx_http_huff_decode.o \
objs/src/http/ngx_http_huff_encode.o \
objs/src/http/ngx_http_write_filter_module.o \
objs/src/http/ngx_http_header_filter_module.o \
objs/src/http/modules/ngx_http_chunked_filter_module.o \
objs/src/http/v2/ngx_http_v2_filter_module.o \
objs/src/http/modules/ngx_http_range_filter_module.o \
objs/src/http/modules/ngx_http_gzip_filter_module.o \
objs/src/http/ngx_http_postpone_filter_module.o \
objs/src/http/modules/ngx_http_ssi_filter_module.o \
objs/src/http/modules/ngx_http_charset_filter_module.o \
objs/src/http/modules/ngx_http_sub_filter_module.o \
objs/src/http/modules/ngx_http_addition_filter_module.o \
objs/src/http/modules/ngx_http_gunzip_filter_module.o \
objs/src/http/modules/ngx_http_userid_filter_module.o \
objs/src/http/modules/ngx_http_headers_filter_module.o \
objs/src/http/ngx_http_copy_filter_module.o \
objs/src/http/modules/ngx_http_not_modified_filter_module.o \
objs/src/http/modules/ngx_http_slice_filter_module.o \
objs/src/http/v2/ngx_http_v2.o \
objs/src/http/v2/ngx_http_v2_table.o \
objs/src/http/v2/ngx_http_v2_encode.o \
objs/src/http/v2/ngx_http_v2_module.o \
objs/src/http/modules/ngx_http_static_module.o \
objs/src/http/modules/ngx_http_gzip_static_module.o \
objs/src/http/modules/ngx_http_dav_module.o \
objs/src/http/modules/ngx_http_autoindex_module.o \
objs/src/http/modules/ngx_http_index_module.o \
objs/src/http/modules/ngx_http_random_index_module.o \
objs/src/http/modules/ngx_http_mirror_module.o \
objs/src/http/modules/ngx_http_try_files_module.o \
objs/src/http/modules/ngx_http_auth_request_module.o \
objs/src/http/modules/ngx_http_auth_basic_module.o \
objs/src/http/modules/ngx_http_access_module.o \
objs/src/http/modules/ngx_http_limit_conn_module.o \
objs/src/http/modules/ngx_http_limit_req_module.o \
objs/src/http/modules/ngx_http_realip_module.o \
objs/src/http/modules/ngx_http_geo_module.o \
objs/src/http/modules/ngx_http_map_module.o \
objs/src/http/modules/ngx_http_split_clients_module.o \
objs/src/http/modules/ngx_http_referer_module.o \
objs/src/http/modules/ngx_http_rewrite_module.o \
objs/src/http/modules/ngx_http_ssl_module.o \
objs/src/http/modules/ngx_http_proxy_module.o \
objs/src/http/modules/ngx_http_fastcgi_module.o \
objs/src/http/modules/ngx_http_uwsgi_module.o \
objs/src/http/modules/ngx_http_scgi_module.o \
objs/src/http/modules/ngx_http_grpc_module.o \
objs/src/http/modules/ngx_http_memcached_module.o \
objs/src/http/modules/ngx_http_empty_gif_module.o \
objs/src/http/modules/ngx_http_browser_module.o \
objs/src/http/modules/ngx_http_secure_link_module.o \
objs/src/http/modules/ngx_http_flv_module.o \
objs/src/http/modules/ngx_http_mp4_module.o \
objs/src/http/modules/ngx_http_upstream_hash_module.o \
objs/src/http/modules/ngx_http_upstream_ip_hash_module.o \
objs/src/http/modules/ngx_http_upstream_least_conn_module.o \
objs/src/http/modules/ngx_http_upstream_random_module.o \
objs/src/http/modules/ngx_http_upstream_keepalive_module.o \
objs/src/http/modules/ngx_http_upstream_zone_module.o \
objs/src/http/modules/ngx_http_stub_status_module.o \
objs/addon/filter/ngx_http_brotli_filter_module.o \
objs/addon/static/ngx_http_brotli_static_module.o \
objs/ngx_modules.o \
-Wl,-Bsymbolic-functions -flto=auto -ffat-lto-objects -Wl,-z,relro -Wl,-z,now -fPIC -lpthread -lcrypt -L/root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/../out -lbrotlienc -lbrotlicommon -lm -lpcre2-8 -lssl -lcrypto -lpthread -lz \
-Wl,-E
cc -o objs/ngx_http_xslt_filter_module.so \
objs/src/http/modules/ngx_http_xslt_filter_module.o \
objs/ngx_http_xslt_filter_module_modules.o \
-Wl,-Bsymbolic-functions -flto=auto -ffat-lto-objects -Wl,-z,relro -Wl,-z,now -fPIC -lxml2 -lxslt -lexslt \
-shared
cc -o objs/ngx_http_image_filter_module.so \
objs/src/http/modules/ngx_http_image_filter_module.o \
objs/ngx_http_image_filter_module_modules.o \
-Wl,-Bsymbolic-functions -flto=auto -ffat-lto-objects -Wl,-z,relro -Wl,-z,now -fPIC -lgd \
-shared
cc -o objs/ngx_http_geoip_module.so \
objs/src/http/modules/ngx_http_geoip_module.o \
objs/ngx_http_geoip_module_modules.o \
-Wl,-Bsymbolic-functions -flto=auto -ffat-lto-objects -Wl,-z,relro -Wl,-z,now -fPIC -lGeoIP \
-shared
cd objs/src/http/modules/perl && /usr/bin/make
make[4]: Entering directory '/root/custom-nginx/nginx-1.24.0/debian/build-bin/objs/src/http/modules/perl'
cp nginx.pm blib/lib/nginx.pm
Running Mkbootstrap for nginx ()
chmod 644 "nginx.bs"
"/usr/bin/perl" "/usr/share/perl/5.38/ExtUtils/xsubpp"  -typemap '/usr/share/perl/5.38/ExtUtils/typemap' -typemap '/root/custom-nginx/nginx-1.24.0/debian/build-bin/objs/src/http/modules/perl/typemap'  nginx.xs > nginx.xsc
mv nginx.xsc nginx.c
"/usr/bin/perl" -MExtUtils::Command::MM -e 'cp_nonempty' -- nginx.bs blib/arch/auto/nginx/nginx.bs 644
x86_64-linux-gnu-gcc -c  -I ../../../../../src/core -I ../../../../../src/event -I ../../../../../src/event/modules -I ../../../../../src/os/unix -I ../../../../../src/http/modules/perl -I /usr/include/libxml2 -I ../../../../../objs -I ../../../../../src/http -I ../../../../../src/http/modules -I ../../../../../src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -D_REENTRANT -D_GNU_SOURCE -DDEBIAN -fwrapv -fno-strict-aliasing -pipe -I/usr/local/include -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64 -I/usr/lib/x86_64-linux-gnu/perl/5.38/CORE -g -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -O   -DVERSION=\"1.24.0\" -DXS_VERSION=\"1.24.0\" -fPIC "-I/usr/lib/x86_64-linux-gnu/perl/5.38/CORE"   nginx.c
rm -f blib/arch/auto/nginx/nginx.so
x86_64-linux-gnu-gcc  -Wl,-Bsymbolic-functions -flto=auto -ffat-lto-objects -Wl,-z,relro -Wl,-z,now -fPIC -shared -L/usr/local/lib -fstack-protector-strong  nginx.o  -o blib/arch/auto/nginx/nginx.so  \
      \

chmod 755 blib/arch/auto/nginx/nginx.so
Manifying 1 pod document
make[4]: Leaving directory '/root/custom-nginx/nginx-1.24.0/debian/build-bin/objs/src/http/modules/perl'
rm -rf objs/install_perl
cc -o objs/ngx_stream_geoip_module.so \
objs/src/stream/ngx_stream_geoip_module.o \
objs/ngx_stream_geoip_module_modules.o \
-Wl,-Bsymbolic-functions -flto=auto -ffat-lto-objects -Wl,-z,relro -Wl,-z,now -fPIC -lGeoIP \
-shared
cc -o objs/ngx_mail_module.so \
objs/src/mail/ngx_mail.o \
objs/src/mail/ngx_mail_core_module.o \
objs/src/mail/ngx_mail_handler.o \
objs/src/mail/ngx_mail_parse.o \
objs/src/mail/ngx_mail_ssl_module.o \
objs/src/mail/ngx_mail_pop3_module.o \
objs/src/mail/ngx_mail_pop3_handler.o \
objs/src/mail/ngx_mail_imap_module.o \
objs/src/mail/ngx_mail_imap_handler.o \
objs/src/mail/ngx_mail_smtp_module.o \
objs/src/mail/ngx_mail_smtp_handler.o \
objs/src/mail/ngx_mail_auth_http_module.o \
objs/src/mail/ngx_mail_proxy_module.o \
objs/src/mail/ngx_mail_realip_module.o \
objs/ngx_mail_module_modules.o \
-Wl,-Bsymbolic-functions -flto=auto -ffat-lto-objects -Wl,-z,relro -Wl,-z,now -fPIC \
-shared
cc -o objs/ngx_stream_module.so \
objs/src/stream/ngx_stream.o \
objs/src/stream/ngx_stream_variables.o \
objs/src/stream/ngx_stream_script.o \
objs/src/stream/ngx_stream_handler.o \
objs/src/stream/ngx_stream_core_module.o \
objs/src/stream/ngx_stream_log_module.o \
objs/src/stream/ngx_stream_proxy_module.o \
objs/src/stream/ngx_stream_upstream.o \
objs/src/stream/ngx_stream_upstream_round_robin.o \
objs/src/stream/ngx_stream_write_filter_module.o \
objs/src/stream/ngx_stream_ssl_module.o \
objs/src/stream/ngx_stream_realip_module.o \
objs/src/stream/ngx_stream_limit_conn_module.o \
objs/src/stream/ngx_stream_access_module.o \
objs/src/stream/ngx_stream_geo_module.o \
objs/src/stream/ngx_stream_map_module.o \
objs/src/stream/ngx_stream_split_clients_module.o \
objs/src/stream/ngx_stream_return_module.o \
objs/src/stream/ngx_stream_set_module.o \
objs/src/stream/ngx_stream_upstream_hash_module.o \
objs/src/stream/ngx_stream_upstream_least_conn_module.o \
objs/src/stream/ngx_stream_upstream_random_module.o \
objs/src/stream/ngx_stream_upstream_zone_module.o \
objs/src/stream/ngx_stream_ssl_preread_module.o \
objs/ngx_stream_module_modules.o \
-Wl,-Bsymbolic-functions -flto=auto -ffat-lto-objects -Wl,-z,relro -Wl,-z,now -fPIC \
-shared
cc -c -fPIC -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=.
-flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -g -O2 -fno-omit-frame-pointer
-mno-omit-leaf-frame-pointer -ffile-prefix-map=/root/custom-nginx/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/root/custom-nginx/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7-custom -fPIC -Wdate-time -D_FORTIFY_SOURCE=3 -Wno-deprecated-declarations -D_REENTRANT -D_GNU_SOURCE -DDEBIAN -fwrapv -fno-strict-aliasing -pipe -I/usr/local/include -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64 -I/usr/lib/x86_64-linux-gnu/perl/5.38/CORE -I src/core -I src/event -I src/event/modules -I src/os/unix -I src/http/modules/perl -I /usr/include/libxml2 -I objs -I src/http -I src/http/modules -I src/http/v2 -I /root/custom-nginx/nginx-1.24.0/ngx_brotli/deps/brotli/c/include -I src/mail -I src/stream \
        -o objs/src/http/modules/perl/ngx_http_perl_module.o \
        src/http/modules/perl/ngx_http_perl_module.c
cc -o objs/ngx_http_perl_module.so \
objs/src/http/modules/perl/ngx_http_perl_module.o \
objs/ngx_http_perl_module_modules.o \
-Wl,-Bsymbolic-functions -flto=auto -ffat-lto-objects -Wl,-z,relro -Wl,-z,now -fPIC -Wl,-E -fstack-protector-strong -L/usr/local/lib -L/usr/lib/x86_64-linux-gnu/perl/5.38/CORE -lperl -ldl -lm -lpthread -lc -lcrypt \
-shared
make[3]: Leaving directory '/root/custom-nginx/nginx-1.24.0/debian/build-bin'
make[2]: Leaving directory '/root/custom-nginx/nginx-1.24.0/debian/build-bin'
dh override_dh_auto_build --without autoreconf
make[1]: Leaving directory '/root/custom-nginx/nginx-1.24.0'
   create-stamp debian/debhelper-build-stamp
   dh_prep
        rm -f -- debian/nginx.substvars debian/nginx-doc.substvars debian/nginx-common.substvars debian/nginx-dev.substvars
debian/nginx-core.substvars debian/nginx-full.substvars debian/nginx-light.substvars debian/nginx-extras.substvars debian/libnginx-mod-http-geoip.substvars debian/libnginx-mod-http-image-filter.substvars debian/libnginx-mod-http-xslt-filter.substvars debian/libnginx-mod-mail.substvars debian/libnginx-mod-stream.substvars debian/libnginx-mod-stream-geoip.substvars debian/libnginx-mod-http-perl.substvars
        rm -fr -- debian/.debhelper/generated/nginx/ debian/nginx/ debian/tmp/ debian/.debhelper/generated/nginx-doc/ debian/nginx-doc/ debian/.debhelper/generated/nginx-common/ debian/nginx-common/ debian/.debhelper/generated/nginx-dev/ debian/nginx-dev/ debian/.debhelper/generated/nginx-core/ debian/nginx-core/ debian/.debhelper/generated/nginx-full/ debian/nginx-full/ debian/.debhelper/generated/nginx-light/ debian/nginx-light/ debian/.debhelper/generated/nginx-extras/ debian/nginx-extras/ debian/.debhelper/generated/libnginx-mod-http-geoip/ debian/libnginx-mod-http-geoip/ debian/.debhelper/generated/libnginx-mod-http-image-filter/ debian/libnginx-mod-http-image-filter/ debian/.debhelper/generated/libnginx-mod-http-xslt-filter/ debian/libnginx-mod-http-xslt-filter/ debian/.debhelper/generated/libnginx-mod-mail/ debian/libnginx-mod-mail/ debian/.debhelper/generated/libnginx-mod-stream/ debian/libnginx-mod-stream/ debian/.debhelper/generated/libnginx-mod-stream-geoip/ debian/libnginx-mod-stream-geoip/ debian/.debhelper/generated/libnginx-mod-http-perl/ debian/libnginx-mod-http-perl/
   dh_installdirs
        install -m0755 -d debian/nginx/usr/sbin
        install -m0755 -d debian/nginx-common/etc/nginx debian/nginx-common/etc/nginx/conf.d debian/nginx-common/etc/nginx/modules-available debian/nginx-common/etc/nginx/modules-enabled debian/nginx-common/etc/nginx/sites-available debian/nginx-common/etc/nginx/sites-enabled debian/nginx-common/etc/ufw/applications.d debian/nginx-common/usr/share/nginx debian/nginx-common/usr/share/vim/addons debian/nginx-common/usr/share/vim/registry debian/nginx-common/var/lib/nginx debian/nginx-common/var/log/nginx
   debian/rules override_dh_install
make[1]: Entering directory '/root/custom-nginx/nginx-1.24.0'
dh_install
        install -m0755 -d debian/nginx/usr/sbin
        cp --reflink=auto -a ./debian/build-bin/objs/nginx debian/nginx/usr/sbin/
        install -m0755 -d debian/nginx-common/etc/nginx
        cp --reflink=auto -a ./debian/conf/fastcgi.conf ./debian/conf/fastcgi_params ./debian/conf/koi-utf ./debian/conf/koi-win ./debian/conf/mime.types ./debian/conf/nginx.conf ./debian/conf/proxy_params ./debian/conf/scgi_params ./debian/conf/sites-available ./debian/conf/snippets ./debian/conf/uwsgi_params ./debian/conf/win-utf debian/nginx-common/etc/nginx/
        install -m0755 -d debian/nginx-common/etc/ufw/applications.d
        cp --reflink=auto -a ./debian/ufw/nginx debian/nginx-common/etc/ufw/applications.d/
        install -m0755 -d debian/nginx-common/usr/share/apport/package-hooks
        cp --reflink=auto -a ./debian/apport/source_nginx.py debian/nginx-common/usr/share/apport/package-hooks/
        install -m0755 -d debian/nginx-common/usr/share/nginx/html/
        cp --reflink=auto -a ./html/index.html debian/nginx-common/usr/share/nginx/html//
        install -m0755 -d debian/nginx-common/usr/share/vim/addons
        cp --reflink=auto -a ./contrib/vim/ftdetect ./contrib/vim/ftplugin ./contrib/vim/indent ./contrib/vim/syntax debian/nginx-common/usr/share/vim/addons/
        install -m0755 -d debian/nginx-common/usr/share/vim/registry
        cp --reflink=auto -a ./debian/vim/nginx.yaml debian/nginx-common/usr/share/vim/registry/
        install -m0755 -d debian/nginx-dev/usr/bin/
        cp --reflink=auto -a ./debian/debhelper/dh_nginx debian/nginx-dev/usr/bin//
        install -m0755 -d debian/nginx-dev/usr/share/debhelper/autoscripts/
        cp --reflink=auto -a ./debian/autoscripts/postinst-nginx ./debian/autoscripts/postrm-nginx ./debian/autoscripts/prerm-nginx debian/nginx-dev/usr/share/debhelper/autoscripts//
        install -m0755 -d debian/nginx-dev/usr/share/nginx/src/
        cp --reflink=auto -a ./debian/build-src/auto ./debian/build-src/conf_flags ./debian/build-src/configure ./debian/build-src/src debian/nginx-dev/usr/share/nginx/src//
        install -m0755 -d debian/nginx-dev/usr/share/nginx/src/debian/
        cp --reflink=auto -a ./debian/libnginx-mod.abisubstvars debian/nginx-dev/usr/share/nginx/src/debian//
        install -m0755 -d debian/nginx-dev/usr/share/perl5/Debian/Debhelper/Buildsystem/
        cp --reflink=auto -a ./debian/debhelper/nginx_mod.pm debian/nginx-dev/usr/share/perl5/Debian/Debhelper/Buildsystem//
        install -m0755 -d debian/nginx-dev/usr/share/perl5/Debian/Debhelper/Sequence/
        cp --reflink=auto -a ./debian/debhelper/nginx.pm debian/nginx-dev/usr/share/perl5/Debian/Debhelper/Sequence//
        install -m0755 -d debian/libnginx-mod-http-perl/usr/lib/x86_64-linux-gnu/perl5/5.38
        cp --reflink=auto -a ./debian/build-bin/objs/src/http/modules/perl/blib/lib/nginx.pm debian/libnginx-mod-http-perl/usr/lib/x86_64-linux-gnu/perl5/5.38/
        install -m0755 -d debian/libnginx-mod-http-perl/usr/lib/x86_64-linux-gnu/perl5/5.38/auto/nginx
        cp --reflink=auto -a ./debian/build-bin/objs/src/http/modules/perl/blib/arch/auto/nginx/nginx.so debian/libnginx-mod-http-perl/usr/lib/x86_64-linux-gnu/perl5/5.38/auto/nginx/
DH_AUTOSCRIPTDIR=/root/custom-nginx/nginx-1.24.0/debian/autoscripts debian/debhelper/dh_nginx --in-nginx-tree
        modules -- debian/build-bin/objs/ngx_http_geoip_module.so --


        Installing module binary debian/build-bin/objs/ngx_http_geoip_module.so into debian/libnginx-mod-http-geoip//usr/lib/nginx/modules/

        mkdir -p debian/libnginx-mod-http-geoip//usr/lib/nginx/modules/
        chmod 755 debian/libnginx-mod-http-geoip//usr/lib/nginx/modules/
        cp debian/build-bin/objs/ngx_http_geoip_module.so debian/libnginx-mod-http-geoip//usr/lib/nginx/modules/
        modules -- debian/libnginx-mod.conf/mod-http-geoip.conf --


        Installing module configuration mod-http-geoip.conf into debian/libnginx-mod-http-geoip/usr/share/nginx/modules-available/ prio:50

        mkdir -p debian/libnginx-mod-http-geoip/usr/share/nginx/modules-available/
        chmod 755 debian/libnginx-mod-http-geoip/usr/share/nginx/modules-available/
        cp debian/libnginx-mod.conf/mod-http-geoip.conf debian/libnginx-mod-http-geoip/usr/share/nginx/modules-available/
        chmod 644 debian/libnginx-mod-http-geoip/usr/share/nginx/modules-available//mod-http-geoip.conf
        mv debian/libnginx-mod-http-geoip.substvars.new debian/libnginx-mod-http-geoip.substvars
        echo "# Automatically added by dh_nginx/UNDECLARED">> debian/libnginx-mod-http-geoip.postinst.debhelper
        sed "s#NAMES#mod-http-geoip.conf:50-mod-http-geoip.conf g; " /root/custom-nginx/nginx-1.24.0/debian/autoscripts/postinst-nginx >> debian/libnginx-mod-http-geoip.postinst.debhelper
        echo '# End automatically added section' >> debian/libnginx-mod-http-geoip.postinst.debhelper
        echo "# Automatically added by dh_nginx/UNDECLARED">> debian/libnginx-mod-http-geoip.prerm.debhelper
        sed "s#NAMES#mod-http-geoip.conf:50-mod-http-geoip.conf g; " /root/custom-nginx/nginx-1.24.0/debian/autoscripts/prerm-nginx >> debian/libnginx-mod-http-geoip.prerm.debhelper
        echo '# End automatically added section' >> debian/libnginx-mod-http-geoip.prerm.debhelper
        echo "# Automatically added by dh_nginx/UNDECLARED">> debian/libnginx-mod-http-geoip.postrm.debhelper
        sed "s#NAMES#mod-http-geoip.conf:50-mod-http-geoip.conf g; " /root/custom-nginx/nginx-1.24.0/debian/autoscripts/postrm-nginx >> debian/libnginx-mod-http-geoip.postrm.debhelper
        echo '# End automatically added section' >> debian/libnginx-mod-http-geoip.postrm.debhelper
        modules -- debian/build-bin/objs/ngx_http_image_filter_module.so --


        Installing module binary debian/build-bin/objs/ngx_http_image_filter_module.so into debian/libnginx-mod-http-image-filter//usr/lib/nginx/modules/

        mkdir -p debian/libnginx-mod-http-image-filter//usr/lib/nginx/modules/
        chmod 755 debian/libnginx-mod-http-image-filter//usr/lib/nginx/modules/
        cp debian/build-bin/objs/ngx_http_image_filter_module.so debian/libnginx-mod-http-image-filter//usr/lib/nginx/modules/
        modules -- debian/libnginx-mod.conf/mod-http-image-filter.conf --


        Installing module configuration mod-http-image-filter.conf into debian/libnginx-mod-http-image-filter/usr/share/nginx/modules-available/ prio:50

        mkdir -p debian/libnginx-mod-http-image-filter/usr/share/nginx/modules-available/
        chmod 755 debian/libnginx-mod-http-image-filter/usr/share/nginx/modules-available/
        cp debian/libnginx-mod.conf/mod-http-image-filter.conf debian/libnginx-mod-http-image-filter/usr/share/nginx/modules-available/
        chmod 644 debian/libnginx-mod-http-image-filter/usr/share/nginx/modules-available//mod-http-image-filter.conf
        mv debian/libnginx-mod-http-image-filter.substvars.new debian/libnginx-mod-http-image-filter.substvars
        echo "# Automatically added by dh_nginx/UNDECLARED">> debian/libnginx-mod-http-image-filter.postinst.debhelper
        sed "s#NAMES#mod-http-image-filter.conf:50-mod-http-image-filter.conf g; " /root/custom-nginx/nginx-1.24.0/debian/autoscripts/postinst-nginx >> debian/libnginx-mod-http-image-filter.postinst.debhelper
        echo '# End automatically added section' >> debian/libnginx-mod-http-image-filter.postinst.debhelper
        echo "# Automatically added by dh_nginx/UNDECLARED">> debian/libnginx-mod-http-image-filter.prerm.debhelper
        sed "s#NAMES#mod-http-image-filter.conf:50-mod-http-image-filter.conf g; " /root/custom-nginx/nginx-1.24.0/debian/autoscripts/prerm-nginx >> debian/libnginx-mod-http-image-filter.prerm.debhelper
        echo '# End automatically added section' >> debian/libnginx-mod-http-image-filter.prerm.debhelper
        echo "# Automatically added by dh_nginx/UNDECLARED">> debian/libnginx-mod-http-image-filter.postrm.debhelper
        sed "s#NAMES#mod-http-image-filter.conf:50-mod-http-image-filter.conf g; " /root/custom-nginx/nginx-1.24.0/debian/autoscripts/postrm-nginx >> debian/libnginx-mod-http-image-filter.postrm.debhelper
        echo '# End automatically added section' >> debian/libnginx-mod-http-image-filter.postrm.debhelper
        modules -- debian/build-bin/objs/ngx_http_xslt_filter_module.so --


        Installing module binary debian/build-bin/objs/ngx_http_xslt_filter_module.so into debian/libnginx-mod-http-xslt-filter//usr/lib/nginx/modules/

        mkdir -p debian/libnginx-mod-http-xslt-filter//usr/lib/nginx/modules/
        chmod 755 debian/libnginx-mod-http-xslt-filter//usr/lib/nginx/modules/
        cp debian/build-bin/objs/ngx_http_xslt_filter_module.so debian/libnginx-mod-http-xslt-filter//usr/lib/nginx/modules/
        modules -- debian/libnginx-mod.conf/mod-http-xslt-filter.conf --


        Installing module configuration mod-http-xslt-filter.conf into debian/libnginx-mod-http-xslt-filter/usr/share/nginx/modules-available/ prio:50

        mkdir -p debian/libnginx-mod-http-xslt-filter/usr/share/nginx/modules-available/
        chmod 755 debian/libnginx-mod-http-xslt-filter/usr/share/nginx/modules-available/
        cp debian/libnginx-mod.conf/mod-http-xslt-filter.conf debian/libnginx-mod-http-xslt-filter/usr/share/nginx/modules-available/
        chmod 644 debian/libnginx-mod-http-xslt-filter/usr/share/nginx/modules-available//mod-http-xslt-filter.conf
        mv debian/libnginx-mod-http-xslt-filter.substvars.new debian/libnginx-mod-http-xslt-filter.substvars
        echo "# Automatically added by dh_nginx/UNDECLARED">> debian/libnginx-mod-http-xslt-filter.postinst.debhelper
        sed "s#NAMES#mod-http-xslt-filter.conf:50-mod-http-xslt-filter.conf g; " /root/custom-nginx/nginx-1.24.0/debian/autoscripts/postinst-nginx >> debian/libnginx-mod-http-xslt-filter.postinst.debhelper
        echo '# End automatically added section' >> debian/libnginx-mod-http-xslt-filter.postinst.debhelper
        echo "# Automatically added by dh_nginx/UNDECLARED">> debian/libnginx-mod-http-xslt-filter.prerm.debhelper
        sed "s#NAMES#mod-http-xslt-filter.conf:50-mod-http-xslt-filter.conf g; " /root/custom-nginx/nginx-1.24.0/debian/autoscripts/prerm-nginx >> debian/libnginx-mod-http-xslt-filter.prerm.debhelper
        echo '# End automatically added section' >> debian/libnginx-mod-http-xslt-filter.prerm.debhelper
        echo "# Automatically added by dh_nginx/UNDECLARED">> debian/libnginx-mod-http-xslt-filter.postrm.debhelper
        sed "s#NAMES#mod-http-xslt-filter.conf:50-mod-http-xslt-filter.conf g; " /root/custom-nginx/nginx-1.24.0/debian/autoscripts/postrm-nginx >> debian/libnginx-mod-http-xslt-filter.postrm.debhelper
        echo '# End automatically added section' >> debian/libnginx-mod-http-xslt-filter.postrm.debhelper
        modules -- debian/build-bin/objs/ngx_mail_module.so --


        Installing module binary debian/build-bin/objs/ngx_mail_module.so into debian/libnginx-mod-mail//usr/lib/nginx/modules/

        mkdir -p debian/libnginx-mod-mail//usr/lib/nginx/modules/
        chmod 755 debian/libnginx-mod-mail//usr/lib/nginx/modules/
        cp debian/build-bin/objs/ngx_mail_module.so debian/libnginx-mod-mail//usr/lib/nginx/modules/
        modules -- debian/libnginx-mod.conf/mod-mail.conf --


        Installing module configuration mod-mail.conf into debian/libnginx-mod-mail/usr/share/nginx/modules-available/ prio:50

        mkdir -p debian/libnginx-mod-mail/usr/share/nginx/modules-available/
        chmod 755 debian/libnginx-mod-mail/usr/share/nginx/modules-available/
        cp debian/libnginx-mod.conf/mod-mail.conf debian/libnginx-mod-mail/usr/share/nginx/modules-available/
        chmod 644 debian/libnginx-mod-mail/usr/share/nginx/modules-available//mod-mail.conf
        mv debian/libnginx-mod-mail.substvars.new debian/libnginx-mod-mail.substvars
        echo "# Automatically added by dh_nginx/UNDECLARED">> debian/libnginx-mod-mail.postinst.debhelper
        sed "s#NAMES#mod-mail.conf:50-mod-mail.conf g; " /root/custom-nginx/nginx-1.24.0/debian/autoscripts/postinst-nginx >> debian/libnginx-mod-mail.postinst.debhelper
        echo '# End automatically added section' >> debian/libnginx-mod-mail.postinst.debhelper
        echo "# Automatically added by dh_nginx/UNDECLARED">> debian/libnginx-mod-mail.prerm.debhelper
        sed "s#NAMES#mod-mail.conf:50-mod-mail.conf g; " /root/custom-nginx/nginx-1.24.0/debian/autoscripts/prerm-nginx >> debian/libnginx-mod-mail.prerm.debhelper
        echo '# End automatically added section' >> debian/libnginx-mod-mail.prerm.debhelper
        echo "# Automatically added by dh_nginx/UNDECLARED">> debian/libnginx-mod-mail.postrm.debhelper
        sed "s#NAMES#mod-mail.conf:50-mod-mail.conf g; " /root/custom-nginx/nginx-1.24.0/debian/autoscripts/postrm-nginx >>
debian/libnginx-mod-mail.postrm.debhelper
        echo '# End automatically added section' >> debian/libnginx-mod-mail.postrm.debhelper
        modules -- debian/build-bin/objs/ngx_stream_module.so --


        Installing module binary debian/build-bin/objs/ngx_stream_module.so into debian/libnginx-mod-stream//usr/lib/nginx/modules/

        mkdir -p debian/libnginx-mod-stream//usr/lib/nginx/modules/
        chmod 755 debian/libnginx-mod-stream//usr/lib/nginx/modules/
        cp debian/build-bin/objs/ngx_stream_module.so debian/libnginx-mod-stream//usr/lib/nginx/modules/
        modules -- debian/libnginx-mod.conf/mod-stream.conf --


        Installing module configuration mod-stream.conf into debian/libnginx-mod-stream/usr/share/nginx/modules-available/ prio:50

        mkdir -p debian/libnginx-mod-stream/usr/share/nginx/modules-available/
        chmod 755 debian/libnginx-mod-stream/usr/share/nginx/modules-available/
        cp debian/libnginx-mod.conf/mod-stream.conf debian/libnginx-mod-stream/usr/share/nginx/modules-available/
        chmod 644 debian/libnginx-mod-stream/usr/share/nginx/modules-available//mod-stream.conf
        mv debian/libnginx-mod-stream.substvars.new debian/libnginx-mod-stream.substvars
        echo "# Automatically added by dh_nginx/UNDECLARED">> debian/libnginx-mod-stream.postinst.debhelper
        sed "s#NAMES#mod-stream.conf:50-mod-stream.conf g; " /root/custom-nginx/nginx-1.24.0/debian/autoscripts/postinst-nginx >> debian/libnginx-mod-stream.postinst.debhelper
        echo '# End automatically added section' >> debian/libnginx-mod-stream.postinst.debhelper
        echo "# Automatically added by dh_nginx/UNDECLARED">> debian/libnginx-mod-stream.prerm.debhelper
        sed "s#NAMES#mod-stream.conf:50-mod-stream.conf g; " /root/custom-nginx/nginx-1.24.0/debian/autoscripts/prerm-nginx
>> debian/libnginx-mod-stream.prerm.debhelper
        echo '# End automatically added section' >> debian/libnginx-mod-stream.prerm.debhelper
        echo "# Automatically added by dh_nginx/UNDECLARED">> debian/libnginx-mod-stream.postrm.debhelper
        sed "s#NAMES#mod-stream.conf:50-mod-stream.conf g; " /root/custom-nginx/nginx-1.24.0/debian/autoscripts/postrm-nginx >> debian/libnginx-mod-stream.postrm.debhelper
        echo '# End automatically added section' >> debian/libnginx-mod-stream.postrm.debhelper
        modules -- debian/build-bin/objs/ngx_stream_geoip_module.so --


        Installing module binary debian/build-bin/objs/ngx_stream_geoip_module.so into debian/libnginx-mod-stream-geoip//usr/lib/nginx/modules/

        mkdir -p debian/libnginx-mod-stream-geoip//usr/lib/nginx/modules/
        chmod 755 debian/libnginx-mod-stream-geoip//usr/lib/nginx/modules/
        cp debian/build-bin/objs/ngx_stream_geoip_module.so debian/libnginx-mod-stream-geoip//usr/lib/nginx/modules/
        modules -- debian/libnginx-mod.conf/mod-stream-geoip.conf -- 70


        Installing module configuration mod-stream-geoip.conf into debian/libnginx-mod-stream-geoip/usr/share/nginx/modules-available/ prio:70

        mkdir -p debian/libnginx-mod-stream-geoip/usr/share/nginx/modules-available/
        chmod 755 debian/libnginx-mod-stream-geoip/usr/share/nginx/modules-available/
        cp debian/libnginx-mod.conf/mod-stream-geoip.conf debian/libnginx-mod-stream-geoip/usr/share/nginx/modules-available/
        chmod 644 debian/libnginx-mod-stream-geoip/usr/share/nginx/modules-available//mod-stream-geoip.conf
        mv debian/libnginx-mod-stream-geoip.substvars.new debian/libnginx-mod-stream-geoip.substvars
        echo "# Automatically added by dh_nginx/UNDECLARED">> debian/libnginx-mod-stream-geoip.postinst.debhelper
        sed "s#NAMES#mod-stream-geoip.conf:70-mod-stream-geoip.conf g; " /root/custom-nginx/nginx-1.24.0/debian/autoscripts/postinst-nginx >> debian/libnginx-mod-stream-geoip.postinst.debhelper
        echo '# End automatically added section' >> debian/libnginx-mod-stream-geoip.postinst.debhelper
        echo "# Automatically added by dh_nginx/UNDECLARED">> debian/libnginx-mod-stream-geoip.prerm.debhelper
        sed "s#NAMES#mod-stream-geoip.conf:70-mod-stream-geoip.conf g; " /root/custom-nginx/nginx-1.24.0/debian/autoscripts/prerm-nginx >> debian/libnginx-mod-stream-geoip.prerm.debhelper
        echo '# End automatically added section' >> debian/libnginx-mod-stream-geoip.prerm.debhelper
        echo "# Automatically added by dh_nginx/UNDECLARED">> debian/libnginx-mod-stream-geoip.postrm.debhelper
        sed "s#NAMES#mod-stream-geoip.conf:70-mod-stream-geoip.conf g; " /root/custom-nginx/nginx-1.24.0/debian/autoscripts/postrm-nginx >> debian/libnginx-mod-stream-geoip.postrm.debhelper
        echo '# End automatically added section' >> debian/libnginx-mod-stream-geoip.postrm.debhelper
        modules -- debian/build-bin/objs/ngx_http_perl_module.so --


        Installing module binary debian/build-bin/objs/ngx_http_perl_module.so into debian/libnginx-mod-http-perl//usr/lib/nginx/modules/

        mkdir -p debian/libnginx-mod-http-perl//usr/lib/nginx/modules/
        chmod 755 debian/libnginx-mod-http-perl//usr/lib/nginx/modules/
        cp debian/build-bin/objs/ngx_http_perl_module.so debian/libnginx-mod-http-perl//usr/lib/nginx/modules/
        modules -- debian/libnginx-mod.conf/mod-http-perl.conf --


        Installing module configuration mod-http-perl.conf into debian/libnginx-mod-http-perl/usr/share/nginx/modules-available/ prio:50

        mkdir -p debian/libnginx-mod-http-perl/usr/share/nginx/modules-available/
        chmod 755 debian/libnginx-mod-http-perl/usr/share/nginx/modules-available/
        cp debian/libnginx-mod.conf/mod-http-perl.conf debian/libnginx-mod-http-perl/usr/share/nginx/modules-available/
        chmod 644 debian/libnginx-mod-http-perl/usr/share/nginx/modules-available//mod-http-perl.conf
        mv debian/libnginx-mod-http-perl.substvars.new debian/libnginx-mod-http-perl.substvars
        echo "# Automatically added by dh_nginx/UNDECLARED">> debian/libnginx-mod-http-perl.postinst.debhelper
        sed "s#NAMES#mod-http-perl.conf:50-mod-http-perl.conf g; " /root/custom-nginx/nginx-1.24.0/debian/autoscripts/postinst-nginx >> debian/libnginx-mod-http-perl.postinst.debhelper
        echo '# End automatically added section' >> debian/libnginx-mod-http-perl.postinst.debhelper
        echo "# Automatically added by dh_nginx/UNDECLARED">> debian/libnginx-mod-http-perl.prerm.debhelper
        sed "s#NAMES#mod-http-perl.conf:50-mod-http-perl.conf g; " /root/custom-nginx/nginx-1.24.0/debian/autoscripts/prerm-nginx >> debian/libnginx-mod-http-perl.prerm.debhelper
        echo '# End automatically added section' >> debian/libnginx-mod-http-perl.prerm.debhelper
        echo "# Automatically added by dh_nginx/UNDECLARED">> debian/libnginx-mod-http-perl.postrm.debhelper
        sed "s#NAMES#mod-http-perl.conf:50-mod-http-perl.conf g; " /root/custom-nginx/nginx-1.24.0/debian/autoscripts/postrm-nginx >> debian/libnginx-mod-http-perl.postrm.debhelper
        echo '# End automatically added section' >> debian/libnginx-mod-http-perl.postrm.debhelper
make[1]: Leaving directory '/root/custom-nginx/nginx-1.24.0'
   dh_installdocs
        install -m0755 -d debian/nginx/usr/share/doc/nginx
        install -p -m0644 debian/copyright debian/nginx/usr/share/doc/nginx/copyright
        install -m0755 -d debian/nginx-doc/usr/share/doc/nginx-doc
        install -m0755 -d debian/nginx-doc/usr/share/doc/nginx
        cp --reflink=auto -a ./README debian/nginx-doc/usr/share/doc/nginx
        cp --reflink=auto -a ./debian/help/docs/fcgiwrap debian/nginx-doc/usr/share/doc/nginx
        cp --reflink=auto -a ./debian/help/docs/php debian/nginx-doc/usr/share/doc/nginx
        cp --reflink=auto -a ./debian/help/docs/support-irc debian/nginx-doc/usr/share/doc/nginx
        cp --reflink=auto -a ./debian/help/docs/upstream debian/nginx-doc/usr/share/doc/nginx
        chmod -R u\+rw,go=rX debian/nginx-doc/usr/share/doc
        install -p -m0644 debian/copyright debian/nginx-doc/usr/share/doc/nginx-doc/copyright
        install -m0755 -d debian/nginx-common/usr/share/doc/nginx-common
        install -p -m0644 debian/copyright debian/nginx-common/usr/share/doc/nginx-common/copyright
        install -m0755 -d debian/nginx-dev/usr/share/doc/nginx-dev
        install -p -m0644 debian/copyright debian/nginx-dev/usr/share/doc/nginx-dev/copyright
        install -m0755 -d debian/nginx-core/usr/share/doc/nginx-core
        install -p -m0644 debian/copyright debian/nginx-core/usr/share/doc/nginx-core/copyright
        install -m0755 -d debian/nginx-full/usr/share/doc/nginx-full
        install -p -m0644 debian/copyright debian/nginx-full/usr/share/doc/nginx-full/copyright
        install -m0755 -d debian/nginx-light/usr/share/doc/nginx-light
        install -p -m0644 debian/copyright debian/nginx-light/usr/share/doc/nginx-light/copyright
        install -m0755 -d debian/nginx-extras/usr/share/doc/nginx-extras
        install -p -m0644 debian/copyright debian/nginx-extras/usr/share/doc/nginx-extras/copyright
        install -m0755 -d debian/libnginx-mod-http-geoip/usr/share/doc/libnginx-mod-http-geoip
        install -p -m0644 debian/copyright debian/libnginx-mod-http-geoip/usr/share/doc/libnginx-mod-http-geoip/copyright
        install -m0755 -d debian/libnginx-mod-http-image-filter/usr/share/doc/libnginx-mod-http-image-filter
        install -p -m0644 debian/copyright debian/libnginx-mod-http-image-filter/usr/share/doc/libnginx-mod-http-image-filter/copyright
        install -m0755 -d debian/libnginx-mod-http-xslt-filter/usr/share/doc/libnginx-mod-http-xslt-filter
        install -p -m0644 debian/copyright debian/libnginx-mod-http-xslt-filter/usr/share/doc/libnginx-mod-http-xslt-filter/copyright
        install -m0755 -d debian/libnginx-mod-mail/usr/share/doc/libnginx-mod-mail
        install -p -m0644 debian/copyright debian/libnginx-mod-mail/usr/share/doc/libnginx-mod-mail/copyright
        install -m0755 -d debian/libnginx-mod-stream/usr/share/doc/libnginx-mod-stream
        install -p -m0644 debian/copyright debian/libnginx-mod-stream/usr/share/doc/libnginx-mod-stream/copyright
        install -m0755 -d debian/libnginx-mod-stream-geoip/usr/share/doc/libnginx-mod-stream-geoip
        install -p -m0644 debian/copyright debian/libnginx-mod-stream-geoip/usr/share/doc/libnginx-mod-stream-geoip/copyright
        install -m0755 -d debian/libnginx-mod-http-perl/usr/share/doc/libnginx-mod-http-perl
        install -p -m0644 debian/copyright debian/libnginx-mod-http-perl/usr/share/doc/libnginx-mod-http-perl/copyright
   dh_installchangelogs
        install -m0755 -d debian/nginx/usr/share/doc/nginx
        install -m0755 -d debian/libnginx-mod-http-geoip/usr/share/doc/libnginx-mod-http-geoip
        install -p -m0644 debian/.debhelper/generated/nginx/dh_installchangelogs.dch.trimmed debian/nginx/usr/share/doc/nginx/changelog.Debian
        install -m0755 -d debian/nginx-doc/usr/share/doc/nginx-doc
        install -p -m0644 debian/.debhelper/generated/nginx-doc/dh_installchangelogs.dch.trimmed debian/nginx-doc/usr/share/doc/nginx-doc/changelog.Debian
        install -m0755 -d debian/nginx-common/usr/share/doc/nginx-common
        install -p -m0644 debian/.debhelper/generated/libnginx-mod-http-geoip/dh_installchangelogs.dch.trimmed debian/libnginx-mod-http-geoip/usr/share/doc/libnginx-mod-http-geoip/changelog.Debian
        install -m0755 -d debian/libnginx-mod-http-image-filter/usr/share/doc/libnginx-mod-http-image-filter
        install -p -m0644 debian/.debhelper/generated/nginx-common/dh_installchangelogs.dch.trimmed debian/nginx-common/usr/share/doc/nginx-common/changelog.Debian
        install -m0755 -d debian/nginx-dev/usr/share/doc/nginx-dev
        install -p -m0644 debian/.debhelper/generated/nginx-dev/dh_installchangelogs.dch.trimmed debian/nginx-dev/usr/share/doc/nginx-dev/changelog.Debian
        install -m0755 -d debian/nginx-core/usr/share/doc/nginx-core
        install -p -m0644 debian/.debhelper/generated/libnginx-mod-http-image-filter/dh_installchangelogs.dch.trimmed debian/libnginx-mod-http-image-filter/usr/share/doc/libnginx-mod-http-image-filter/changelog.Debian
        install -m0755 -d debian/libnginx-mod-http-xslt-filter/usr/share/doc/libnginx-mod-http-xslt-filter
        install -p -m0644 debian/.debhelper/generated/nginx-core/dh_installchangelogs.dch.trimmed debian/nginx-core/usr/share/doc/nginx-core/changelog.Debian
        install -p -m0644 debian/.debhelper/generated/nginx-core/dh_installchangelogs.news.trimmed debian/nginx-core/usr/share/doc/nginx-core/NEWS.Debian
        install -m0755 -d debian/nginx-full/usr/share/doc/nginx-full
        install -p -m0644 debian/.debhelper/generated/libnginx-mod-http-xslt-filter/dh_installchangelogs.dch.trimmed debian/libnginx-mod-http-xslt-filter/usr/share/doc/libnginx-mod-http-xslt-filter/changelog.Debian
        install -m0755 -d debian/libnginx-mod-mail/usr/share/doc/libnginx-mod-mail
        install -p -m0644 debian/.debhelper/generated/nginx-full/dh_installchangelogs.dch.trimmed debian/nginx-full/usr/share/doc/nginx-full/changelog.Debian
        install -p -m0644 debian/.debhelper/generated/nginx-full/dh_installchangelogs.news.trimmed debian/nginx-full/usr/share/doc/nginx-full/NEWS.Debian
        install -m0755 -d debian/nginx-light/usr/share/doc/nginx-light
        install -p -m0644 debian/.debhelper/generated/libnginx-mod-mail/dh_installchangelogs.dch.trimmed debian/libnginx-mod-mail/usr/share/doc/libnginx-mod-mail/changelog.Debian
        install -m0755 -d debian/libnginx-mod-stream/usr/share/doc/libnginx-mod-stream
        install -p -m0644 debian/.debhelper/generated/nginx-light/dh_installchangelogs.dch.trimmed debian/nginx-light/usr/share/doc/nginx-light/changelog.Debian
        install -p -m0644 debian/.debhelper/generated/nginx-light/dh_installchangelogs.news.trimmed debian/nginx-light/usr/share/doc/nginx-light/NEWS.Debian
        install -m0755 -d debian/nginx-extras/usr/share/doc/nginx-extras
        install -p -m0644 debian/.debhelper/generated/libnginx-mod-stream/dh_installchangelogs.dch.trimmed debian/libnginx-mod-stream/usr/share/doc/libnginx-mod-stream/changelog.Debian
        install -m0755 -d debian/libnginx-mod-stream-geoip/usr/share/doc/libnginx-mod-stream-geoip
        install -p -m0644 debian/.debhelper/generated/nginx-extras/dh_installchangelogs.dch.trimmed debian/nginx-extras/usr/share/doc/nginx-extras/changelog.Debian
        install -p -m0644 debian/.debhelper/generated/nginx-extras/dh_installchangelogs.news.trimmed debian/nginx-extras/usr/share/doc/nginx-extras/NEWS.Debian
        install -p -m0644 debian/.debhelper/generated/libnginx-mod-stream-geoip/dh_installchangelogs.dch.trimmed debian/libnginx-mod-stream-geoip/usr/share/doc/libnginx-mod-stream-geoip/changelog.Debian
        install -m0755 -d debian/libnginx-mod-http-perl/usr/share/doc/libnginx-mod-http-perl
        install -p -m0644 debian/.debhelper/generated/libnginx-mod-http-perl/dh_installchangelogs.dch.trimmed debian/libnginx-mod-http-perl/usr/share/doc/libnginx-mod-http-perl/changelog.Debian
   dh_installexamples
        install -m0755 -d debian/nginx-doc/usr/share/doc/nginx/examples
        cp --reflink=auto -a ./debian/help/examples/drupal debian/nginx-doc/usr/share/doc/nginx/examples
        cp --reflink=auto -a ./debian/help/examples/http debian/nginx-doc/usr/share/doc/nginx/examples
        cp --reflink=auto -a ./debian/help/examples/mail debian/nginx-doc/usr/share/doc/nginx/examples
        cp --reflink=auto -a ./debian/help/examples/mailman debian/nginx-doc/usr/share/doc/nginx/examples
        cp --reflink=auto -a ./debian/help/examples/nginx.conf debian/nginx-doc/usr/share/doc/nginx/examples
        cp --reflink=auto -a ./debian/help/examples/nginx_modsite debian/nginx-doc/usr/share/doc/nginx/examples
        cp --reflink=auto -a ./debian/help/examples/virtual_hosts debian/nginx-doc/usr/share/doc/nginx/examples
        cp --reflink=auto -a ./debian/help/examples/wordpress debian/nginx-doc/usr/share/doc/nginx/examples
   dh_installman
        install -m0755 -d debian/nginx/usr/share/man/man8/
        install -p -m0644 ./debian/build-bin/objs/nginx.8 debian/nginx/usr/share/man/man8/nginx.8
        install -m0755 -d debian/nginx-dev/usr/share/man/man1/
        install -p -m0644 ./debian/build-src/dh_nginx.1 debian/nginx-dev/usr/share/man/man1/dh_nginx.1
        man-recode --to-code UTF-8 --suffix .dh-new debian/nginx/usr/share/man/man8/nginx.8
        man-recode --to-code UTF-8 --suffix .dh-new debian/nginx-dev/usr/share/man/man1/dh_nginx.1
        mv debian/nginx-dev/usr/share/man/man1/dh_nginx.1.dh-new debian/nginx-dev/usr/share/man/man1/dh_nginx.1
        chmod 0644 -- debian/nginx-dev/usr/share/man/man1/dh_nginx.1
        mv debian/nginx/usr/share/man/man8/nginx.8.dh-new debian/nginx/usr/share/man/man8/nginx.8
        chmod 0644 -- debian/nginx/usr/share/man/man8/nginx.8
   dh_installdebconf
        install -m0755 -d debian/nginx/DEBIAN
        install -m0755 -d debian/nginx-doc/DEBIAN
        install -m0755 -d debian/nginx-common/DEBIAN
        cp -f debian/nginx-common.config debian/nginx-common/DEBIAN/config
        [META] Replace #TOKEN#s in "debian/nginx-common/DEBIAN/config"
        chmod 0755 -- debian/nginx-common/DEBIAN/config
        po2debconf  debian/nginx-common.templates > debian/nginx-common/DEBIAN/templates
        mv debian/nginx-common.substvars.new debian/nginx-common.substvars
        [META] Append autosnippet "postrm-debconf" to postrm [debian/nginx-common.postrm.debhelper]
        install -m0755 -d debian/nginx-dev/DEBIAN
        install -m0755 -d debian/nginx-core/DEBIAN
        install -m0755 -d debian/nginx-full/DEBIAN
        install -m0755 -d debian/nginx-light/DEBIAN
        install -m0755 -d debian/nginx-extras/DEBIAN
        install -m0755 -d debian/libnginx-mod-http-geoip/DEBIAN
        install -m0755 -d debian/libnginx-mod-http-image-filter/DEBIAN
        install -m0755 -d debian/libnginx-mod-http-xslt-filter/DEBIAN
        install -m0755 -d debian/libnginx-mod-mail/DEBIAN
        install -m0755 -d debian/libnginx-mod-stream/DEBIAN
        install -m0755 -d debian/libnginx-mod-stream-geoip/DEBIAN
        install -m0755 -d debian/libnginx-mod-http-perl/DEBIAN
        rm -f debian/libnginx-mod-http-geoip.debhelper.log debian/libnginx-mod-http-image-filter.debhelper.log debian/libnginx-mod-http-perl.debhelper.log debian/libnginx-mod-http-xslt-filter.debhelper.log debian/libnginx-mod-mail.debhelper.log debian/libnginx-mod-stream-geoip.debhelper.log debian/libnginx-mod-stream.debhelper.log debian/nginx-common.debhelper.log debian/nginx-core.debhelper.log debian/nginx-dev.debhelper.log debian/nginx-doc.debhelper.log debian/nginx-extras.debhelper.log debian/nginx-full.debhelper.log debian/nginx-light.debhelper.log debian/nginx.debhelper.log
   debian/rules override_dh_installinit
make[1]: Entering directory '/root/custom-nginx/nginx-1.24.0'
dh_installinit --no-stop-on-upgrade --no-start --name=nginx
        install -m0755 -d debian/nginx-common/etc/default
        install -p -m0644 debian/nginx-common.nginx.default debian/nginx-common/etc/default/nginx
        install -m0755 -d debian/nginx-common/etc/init.d
        install -p -m0755 debian/nginx-common.nginx.init debian/nginx-common/etc/init.d/nginx
        [META] Append autosnippet "preinst-init-chmod" to preinst [debian/.debhelper/generated/nginx-common/preinst.service]
        [META] Append autosnippet "postinst-init-nostart" to postinst [debian/.debhelper/generated/nginx-common/postinst.service]
        [META] Append autosnippet "postrm-init" to postrm [debian/.debhelper/generated/nginx-common/postrm.service]
make[1]: Leaving directory '/root/custom-nginx/nginx-1.24.0'
        rm -f debian/libnginx-mod-http-geoip.debhelper.log debian/libnginx-mod-http-image-filter.debhelper.log debian/libnginx-mod-http-perl.debhelper.log debian/libnginx-mod-http-xslt-filter.debhelper.log debian/libnginx-mod-mail.debhelper.log debian/libnginx-mod-stream-geoip.debhelper.log debian/libnginx-mod-stream.debhelper.log debian/nginx-common.debhelper.log debian/nginx-core.debhelper.log debian/nginx-dev.debhelper.log debian/nginx-doc.debhelper.log debian/nginx-extras.debhelper.log debian/nginx-full.debhelper.log debian/nginx-light.debhelper.log debian/nginx.debhelper.log
   debian/rules override_dh_installsystemd
make[1]: Entering directory '/root/custom-nginx/nginx-1.24.0'
dh_installsystemd --name=nginx
        install -m0755 -d debian/nginx-common/usr/lib/systemd/system
        install -p -m0644 debian/nginx-common.nginx.service debian/nginx-common/usr/lib/systemd/system/nginx.service
        [META] Append autosnippet "postinst-systemd-enable" to postinst [debian/.debhelper/generated/nginx-common/postinst.service]
        [META] Prepend autosnippet "postrm-systemd" to postrm [debian/nginx-common.postrm.debhelper.new]
        mv debian/nginx-common.postrm.debhelper.new debian/nginx-common.postrm.debhelper
        [META] Append autosnippet "postinst-systemd-restart" to postinst [debian/.debhelper/generated/nginx-common/postinst.service]
        [META] Append autosnippet "prerm-systemd-restart" to prerm [debian/.debhelper/generated/nginx-common/prerm.service]
        [META] Prepend autosnippet "postrm-systemd-reload-only" to postrm [debian/nginx-common.postrm.debhelper.new]
        mv debian/nginx-common.postrm.debhelper.new debian/nginx-common.postrm.debhelper
make[1]: Leaving directory '/root/custom-nginx/nginx-1.24.0'
        rm -f debian/libnginx-mod-http-geoip.debhelper.log debian/libnginx-mod-http-image-filter.debhelper.log debian/libnginx-mod-http-perl.debhelper.log debian/libnginx-mod-http-xslt-filter.debhelper.log debian/libnginx-mod-mail.debhelper.log debian/libnginx-mod-stream-geoip.debhelper.log debian/libnginx-mod-stream.debhelper.log debian/nginx-common.debhelper.log debian/nginx-core.debhelper.log debian/nginx-dev.debhelper.log debian/nginx-doc.debhelper.log debian/nginx-extras.debhelper.log debian/nginx-full.debhelper.log debian/nginx-light.debhelper.log debian/nginx.debhelper.log
   debian/rules override_dh_installlogrotate
make[1]: Entering directory '/root/custom-nginx/nginx-1.24.0'
dh_installlogrotate --package nginx-common --name=nginx
        install -m0755 -d debian/nginx-common/etc/logrotate.d
        install -p -m0644 debian/nginx-common.nginx.logrotate debian/nginx-common/etc/logrotate.d/nginx
make[1]: Leaving directory '/root/custom-nginx/nginx-1.24.0'
   dh_perl
        mv debian/nginx-dev.substvars.new debian/nginx-dev.substvars
        rmdir --ignore-fail-on-non-empty --parents debian/nginx-dev/usr/share/perl5
        mv debian/libnginx-mod-http-perl.substvars.new debian/libnginx-mod-http-perl.substvars
        mv debian/libnginx-mod-http-perl.substvars.new debian/libnginx-mod-http-perl.substvars
        rmdir --ignore-fail-on-non-empty --parents debian/libnginx-mod-http-perl/usr/lib/x86_64-linux-gnu/perl5/5.38
   dh_link
        install -m0755 -d debian/nginx-common/usr/share/nginx
        rm -f debian/nginx-common/usr/share/nginx/modules
        ln -s ../../lib/nginx/modules debian/nginx-common/usr/share/nginx/modules
   dh_strip_nondeterminism
   dh_compress
        cd debian/nginx
        cd debian/libnginx-mod-http-geoip
        chmod a-x usr/share/doc/nginx/changelog.Debian usr/share/man/man8/nginx.8
        chmod a-x usr/share/doc/libnginx-mod-http-geoip/changelog.Debian
        gzip -9nf usr/share/doc/nginx/changelog.Debian usr/share/man/man8/nginx.8
        gzip -9nf usr/share/doc/libnginx-mod-http-geoip/changelog.Debian
        cd '/root/custom-nginx/nginx-1.24.0'
        cd '/root/custom-nginx/nginx-1.24.0'
        cd debian/nginx-doc
        cd debian/libnginx-mod-http-image-filter
        chmod a-x usr/share/doc/nginx-doc/changelog.Debian
        gzip -9nf usr/share/doc/nginx-doc/changelog.Debian
        chmod a-x usr/share/doc/libnginx-mod-http-image-filter/changelog.Debian
        cd '/root/custom-nginx/nginx-1.24.0'
        gzip -9nf usr/share/doc/libnginx-mod-http-image-filter/changelog.Debian
        cd debian/nginx-common
        cd '/root/custom-nginx/nginx-1.24.0'
        cd debian/libnginx-mod-http-xslt-filter
        chmod a-x usr/share/doc/nginx-common/changelog.Debian
        gzip -9nf usr/share/doc/nginx-common/changelog.Debian
        chmod a-x usr/share/doc/libnginx-mod-http-xslt-filter/changelog.Debian
        cd '/root/custom-nginx/nginx-1.24.0'
        gzip -9nf usr/share/doc/libnginx-mod-http-xslt-filter/changelog.Debian
        cd debian/nginx-dev
        cd '/root/custom-nginx/nginx-1.24.0'
        cd debian/libnginx-mod-mail
        chmod a-x usr/share/doc/nginx-dev/changelog.Debian usr/share/man/man1/dh_nginx.1
        gzip -9nf usr/share/doc/nginx-dev/changelog.Debian usr/share/man/man1/dh_nginx.1
        cd '/root/custom-nginx/nginx-1.24.0'
        chmod a-x usr/share/doc/libnginx-mod-mail/changelog.Debian
        gzip -9nf usr/share/doc/libnginx-mod-mail/changelog.Debian
        cd debian/nginx-core
        cd '/root/custom-nginx/nginx-1.24.0'
        cd debian/libnginx-mod-stream
        chmod a-x usr/share/doc/nginx-core/NEWS.Debian usr/share/doc/nginx-core/changelog.Debian
        gzip -9nf usr/share/doc/nginx-core/NEWS.Debian usr/share/doc/nginx-core/changelog.Debian
        cd '/root/custom-nginx/nginx-1.24.0'
        chmod a-x usr/share/doc/libnginx-mod-stream/changelog.Debian
        gzip -9nf usr/share/doc/libnginx-mod-stream/changelog.Debian
        cd debian/nginx-full
        cd '/root/custom-nginx/nginx-1.24.0'
        cd debian/libnginx-mod-stream-geoip
        chmod a-x usr/share/doc/nginx-full/NEWS.Debian usr/share/doc/nginx-full/changelog.Debian
        gzip -9nf usr/share/doc/nginx-full/NEWS.Debian usr/share/doc/nginx-full/changelog.Debian
        chmod a-x usr/share/doc/libnginx-mod-stream-geoip/changelog.Debian
        gzip -9nf usr/share/doc/libnginx-mod-stream-geoip/changelog.Debian
        cd '/root/custom-nginx/nginx-1.24.0'
        cd debian/nginx-light
        cd '/root/custom-nginx/nginx-1.24.0'
        cd debian/libnginx-mod-http-perl
        chmod a-x usr/share/doc/nginx-light/NEWS.Debian usr/share/doc/nginx-light/changelog.Debian
        chmod a-x usr/share/doc/libnginx-mod-http-perl/changelog.Debian
        gzip -9nf usr/share/doc/nginx-light/NEWS.Debian usr/share/doc/nginx-light/changelog.Debian
        gzip -9nf usr/share/doc/libnginx-mod-http-perl/changelog.Debian
        cd '/root/custom-nginx/nginx-1.24.0'
        cd '/root/custom-nginx/nginx-1.24.0'
        cd debian/nginx-extras
        chmod a-x usr/share/doc/nginx-extras/NEWS.Debian usr/share/doc/nginx-extras/changelog.Debian
        gzip -9nf usr/share/doc/nginx-extras/NEWS.Debian usr/share/doc/nginx-extras/changelog.Debian
        cd '/root/custom-nginx/nginx-1.24.0'
   dh_fixperms
        find debian/nginx ! -type l -a -true -a -true -print0 2>/dev/null | xargs -0r chmod go=rX,u+rw,a-s
        find debian/libnginx-mod-http-geoip ! -type l -a -true -a -true -print0 2>/dev/null | xargs -0r chmod go=rX,u+rw,a-s
        find debian/nginx/usr/share/doc -type f -a -true -a ! -regex 'debian/nginx/usr/share/doc/[^/]*/examples/.*' -print0
2>/dev/null | xargs -0r chmod 0644
        find debian/libnginx-mod-http-geoip/usr/share/doc -type f -a -true -a ! -regex 'debian/libnginx-mod-http-geoip/usr/share/doc/[^/]*/examples/.*' -print0 2>/dev/null | xargs -0r chmod 0644
        find debian/nginx/usr/share/doc -type d -a -true -a -true -print0 2>/dev/null | xargs -0r chmod 0755
        find debian/libnginx-mod-http-geoip/usr/share/doc -type d -a -true -a -true -print0 2>/dev/null | xargs -0r chmod 0755
        find debian/nginx/usr/share/man -type f -a -true -a -true -print0 2>/dev/null | xargs -0r chmod 0644
        find debian/libnginx-mod-http-geoip -type f \( -name '*.so.*' -o -name '*.so' -o -name '*.la' -o -name '*.a' -o -name '*.js' -o -name '*.css' -o -name '*.scss' -o -name '*.sass' -o -name '*.jpeg' -o -name '*.jpg' -o -name '*.png' -o -name '*.gif' -o -name '*.cmxs' -o -name '*.node' \) -a -true -a -true -print0 2>/dev/null | xargs -0r chmod 0644
        find debian/nginx -type f \( -name '*.so.*' -o -name '*.so' -o -name '*.la' -o -name '*.a' -o -name '*.js' -o -name
'*.css' -o -name '*.scss' -o -name '*.sass' -o -name '*.jpeg' -o -name '*.jpg' -o -name '*.png' -o -name '*.gif' -o -name '*.cmxs' -o -name '*.node' \) -a -true -a -true -print0 2>/dev/null | xargs -0r chmod 0644
        find debian/nginx/usr/sbin -type f -a -true -a -true -print0 2>/dev/null | xargs -0r chmod a+x
        find debian/libnginx-mod-http-geoip/usr/lib -type f -name '*.ali' -a -true -a -true -print0 2>/dev/null | xargs -0r
chmod uga-w
        find debian/libnginx-mod-http-image-filter ! -type l -a -true -a -true -print0 2>/dev/null | xargs -0r chmod go=rX,u+rw,a-s
        find debian/nginx-doc ! -type l -a -true -a -true -print0 2>/dev/null | xargs -0r chmod go=rX,u+rw,a-s
        find debian/nginx-doc/usr/share/doc -type f -a -true -a ! -regex 'debian/nginx-doc/usr/share/doc/[^/]*/examples/.*'
-print0 2>/dev/null | xargs -0r chmod 0644
        find debian/libnginx-mod-http-image-filter/usr/share/doc -type f -a -true -a ! -regex 'debian/libnginx-mod-http-image-filter/usr/share/doc/[^/]*/examples/.*' -print0 2>/dev/null | xargs -0r chmod 0644
        find debian/nginx-doc/usr/share/doc -type d -a -true -a -true -print0 2>/dev/null | xargs -0r chmod 0755
        find debian/libnginx-mod-http-image-filter/usr/share/doc -type d -a -true -a -true -print0 2>/dev/null | xargs -0r chmod 0755
        find debian/nginx-doc -type f \( -name '*.so.*' -o -name '*.so' -o -name '*.la' -o -name '*.a' -o -name '*.js' -o -name '*.css' -o -name '*.scss' -o -name '*.sass' -o -name '*.jpeg' -o -name '*.jpg' -o -name '*.png' -o -name '*.gif' -o -name '*.cmxs' -o -name '*.node' \) -a -true -a -true -print0 2>/dev/null | xargs -0r chmod 0644
        find debian/libnginx-mod-http-image-filter -type f \( -name '*.so.*' -o -name '*.so' -o -name '*.la' -o -name '*.a'
-o -name '*.js' -o -name '*.css' -o -name '*.scss' -o -name '*.sass' -o -name '*.jpeg' -o -name '*.jpg' -o -name '*.png' -o
-name '*.gif' -o -name '*.cmxs' -o -name '*.node' \) -a -true -a -true -print0 2>/dev/null | xargs -0r chmod 0644
        find debian/nginx-common ! -type l -a -true -a -true -print0 2>/dev/null | xargs -0r chmod go=rX,u+rw,a-s
        find debian/libnginx-mod-http-image-filter/usr/lib -type f -name '*.ali' -a -true -a -true -print0 2>/dev/null | xargs -0r chmod uga-w
        find debian/libnginx-mod-http-xslt-filter ! -type l -a -true -a -true -print0 2>/dev/null | xargs -0r chmod go=rX,u+rw,a-s
        find debian/nginx-common/usr/share/doc -type f -a -true -a ! -regex 'debian/nginx-common/usr/share/doc/[^/]*/examples/.*' -print0 2>/dev/null | xargs -0r chmod 0644
        find debian/libnginx-mod-http-xslt-filter/usr/share/doc -type f -a -true -a ! -regex 'debian/libnginx-mod-http-xslt-filter/usr/share/doc/[^/]*/examples/.*' -print0 2>/dev/null | xargs -0r chmod 0644
        find debian/nginx-common/usr/share/doc -type d -a -true -a -true -print0 2>/dev/null | xargs -0r chmod 0755
        find debian/nginx-common -type f \( -name '*.so.*' -o -name '*.so' -o -name '*.la' -o -name '*.a' -o -name '*.js' -o -name '*.css' -o -name '*.scss' -o -name '*.sass' -o -name '*.jpeg' -o -name '*.jpg' -o -name '*.png' -o -name '*.gif' -o -name '*.cmxs' -o -name '*.node' \) -a -true -a -true -print0 2>/dev/null | xargs -0r chmod 0644
        find debian/libnginx-mod-http-xslt-filter/usr/share/doc -type d -a -true -a -true -print0 2>/dev/null | xargs -0r chmod 0755
        find debian/libnginx-mod-http-xslt-filter -type f \( -name '*.so.*' -o -name '*.so' -o -name '*.la' -o -name '*.a' -o -name '*.js' -o -name '*.css' -o -name '*.scss' -o -name '*.sass' -o -name '*.jpeg' -o -name '*.jpg' -o -name '*.png' -o -name '*.gif' -o -name '*.cmxs' -o -name '*.node' \) -a -true -a -true -print0 2>/dev/null | xargs -0r chmod 0644
        find debian/nginx-common/etc/init.d -type f -a -true -a -true -print0 2>/dev/null | xargs -0r chmod a+x
        find debian/nginx-common/usr/lib -type f -name '*.ali' -a -true -a -true -print0 2>/dev/null | xargs -0r chmod uga-w
        find debian/libnginx-mod-http-xslt-filter/usr/lib -type f -name '*.ali' -a -true -a -true -print0 2>/dev/null | xargs -0r chmod uga-w
        find debian/nginx-dev ! -type l -a -true -a -true -print0 2>/dev/null | xargs -0r chmod go=rX,u+rw,a-s
        find debian/libnginx-mod-mail ! -type l -a -true -a -true -print0 2>/dev/null | xargs -0r chmod go=rX,u+rw,a-s
        find debian/libnginx-mod-mail/usr/share/doc -type f -a -true -a ! -regex 'debian/libnginx-mod-mail/usr/share/doc/[^/]*/examples/.*' -print0 2>/dev/null | xargs -0r chmod 0644
        find debian/nginx-dev/usr/share/doc -type f -a -true -a ! -regex 'debian/nginx-dev/usr/share/doc/[^/]*/examples/.*'
-print0 2>/dev/null | xargs -0r chmod 0644
        find debian/libnginx-mod-mail/usr/share/doc -type d -a -true -a -true -print0 2>/dev/null | xargs -0r chmod 0755
        find debian/nginx-dev/usr/share/doc -type d -a -true -a -true -print0 2>/dev/null | xargs -0r chmod 0755
        find debian/nginx-dev/usr/share/man -type f -a -true -a -true -print0 2>/dev/null | xargs -0r chmod 0644
        find debian/libnginx-mod-mail -type f \( -name '*.so.*' -o -name '*.so' -o -name '*.la' -o -name '*.a' -o -name '*.js' -o -name '*.css' -o -name '*.scss' -o -name '*.sass' -o -name '*.jpeg' -o -name '*.jpg' -o -name '*.png' -o -name '*.gif' -o -name '*.cmxs' -o -name '*.node' \) -a -true -a -true -print0 2>/dev/null | xargs -0r chmod 0644
        find debian/libnginx-mod-mail/usr/lib -type f -name '*.ali' -a -true -a -true -print0 2>/dev/null | xargs -0r chmod
uga-w
        find debian/nginx-dev/usr/share/perl5 -type f -perm -5 -name '*.pm' -a -true -a -true -print0 2>/dev/null | xargs -0r chmod a-X
        find debian/nginx-dev -type f \( -name '*.so.*' -o -name '*.so' -o -name '*.la' -o -name '*.a' -o -name '*.js' -o -name '*.css' -o -name '*.scss' -o -name '*.sass' -o -name '*.jpeg' -o -name '*.jpg' -o -name '*.png' -o -name '*.gif' -o -name '*.cmxs' -o -name '*.node' \) -a -true -a -true -print0 2>/dev/null | xargs -0r chmod 0644
        find debian/libnginx-mod-stream ! -type l -a -true -a -true -print0 2>/dev/null | xargs -0r chmod go=rX,u+rw,a-s
        find debian/nginx-dev/usr/bin -type f -a -true -a -true -print0 2>/dev/null | xargs -0r chmod a+x
        find debian/libnginx-mod-stream/usr/share/doc -type f -a -true -a ! -regex 'debian/libnginx-mod-stream/usr/share/doc/[^/]*/examples/.*' -print0 2>/dev/null | xargs -0r chmod 0644
        find debian/libnginx-mod-stream/usr/share/doc -type d -a -true -a -true -print0 2>/dev/null | xargs -0r chmod 0755
        find debian/nginx-core ! -type l -a -true -a -true -print0 2>/dev/null | xargs -0r chmod go=rX,u+rw,a-s
        find debian/nginx-core/usr/share/doc -type f -a -true -a ! -regex 'debian/nginx-core/usr/share/doc/[^/]*/examples/.*' -print0 2>/dev/null | xargs -0r chmod 0644
        find debian/libnginx-mod-stream -type f \( -name '*.so.*' -o -name '*.so' -o -name '*.la' -o -name '*.a' -o -name '*.js' -o -name '*.css' -o -name '*.scss' -o -name '*.sass' -o -name '*.jpeg' -o -name '*.jpg' -o -name '*.png' -o -name '*.gif' -o -name '*.cmxs' -o -name '*.node' \) -a -true -a -true -print0 2>/dev/null | xargs -0r chmod 0644
        find debian/nginx-core/usr/share/doc -type d -a -true -a -true -print0 2>/dev/null | xargs -0r chmod 0755
        find debian/libnginx-mod-stream/usr/lib -type f -name '*.ali' -a -true -a -true -print0 2>/dev/null | xargs -0r chmod uga-w
        find debian/libnginx-mod-stream-geoip ! -type l -a -true -a -true -print0 2>/dev/null | xargs -0r chmod go=rX,u+rw,a-s
        find debian/nginx-core -type f \( -name '*.so.*' -o -name '*.so' -o -name '*.la' -o -name '*.a' -o -name '*.js' -o -name '*.css' -o -name '*.scss' -o -name '*.sass' -o -name '*.jpeg' -o -name '*.jpg' -o -name '*.png' -o -name '*.gif' -o -name '*.cmxs' -o -name '*.node' \) -a -true -a -true -print0 2>/dev/null | xargs -0r chmod 0644
        find debian/nginx-full ! -type l -a -true -a -true -print0 2>/dev/null | xargs -0r chmod go=rX,u+rw,a-s
        find debian/libnginx-mod-stream-geoip/usr/share/doc -type f -a -true -a ! -regex 'debian/libnginx-mod-stream-geoip/usr/share/doc/[^/]*/examples/.*' -print0 2>/dev/null | xargs -0r chmod 0644
        find debian/nginx-full/usr/share/doc -type f -a -true -a ! -regex 'debian/nginx-full/usr/share/doc/[^/]*/examples/.*' -print0 2>/dev/null | xargs -0r chmod 0644
        find debian/libnginx-mod-stream-geoip/usr/share/doc -type d -a -true -a -true -print0 2>/dev/null | xargs -0r chmod
0755
        find debian/nginx-full/usr/share/doc -type d -a -true -a -true -print0 2>/dev/null | xargs -0r chmod 0755
        find debian/libnginx-mod-stream-geoip -type f \( -name '*.so.*' -o -name '*.so' -o -name '*.la' -o -name '*.a' -o -name '*.js' -o -name '*.css' -o -name '*.scss' -o -name '*.sass' -o -name '*.jpeg' -o -name '*.jpg' -o -name '*.png' -o -name '*.gif' -o -name '*.cmxs' -o -name '*.node' \) -a -true -a -true -print0 2>/dev/null | xargs -0r chmod 0644
        find debian/nginx-full -type f \( -name '*.so.*' -o -name '*.so' -o -name '*.la' -o -name '*.a' -o -name '*.js' -o -name '*.css' -o -name '*.scss' -o -name '*.sass' -o -name '*.jpeg' -o -name '*.jpg' -o -name '*.png' -o -name '*.gif' -o -name '*.cmxs' -o -name '*.node' \) -a -true -a -true -print0 2>/dev/null | xargs -0r chmod 0644
        find debian/libnginx-mod-stream-geoip/usr/lib -type f -name '*.ali' -a -true -a -true -print0 2>/dev/null | xargs -0r chmod uga-w
        find debian/nginx-light ! -type l -a -true -a -true -print0 2>/dev/null | xargs -0r chmod go=rX,u+rw,a-s
        find debian/libnginx-mod-http-perl ! -type l -a -true -a -true -print0 2>/dev/null | xargs -0r chmod go=rX,u+rw,a-s
        find debian/nginx-light/usr/share/doc -type f -a -true -a ! -regex 'debian/nginx-light/usr/share/doc/[^/]*/examples/.*' -print0 2>/dev/null | xargs -0r chmod 0644
        find debian/libnginx-mod-http-perl/usr/share/doc -type f -a -true -a ! -regex 'debian/libnginx-mod-http-perl/usr/share/doc/[^/]*/examples/.*' -print0 2>/dev/null | xargs -0r chmod 0644
        find debian/nginx-light/usr/share/doc -type d -a -true -a -true -print0 2>/dev/null | xargs -0r chmod 0755
        find debian/libnginx-mod-http-perl/usr/share/doc -type d -a -true -a -true -print0 2>/dev/null | xargs -0r chmod 0755
        find debian/libnginx-mod-http-perl/usr/lib/x86_64-linux-gnu/perl5/5.38 -type f -perm -5 -name '*.pm' -a -true -a -true -print0 2>/dev/null | xargs -0r chmod a-X
        find debian/nginx-light -type f \( -name '*.so.*' -o -name '*.so' -o -name '*.la' -o -name '*.a' -o -name '*.js' -o
-name '*.css' -o -name '*.scss' -o -name '*.sass' -o -name '*.jpeg' -o -name '*.jpg' -o -name '*.png' -o -name '*.gif' -o -name '*.cmxs' -o -name '*.node' \) -a -true -a -true -print0 2>/dev/null | xargs -0r chmod 0644
        find debian/libnginx-mod-http-perl -type f \( -name '*.so.*' -o -name '*.so' -o -name '*.la' -o -name '*.a' -o -name '*.js' -o -name '*.css' -o -name '*.scss' -o -name '*.sass' -o -name '*.jpeg' -o -name '*.jpg' -o -name '*.png' -o -name '*.gif' -o -name '*.cmxs' -o -name '*.node' \) -a -true -a -true -print0 2>/dev/null | xargs -0r chmod 0644
        find debian/nginx-extras ! -type l -a -true -a -true -print0 2>/dev/null | xargs -0r chmod go=rX,u+rw,a-s
        find debian/nginx-extras/usr/share/doc -type f -a -true -a ! -regex 'debian/nginx-extras/usr/share/doc/[^/]*/examples/.*' -print0 2>/dev/null | xargs -0r chmod 0644
        find debian/libnginx-mod-http-perl/usr/lib -type f -name '*.ali' -a -true -a -true -print0 2>/dev/null | xargs -0r chmod uga-w
        find debian/nginx-extras/usr/share/doc -type d -a -true -a -true -print0 2>/dev/null | xargs -0r chmod 0755
        find debian/nginx-extras -type f \( -name '*.so.*' -o -name '*.so' -o -name '*.la' -o -name '*.a' -o -name '*.js' -o -name '*.css' -o -name '*.scss' -o -name '*.sass' -o -name '*.jpeg' -o -name '*.jpg' -o -name '*.png' -o -name '*.gif' -o -name '*.cmxs' -o -name '*.node' \) -a -true -a -true -print0 2>/dev/null | xargs -0r chmod 0644
   dh_missing
   dh_dwz -a
        dwz -- debian/nginx/usr/sbin/nginx
        dwz -- debian/libnginx-mod-mail/usr/lib/nginx/modules/ngx_mail_module.so
        dwz -- debian/libnginx-mod-stream/usr/lib/nginx/modules/ngx_stream_module.so
        dwz -- debian/libnginx-mod-stream-geoip/usr/lib/nginx/modules/ngx_stream_geoip_module.so
        install -m0755 -d debian/libnginx-mod-http-perl/usr/lib/debug/.dwz/x86_64-linux-gnu
        dwz -mdebian/libnginx-mod-http-perl/usr/lib/debug/.dwz/x86_64-linux-gnu/libnginx-mod-http-perl.debug -M/usr/lib/debug/.dwz/x86_64-linux-gnu/libnginx-mod-http-perl.debug -- debian/libnginx-mod-http-perl/usr/lib/nginx/modules/ngx_http_perl_module.so debian/libnginx-mod-http-perl/usr/lib/x86_64-linux-gnu/perl5/5.38/auto/nginx/nginx.so
        objcopy --compress-debug-sections debian/libnginx-mod-http-perl/usr/lib/debug/.dwz/x86_64-linux-gnu/libnginx-mod-http-perl.debug
        chmod 0644 -- debian/libnginx-mod-http-perl/usr/lib/debug/.dwz/x86_64-linux-gnu/libnginx-mod-http-perl.debug
        dwz -- debian/libnginx-mod-http-geoip/usr/lib/nginx/modules/ngx_http_geoip_module.so
        dwz -- debian/libnginx-mod-http-image-filter/usr/lib/nginx/modules/ngx_http_image_filter_module.so
        dwz -- debian/libnginx-mod-http-xslt-filter/usr/lib/nginx/modules/ngx_http_xslt_filter_module.so
   dh_strip -a
        debugedit --build-id --build-id-seed=libnginx-mod-mail/1.24.0-2ubuntu7-custom debian/libnginx-mod-mail/usr/lib/nginx/modules/ngx_mail_module.so
        debugedit --build-id --build-id-seed=nginx/1.24.0-2ubuntu7-custom debian/nginx/usr/sbin/nginx
56dabdce2de573fe92fc51be00a55be7d1c2c6d7
        install -m0755 -d debian/.debhelper/libnginx-mod-mail/dbgsym-root/usr/lib/debug/.build-id/56
        objcopy --only-keep-debug --compress-debug-sections debian/libnginx-mod-mail/usr/lib/nginx/modules/ngx_mail_module.so debian/.debhelper/libnginx-mod-mail/dbgsym-root/usr/lib/debug/.build-id/56/dabdce2de573fe92fc51be00a55be7d1c2c6d7.debug
        chmod 0644 -- debian/.debhelper/libnginx-mod-mail/dbgsym-root/usr/lib/debug/.build-id/56/dabdce2de573fe92fc51be00a55be7d1c2c6d7.debug
        strip --remove-section=.comment --remove-section=.note --strip-unneeded -o /tmp/MFAcdnpxXY/stripLkm8tC debian/libnginx-mod-mail/usr/lib/nginx/modules/ngx_mail_module.so
        chmod --reference debian/libnginx-mod-mail/usr/lib/nginx/modules/ngx_mail_module.so /tmp/MFAcdnpxXY/stripLkm8tC
        cat '/tmp/MFAcdnpxXY/stripLkm8tC' > 'debian/libnginx-mod-mail/usr/lib/nginx/modules/ngx_mail_module.so'
        chmod --reference /tmp/MFAcdnpxXY/stripLkm8tC debian/libnginx-mod-mail/usr/lib/nginx/modules/ngx_mail_module.so
        objcopy --add-gnu-debuglink debian/.debhelper/libnginx-mod-mail/dbgsym-root/usr/lib/debug/.build-id/56/dabdce2de573fe92fc51be00a55be7d1c2c6d7.debug debian/libnginx-mod-mail/usr/lib/nginx/modules/ngx_mail_module.so /tmp/MFAcdnpxXY/objcopyM7jC_q
        chmod --reference debian/libnginx-mod-mail/usr/lib/nginx/modules/ngx_mail_module.so /tmp/MFAcdnpxXY/objcopyM7jC_q
        cat '/tmp/MFAcdnpxXY/objcopyM7jC_q' > 'debian/libnginx-mod-mail/usr/lib/nginx/modules/ngx_mail_module.so'
        chmod --reference /tmp/MFAcdnpxXY/objcopyM7jC_q debian/libnginx-mod-mail/usr/lib/nginx/modules/ngx_mail_module.so
        install -m0755 -d debian/.debhelper/libnginx-mod-mail/dbgsym-root/usr/share/doc
        ln -s libnginx-mod-mail debian/.debhelper/libnginx-mod-mail/dbgsym-root/usr/share/doc/libnginx-mod-mail-dbgsym
        install -m0755 -d debian/.debhelper/libnginx-mod-mail
        debugedit --build-id --build-id-seed=libnginx-mod-stream/1.24.0-2ubuntu7-custom debian/libnginx-mod-stream/usr/lib/nginx/modules/ngx_stream_module.so
9b936a69c2e956bf0014fdcbd6dd9d89468aac27
ebd42074efb069d4f4d7fa5c01531c7c57339898
        install -m0755 -d debian/.debhelper/libnginx-mod-stream/dbgsym-root/usr/lib/debug/.build-id/eb
        objcopy --only-keep-debug --compress-debug-sections debian/libnginx-mod-stream/usr/lib/nginx/modules/ngx_stream_module.so debian/.debhelper/libnginx-mod-stream/dbgsym-root/usr/lib/debug/.build-id/eb/d42074efb069d4f4d7fa5c01531c7c57339898.debug
        install -m0755 -d debian/.debhelper/nginx/dbgsym-root/usr/lib/debug/.build-id/9b
        objcopy --only-keep-debug --compress-debug-sections debian/nginx/usr/sbin/nginx debian/.debhelper/nginx/dbgsym-root/usr/lib/debug/.build-id/9b/936a69c2e956bf0014fdcbd6dd9d89468aac27.debug
        chmod 0644 -- debian/.debhelper/libnginx-mod-stream/dbgsym-root/usr/lib/debug/.build-id/eb/d42074efb069d4f4d7fa5c01531c7c57339898.debug
        strip --remove-section=.comment --remove-section=.note --strip-unneeded -o /tmp/7_U25K7ScO/stripHvBDAt debian/libnginx-mod-stream/usr/lib/nginx/modules/ngx_stream_module.so
        chmod --reference debian/libnginx-mod-stream/usr/lib/nginx/modules/ngx_stream_module.so /tmp/7_U25K7ScO/stripHvBDAt
        cat '/tmp/7_U25K7ScO/stripHvBDAt' > 'debian/libnginx-mod-stream/usr/lib/nginx/modules/ngx_stream_module.so'
        chmod --reference /tmp/7_U25K7ScO/stripHvBDAt debian/libnginx-mod-stream/usr/lib/nginx/modules/ngx_stream_module.so
        objcopy --add-gnu-debuglink debian/.debhelper/libnginx-mod-stream/dbgsym-root/usr/lib/debug/.build-id/eb/d42074efb069d4f4d7fa5c01531c7c57339898.debug debian/libnginx-mod-stream/usr/lib/nginx/modules/ngx_stream_module.so /tmp/7_U25K7ScO/objcopydo9Q3i
        chmod --reference debian/libnginx-mod-stream/usr/lib/nginx/modules/ngx_stream_module.so /tmp/7_U25K7ScO/objcopydo9Q3i
        cat '/tmp/7_U25K7ScO/objcopydo9Q3i' > 'debian/libnginx-mod-stream/usr/lib/nginx/modules/ngx_stream_module.so'
        chmod --reference /tmp/7_U25K7ScO/objcopydo9Q3i debian/libnginx-mod-stream/usr/lib/nginx/modules/ngx_stream_module.so
        install -m0755 -d debian/.debhelper/libnginx-mod-stream/dbgsym-root/usr/share/doc
        ln -s libnginx-mod-stream debian/.debhelper/libnginx-mod-stream/dbgsym-root/usr/share/doc/libnginx-mod-stream-dbgsym
        install -m0755 -d debian/.debhelper/libnginx-mod-stream
        debugedit --build-id --build-id-seed=libnginx-mod-stream-geoip/1.24.0-2ubuntu7-custom debian/libnginx-mod-stream-geoip/usr/lib/nginx/modules/ngx_stream_geoip_module.so
28762fc6e00a08f00fc90b7b67dc8e5d140f8181
        install -m0755 -d debian/.debhelper/libnginx-mod-stream-geoip/dbgsym-root/usr/lib/debug/.build-id/28
        objcopy --only-keep-debug --compress-debug-sections debian/libnginx-mod-stream-geoip/usr/lib/nginx/modules/ngx_stream_geoip_module.so debian/.debhelper/libnginx-mod-stream-geoip/dbgsym-root/usr/lib/debug/.build-id/28/762fc6e00a08f00fc90b7b67dc8e5d140f8181.debug
        chmod 0644 -- debian/.debhelper/libnginx-mod-stream-geoip/dbgsym-root/usr/lib/debug/.build-id/28/762fc6e00a08f00fc90b7b67dc8e5d140f8181.debug
        strip --remove-section=.comment --remove-section=.note --strip-unneeded -o /tmp/cC_k7znvXZ/stripo8rxd_ debian/libnginx-mod-stream-geoip/usr/lib/nginx/modules/ngx_stream_geoip_module.so
        chmod --reference debian/libnginx-mod-stream-geoip/usr/lib/nginx/modules/ngx_stream_geoip_module.so /tmp/cC_k7znvXZ/stripo8rxd_
        cat '/tmp/cC_k7znvXZ/stripo8rxd_' > 'debian/libnginx-mod-stream-geoip/usr/lib/nginx/modules/ngx_stream_geoip_module.so'
        chmod --reference /tmp/cC_k7znvXZ/stripo8rxd_ debian/libnginx-mod-stream-geoip/usr/lib/nginx/modules/ngx_stream_geoip_module.so
        objcopy --add-gnu-debuglink debian/.debhelper/libnginx-mod-stream-geoip/dbgsym-root/usr/lib/debug/.build-id/28/762fc6e00a08f00fc90b7b67dc8e5d140f8181.debug debian/libnginx-mod-stream-geoip/usr/lib/nginx/modules/ngx_stream_geoip_module.so /tmp/cC_k7znvXZ/objcopyQlajKk
        chmod --reference debian/libnginx-mod-stream-geoip/usr/lib/nginx/modules/ngx_stream_geoip_module.so /tmp/cC_k7znvXZ/objcopyQlajKk
        cat '/tmp/cC_k7znvXZ/objcopyQlajKk' > 'debian/libnginx-mod-stream-geoip/usr/lib/nginx/modules/ngx_stream_geoip_module.so'
        chmod --reference /tmp/cC_k7znvXZ/objcopyQlajKk debian/libnginx-mod-stream-geoip/usr/lib/nginx/modules/ngx_stream_geoip_module.so
        install -m0755 -d debian/.debhelper/libnginx-mod-stream-geoip/dbgsym-root/usr/share/doc
        ln -s libnginx-mod-stream-geoip debian/.debhelper/libnginx-mod-stream-geoip/dbgsym-root/usr/share/doc/libnginx-mod-stream-geoip-dbgsym
        install -m0755 -d debian/.debhelper/libnginx-mod-stream-geoip
        debugedit --build-id --build-id-seed=libnginx-mod-http-perl/1.24.0-2ubuntu7-custom debian/libnginx-mod-http-perl/usr/lib/nginx/modules/ngx_http_perl_module.so
debugedit: debian/libnginx-mod-http-perl/usr/lib/nginx/modules/ngx_http_perl_module.so: Unknown DWARF DW_FORM_0x1f21
74a90fa4ef92e050798e5dd0e2c0c8b095918652
        install -m0755 -d debian/.debhelper/libnginx-mod-http-perl/dbgsym-root/usr/lib/debug/.build-id/74
        objcopy --only-keep-debug --compress-debug-sections debian/libnginx-mod-http-perl/usr/lib/nginx/modules/ngx_http_perl_module.so debian/.debhelper/libnginx-mod-http-perl/dbgsym-root/usr/lib/debug/.build-id/74/a90fa4ef92e050798e5dd0e2c0c8b095918652.debug
        chmod 0644 -- debian/.debhelper/libnginx-mod-http-perl/dbgsym-root/usr/lib/debug/.build-id/74/a90fa4ef92e050798e5dd0e2c0c8b095918652.debug
        strip --remove-section=.comment --remove-section=.note --strip-unneeded -o /tmp/jmoWoCQm8z/stripAoUy43 debian/libnginx-mod-http-perl/usr/lib/nginx/modules/ngx_http_perl_module.so
        chmod --reference debian/libnginx-mod-http-perl/usr/lib/nginx/modules/ngx_http_perl_module.so /tmp/jmoWoCQm8z/stripAoUy43
        cat '/tmp/jmoWoCQm8z/stripAoUy43' > 'debian/libnginx-mod-http-perl/usr/lib/nginx/modules/ngx_http_perl_module.so'
        chmod --reference /tmp/jmoWoCQm8z/stripAoUy43 debian/libnginx-mod-http-perl/usr/lib/nginx/modules/ngx_http_perl_module.so
        objcopy --add-gnu-debuglink debian/.debhelper/libnginx-mod-http-perl/dbgsym-root/usr/lib/debug/.build-id/74/a90fa4ef92e050798e5dd0e2c0c8b095918652.debug debian/libnginx-mod-http-perl/usr/lib/nginx/modules/ngx_http_perl_module.so /tmp/jmoWoCQm8z/objcopyHIQUeM
        chmod --reference debian/libnginx-mod-http-perl/usr/lib/nginx/modules/ngx_http_perl_module.so /tmp/jmoWoCQm8z/objcopyHIQUeM
        cat '/tmp/jmoWoCQm8z/objcopyHIQUeM' > 'debian/libnginx-mod-http-perl/usr/lib/nginx/modules/ngx_http_perl_module.so'
        chmod --reference /tmp/jmoWoCQm8z/objcopyHIQUeM debian/libnginx-mod-http-perl/usr/lib/nginx/modules/ngx_http_perl_module.so
        debugedit --build-id --build-id-seed=libnginx-mod-http-perl/1.24.0-2ubuntu7-custom debian/libnginx-mod-http-perl/usr/lib/x86_64-linux-gnu/perl5/5.38/auto/nginx/nginx.so
debugedit: debian/libnginx-mod-http-perl/usr/lib/x86_64-linux-gnu/perl5/5.38/auto/nginx/nginx.so: Unknown DWARF DW_FORM_0x1f20
19596dea4b0b382c8e0e77434aa0b7d69d117d17
        install -m0755 -d debian/.debhelper/libnginx-mod-http-perl/dbgsym-root/usr/lib/debug/.build-id/19
        objcopy --only-keep-debug --compress-debug-sections debian/libnginx-mod-http-perl/usr/lib/x86_64-linux-gnu/perl5/5.38/auto/nginx/nginx.so debian/.debhelper/libnginx-mod-http-perl/dbgsym-root/usr/lib/debug/.build-id/19/596dea4b0b382c8e0e77434aa0b7d69d117d17.debug
        chmod 0644 -- debian/.debhelper/libnginx-mod-http-perl/dbgsym-root/usr/lib/debug/.build-id/19/596dea4b0b382c8e0e77434aa0b7d69d117d17.debug
        strip --remove-section=.comment --remove-section=.note --strip-unneeded -o /tmp/jmoWoCQm8z/stripNCXS1_ debian/libnginx-mod-http-perl/usr/lib/x86_64-linux-gnu/perl5/5.38/auto/nginx/nginx.so
        chmod --reference debian/libnginx-mod-http-perl/usr/lib/x86_64-linux-gnu/perl5/5.38/auto/nginx/nginx.so /tmp/jmoWoCQm8z/stripNCXS1_
        cat '/tmp/jmoWoCQm8z/stripNCXS1_' > 'debian/libnginx-mod-http-perl/usr/lib/x86_64-linux-gnu/perl5/5.38/auto/nginx/nginx.so'
        chmod --reference /tmp/jmoWoCQm8z/stripNCXS1_ debian/libnginx-mod-http-perl/usr/lib/x86_64-linux-gnu/perl5/5.38/auto/nginx/nginx.so
        objcopy --add-gnu-debuglink debian/.debhelper/libnginx-mod-http-perl/dbgsym-root/usr/lib/debug/.build-id/19/596dea4b0b382c8e0e77434aa0b7d69d117d17.debug debian/libnginx-mod-http-perl/usr/lib/x86_64-linux-gnu/perl5/5.38/auto/nginx/nginx.so /tmp/jmoWoCQm8z/objcopyV3CM5o
        chmod --reference debian/libnginx-mod-http-perl/usr/lib/x86_64-linux-gnu/perl5/5.38/auto/nginx/nginx.so /tmp/jmoWoCQm8z/objcopyV3CM5o
        cat '/tmp/jmoWoCQm8z/objcopyV3CM5o' > 'debian/libnginx-mod-http-perl/usr/lib/x86_64-linux-gnu/perl5/5.38/auto/nginx/nginx.so'
        chmod --reference /tmp/jmoWoCQm8z/objcopyV3CM5o debian/libnginx-mod-http-perl/usr/lib/x86_64-linux-gnu/perl5/5.38/auto/nginx/nginx.so
        install -m0755 -d debian/.debhelper/libnginx-mod-http-perl/dbgsym-root/usr/lib/debug/.dwz
        cp --reflink=auto -a debian/libnginx-mod-http-perl/usr/lib/debug/.dwz/x86_64-linux-gnu debian/.debhelper/libnginx-mod-http-perl/dbgsym-root/usr/lib/debug/.dwz
        rm -fr debian/libnginx-mod-http-perl/usr/lib/debug/.dwz
        rmdir -p --ignore-fail-on-non-empty debian/libnginx-mod-http-perl/usr/lib/debug
        install -m0755 -d debian/.debhelper/libnginx-mod-http-perl/dbgsym-root/usr/share/doc
        ln -s libnginx-mod-http-perl debian/.debhelper/libnginx-mod-http-perl/dbgsym-root/usr/share/doc/libnginx-mod-http-perl-dbgsym
        install -m0755 -d debian/.debhelper/libnginx-mod-http-perl
        chmod 0644 -- debian/.debhelper/nginx/dbgsym-root/usr/lib/debug/.build-id/9b/936a69c2e956bf0014fdcbd6dd9d89468aac27.debug
        strip --remove-section=.comment --remove-section=.note -o /tmp/WMzRdpPsgh/stripZonkt4 debian/nginx/usr/sbin/nginx
        chmod --reference debian/nginx/usr/sbin/nginx /tmp/WMzRdpPsgh/stripZonkt4
        cat '/tmp/WMzRdpPsgh/stripZonkt4' > 'debian/nginx/usr/sbin/nginx'
        chmod --reference /tmp/WMzRdpPsgh/stripZonkt4 debian/nginx/usr/sbin/nginx
        objcopy --add-gnu-debuglink debian/.debhelper/nginx/dbgsym-root/usr/lib/debug/.build-id/9b/936a69c2e956bf0014fdcbd6dd9d89468aac27.debug debian/nginx/usr/sbin/nginx /tmp/WMzRdpPsgh/objcopyGLA1Qr
        chmod --reference debian/nginx/usr/sbin/nginx /tmp/WMzRdpPsgh/objcopyGLA1Qr
        cat '/tmp/WMzRdpPsgh/objcopyGLA1Qr' > 'debian/nginx/usr/sbin/nginx'
        chmod --reference /tmp/WMzRdpPsgh/objcopyGLA1Qr debian/nginx/usr/sbin/nginx
        install -m0755 -d debian/.debhelper/nginx/dbgsym-root/usr/share/doc
        ln -s nginx debian/.debhelper/nginx/dbgsym-root/usr/share/doc/nginx-dbgsym
        install -m0755 -d debian/.debhelper/nginx
        debugedit --build-id --build-id-seed=libnginx-mod-http-geoip/1.24.0-2ubuntu7-custom debian/libnginx-mod-http-geoip/usr/lib/nginx/modules/ngx_http_geoip_module.so
7b5b4ca2d6b7c6e4310dfac70d4356052a46db4b
        install -m0755 -d debian/.debhelper/libnginx-mod-http-geoip/dbgsym-root/usr/lib/debug/.build-id/7b
        objcopy --only-keep-debug --compress-debug-sections debian/libnginx-mod-http-geoip/usr/lib/nginx/modules/ngx_http_geoip_module.so debian/.debhelper/libnginx-mod-http-geoip/dbgsym-root/usr/lib/debug/.build-id/7b/5b4ca2d6b7c6e4310dfac70d4356052a46db4b.debug
        chmod 0644 -- debian/.debhelper/libnginx-mod-http-geoip/dbgsym-root/usr/lib/debug/.build-id/7b/5b4ca2d6b7c6e4310dfac70d4356052a46db4b.debug
        strip --remove-section=.comment --remove-section=.note --strip-unneeded -o /tmp/ytfquMMlfK/strippTTziI debian/libnginx-mod-http-geoip/usr/lib/nginx/modules/ngx_http_geoip_module.so
        chmod --reference debian/libnginx-mod-http-geoip/usr/lib/nginx/modules/ngx_http_geoip_module.so /tmp/ytfquMMlfK/strippTTziI
        cat '/tmp/ytfquMMlfK/strippTTziI' > 'debian/libnginx-mod-http-geoip/usr/lib/nginx/modules/ngx_http_geoip_module.so'
        chmod --reference /tmp/ytfquMMlfK/strippTTziI debian/libnginx-mod-http-geoip/usr/lib/nginx/modules/ngx_http_geoip_module.so
        objcopy --add-gnu-debuglink debian/.debhelper/libnginx-mod-http-geoip/dbgsym-root/usr/lib/debug/.build-id/7b/5b4ca2d6b7c6e4310dfac70d4356052a46db4b.debug debian/libnginx-mod-http-geoip/usr/lib/nginx/modules/ngx_http_geoip_module.so /tmp/ytfquMMlfK/objcopyFhTacl
        chmod --reference debian/libnginx-mod-http-geoip/usr/lib/nginx/modules/ngx_http_geoip_module.so /tmp/ytfquMMlfK/objcopyFhTacl
        cat '/tmp/ytfquMMlfK/objcopyFhTacl' > 'debian/libnginx-mod-http-geoip/usr/lib/nginx/modules/ngx_http_geoip_module.so'
        chmod --reference /tmp/ytfquMMlfK/objcopyFhTacl debian/libnginx-mod-http-geoip/usr/lib/nginx/modules/ngx_http_geoip_module.so
        install -m0755 -d debian/.debhelper/libnginx-mod-http-geoip/dbgsym-root/usr/share/doc
        ln -s libnginx-mod-http-geoip debian/.debhelper/libnginx-mod-http-geoip/dbgsym-root/usr/share/doc/libnginx-mod-http-geoip-dbgsym
        install -m0755 -d debian/.debhelper/libnginx-mod-http-geoip
        debugedit --build-id --build-id-seed=libnginx-mod-http-image-filter/1.24.0-2ubuntu7-custom debian/libnginx-mod-http-image-filter/usr/lib/nginx/modules/ngx_http_image_filter_module.so
8cd906362ba63ff4670d057c2ff40758334f2eff
        install -m0755 -d debian/.debhelper/libnginx-mod-http-image-filter/dbgsym-root/usr/lib/debug/.build-id/8c
        objcopy --only-keep-debug --compress-debug-sections debian/libnginx-mod-http-image-filter/usr/lib/nginx/modules/ngx_http_image_filter_module.so debian/.debhelper/libnginx-mod-http-image-filter/dbgsym-root/usr/lib/debug/.build-id/8c/d906362ba63ff4670d057c2ff40758334f2eff.debug
        chmod 0644 -- debian/.debhelper/libnginx-mod-http-image-filter/dbgsym-root/usr/lib/debug/.build-id/8c/d906362ba63ff4670d057c2ff40758334f2eff.debug
        strip --remove-section=.comment --remove-section=.note --strip-unneeded -o /tmp/qsaY0yKmgY/strip8W2mcX debian/libnginx-mod-http-image-filter/usr/lib/nginx/modules/ngx_http_image_filter_module.so
        chmod --reference debian/libnginx-mod-http-image-filter/usr/lib/nginx/modules/ngx_http_image_filter_module.so /tmp/qsaY0yKmgY/strip8W2mcX
        cat '/tmp/qsaY0yKmgY/strip8W2mcX' > 'debian/libnginx-mod-http-image-filter/usr/lib/nginx/modules/ngx_http_image_filter_module.so'
        chmod --reference /tmp/qsaY0yKmgY/strip8W2mcX debian/libnginx-mod-http-image-filter/usr/lib/nginx/modules/ngx_http_image_filter_module.so
        objcopy --add-gnu-debuglink debian/.debhelper/libnginx-mod-http-image-filter/dbgsym-root/usr/lib/debug/.build-id/8c/d906362ba63ff4670d057c2ff40758334f2eff.debug debian/libnginx-mod-http-image-filter/usr/lib/nginx/modules/ngx_http_image_filter_module.so /tmp/qsaY0yKmgY/objcopySRwHLX
        chmod --reference debian/libnginx-mod-http-image-filter/usr/lib/nginx/modules/ngx_http_image_filter_module.so /tmp/qsaY0yKmgY/objcopySRwHLX
        cat '/tmp/qsaY0yKmgY/objcopySRwHLX' > 'debian/libnginx-mod-http-image-filter/usr/lib/nginx/modules/ngx_http_image_filter_module.so'
        chmod --reference /tmp/qsaY0yKmgY/objcopySRwHLX debian/libnginx-mod-http-image-filter/usr/lib/nginx/modules/ngx_http_image_filter_module.so
        install -m0755 -d debian/.debhelper/libnginx-mod-http-image-filter/dbgsym-root/usr/share/doc
        ln -s libnginx-mod-http-image-filter debian/.debhelper/libnginx-mod-http-image-filter/dbgsym-root/usr/share/doc/libnginx-mod-http-image-filter-dbgsym
        install -m0755 -d debian/.debhelper/libnginx-mod-http-image-filter
        debugedit --build-id --build-id-seed=libnginx-mod-http-xslt-filter/1.24.0-2ubuntu7-custom debian/libnginx-mod-http-xslt-filter/usr/lib/nginx/modules/ngx_http_xslt_filter_module.so
d4d72ce7d560ce5865350938fcf83a30d2867ee5
        install -m0755 -d debian/.debhelper/libnginx-mod-http-xslt-filter/dbgsym-root/usr/lib/debug/.build-id/d4
        objcopy --only-keep-debug --compress-debug-sections debian/libnginx-mod-http-xslt-filter/usr/lib/nginx/modules/ngx_http_xslt_filter_module.so debian/.debhelper/libnginx-mod-http-xslt-filter/dbgsym-root/usr/lib/debug/.build-id/d4/d72ce7d560ce5865350938fcf83a30d2867ee5.debug
        chmod 0644 -- debian/.debhelper/libnginx-mod-http-xslt-filter/dbgsym-root/usr/lib/debug/.build-id/d4/d72ce7d560ce5865350938fcf83a30d2867ee5.debug
        strip --remove-section=.comment --remove-section=.note --strip-unneeded -o /tmp/jg0zpG9ROC/striphPKGux debian/libnginx-mod-http-xslt-filter/usr/lib/nginx/modules/ngx_http_xslt_filter_module.so
        chmod --reference debian/libnginx-mod-http-xslt-filter/usr/lib/nginx/modules/ngx_http_xslt_filter_module.so /tmp/jg0zpG9ROC/striphPKGux
        cat '/tmp/jg0zpG9ROC/striphPKGux' > 'debian/libnginx-mod-http-xslt-filter/usr/lib/nginx/modules/ngx_http_xslt_filter_module.so'
        chmod --reference /tmp/jg0zpG9ROC/striphPKGux debian/libnginx-mod-http-xslt-filter/usr/lib/nginx/modules/ngx_http_xslt_filter_module.so
        objcopy --add-gnu-debuglink debian/.debhelper/libnginx-mod-http-xslt-filter/dbgsym-root/usr/lib/debug/.build-id/d4/d72ce7d560ce5865350938fcf83a30d2867ee5.debug debian/libnginx-mod-http-xslt-filter/usr/lib/nginx/modules/ngx_http_xslt_filter_module.so /tmp/jg0zpG9ROC/objcopyoDz9kt
        chmod --reference debian/libnginx-mod-http-xslt-filter/usr/lib/nginx/modules/ngx_http_xslt_filter_module.so /tmp/jg0zpG9ROC/objcopyoDz9kt
        cat '/tmp/jg0zpG9ROC/objcopyoDz9kt' > 'debian/libnginx-mod-http-xslt-filter/usr/lib/nginx/modules/ngx_http_xslt_filter_module.so'
        chmod --reference /tmp/jg0zpG9ROC/objcopyoDz9kt debian/libnginx-mod-http-xslt-filter/usr/lib/nginx/modules/ngx_http_xslt_filter_module.so
        install -m0755 -d debian/.debhelper/libnginx-mod-http-xslt-filter/dbgsym-root/usr/share/doc
        ln -s libnginx-mod-http-xslt-filter debian/.debhelper/libnginx-mod-http-xslt-filter/dbgsym-root/usr/share/doc/libnginx-mod-http-xslt-filter-dbgsym
        install -m0755 -d debian/.debhelper/libnginx-mod-http-xslt-filter
   dh_makeshlibs -a
        rm -f debian/nginx/DEBIAN/shlibs
        rm -f debian/nginx-extras/DEBIAN/shlibs
        rm -f debian/libnginx-mod-http-geoip/DEBIAN/shlibs
        rm -f debian/libnginx-mod-http-image-filter/DEBIAN/shlibs
        rm -f debian/libnginx-mod-http-xslt-filter/DEBIAN/shlibs
        rm -f debian/libnginx-mod-mail/DEBIAN/shlibs
        rm -f debian/libnginx-mod-stream/DEBIAN/shlibs
        rm -f debian/libnginx-mod-stream-geoip/DEBIAN/shlibs
        rm -f debian/libnginx-mod-http-perl/DEBIAN/shlibs
   dh_shlibdeps -a
        install -m0755 -d debian/libnginx-mod-mail/DEBIAN
        install -m0755 -d debian/nginx/DEBIAN
        dpkg-shlibdeps -Tdebian/libnginx-mod-mail.substvars debian/libnginx-mod-mail/usr/lib/nginx/modules/ngx_mail_module.so
        dpkg-shlibdeps -Tdebian/nginx.substvars debian/nginx/usr/sbin/nginx
        install -m0755 -d debian/libnginx-mod-stream/DEBIAN
        dpkg-shlibdeps -Tdebian/libnginx-mod-stream.substvars debian/libnginx-mod-stream/usr/lib/nginx/modules/ngx_stream_module.so
        install -m0755 -d debian/libnginx-mod-stream-geoip/DEBIAN
        dpkg-shlibdeps -Tdebian/libnginx-mod-stream-geoip.substvars debian/libnginx-mod-stream-geoip/usr/lib/nginx/modules/ngx_stream_geoip_module.so
        install -m0755 -d debian/libnginx-mod-http-geoip/DEBIAN
        dpkg-shlibdeps -Tdebian/libnginx-mod-http-geoip.substvars debian/libnginx-mod-http-geoip/usr/lib/nginx/modules/ngx_http_geoip_module.so
        install -m0755 -d debian/libnginx-mod-http-perl/DEBIAN
        dpkg-shlibdeps -Tdebian/libnginx-mod-http-perl.substvars debian/libnginx-mod-http-perl/usr/lib/nginx/modules/ngx_http_perl_module.so debian/libnginx-mod-http-perl/usr/lib/x86_64-linux-gnu/perl5/5.38/auto/nginx/nginx.so
        install -m0755 -d debian/libnginx-mod-http-image-filter/DEBIAN
        dpkg-shlibdeps -Tdebian/libnginx-mod-http-image-filter.substvars debian/libnginx-mod-http-image-filter/usr/lib/nginx/modules/ngx_http_image_filter_module.so
dpkg-shlibdeps: warning: diversions involved - output may be incorrect
 diversion by libc6 from: /lib64/ld-linux-x86-64.so.2
dpkg-shlibdeps: warning: diversions involved - output may be incorrect
 diversion by libc6 to: /lib64/ld-linux-x86-64.so.2.usr-is-merged
        install -m0755 -d debian/libnginx-mod-http-xslt-filter/DEBIAN
        dpkg-shlibdeps -Tdebian/libnginx-mod-http-xslt-filter.substvars debian/libnginx-mod-http-xslt-filter/usr/lib/nginx/modules/ngx_http_xslt_filter_module.so
   dh_installdeb
        install -m0755 -d debian/nginx/DEBIAN
        cp -f debian/nginx.postinst debian/nginx/DEBIAN/postinst
        [META] Replace #TOKEN#s in "debian/nginx/DEBIAN/postinst"
        chmod 0755 -- debian/nginx/DEBIAN/postinst
        cp -f debian/nginx.prerm debian/nginx/DEBIAN/prerm
        [META] Replace #TOKEN#s in "debian/nginx/DEBIAN/prerm"
        chmod 0755 -- debian/nginx/DEBIAN/prerm
        install -p -m0644 debian/nginx.triggers debian/nginx/DEBIAN/triggers
        install -m0755 -d debian/nginx-doc/DEBIAN
        install -m0755 -d debian/nginx-common/DEBIAN
        cp -f debian/nginx-common.postinst debian/nginx-common/DEBIAN/postinst
        [META] Replace #TOKEN#s in "debian/nginx-common/DEBIAN/postinst"
        chmod 0755 -- debian/nginx-common/DEBIAN/postinst
        cp -f debian/nginx-common.preinst debian/nginx-common/DEBIAN/preinst
        [META] Replace #TOKEN#s in "debian/nginx-common/DEBIAN/preinst"
        chmod 0755 -- debian/nginx-common/DEBIAN/preinst
        printf '#!/bin/sh\nset -e\n' > debian/nginx-common/DEBIAN/prerm
        cat debian/.debhelper/generated/nginx-common/prerm.service >> debian/nginx-common/DEBIAN/prerm
        chmod 0755 -- debian/nginx-common/DEBIAN/prerm
        cp -f debian/nginx-common.postrm debian/nginx-common/DEBIAN/postrm
        [META] Replace #TOKEN#s in "debian/nginx-common/DEBIAN/postrm"
        chmod 0755 -- debian/nginx-common/DEBIAN/postrm
        find debian/nginx-common/etc -type f -printf '/etc/%P
' | LC_ALL=C sort >> debian/nginx-common/DEBIAN/conffiles
        chmod 0644 -- debian/nginx-common/DEBIAN/conffiles
        install -m0755 -d debian/nginx-dev/DEBIAN
        install -m0755 -d debian/nginx-core/DEBIAN
        install -m0755 -d debian/nginx-full/DEBIAN
        install -m0755 -d debian/nginx-light/DEBIAN
        install -m0755 -d debian/nginx-extras/DEBIAN
        install -m0755 -d debian/libnginx-mod-http-geoip/DEBIAN
        printf '#!/bin/sh\nset -e\n' > debian/libnginx-mod-http-geoip/DEBIAN/postinst
        cat debian/libnginx-mod-http-geoip.postinst.debhelper >> debian/libnginx-mod-http-geoip/DEBIAN/postinst
        chmod 0755 -- debian/libnginx-mod-http-geoip/DEBIAN/postinst
        printf '#!/bin/sh\nset -e\n' > debian/libnginx-mod-http-geoip/DEBIAN/prerm
        cat debian/libnginx-mod-http-geoip.prerm.debhelper >> debian/libnginx-mod-http-geoip/DEBIAN/prerm
        chmod 0755 -- debian/libnginx-mod-http-geoip/DEBIAN/prerm
        printf '#!/bin/sh\nset -e\n' > debian/libnginx-mod-http-geoip/DEBIAN/postrm
        cat debian/libnginx-mod-http-geoip.postrm.debhelper >> debian/libnginx-mod-http-geoip/DEBIAN/postrm
        chmod 0755 -- debian/libnginx-mod-http-geoip/DEBIAN/postrm
        install -m0755 -d debian/libnginx-mod-http-image-filter/DEBIAN
        printf '#!/bin/sh\nset -e\n' > debian/libnginx-mod-http-image-filter/DEBIAN/postinst
        cat debian/libnginx-mod-http-image-filter.postinst.debhelper >> debian/libnginx-mod-http-image-filter/DEBIAN/postinst
        chmod 0755 -- debian/libnginx-mod-http-image-filter/DEBIAN/postinst
        printf '#!/bin/sh\nset -e\n' > debian/libnginx-mod-http-image-filter/DEBIAN/prerm
        cat debian/libnginx-mod-http-image-filter.prerm.debhelper >> debian/libnginx-mod-http-image-filter/DEBIAN/prerm
        chmod 0755 -- debian/libnginx-mod-http-image-filter/DEBIAN/prerm
        printf '#!/bin/sh\nset -e\n' > debian/libnginx-mod-http-image-filter/DEBIAN/postrm
        cat debian/libnginx-mod-http-image-filter.postrm.debhelper >> debian/libnginx-mod-http-image-filter/DEBIAN/postrm
        chmod 0755 -- debian/libnginx-mod-http-image-filter/DEBIAN/postrm
        install -m0755 -d debian/libnginx-mod-http-xslt-filter/DEBIAN
        printf '#!/bin/sh\nset -e\n' > debian/libnginx-mod-http-xslt-filter/DEBIAN/postinst
        cat debian/libnginx-mod-http-xslt-filter.postinst.debhelper >> debian/libnginx-mod-http-xslt-filter/DEBIAN/postinst
        chmod 0755 -- debian/libnginx-mod-http-xslt-filter/DEBIAN/postinst
        printf '#!/bin/sh\nset -e\n' > debian/libnginx-mod-http-xslt-filter/DEBIAN/prerm
        cat debian/libnginx-mod-http-xslt-filter.prerm.debhelper >> debian/libnginx-mod-http-xslt-filter/DEBIAN/prerm
        chmod 0755 -- debian/libnginx-mod-http-xslt-filter/DEBIAN/prerm
        printf '#!/bin/sh\nset -e\n' > debian/libnginx-mod-http-xslt-filter/DEBIAN/postrm
        cat debian/libnginx-mod-http-xslt-filter.postrm.debhelper >> debian/libnginx-mod-http-xslt-filter/DEBIAN/postrm
        chmod 0755 -- debian/libnginx-mod-http-xslt-filter/DEBIAN/postrm
        install -m0755 -d debian/libnginx-mod-mail/DEBIAN
        printf '#!/bin/sh\nset -e\n' > debian/libnginx-mod-mail/DEBIAN/postinst
        cat debian/libnginx-mod-mail.postinst.debhelper >> debian/libnginx-mod-mail/DEBIAN/postinst
        chmod 0755 -- debian/libnginx-mod-mail/DEBIAN/postinst
        printf '#!/bin/sh\nset -e\n' > debian/libnginx-mod-mail/DEBIAN/prerm
        cat debian/libnginx-mod-mail.prerm.debhelper >> debian/libnginx-mod-mail/DEBIAN/prerm
        chmod 0755 -- debian/libnginx-mod-mail/DEBIAN/prerm
        printf '#!/bin/sh\nset -e\n' > debian/libnginx-mod-mail/DEBIAN/postrm
        cat debian/libnginx-mod-mail.postrm.debhelper >> debian/libnginx-mod-mail/DEBIAN/postrm
        chmod 0755 -- debian/libnginx-mod-mail/DEBIAN/postrm
        install -m0755 -d debian/libnginx-mod-stream/DEBIAN
        printf '#!/bin/sh\nset -e\n' > debian/libnginx-mod-stream/DEBIAN/postinst
        cat debian/libnginx-mod-stream.postinst.debhelper >> debian/libnginx-mod-stream/DEBIAN/postinst
        chmod 0755 -- debian/libnginx-mod-stream/DEBIAN/postinst
        printf '#!/bin/sh\nset -e\n' > debian/libnginx-mod-stream/DEBIAN/prerm
        cat debian/libnginx-mod-stream.prerm.debhelper >> debian/libnginx-mod-stream/DEBIAN/prerm
        chmod 0755 -- debian/libnginx-mod-stream/DEBIAN/prerm
        printf '#!/bin/sh\nset -e\n' > debian/libnginx-mod-stream/DEBIAN/postrm
        cat debian/libnginx-mod-stream.postrm.debhelper >> debian/libnginx-mod-stream/DEBIAN/postrm
        chmod 0755 -- debian/libnginx-mod-stream/DEBIAN/postrm
        install -m0755 -d debian/libnginx-mod-stream-geoip/DEBIAN
        printf '#!/bin/sh\nset -e\n' > debian/libnginx-mod-stream-geoip/DEBIAN/postinst
        cat debian/libnginx-mod-stream-geoip.postinst.debhelper >> debian/libnginx-mod-stream-geoip/DEBIAN/postinst
        chmod 0755 -- debian/libnginx-mod-stream-geoip/DEBIAN/postinst
        printf '#!/bin/sh\nset -e\n' > debian/libnginx-mod-stream-geoip/DEBIAN/prerm
        cat debian/libnginx-mod-stream-geoip.prerm.debhelper >> debian/libnginx-mod-stream-geoip/DEBIAN/prerm
        chmod 0755 -- debian/libnginx-mod-stream-geoip/DEBIAN/prerm
        printf '#!/bin/sh\nset -e\n' > debian/libnginx-mod-stream-geoip/DEBIAN/postrm
        cat debian/libnginx-mod-stream-geoip.postrm.debhelper >> debian/libnginx-mod-stream-geoip/DEBIAN/postrm
        chmod 0755 -- debian/libnginx-mod-stream-geoip/DEBIAN/postrm
        install -m0755 -d debian/libnginx-mod-http-perl/DEBIAN
        printf '#!/bin/sh\nset -e\n' > debian/libnginx-mod-http-perl/DEBIAN/postinst
        cat debian/libnginx-mod-http-perl.postinst.debhelper >> debian/libnginx-mod-http-perl/DEBIAN/postinst
        chmod 0755 -- debian/libnginx-mod-http-perl/DEBIAN/postinst
        printf '#!/bin/sh\nset -e\n' > debian/libnginx-mod-http-perl/DEBIAN/prerm
        cat debian/libnginx-mod-http-perl.prerm.debhelper >> debian/libnginx-mod-http-perl/DEBIAN/prerm
        chmod 0755 -- debian/libnginx-mod-http-perl/DEBIAN/prerm
        printf '#!/bin/sh\nset -e\n' > debian/libnginx-mod-http-perl/DEBIAN/postrm
        cat debian/libnginx-mod-http-perl.postrm.debhelper >> debian/libnginx-mod-http-perl/DEBIAN/postrm
        chmod 0755 -- debian/libnginx-mod-http-perl/DEBIAN/postrm
        rm -f debian/nginx-common.debhelper.log
   debian/rules override_dh_gencontrol
make[1]: Entering directory '/root/custom-nginx/nginx-1.24.0'
dh_gencontrol -- -Tdebian/substvars -Tdebian/libnginx-mod.abisubstvars
        install -m0755 -d debian/nginx/DEBIAN
        install -m0755 -d debian/libnginx-mod-http-geoip/DEBIAN
        echo misc:Pre-Depends= >> debian/libnginx-mod-http-geoip.substvars
        install -m0755 -d debian/.debhelper/libnginx-mod-http-geoip/dbgsym-root/DEBIAN
        dpkg-gencontrol -plibnginx-mod-http-geoip -ldebian/changelog -Tdebian/libnginx-mod-http-geoip.substvars -Pdebian/.debhelper/libnginx-mod-http-geoip/dbgsym-root -Tdebian/substvars -Tdebian/libnginx-mod.abisubstvars -UPre-Depends -URecommends -USuggests -UEnhances -UProvides -UEssential -UConflicts -DPriority=optional -UHomepage -UImportant -DAuto-Built-Package=debug-symbols -UProtected -UBuilt-Using -UStatic-Built-Using -DPackage=libnginx-mod-http-geoip-dbgsym "-DDepends=libnginx-mod-http-geoip (= \${binary:Version})" "-DDescription=debug symbols for libnginx-mod-http-geoip" -DBuild-Ids=7b5b4ca2d6b7c6e4310dfac70d4356052a46db4b -DSection=debug -DPackage-Type=ddeb -UMulti-Arch -UReplaces -UBreaks
        echo misc:Depends= >> debian/nginx.substvars
        echo misc:Pre-Depends= >> debian/nginx.substvars
        install -m0755 -d debian/.debhelper/nginx/dbgsym-root/DEBIAN
        dpkg-gencontrol -pnginx -ldebian/changelog -Tdebian/nginx.substvars -Pdebian/.debhelper/nginx/dbgsym-root -Tdebian/substvars -Tdebian/libnginx-mod.abisubstvars -UPre-Depends -URecommends -USuggests -UEnhances -UProvides -UEssential -UConflicts -DPriority=optional -UHomepage -UImportant -DAuto-Built-Package=debug-symbols -UProtected -UBuilt-Using -UStatic-Built-Using -DPackage=nginx-dbgsym "-DDepends=nginx (= \${binary:Version})" "-DDescription=debug symbols for nginx" -DBuild-Ids=9b936a69c2e956bf0014fdcbd6dd9d89468aac27 -DSection=debug -DPackage-Type=ddeb -UMulti-Arch -UReplaces -UBreaks
dpkg-gencontrol: warning: package nginx: substitution variable ${libnginx-mod-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx: substitution variable ${libnginx-mod-http-auth-pam:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx: substitution variable ${libnginx-mod-http-cache-purge:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx: substitution variable ${libnginx-mod-http-dav-ext:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx: substitution variable ${libnginx-mod-http-echo:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-geoip: substitution variable ${libnginx-mod-geoip2:Version} unused, but
is defined
dpkg-gencontrol: warning: package libnginx-mod-http-geoip: substitution variable ${libnginx-mod-http-auth-pam:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-geoip: substitution variable ${libnginx-mod-http-cache-purge:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-geoip: substitution variable ${libnginx-mod-http-dav-ext:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-geoip: substitution variable ${libnginx-mod-http-echo:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-geoip: substitution variable ${libnginx-mod-http-fancyindex:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx: substitution variable ${libnginx-mod-http-fancyindex:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-geoip: substitution variable ${libnginx-mod-http-geoip2:Version} unused,
but is defined
dpkg-gencontrol: warning: package nginx: substitution variable ${libnginx-mod-http-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-geoip: substitution variable ${libnginx-mod-http-headers-more-filter:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx: substitution variable ${libnginx-mod-http-headers-more-filter:Version} unused, but
is defined
dpkg-gencontrol: warning: package libnginx-mod-http-geoip: substitution variable ${libnginx-mod-http-lua:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-geoip: substitution variable ${libnginx-mod-http-subs-filter:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-geoip: substitution variable ${libnginx-mod-http-uploadprogress:Version}
unused, but is defined
dpkg-gencontrol: warning: package nginx: substitution variable ${libnginx-mod-http-lua:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-geoip: substitution variable ${libnginx-mod-http-upstream-fair:Version}
unused, but is defined
dpkg-gencontrol: warning: package nginx: substitution variable ${libnginx-mod-http-subs-filter:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-geoip: substitution variable ${libnginx-mod-nchan:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx: substitution variable ${libnginx-mod-http-uploadprogress:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-geoip: substitution variable ${nginx:abi} unused, but is defined
dpkg-gencontrol: warning: package nginx: substitution variable ${libnginx-mod-http-upstream-fair:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx: substitution variable ${libnginx-mod-nchan:Version} unused, but is defined
        chmod 0644 -- debian/.debhelper/libnginx-mod-http-geoip/dbgsym-root/DEBIAN/control
        dpkg-gencontrol -plibnginx-mod-http-geoip -ldebian/changelog -Tdebian/libnginx-mod-http-geoip.substvars -Pdebian/libnginx-mod-http-geoip -Tdebian/substvars -Tdebian/libnginx-mod.abisubstvars
        chmod 0644 -- debian/.debhelper/nginx/dbgsym-root/DEBIAN/control
        dpkg-gencontrol -pnginx -ldebian/changelog -Tdebian/nginx.substvars -Pdebian/nginx -Tdebian/substvars -Tdebian/libnginx-mod.abisubstvars
dpkg-gencontrol: warning: package nginx: substitution variable ${libnginx-mod-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx: substitution variable ${libnginx-mod-http-auth-pam:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx: substitution variable ${libnginx-mod-http-cache-purge:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-geoip: substitution variable ${libnginx-mod-geoip2:Version} unused, but
is defined
dpkg-gencontrol: warning: package libnginx-mod-http-geoip: substitution variable ${libnginx-mod-http-auth-pam:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-geoip: substitution variable ${libnginx-mod-http-cache-purge:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-geoip: substitution variable ${libnginx-mod-http-dav-ext:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-geoip: substitution variable ${libnginx-mod-http-echo:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-geoip: substitution variable ${libnginx-mod-http-fancyindex:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-geoip: substitution variable ${libnginx-mod-http-geoip2:Version} unused,
but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-geoip: substitution variable ${libnginx-mod-http-headers-more-filter:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-geoip: substitution variable ${libnginx-mod-http-lua:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-geoip: substitution variable ${libnginx-mod-http-subs-filter:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-geoip: substitution variable ${libnginx-mod-http-uploadprogress:Version}
unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-geoip: substitution variable ${libnginx-mod-http-upstream-fair:Version}
unused, but is defined
dpkg-gencontrol: warning: package nginx: substitution variable ${libnginx-mod-http-dav-ext:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-geoip: substitution variable ${libnginx-mod-nchan:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-geoip: substitution variable ${nginx:abi} unused, but is defined
dpkg-gencontrol: warning: package nginx: substitution variable ${libnginx-mod-http-echo:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx: substitution variable ${libnginx-mod-http-fancyindex:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx: substitution variable ${libnginx-mod-http-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx: substitution variable ${libnginx-mod-http-headers-more-filter:Version} unused, but
is defined
dpkg-gencontrol: warning: package nginx: substitution variable ${libnginx-mod-http-lua:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx: substitution variable ${libnginx-mod-http-subs-filter:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx: substitution variable ${libnginx-mod-http-uploadprogress:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx: substitution variable ${libnginx-mod-http-upstream-fair:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx: substitution variable ${libnginx-mod-nchan:Version} unused, but is defined
        chmod 0644 -- debian/libnginx-mod-http-geoip/DEBIAN/control
        install -m0755 -d debian/libnginx-mod-http-image-filter/DEBIAN
        echo misc:Pre-Depends= >> debian/libnginx-mod-http-image-filter.substvars
        install -m0755 -d debian/.debhelper/libnginx-mod-http-image-filter/dbgsym-root/DEBIAN
        dpkg-gencontrol -plibnginx-mod-http-image-filter -ldebian/changelog -Tdebian/libnginx-mod-http-image-filter.substvars -Pdebian/.debhelper/libnginx-mod-http-image-filter/dbgsym-root -Tdebian/substvars -Tdebian/libnginx-mod.abisubstvars -UPre-Depends -URecommends -USuggests -UEnhances -UProvides -UEssential -UConflicts -DPriority=optional -UHomepage -UImportant -DAuto-Built-Package=debug-symbols -UProtected -UBuilt-Using -UStatic-Built-Using -DPackage=libnginx-mod-http-image-filter-dbgsym
"-DDepends=libnginx-mod-http-image-filter (= \${binary:Version})" "-DDescription=debug symbols for libnginx-mod-http-image-filter" -DBuild-Ids=8cd906362ba63ff4670d057c2ff40758334f2eff -DSection=debug -DPackage-Type=ddeb -UMulti-Arch -UReplaces
-UBreaks
        chmod 0644 -- debian/nginx/DEBIAN/control
        install -m0755 -d debian/nginx-doc/DEBIAN
        echo misc:Depends= >> debian/nginx-doc.substvars
        echo misc:Pre-Depends= >> debian/nginx-doc.substvars
        dpkg-gencontrol -pnginx-doc -ldebian/changelog -Tdebian/nginx-doc.substvars -Pdebian/nginx-doc -Tdebian/substvars -Tdebian/libnginx-mod.abisubstvars
dpkg-gencontrol: warning: package libnginx-mod-http-image-filter: substitution variable ${libnginx-mod-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-image-filter: substitution variable ${libnginx-mod-http-auth-pam:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-image-filter: substitution variable ${libnginx-mod-http-cache-purge:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-image-filter: substitution variable ${libnginx-mod-http-dav-ext:Version}
unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-image-filter: substitution variable ${libnginx-mod-http-echo:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-image-filter: substitution variable ${libnginx-mod-http-fancyindex:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-image-filter: substitution variable ${libnginx-mod-http-geoip2:Version}
unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-image-filter: substitution variable ${libnginx-mod-http-headers-more-filter:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-image-filter: substitution variable ${libnginx-mod-http-lua:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-image-filter: substitution variable ${libnginx-mod-http-subs-filter:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-image-filter: substitution variable ${libnginx-mod-http-uploadprogress:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-image-filter: substitution variable ${libnginx-mod-http-upstream-fair:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-image-filter: substitution variable ${libnginx-mod-nchan:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-image-filter: substitution variable ${nginx:abi} unused, but is defined
        chmod 0644 -- debian/.debhelper/libnginx-mod-http-image-filter/dbgsym-root/DEBIAN/control
        dpkg-gencontrol -plibnginx-mod-http-image-filter -ldebian/changelog -Tdebian/libnginx-mod-http-image-filter.substvars -Pdebian/libnginx-mod-http-image-filter -Tdebian/substvars -Tdebian/libnginx-mod.abisubstvars
dpkg-gencontrol: warning: package nginx-doc: substitution variable ${libnginx-mod-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-doc: substitution variable ${libnginx-mod-http-auth-pam:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-doc: substitution variable ${libnginx-mod-http-cache-purge:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-doc: substitution variable ${libnginx-mod-http-dav-ext:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-doc: substitution variable ${libnginx-mod-http-echo:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-doc: substitution variable ${libnginx-mod-http-fancyindex:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-doc: substitution variable ${libnginx-mod-http-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-doc: substitution variable ${libnginx-mod-http-headers-more-filter:Version} unused,
but is defined
dpkg-gencontrol: warning: package nginx-doc: substitution variable ${libnginx-mod-http-lua:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-doc: substitution variable ${libnginx-mod-http-subs-filter:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-doc: substitution variable ${libnginx-mod-http-uploadprogress:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-doc: substitution variable ${libnginx-mod-http-upstream-fair:Version} unused, but is
defined
dpkg-gencontrol: warning: package nginx-doc: substitution variable ${libnginx-mod-nchan:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-doc: substitution variable ${nginx:abi} unused, but is defined
        chmod 0644 -- debian/nginx-doc/DEBIAN/control
        install -m0755 -d debian/nginx-common/DEBIAN
        echo misc:Pre-Depends= >> debian/nginx-common.substvars
        dpkg-gencontrol -pnginx-common -ldebian/changelog -Tdebian/nginx-common.substvars -Pdebian/nginx-common -Tdebian/substvars -Tdebian/libnginx-mod.abisubstvars
dpkg-gencontrol: warning: package libnginx-mod-http-image-filter: substitution variable ${libnginx-mod-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-image-filter: substitution variable ${libnginx-mod-http-auth-pam:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-image-filter: substitution variable ${libnginx-mod-http-cache-purge:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-image-filter: substitution variable ${libnginx-mod-http-dav-ext:Version}
unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-image-filter: substitution variable ${libnginx-mod-http-echo:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-image-filter: substitution variable ${libnginx-mod-http-fancyindex:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-image-filter: substitution variable ${libnginx-mod-http-geoip2:Version}
unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-image-filter: substitution variable ${libnginx-mod-http-headers-more-filter:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-image-filter: substitution variable ${libnginx-mod-http-lua:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-image-filter: substitution variable ${libnginx-mod-http-subs-filter:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-image-filter: substitution variable ${libnginx-mod-http-uploadprogress:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-image-filter: substitution variable ${libnginx-mod-http-upstream-fair:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-image-filter: substitution variable ${libnginx-mod-nchan:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-image-filter: substitution variable ${nginx:abi} unused, but is defined
        chmod 0644 -- debian/libnginx-mod-http-image-filter/DEBIAN/control
        install -m0755 -d debian/libnginx-mod-http-xslt-filter/DEBIAN
        echo misc:Pre-Depends= >> debian/libnginx-mod-http-xslt-filter.substvars
        install -m0755 -d debian/.debhelper/libnginx-mod-http-xslt-filter/dbgsym-root/DEBIAN
        dpkg-gencontrol -plibnginx-mod-http-xslt-filter -ldebian/changelog -Tdebian/libnginx-mod-http-xslt-filter.substvars
-Pdebian/.debhelper/libnginx-mod-http-xslt-filter/dbgsym-root -Tdebian/substvars -Tdebian/libnginx-mod.abisubstvars -UPre-Depends -URecommends -USuggests -UEnhances -UProvides -UEssential -UConflicts -DPriority=optional -UHomepage -UImportant -DAuto-Built-Package=debug-symbols -UProtected -UBuilt-Using -UStatic-Built-Using -DPackage=libnginx-mod-http-xslt-filter-dbgsym
"-DDepends=libnginx-mod-http-xslt-filter (= \${binary:Version})" "-DDescription=debug symbols for libnginx-mod-http-xslt-filter" -DBuild-Ids=d4d72ce7d560ce5865350938fcf83a30d2867ee5 -DSection=debug -DPackage-Type=ddeb -UMulti-Arch -UReplaces -UBreaks
dpkg-gencontrol: warning: package nginx-common: substitution variable ${libnginx-mod-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-common: substitution variable ${libnginx-mod-http-auth-pam:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-common: substitution variable ${libnginx-mod-http-cache-purge:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-common: substitution variable ${libnginx-mod-http-dav-ext:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-common: substitution variable ${libnginx-mod-http-echo:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-common: substitution variable ${libnginx-mod-http-fancyindex:Version} unused, but is
defined
dpkg-gencontrol: warning: package nginx-common: substitution variable ${libnginx-mod-http-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-common: substitution variable ${libnginx-mod-http-headers-more-filter:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-common: substitution variable ${libnginx-mod-http-lua:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-common: substitution variable ${libnginx-mod-http-subs-filter:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-common: substitution variable ${libnginx-mod-http-uploadprogress:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-common: substitution variable ${libnginx-mod-http-upstream-fair:Version} unused, but
is defined
dpkg-gencontrol: warning: package nginx-common: substitution variable ${libnginx-mod-nchan:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-common: substitution variable ${nginx:abi} unused, but is defined
        chmod 0644 -- debian/nginx-common/DEBIAN/control
        install -m0755 -d debian/nginx-dev/DEBIAN
        echo misc:Depends= >> debian/nginx-dev.substvars
        echo misc:Pre-Depends= >> debian/nginx-dev.substvars
        dpkg-gencontrol -pnginx-dev -ldebian/changelog -Tdebian/nginx-dev.substvars -Pdebian/nginx-dev -Tdebian/substvars -Tdebian/libnginx-mod.abisubstvars
dpkg-gencontrol: warning: package libnginx-mod-http-xslt-filter: substitution variable ${libnginx-mod-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-xslt-filter: substitution variable ${libnginx-mod-http-auth-pam:Version}
unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-xslt-filter: substitution variable ${libnginx-mod-http-cache-purge:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-xslt-filter: substitution variable ${libnginx-mod-http-dav-ext:Version}
unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-xslt-filter: substitution variable ${libnginx-mod-http-echo:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-xslt-filter: substitution variable ${libnginx-mod-http-fancyindex:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-xslt-filter: substitution variable ${libnginx-mod-http-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-xslt-filter: substitution variable ${libnginx-mod-http-headers-more-filter:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-xslt-filter: substitution variable ${libnginx-mod-http-lua:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-xslt-filter: substitution variable ${libnginx-mod-http-subs-filter:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-xslt-filter: substitution variable ${libnginx-mod-http-uploadprogress:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-xslt-filter: substitution variable ${libnginx-mod-http-upstream-fair:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-xslt-filter: substitution variable ${libnginx-mod-nchan:Version} unused,
but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-xslt-filter: substitution variable ${nginx:abi} unused, but is defined
        chmod 0644 -- debian/.debhelper/libnginx-mod-http-xslt-filter/dbgsym-root/DEBIAN/control
        dpkg-gencontrol -plibnginx-mod-http-xslt-filter -ldebian/changelog -Tdebian/libnginx-mod-http-xslt-filter.substvars
-Pdebian/libnginx-mod-http-xslt-filter -Tdebian/substvars -Tdebian/libnginx-mod.abisubstvars
dpkg-gencontrol: warning: package nginx-dev: substitution variable ${libnginx-mod-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-dev: substitution variable ${libnginx-mod-http-auth-pam:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-dev: substitution variable ${libnginx-mod-http-cache-purge:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-dev: substitution variable ${libnginx-mod-http-dav-ext:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-dev: substitution variable ${libnginx-mod-http-echo:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-dev: substitution variable ${libnginx-mod-http-fancyindex:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-dev: substitution variable ${libnginx-mod-http-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-dev: substitution variable ${libnginx-mod-http-headers-more-filter:Version} unused,
but is defined
dpkg-gencontrol: warning: package nginx-dev: substitution variable ${libnginx-mod-http-lua:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-dev: substitution variable ${libnginx-mod-http-subs-filter:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-dev: substitution variable ${libnginx-mod-http-uploadprogress:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-dev: substitution variable ${libnginx-mod-http-upstream-fair:Version} unused, but is
defined
dpkg-gencontrol: warning: package nginx-dev: substitution variable ${libnginx-mod-nchan:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-dev: substitution variable ${perl:Depends} unused, but is defined
        chmod 0644 -- debian/nginx-dev/DEBIAN/control
        install -m0755 -d debian/nginx-core/DEBIAN
        echo misc:Depends= >> debian/nginx-core.substvars
        echo misc:Pre-Depends= >> debian/nginx-core.substvars
        dpkg-gencontrol -pnginx-core -ldebian/changelog -Tdebian/nginx-core.substvars -Pdebian/nginx-core -Tdebian/substvars
-Tdebian/libnginx-mod.abisubstvars
dpkg-gencontrol: warning: Depends field of package nginx-core: substitution variable ${shlibs:Depends} used, but is not defined
dpkg-gencontrol: warning: package libnginx-mod-http-xslt-filter: substitution variable ${libnginx-mod-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-xslt-filter: substitution variable ${libnginx-mod-http-auth-pam:Version}
unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-xslt-filter: substitution variable ${libnginx-mod-http-cache-purge:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-xslt-filter: substitution variable ${libnginx-mod-http-dav-ext:Version}
unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-xslt-filter: substitution variable ${libnginx-mod-http-echo:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-xslt-filter: substitution variable ${libnginx-mod-http-fancyindex:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-xslt-filter: substitution variable ${libnginx-mod-http-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-xslt-filter: substitution variable ${libnginx-mod-http-headers-more-filter:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-xslt-filter: substitution variable ${libnginx-mod-http-lua:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-xslt-filter: substitution variable ${libnginx-mod-http-subs-filter:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-xslt-filter: substitution variable ${libnginx-mod-http-uploadprogress:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-xslt-filter: substitution variable ${libnginx-mod-http-upstream-fair:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-xslt-filter: substitution variable ${libnginx-mod-nchan:Version} unused,
but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-xslt-filter: substitution variable ${nginx:abi} unused, but is defined
        chmod 0644 -- debian/libnginx-mod-http-xslt-filter/DEBIAN/control
        install -m0755 -d debian/libnginx-mod-mail/DEBIAN
        echo misc:Pre-Depends= >> debian/libnginx-mod-mail.substvars
        install -m0755 -d debian/.debhelper/libnginx-mod-mail/dbgsym-root/DEBIAN
        dpkg-gencontrol -plibnginx-mod-mail -ldebian/changelog -Tdebian/libnginx-mod-mail.substvars -Pdebian/.debhelper/libnginx-mod-mail/dbgsym-root -Tdebian/substvars -Tdebian/libnginx-mod.abisubstvars -UPre-Depends -URecommends -USuggests -UEnhances -UProvides -UEssential -UConflicts -DPriority=optional -UHomepage -UImportant -DAuto-Built-Package=debug-symbols -UProtected -UBuilt-Using -UStatic-Built-Using -DPackage=libnginx-mod-mail-dbgsym "-DDepends=libnginx-mod-mail (= \${binary:Version})" "-DDescription=debug symbols for libnginx-mod-mail" -DBuild-Ids=56dabdce2de573fe92fc51be00a55be7d1c2c6d7 -DSection=debug -DPackage-Type=ddeb -UMulti-Arch -UReplaces -UBreaks
dpkg-gencontrol: warning: package nginx-core: substitution variable ${libnginx-mod-http-auth-pam:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-core: substitution variable ${libnginx-mod-http-cache-purge:Version} unused, but is
defined
dpkg-gencontrol: warning: package nginx-core: substitution variable ${libnginx-mod-http-dav-ext:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-core: substitution variable ${libnginx-mod-http-echo:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-core: substitution variable ${libnginx-mod-http-fancyindex:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-core: substitution variable ${libnginx-mod-http-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-core: substitution variable ${libnginx-mod-http-headers-more-filter:Version} unused,
but is defined
dpkg-gencontrol: warning: package nginx-core: substitution variable ${libnginx-mod-http-lua:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-core: substitution variable ${libnginx-mod-http-subs-filter:Version} unused, but is
defined
dpkg-gencontrol: warning: package nginx-core: substitution variable ${libnginx-mod-http-uploadprogress:Version} unused, but
is defined
dpkg-gencontrol: warning: package nginx-core: substitution variable ${libnginx-mod-http-upstream-fair:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-core: substitution variable ${libnginx-mod-nchan:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-core: substitution variable ${nginx:abi} unused, but is defined
        chmod 0644 -- debian/nginx-core/DEBIAN/control
        install -m0755 -d debian/nginx-full/DEBIAN
        echo misc:Depends= >> debian/nginx-full.substvars
        echo misc:Pre-Depends= >> debian/nginx-full.substvars
        dpkg-gencontrol -pnginx-full -ldebian/changelog -Tdebian/nginx-full.substvars -Pdebian/nginx-full -Tdebian/substvars
-Tdebian/libnginx-mod.abisubstvars
dpkg-gencontrol: warning: Depends field of package nginx-full: substitution variable ${shlibs:Depends} used, but is not defined
dpkg-gencontrol: warning: package nginx-full: substitution variable ${libnginx-mod-http-cache-purge:Version} unused, but is
defined
dpkg-gencontrol: warning: package nginx-full: substitution variable ${libnginx-mod-http-fancyindex:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-full: substitution variable ${libnginx-mod-http-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-full: substitution variable ${libnginx-mod-http-headers-more-filter:Version} unused,
but is defined
dpkg-gencontrol: warning: package nginx-full: substitution variable ${libnginx-mod-http-lua:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-full: substitution variable ${libnginx-mod-http-uploadprogress:Version} unused, but
is defined
dpkg-gencontrol: warning: package nginx-full: substitution variable ${libnginx-mod-nchan:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-full: substitution variable ${nginx:abi} unused, but is defined
        chmod 0644 -- debian/nginx-full/DEBIAN/control
        install -m0755 -d debian/nginx-light/DEBIAN
        echo misc:Depends= >> debian/nginx-light.substvars
        echo misc:Pre-Depends= >> debian/nginx-light.substvars
        dpkg-gencontrol -pnginx-light -ldebian/changelog -Tdebian/nginx-light.substvars -Pdebian/nginx-light -Tdebian/substvars -Tdebian/libnginx-mod.abisubstvars
dpkg-gencontrol: warning: package libnginx-mod-mail: substitution variable ${libnginx-mod-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-mail: substitution variable ${libnginx-mod-http-auth-pam:Version} unused, but
is defined
dpkg-gencontrol: warning: package libnginx-mod-mail: substitution variable ${libnginx-mod-http-cache-purge:Version} unused,
but is defined
dpkg-gencontrol: warning: package libnginx-mod-mail: substitution variable ${libnginx-mod-http-dav-ext:Version} unused, but
is defined
dpkg-gencontrol: warning: package libnginx-mod-mail: substitution variable ${libnginx-mod-http-echo:Version} unused, but is
defined
dpkg-gencontrol: warning: package libnginx-mod-mail: substitution variable ${libnginx-mod-http-fancyindex:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-mail: substitution variable ${libnginx-mod-http-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-mail: substitution variable ${libnginx-mod-http-headers-more-filter:Version}
unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-mail: substitution variable ${libnginx-mod-http-lua:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-mail: substitution variable ${libnginx-mod-http-subs-filter:Version} unused,
but is defined
dpkg-gencontrol: warning: package libnginx-mod-mail: substitution variable ${libnginx-mod-http-uploadprogress:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-mail: substitution variable ${libnginx-mod-http-upstream-fair:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-mail: substitution variable ${libnginx-mod-nchan:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-mail: substitution variable ${nginx:abi} unused, but is defined
        chmod 0644 -- debian/.debhelper/libnginx-mod-mail/dbgsym-root/DEBIAN/control
        dpkg-gencontrol -plibnginx-mod-mail -ldebian/changelog -Tdebian/libnginx-mod-mail.substvars -Pdebian/libnginx-mod-mail -Tdebian/substvars -Tdebian/libnginx-mod.abisubstvars
dpkg-gencontrol: warning: Depends field of package nginx-light: substitution variable ${shlibs:Depends} used, but is not defined
dpkg-gencontrol: warning: package nginx-light: substitution variable ${libnginx-mod-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-light: substitution variable ${libnginx-mod-http-auth-pam:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-light: substitution variable ${libnginx-mod-http-cache-purge:Version} unused, but is
defined
dpkg-gencontrol: warning: package nginx-light: substitution variable ${libnginx-mod-http-dav-ext:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-light: substitution variable ${libnginx-mod-http-fancyindex:Version} unused, but is
defined
dpkg-gencontrol: warning: package nginx-light: substitution variable ${libnginx-mod-http-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-light: substitution variable ${libnginx-mod-http-headers-more-filter:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-light: substitution variable ${libnginx-mod-http-lua:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-light: substitution variable ${libnginx-mod-http-subs-filter:Version} unused, but is
defined
dpkg-gencontrol: warning: package nginx-light: substitution variable ${libnginx-mod-http-uploadprogress:Version} unused, but
is defined
dpkg-gencontrol: warning: package nginx-light: substitution variable ${libnginx-mod-http-upstream-fair:Version} unused, but
is defined
dpkg-gencontrol: warning: package nginx-light: substitution variable ${libnginx-mod-nchan:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-light: substitution variable ${nginx:abi} unused, but is defined
        chmod 0644 -- debian/nginx-light/DEBIAN/control
        install -m0755 -d debian/nginx-extras/DEBIAN
        echo misc:Depends= >> debian/nginx-extras.substvars
        echo misc:Pre-Depends= >> debian/nginx-extras.substvars
        dpkg-gencontrol -pnginx-extras -ldebian/changelog -Tdebian/nginx-extras.substvars -Pdebian/nginx-extras -Tdebian/substvars -Tdebian/libnginx-mod.abisubstvars
dpkg-gencontrol: warning: package libnginx-mod-mail: substitution variable ${libnginx-mod-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-mail: substitution variable ${libnginx-mod-http-auth-pam:Version} unused, but
is defined
dpkg-gencontrol: warning: package libnginx-mod-mail: substitution variable ${libnginx-mod-http-cache-purge:Version} unused,
but is defined
dpkg-gencontrol: warning: package libnginx-mod-mail: substitution variable ${libnginx-mod-http-dav-ext:Version} unused, but
is defined
dpkg-gencontrol: warning: package libnginx-mod-mail: substitution variable ${libnginx-mod-http-echo:Version} unused, but is
defined
dpkg-gencontrol: warning: package libnginx-mod-mail: substitution variable ${libnginx-mod-http-fancyindex:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-mail: substitution variable ${libnginx-mod-http-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-mail: substitution variable ${libnginx-mod-http-headers-more-filter:Version}
unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-mail: substitution variable ${libnginx-mod-http-lua:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-mail: substitution variable ${libnginx-mod-http-subs-filter:Version} unused,
but is defined
dpkg-gencontrol: warning: package libnginx-mod-mail: substitution variable ${libnginx-mod-http-uploadprogress:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-mail: substitution variable ${libnginx-mod-http-upstream-fair:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-mail: substitution variable ${libnginx-mod-nchan:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-mail: substitution variable ${nginx:abi} unused, but is defined
        chmod 0644 -- debian/libnginx-mod-mail/DEBIAN/control
        install -m0755 -d debian/libnginx-mod-stream/DEBIAN
        echo misc:Pre-Depends= >> debian/libnginx-mod-stream.substvars
        install -m0755 -d debian/.debhelper/libnginx-mod-stream/dbgsym-root/DEBIAN
        dpkg-gencontrol -plibnginx-mod-stream -ldebian/changelog -Tdebian/libnginx-mod-stream.substvars -Pdebian/.debhelper/libnginx-mod-stream/dbgsym-root -Tdebian/substvars -Tdebian/libnginx-mod.abisubstvars -UPre-Depends -URecommends -USuggests
-UEnhances -UProvides -UEssential -UConflicts -DPriority=optional -UHomepage -UImportant -DAuto-Built-Package=debug-symbols
-UProtected -UBuilt-Using -UStatic-Built-Using -DPackage=libnginx-mod-stream-dbgsym "-DDepends=libnginx-mod-stream (= \${binary:Version})" "-DDescription=debug symbols for libnginx-mod-stream" -DBuild-Ids=ebd42074efb069d4f4d7fa5c01531c7c57339898 -DSection=debug -DPackage-Type=ddeb -UMulti-Arch -UReplaces -UBreaks
dpkg-gencontrol: warning: Depends field of package nginx-extras: substitution variable ${shlibs:Depends} used, but is not defined
dpkg-gencontrol: warning: package nginx-extras: substitution variable ${libnginx-mod-http-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-extras: substitution variable ${libnginx-mod-http-lua:Version} unused, but is defined
dpkg-gencontrol: warning: package nginx-extras: substitution variable ${nginx:abi} unused, but is defined
        chmod 0644 -- debian/nginx-extras/DEBIAN/control
dpkg-gencontrol: warning: package libnginx-mod-stream: substitution variable ${libnginx-mod-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream: substitution variable ${libnginx-mod-http-auth-pam:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream: substitution variable ${libnginx-mod-http-cache-purge:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream: substitution variable ${libnginx-mod-http-dav-ext:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream: substitution variable ${libnginx-mod-http-echo:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream: substitution variable ${libnginx-mod-http-fancyindex:Version} unused,
but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream: substitution variable ${libnginx-mod-http-geoip2:Version} unused, but
is defined
dpkg-gencontrol: warning: package libnginx-mod-stream: substitution variable ${libnginx-mod-http-headers-more-filter:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream: substitution variable ${libnginx-mod-http-lua:Version} unused, but is
defined
dpkg-gencontrol: warning: package libnginx-mod-stream: substitution variable ${libnginx-mod-http-subs-filter:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream: substitution variable ${libnginx-mod-http-uploadprogress:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream: substitution variable ${libnginx-mod-http-upstream-fair:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream: substitution variable ${libnginx-mod-nchan:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream: substitution variable ${nginx:abi} unused, but is defined
        chmod 0644 -- debian/.debhelper/libnginx-mod-stream/dbgsym-root/DEBIAN/control
        dpkg-gencontrol -plibnginx-mod-stream -ldebian/changelog -Tdebian/libnginx-mod-stream.substvars -Pdebian/libnginx-mod-stream -Tdebian/substvars -Tdebian/libnginx-mod.abisubstvars
dpkg-gencontrol: warning: package libnginx-mod-stream: substitution variable ${libnginx-mod-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream: substitution variable ${libnginx-mod-http-auth-pam:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream: substitution variable ${libnginx-mod-http-cache-purge:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream: substitution variable ${libnginx-mod-http-dav-ext:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream: substitution variable ${libnginx-mod-http-echo:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream: substitution variable ${libnginx-mod-http-fancyindex:Version} unused,
but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream: substitution variable ${libnginx-mod-http-geoip2:Version} unused, but
is defined
dpkg-gencontrol: warning: package libnginx-mod-stream: substitution variable ${libnginx-mod-http-headers-more-filter:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream: substitution variable ${libnginx-mod-http-lua:Version} unused, but is
defined
dpkg-gencontrol: warning: package libnginx-mod-stream: substitution variable ${libnginx-mod-http-subs-filter:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream: substitution variable ${libnginx-mod-http-uploadprogress:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream: substitution variable ${libnginx-mod-http-upstream-fair:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream: substitution variable ${libnginx-mod-nchan:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream: substitution variable ${nginx:abi} unused, but is defined
        chmod 0644 -- debian/libnginx-mod-stream/DEBIAN/control
        install -m0755 -d debian/libnginx-mod-stream-geoip/DEBIAN
        echo misc:Pre-Depends= >> debian/libnginx-mod-stream-geoip.substvars
        install -m0755 -d debian/.debhelper/libnginx-mod-stream-geoip/dbgsym-root/DEBIAN
        dpkg-gencontrol -plibnginx-mod-stream-geoip -ldebian/changelog -Tdebian/libnginx-mod-stream-geoip.substvars -Pdebian/.debhelper/libnginx-mod-stream-geoip/dbgsym-root -Tdebian/substvars -Tdebian/libnginx-mod.abisubstvars -UPre-Depends -URecommends -USuggests -UEnhances -UProvides -UEssential -UConflicts -DPriority=optional -UHomepage -UImportant -DAuto-Built-Package=debug-symbols -UProtected -UBuilt-Using -UStatic-Built-Using -DPackage=libnginx-mod-stream-geoip-dbgsym "-DDepends=libnginx-mod-stream-geoip (= \${binary:Version})" "-DDescription=debug symbols for libnginx-mod-stream-geoip" -DBuild-Ids=28762fc6e00a08f00fc90b7b67dc8e5d140f8181 -DSection=debug -DPackage-Type=ddeb -UMulti-Arch -UReplaces -UBreaks
dpkg-gencontrol: warning: package libnginx-mod-stream-geoip: substitution variable ${libnginx-mod-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream-geoip: substitution variable ${libnginx-mod-http-auth-pam:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream-geoip: substitution variable ${libnginx-mod-http-cache-purge:Version}
unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream-geoip: substitution variable ${libnginx-mod-http-dav-ext:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream-geoip: substitution variable ${libnginx-mod-http-echo:Version} unused,
but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream-geoip: substitution variable ${libnginx-mod-http-fancyindex:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream-geoip: substitution variable ${libnginx-mod-http-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream-geoip: substitution variable ${libnginx-mod-http-headers-more-filter:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream-geoip: substitution variable ${libnginx-mod-http-lua:Version} unused,
but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream-geoip: substitution variable ${libnginx-mod-http-subs-filter:Version}
unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream-geoip: substitution variable ${libnginx-mod-http-uploadprogress:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream-geoip: substitution variable ${libnginx-mod-http-upstream-fair:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream-geoip: substitution variable ${libnginx-mod-nchan:Version} unused, but
is defined
dpkg-gencontrol: warning: package libnginx-mod-stream-geoip: substitution variable ${nginx:abi} unused, but is defined
        chmod 0644 -- debian/.debhelper/libnginx-mod-stream-geoip/dbgsym-root/DEBIAN/control
        dpkg-gencontrol -plibnginx-mod-stream-geoip -ldebian/changelog -Tdebian/libnginx-mod-stream-geoip.substvars -Pdebian/libnginx-mod-stream-geoip -Tdebian/substvars -Tdebian/libnginx-mod.abisubstvars
dpkg-gencontrol: warning: package libnginx-mod-stream-geoip: substitution variable ${libnginx-mod-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream-geoip: substitution variable ${libnginx-mod-http-auth-pam:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream-geoip: substitution variable ${libnginx-mod-http-cache-purge:Version}
unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream-geoip: substitution variable ${libnginx-mod-http-dav-ext:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream-geoip: substitution variable ${libnginx-mod-http-echo:Version} unused,
but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream-geoip: substitution variable ${libnginx-mod-http-fancyindex:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream-geoip: substitution variable ${libnginx-mod-http-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream-geoip: substitution variable ${libnginx-mod-http-headers-more-filter:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream-geoip: substitution variable ${libnginx-mod-http-lua:Version} unused,
but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream-geoip: substitution variable ${libnginx-mod-http-subs-filter:Version}
unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream-geoip: substitution variable ${libnginx-mod-http-uploadprogress:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream-geoip: substitution variable ${libnginx-mod-http-upstream-fair:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-stream-geoip: substitution variable ${libnginx-mod-nchan:Version} unused, but
is defined
dpkg-gencontrol: warning: package libnginx-mod-stream-geoip: substitution variable ${nginx:abi} unused, but is defined
        chmod 0644 -- debian/libnginx-mod-stream-geoip/DEBIAN/control
        install -m0755 -d debian/libnginx-mod-http-perl/DEBIAN
        echo misc:Pre-Depends= >> debian/libnginx-mod-http-perl.substvars
        install -m0755 -d debian/.debhelper/libnginx-mod-http-perl/dbgsym-root/DEBIAN
        dpkg-gencontrol -plibnginx-mod-http-perl -ldebian/changelog -Tdebian/libnginx-mod-http-perl.substvars -Pdebian/.debhelper/libnginx-mod-http-perl/dbgsym-root -Tdebian/substvars -Tdebian/libnginx-mod.abisubstvars -UPre-Depends -URecommends -USuggests -UEnhances -UProvides -UEssential -UConflicts -DPriority=optional -UHomepage -UImportant -DAuto-Built-Package=debug-symbols -UProtected -UBuilt-Using -UStatic-Built-Using -DPackage=libnginx-mod-http-perl-dbgsym "-DDepends=libnginx-mod-http-perl (= \${binary:Version})" "-DDescription=debug symbols for libnginx-mod-http-perl" "-DBuild-Ids=19596dea4b0b382c8e0e77434aa0b7d69d117d17 74a90fa4ef92e050798e5dd0e2c0c8b095918652" -DSection=debug -DPackage-Type=ddeb -UMulti-Arch -UReplaces -UBreaks
dpkg-gencontrol: warning: package libnginx-mod-http-perl: substitution variable ${libnginx-mod-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-perl: substitution variable ${libnginx-mod-http-auth-pam:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-perl: substitution variable ${libnginx-mod-http-cache-purge:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-perl: substitution variable ${libnginx-mod-http-dav-ext:Version} unused,
but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-perl: substitution variable ${libnginx-mod-http-echo:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-perl: substitution variable ${libnginx-mod-http-fancyindex:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-perl: substitution variable ${libnginx-mod-http-geoip2:Version} unused,
but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-perl: substitution variable ${libnginx-mod-http-headers-more-filter:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-perl: substitution variable ${libnginx-mod-http-lua:Version} unused, but
is defined
dpkg-gencontrol: warning: package libnginx-mod-http-perl: substitution variable ${libnginx-mod-http-subs-filter:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-perl: substitution variable ${libnginx-mod-http-uploadprogress:Version}
unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-perl: substitution variable ${libnginx-mod-http-upstream-fair:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-perl: substitution variable ${libnginx-mod-nchan:Version} unused, but is
defined
dpkg-gencontrol: warning: package libnginx-mod-http-perl: substitution variable ${nginx:abi} unused, but is defined
        chmod 0644 -- debian/.debhelper/libnginx-mod-http-perl/dbgsym-root/DEBIAN/control
        dpkg-gencontrol -plibnginx-mod-http-perl -ldebian/changelog -Tdebian/libnginx-mod-http-perl.substvars -Pdebian/libnginx-mod-http-perl -Tdebian/substvars -Tdebian/libnginx-mod.abisubstvars
dpkg-gencontrol: warning: package libnginx-mod-http-perl: substitution variable ${libnginx-mod-geoip2:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-perl: substitution variable ${libnginx-mod-http-auth-pam:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-perl: substitution variable ${libnginx-mod-http-cache-purge:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-perl: substitution variable ${libnginx-mod-http-dav-ext:Version} unused,
but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-perl: substitution variable ${libnginx-mod-http-echo:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-perl: substitution variable ${libnginx-mod-http-fancyindex:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-perl: substitution variable ${libnginx-mod-http-geoip2:Version} unused,
but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-perl: substitution variable ${libnginx-mod-http-headers-more-filter:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-perl: substitution variable ${libnginx-mod-http-lua:Version} unused, but
is defined
dpkg-gencontrol: warning: package libnginx-mod-http-perl: substitution variable ${libnginx-mod-http-subs-filter:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-perl: substitution variable ${libnginx-mod-http-uploadprogress:Version}
unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-perl: substitution variable ${libnginx-mod-http-upstream-fair:Version} unused, but is defined
dpkg-gencontrol: warning: package libnginx-mod-http-perl: substitution variable ${libnginx-mod-nchan:Version} unused, but is
defined
dpkg-gencontrol: warning: package libnginx-mod-http-perl: substitution variable ${nginx:abi} unused, but is defined
        chmod 0644 -- debian/libnginx-mod-http-perl/DEBIAN/control
make[1]: Leaving directory '/root/custom-nginx/nginx-1.24.0'
   dh_md5sums
        install -m0755 -d debian/nginx/DEBIAN
        install -m0755 -d debian/libnginx-mod-http-geoip/DEBIAN
        cd debian/nginx >/dev/null && xargs -r0 md5sum | perl -pe 'if (s@^\\@@) { s/\\\\/\\/g; }' > DEBIAN/md5sums
        cd debian/libnginx-mod-http-geoip >/dev/null && xargs -r0 md5sum | perl -pe 'if (s@^\\@@) { s/\\\\/\\/g; }' > DEBIAN/md5sums
        chmod 0644 -- debian/nginx/DEBIAN/md5sums
        install -m0755 -d debian/.debhelper/nginx/dbgsym-root/DEBIAN
        chmod 0644 -- debian/libnginx-mod-http-geoip/DEBIAN/md5sums
        install -m0755 -d debian/.debhelper/libnginx-mod-http-geoip/dbgsym-root/DEBIAN
        cd debian/.debhelper/libnginx-mod-http-geoip/dbgsym-root >/dev/null && xargs -r0 md5sum | perl -pe 'if (s@^\\@@) { s/\\\\/\\/g; }' > DEBIAN/md5sums
        cd debian/.debhelper/nginx/dbgsym-root >/dev/null && xargs -r0 md5sum | perl -pe 'if (s@^\\@@) { s/\\\\/\\/g; }' > DEBIAN/md5sums
        chmod 0644 -- debian/.debhelper/nginx/dbgsym-root/DEBIAN/md5sums
        install -m0755 -d debian/nginx-doc/DEBIAN
        chmod 0644 -- debian/.debhelper/libnginx-mod-http-geoip/dbgsym-root/DEBIAN/md5sums
        cd debian/nginx-doc >/dev/null && xargs -r0 md5sum | perl -pe 'if (s@^\\@@) { s/\\\\/\\/g; }' > DEBIAN/md5sums
        install -m0755 -d debian/libnginx-mod-http-image-filter/DEBIAN
        cd debian/libnginx-mod-http-image-filter >/dev/null && xargs -r0 md5sum | perl -pe 'if (s@^\\@@) { s/\\\\/\\/g; }' >
DEBIAN/md5sums
        chmod 0644 -- debian/libnginx-mod-http-image-filter/DEBIAN/md5sums
        install -m0755 -d debian/.debhelper/libnginx-mod-http-image-filter/dbgsym-root/DEBIAN
        cd debian/.debhelper/libnginx-mod-http-image-filter/dbgsym-root >/dev/null && xargs -r0 md5sum | perl -pe 'if (s@^\\@@) { s/\\\\/\\/g; }' > DEBIAN/md5sums
        chmod 0644 -- debian/nginx-doc/DEBIAN/md5sums
        install -m0755 -d debian/nginx-common/DEBIAN
        cd debian/nginx-common >/dev/null && xargs -r0 md5sum | perl -pe 'if (s@^\\@@) { s/\\\\/\\/g; }' > DEBIAN/md5sums
        chmod 0644 -- debian/.debhelper/libnginx-mod-http-image-filter/dbgsym-root/DEBIAN/md5sums
        install -m0755 -d debian/libnginx-mod-http-xslt-filter/DEBIAN
        cd debian/libnginx-mod-http-xslt-filter >/dev/null && xargs -r0 md5sum | perl -pe 'if (s@^\\@@) { s/\\\\/\\/g; }' >
DEBIAN/md5sums
        chmod 0644 -- debian/libnginx-mod-http-xslt-filter/DEBIAN/md5sums
        install -m0755 -d debian/.debhelper/libnginx-mod-http-xslt-filter/dbgsym-root/DEBIAN
        cd debian/.debhelper/libnginx-mod-http-xslt-filter/dbgsym-root >/dev/null && xargs -r0 md5sum | perl -pe 'if (s@^\\@@) { s/\\\\/\\/g; }' > DEBIAN/md5sums
        chmod 0644 -- debian/nginx-common/DEBIAN/md5sums
        install -m0755 -d debian/nginx-dev/DEBIAN
        cd debian/nginx-dev >/dev/null && xargs -r0 md5sum | perl -pe 'if (s@^\\@@) { s/\\\\/\\/g; }' > DEBIAN/md5sums
        chmod 0644 -- debian/.debhelper/libnginx-mod-http-xslt-filter/dbgsym-root/DEBIAN/md5sums
        install -m0755 -d debian/libnginx-mod-mail/DEBIAN
        cd debian/libnginx-mod-mail >/dev/null && xargs -r0 md5sum | perl -pe 'if (s@^\\@@) { s/\\\\/\\/g; }' > DEBIAN/md5sums
        chmod 0644 -- debian/libnginx-mod-mail/DEBIAN/md5sums
        install -m0755 -d debian/.debhelper/libnginx-mod-mail/dbgsym-root/DEBIAN
        cd debian/.debhelper/libnginx-mod-mail/dbgsym-root >/dev/null && xargs -r0 md5sum | perl -pe 'if (s@^\\@@) { s/\\\\/\\/g; }' > DEBIAN/md5sums
        chmod 0644 -- debian/.debhelper/libnginx-mod-mail/dbgsym-root/DEBIAN/md5sums
        install -m0755 -d debian/libnginx-mod-stream/DEBIAN
        cd debian/libnginx-mod-stream >/dev/null && xargs -r0 md5sum | perl -pe 'if (s@^\\@@) { s/\\\\/\\/g; }' > DEBIAN/md5sums
        chmod 0644 -- debian/nginx-dev/DEBIAN/md5sums
        install -m0755 -d debian/nginx-core/DEBIAN
        chmod 0644 -- debian/libnginx-mod-stream/DEBIAN/md5sums
        install -m0755 -d debian/.debhelper/libnginx-mod-stream/dbgsym-root/DEBIAN
        cd debian/nginx-core >/dev/null && xargs -r0 md5sum | perl -pe 'if (s@^\\@@) { s/\\\\/\\/g; }' > DEBIAN/md5sums
        cd debian/.debhelper/libnginx-mod-stream/dbgsym-root >/dev/null && xargs -r0 md5sum | perl -pe 'if (s@^\\@@) { s/\\\\/\\/g; }' > DEBIAN/md5sums
        chmod 0644 -- debian/nginx-core/DEBIAN/md5sums
        install -m0755 -d debian/nginx-full/DEBIAN
        chmod 0644 -- debian/.debhelper/libnginx-mod-stream/dbgsym-root/DEBIAN/md5sums
        install -m0755 -d debian/libnginx-mod-stream-geoip/DEBIAN
        cd debian/libnginx-mod-stream-geoip >/dev/null && xargs -r0 md5sum | perl -pe 'if (s@^\\@@) { s/\\\\/\\/g; }' > DEBIAN/md5sums
        cd debian/nginx-full >/dev/null && xargs -r0 md5sum | perl -pe 'if (s@^\\@@) { s/\\\\/\\/g; }' > DEBIAN/md5sums
        chmod 0644 -- debian/libnginx-mod-stream-geoip/DEBIAN/md5sums
        chmod 0644 -- debian/nginx-full/DEBIAN/md5sums
        install -m0755 -d debian/nginx-light/DEBIAN
        install -m0755 -d debian/.debhelper/libnginx-mod-stream-geoip/dbgsym-root/DEBIAN
        cd debian/nginx-light >/dev/null && xargs -r0 md5sum | perl -pe 'if (s@^\\@@) { s/\\\\/\\/g; }' > DEBIAN/md5sums
        cd debian/.debhelper/libnginx-mod-stream-geoip/dbgsym-root >/dev/null && xargs -r0 md5sum | perl -pe 'if (s@^\\@@) {
s/\\\\/\\/g; }' > DEBIAN/md5sums
        chmod 0644 -- debian/nginx-light/DEBIAN/md5sums
        install -m0755 -d debian/nginx-extras/DEBIAN
        chmod 0644 -- debian/.debhelper/libnginx-mod-stream-geoip/dbgsym-root/DEBIAN/md5sums
        cd debian/nginx-extras >/dev/null && xargs -r0 md5sum | perl -pe 'if (s@^\\@@) { s/\\\\/\\/g; }' > DEBIAN/md5sums
        install -m0755 -d debian/libnginx-mod-http-perl/DEBIAN
        cd debian/libnginx-mod-http-perl >/dev/null && xargs -r0 md5sum | perl -pe 'if (s@^\\@@) { s/\\\\/\\/g; }' > DEBIAN/md5sums
        chmod 0644 -- debian/nginx-extras/DEBIAN/md5sums
        chmod 0644 -- debian/libnginx-mod-http-perl/DEBIAN/md5sums
        install -m0755 -d debian/.debhelper/libnginx-mod-http-perl/dbgsym-root/DEBIAN
        cd debian/.debhelper/libnginx-mod-http-perl/dbgsym-root >/dev/null && xargs -r0 md5sum | perl -pe 'if (s@^\\@@) { s/\\\\/\\/g; }' > DEBIAN/md5sums
        chmod 0644 -- debian/.debhelper/libnginx-mod-http-perl/dbgsym-root/DEBIAN/md5sums
   dh_builddeb
        dpkg-deb --root-owner-group --build debian/nginx ..
dpkg-deb: сборка пакета «nginx» в «../nginx_1.24.0-2ubuntu7-custom_amd64.deb».
        dpkg-deb --root-owner-group --build debian/.debhelper/libnginx-mod-http-image-filter/dbgsym-root debian/.debhelper/scratch-space/build-libnginx-mod-http-image-filter
dpkg-deb: сборка пакета «libnginx-mod-http-image-filter-dbgsym» в «debian/.debhelper/scratch-space/build-libnginx-mod-http-image-filter/libnginx-mod-http-image-filter-dbgsym_1.24.0-2ubuntu7-custom_amd64.deb».
        Renaming libnginx-mod-http-image-filter-dbgsym_1.24.0-2ubuntu7-custom_amd64.deb to libnginx-mod-http-image-filter-dbgsym_1.24.0-2ubuntu7-custom_amd64.ddeb
        mv debian/.debhelper/scratch-space/build-libnginx-mod-http-image-filter/libnginx-mod-http-image-filter-dbgsym_1.24.0-2ubuntu7-custom_amd64.deb ../libnginx-mod-http-image-filter-dbgsym_1.24.0-2ubuntu7-custom_amd64.ddeb
        dpkg-deb --root-owner-group --build debian/libnginx-mod-http-xslt-filter ..
dpkg-deb: сборка пакета «libnginx-mod-http-xslt-filter» в «../libnginx-mod-http-xslt-filter_1.24.0-2ubuntu7-custom_amd64.deb».
        dpkg-deb --root-owner-group --build debian/.debhelper/libnginx-mod-http-xslt-filter/dbgsym-root debian/.debhelper/scratch-space/build-libnginx-mod-http-xslt-filter
dpkg-deb: сборка пакета «libnginx-mod-http-xslt-filter-dbgsym» в «debian/.debhelper/scratch-space/build-libnginx-mod-http-xslt-filter/libnginx-mod-http-xslt-filter-dbgsym_1.24.0-2ubuntu7-custom_amd64.deb».
        Renaming libnginx-mod-http-xslt-filter-dbgsym_1.24.0-2ubuntu7-custom_amd64.deb to libnginx-mod-http-xslt-filter-dbgsym_1.24.0-2ubuntu7-custom_amd64.ddeb
        mv debian/.debhelper/scratch-space/build-libnginx-mod-http-xslt-filter/libnginx-mod-http-xslt-filter-dbgsym_1.24.0-2ubuntu7-custom_amd64.deb ../libnginx-mod-http-xslt-filter-dbgsym_1.24.0-2ubuntu7-custom_amd64.ddeb
        dpkg-deb --root-owner-group --build debian/libnginx-mod-mail ..
dpkg-deb: сборка пакета «libnginx-mod-mail» в «../libnginx-mod-mail_1.24.0-2ubuntu7-custom_amd64.deb».
        dpkg-deb --root-owner-group --build debian/.debhelper/libnginx-mod-mail/dbgsym-root debian/.debhelper/scratch-space/build-libnginx-mod-mail
dpkg-deb: сборка пакета «libnginx-mod-mail-dbgsym» в «debian/.debhelper/scratch-space/build-libnginx-mod-mail/libnginx-mod-mail-dbgsym_1.24.0-2ubuntu7-custom_amd64.deb».
        Renaming libnginx-mod-mail-dbgsym_1.24.0-2ubuntu7-custom_amd64.deb to libnginx-mod-mail-dbgsym_1.24.0-2ubuntu7-custom_amd64.ddeb
        mv debian/.debhelper/scratch-space/build-libnginx-mod-mail/libnginx-mod-mail-dbgsym_1.24.0-2ubuntu7-custom_amd64.deb
../libnginx-mod-mail-dbgsym_1.24.0-2ubuntu7-custom_amd64.ddeb
        dpkg-deb --root-owner-group --build debian/libnginx-mod-stream ..
dpkg-deb: сборка пакета «libnginx-mod-stream» в «../libnginx-mod-stream_1.24.0-2ubuntu7-custom_amd64.deb».
        dpkg-deb --root-owner-group --build debian/.debhelper/libnginx-mod-stream/dbgsym-root debian/.debhelper/scratch-space/build-libnginx-mod-stream
dpkg-deb: сборка пакета «libnginx-mod-stream-dbgsym» в «debian/.debhelper/scratch-space/build-libnginx-mod-stream/libnginx-mod-stream-dbgsym_1.24.0-2ubuntu7-custom_amd64.deb».
        Renaming libnginx-mod-stream-dbgsym_1.24.0-2ubuntu7-custom_amd64.deb to libnginx-mod-stream-dbgsym_1.24.0-2ubuntu7-custom_amd64.ddeb
        mv debian/.debhelper/scratch-space/build-libnginx-mod-stream/libnginx-mod-stream-dbgsym_1.24.0-2ubuntu7-custom_amd64.deb ../libnginx-mod-stream-dbgsym_1.24.0-2ubuntu7-custom_amd64.ddeb
        dpkg-deb --root-owner-group --build debian/libnginx-mod-stream-geoip ..
dpkg-deb: сборка пакета «libnginx-mod-stream-geoip» в «../libnginx-mod-stream-geoip_1.24.0-2ubuntu7-custom_amd64.deb».
        dpkg-deb --root-owner-group --build debian/.debhelper/libnginx-mod-stream-geoip/dbgsym-root debian/.debhelper/scratch-space/build-libnginx-mod-stream-geoip
dpkg-deb: сборка пакета «libnginx-mod-stream-geoip-dbgsym» в «debian/.debhelper/scratch-space/build-libnginx-mod-stream-geoip/libnginx-mod-stream-geoip-dbgsym_1.24.0-2ubuntu7-custom_amd64.deb».
        Renaming libnginx-mod-stream-geoip-dbgsym_1.24.0-2ubuntu7-custom_amd64.deb to libnginx-mod-stream-geoip-dbgsym_1.24.0-2ubuntu7-custom_amd64.ddeb
        mv debian/.debhelper/scratch-space/build-libnginx-mod-stream-geoip/libnginx-mod-stream-geoip-dbgsym_1.24.0-2ubuntu7-custom_amd64.deb ../libnginx-mod-stream-geoip-dbgsym_1.24.0-2ubuntu7-custom_amd64.ddeb
        dpkg-deb --root-owner-group --build debian/libnginx-mod-http-perl ..
dpkg-deb: сборка пакета «libnginx-mod-http-perl» в «../libnginx-mod-http-perl_1.24.0-2ubuntu7-custom_amd64.deb».
        dpkg-deb --root-owner-group --build debian/.debhelper/libnginx-mod-http-perl/dbgsym-root debian/.debhelper/scratch-space/build-libnginx-mod-http-perl
dpkg-deb: сборка пакета «libnginx-mod-http-perl-dbgsym» в «debian/.debhelper/scratch-space/build-libnginx-mod-http-perl/libnginx-mod-http-perl-dbgsym_1.24.0-2ubuntu7-custom_amd64.deb».
        Renaming libnginx-mod-http-perl-dbgsym_1.24.0-2ubuntu7-custom_amd64.deb to libnginx-mod-http-perl-dbgsym_1.24.0-2ubuntu7-custom_amd64.ddeb
        mv debian/.debhelper/scratch-space/build-libnginx-mod-http-perl/libnginx-mod-http-perl-dbgsym_1.24.0-2ubuntu7-custom_amd64.deb ../libnginx-mod-http-perl-dbgsym_1.24.0-2ubuntu7-custom_amd64.ddeb
        dpkg-deb --root-owner-group --build debian/.debhelper/nginx/dbgsym-root debian/.debhelper/scratch-space/build-nginx
dpkg-deb: сборка пакета «nginx-dbgsym» в «debian/.debhelper/scratch-space/build-nginx/nginx-dbgsym_1.24.0-2ubuntu7-custom_amd64.deb».
        Renaming nginx-dbgsym_1.24.0-2ubuntu7-custom_amd64.deb to nginx-dbgsym_1.24.0-2ubuntu7-custom_amd64.ddeb
        mv debian/.debhelper/scratch-space/build-nginx/nginx-dbgsym_1.24.0-2ubuntu7-custom_amd64.deb ../nginx-dbgsym_1.24.0-2ubuntu7-custom_amd64.ddeb
        dpkg-deb --root-owner-group --build debian/nginx-doc ..
dpkg-deb: сборка пакета «nginx-doc» в «../nginx-doc_1.24.0-2ubuntu7-custom_all.deb».
        dpkg-deb --root-owner-group --build debian/nginx-common ..
dpkg-deb: сборка пакета «nginx-common» в «../nginx-common_1.24.0-2ubuntu7-custom_all.deb».
        dpkg-deb --root-owner-group --build debian/nginx-dev ..
dpkg-deb: сборка пакета «nginx-dev» в «../nginx-dev_1.24.0-2ubuntu7-custom_all.deb».
        dpkg-deb --root-owner-group --build debian/nginx-core ..
dpkg-deb: сборка пакета «nginx-core» в «../nginx-core_1.24.0-2ubuntu7-custom_all.deb».
        dpkg-deb --root-owner-group --build debian/nginx-full ..
dpkg-deb: сборка пакета «nginx-full» в «../nginx-full_1.24.0-2ubuntu7-custom_all.deb».
        dpkg-deb --root-owner-group --build debian/nginx-light ..
dpkg-deb: сборка пакета «nginx-light» в «../nginx-light_1.24.0-2ubuntu7-custom_all.deb».
        dpkg-deb --root-owner-group --build debian/nginx-extras ..
dpkg-deb: сборка пакета «nginx-extras» в «../nginx-extras_1.24.0-2ubuntu7-custom_amd64.deb».
        dpkg-deb --root-owner-group --build debian/libnginx-mod-http-geoip ..
dpkg-deb: сборка пакета «libnginx-mod-http-geoip» в «../libnginx-mod-http-geoip_1.24.0-2ubuntu7-custom_amd64.deb».
        dpkg-deb --root-owner-group --build debian/.debhelper/libnginx-mod-http-geoip/dbgsym-root debian/.debhelper/scratch-space/build-libnginx-mod-http-geoip
dpkg-deb: сборка пакета «libnginx-mod-http-geoip-dbgsym» в «debian/.debhelper/scratch-space/build-libnginx-mod-http-geoip/libnginx-mod-http-geoip-dbgsym_1.24.0-2ubuntu7-custom_amd64.deb».
        Renaming libnginx-mod-http-geoip-dbgsym_1.24.0-2ubuntu7-custom_amd64.deb to libnginx-mod-http-geoip-dbgsym_1.24.0-2ubuntu7-custom_amd64.ddeb
        mv debian/.debhelper/scratch-space/build-libnginx-mod-http-geoip/libnginx-mod-http-geoip-dbgsym_1.24.0-2ubuntu7-custom_amd64.deb ../libnginx-mod-http-geoip-dbgsym_1.24.0-2ubuntu7-custom_amd64.ddeb
        dpkg-deb --root-owner-group --build debian/libnginx-mod-http-image-filter ..
dpkg-deb: сборка пакета «libnginx-mod-http-image-filter» в «../libnginx-mod-http-image-filter_1.24.0-2ubuntu7-custom_amd64.deb».
 dpkg-genbuildinfo --build=binary -O../nginx_1.24.0-2ubuntu7-custom_amd64.buildinfo
 dpkg-genchanges --build=binary -O../nginx_1.24.0-2ubuntu7-custom_amd64.changes
dpkg-genchanges: info: binary-only upload (no source code included)
 dpkg-source --after-build .
dpkg-buildpackage: info: binary-only upload (no source included)</code>
</details>

---
Пакеты собраны
```
root@UbuntuTestVirt:~/custom-nginx/nginx-1.24.0# cd ..
root@UbuntuTestVirt:~/custom-nginx# ll
total 4776
drwxr-xr-x  3 root root    4096 июн 30 00:59 ./
drwx------  9 root root    4096 июн 29 22:43 ../
-rw-r--r--  1 root root   21684 июн 30 00:59 libnginx-mod-http-geoip_1.24.0-2ubuntu7-custom_amd64.deb
-rw-r--r--  1 root root   36308 июн 30 00:59 libnginx-mod-http-geoip-dbgsym_1.24.0-2ubuntu7-custom_amd64.ddeb
-rw-r--r--  1 root root   25728 июн 30 00:59 libnginx-mod-http-image-filter_1.24.0-2ubuntu7-custom_amd64.deb
-rw-r--r--  1 root root   43638 июн 30 00:59 libnginx-mod-http-image-filter-dbgsym_1.24.0-2ubuntu7-custom_amd64.ddeb
-rw-r--r--  1 root root   34316 июн 30 00:59 libnginx-mod-http-perl_1.24.0-2ubuntu7-custom_amd64.deb
-rw-r--r--  1 root root  107872 июн 30 00:59 libnginx-mod-http-perl-dbgsym_1.24.0-2ubuntu7-custom_amd64.ddeb
-rw-r--r--  1 root root   24128 июн 30 00:59 libnginx-mod-http-xslt-filter_1.24.0-2ubuntu7-custom_amd64.deb
-rw-r--r--  1 root root   53386 июн 30 00:59 libnginx-mod-http-xslt-filter-dbgsym_1.24.0-2ubuntu7-custom_amd64.ddeb
-rw-r--r--  1 root root   56964 июн 30 00:59 libnginx-mod-mail_1.24.0-2ubuntu7-custom_amd64.deb
-rw-r--r--  1 root root  106134 июн 30 00:59 libnginx-mod-mail-dbgsym_1.24.0-2ubuntu7-custom_amd64.ddeb
-rw-r--r--  1 root root   84210 июн 30 00:59 libnginx-mod-stream_1.24.0-2ubuntu7-custom_amd64.deb
-rw-r--r--  1 root root  174400 июн 30 00:59 libnginx-mod-stream-dbgsym_1.24.0-2ubuntu7-custom_amd64.ddeb
-rw-r--r--  1 root root   20786 июн 30 00:59 libnginx-mod-stream-geoip_1.24.0-2ubuntu7-custom_amd64.deb
-rw-r--r--  1 root root   22332 июн 30 00:59 libnginx-mod-stream-geoip-dbgsym_1.24.0-2ubuntu7-custom_amd64.ddeb
drwxr-xr-x 11 root root    4096 июн 29 23:50 nginx-1.24.0/
-rw-r--r--  1 root root   16788 июн 30 00:59 nginx_1.24.0-2ubuntu7-custom_amd64.buildinfo
-rw-r--r--  1 root root    9819 июн 30 00:59 nginx_1.24.0-2ubuntu7-custom_amd64.changes
-rw-r--r--  1 root root  963960 июн 30 00:59 nginx_1.24.0-2ubuntu7-custom_amd64.deb
-rw-r--r--  1 root root   81792 мар 31  2024 nginx_1.24.0-2ubuntu7.debian.tar.xz
-rw-r--r--  1 root root    3646 мар 31  2024 nginx_1.24.0-2ubuntu7.dsc
-rw-r--r--  1 root root 1112471 июн 28  2023 nginx_1.24.0.orig.tar.gz
-rw-r--r--  1 root root   42922 июн 30 00:59 nginx-common_1.24.0-2ubuntu7-custom_all.deb
-rw-r--r--  1 root root   16364 июн 30 00:59 nginx-core_1.24.0-2ubuntu7-custom_all.deb
-rw-r--r--  1 root root 1571256 июн 30 00:59 nginx-dbgsym_1.24.0-2ubuntu7-custom_amd64.ddeb
-rw-r--r--  1 root root  121788 июн 30 00:59 nginx-dev_1.24.0-2ubuntu7-custom_all.deb
-rw-r--r--  1 root root   24376 июн 30 00:59 nginx-doc_1.24.0-2ubuntu7-custom_all.deb
-rw-r--r--  1 root root   16612 июн 30 00:59 nginx-extras_1.24.0-2ubuntu7-custom_amd64.deb
-rw-r--r--  1 root root   16428 июн 30 00:59 nginx-full_1.24.0-2ubuntu7-custom_all.deb
-rw-r--r--  1 root root   16140 июн 30 00:59 nginx-light_1.24.0-2ubuntu7-custom_all.deb
```
---
Устанавливам собраный пакет
```
root@UbuntuTestVirt:~/custom-nginx# dpkg -i nginx-common_*.deb nginx_*.deb
root@UbuntuTestVirt:~/custom-nginx# apt-get install -f
```
---
Проверяем наличие установленного модуля в пекете nginx
```
root@UbuntuTestVirt:~/custom-nginx# nginx -V 2>&1 | grep -o brotli
brotli
```
