# ProfHW7
Домашнее задание №7. Сборка RPM-пакета и создание репозитория.

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
Создаём папку out (я так и не понял для чего) и запускаем сборку brotli
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


```
apt install libexpat-dev libgd-dev libgeoip-dev libpcre2-dev libperl-dev libssl-dev libxslt1-dev po-debconf debhelper-compat
```
