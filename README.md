# Changes after forking the original code

1.Adding Patch to Aria2

2.Adding code in build.sh to install git before start the compile .

3.The time to schedule the action is changed.

# aria2-static-build

![Build and Release](https://github.com/abcfy2/aria2-static-build/actions/workflows/build_and_release.yml/badge.svg)

aria2 static build using musl that supports many platforms.

## Download

You can download from [Continuous Build](https://github.com/abcfy2/aria2-static-build/releases/tag/continuous) (Weekly build from aria2 master branch with latest dependencies).

Or download from [latest release](https://github.com/abcfy2/aria2-static-build/releases/latest) build (Build from aria2 latest release version).

## Android users NOTE

If you are executing in an Android environment (such as x86_64, arm, or aarch64), please follow the official aria2 Android README: https://github.com/aria2/aria2/blob/master/README.android

Here is a sample:

```sh
cat /etc/security/cacerts/* | ./aria2c --ca-certificate=/proc/self/fd/0 --async-dns-server=1.1.1.1 https://github.com/
```

Please note that `getprop net.dns1` does not work since Android 8, so you must set a valid DNS manually.

## https certificates NOTE (Linux Only)

SSL certificates location may vary from different distributions. E.g: Ubuntu uses `/etc/ssl/certs/ca-certificates.crt`, but CentOS uses `/etc/pki/tls/certs/ca-bundle.crt`.

It's impossible to detect the certificate locations in all distributions. See issue: [openssl/openssl#7481](https://github.com/openssl/openssl/issues/7481). Fortunately, most distributions may contain a symbolic link `/etc/ssl/cert.pem` that points to the actual file path.

So I set compile options `--openssldir=/etc/ssl/` for openssl/libressl. Which works for most distributions.

If your environment contains the file `/etc/ssl/openssl.cnf` or `/etc/ssl/cert.pem`, you are lucky and can use my build out of the box.

If your environment does not contain any of these files, you must do one of the following to ensure HTTPS requests work.

- add `--ca-certificate=/path/to/your/certificate` to `aria2c` or set `ca-certificate=/path/to/your/certificate` in `~/.aria2/aria2.conf`. E.g: `./aria2c --ca-certificate=/etc/pki/tls/certs/ca-bundle.crt https://github.com/`
- Or add `SSL_CERT_FILE=/path/to/your/certificate` environment variable before you run `aria2c`. E.g: `export SSL_CERT_FILE=/etc/pki/tls/certs/ca-bundle.crt; ./aria2c https://github.com/` or `SSL_CERT_FILE=/etc/pki/tls/certs/ca-bundle.crt ./aria2c https://github.com/`

> Reference for different distribution certificates locations: https://gitlab.com/probono/platformissues/blob/master/README.md#certificates

## Fedora users NOTE

Fedora's openssl may contains some non-official patches and contains some configurations not support by this build openssl.

For this scenario, you can force set an `OPENSSL_CONF` environment variable to point to an invalid path before run `aria2c`. E.g:

```sh
OPENSSL_CONF=/tmp ./aria2c https://github.com/
```

This will not be interfered with by Fedora's openssl configuration.

## Build locally yourself

Requirements:

- docker

```sh
docker run --rm -v `pwd`:/build abcfy2/musl-cross-toolchain-ubuntu:${CROSS_HOST} /build/build.sh
```

> You can also use my GHCR image as alternative: [ghcr.io/abcfy2/musl-cross-toolchain-ubuntu](https://github.com/abcfy2/docker-musl-cross-toolchain-ubuntu/pkgs/container/musl-cross-toolchain-ubuntu)

All avaliable `CROSS_HOST` can be found in [Tags](https://hub.docker.com/r/abcfy2/musl-cross-toolchain-ubuntu/tags) page.

**NOTE**: Currently I only tested these tags:

- arm-unknown-linux-musleabi
- aarch64-unknown-linux-musl
- loongarch64-unknown-linux-musl
- mips-unknown-linux-musl
- mipsel-unknown-linux-musl
- mips64-unknown-linux-musl
- x86_64-unknown-linux-musl
- i686-unknown-linux-musl
- x86_64-w64-mingw32
- i686-w64-mingw32

If you want to build for other platform, you may have to modify `build.sh` to suitable for your platform.

Cached build dependencies (`downloads/`), `build_info.md` and `aria2c` will be found in current directory.

You can set more optional environment variables in `docker` command like:

```sh
docker run --rm -v `pwd`:/build -e USE_ZLIB_NG=0 -e USE_LIBRESSL=1 abcfy2/muslcc-toolchain-ubuntu:${CROSS_HOST} /build/build.sh
```

Optional environment variables:

- `ARIA2_VER`: build specific version of aria2, e.g: `1.36.0`. Default: `master`.
- `USE_CHINA_MIRROR`: set to `1` will use China mirrors, if you were located in China, please set to `1`. Default: `0`.
- `USE_ZLIB_NG`: use [zlib-ng](https://github.com/zlib-ng/zlib-ng) instead of [zlib](https://zlib.net/). Default: `1`
- `USE_LIBRESSL`: use [LibreSSL](https://www.libressl.org/) instead of [OpenSSL](https://www.openssl.org/). Default: `0`. **_NOTE_**, if `CROSS_HOST=x86_64-w64-mingw32` will not use openssl or libressl because aria2 and all dependencies will use WinTLS instead.
