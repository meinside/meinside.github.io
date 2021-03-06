---
layout: post
title: Build Cross Compile Tools for Raspberry Pi (Raspbian Stretch) on macOS
tags:
- raspberry pi
- cross compilation
- macOS
published: true
---

This guide is based on [this article](https://medium.com/@zw3rk/making-a-raspbian-cross-compilation-sdk-830fe56d75ba).

Written and tested with:

* Raspberry Pi 3 B+
  * Raspbian Stretch (GCC 6.3.0 20170516)
* macOS 10.13.3
  * clang+llvm 6.0.0
  * binutils 2.30

----

## Create a SDK directory

Create a SDK directory for not messing other things up:

{% highlight bash %}
$ mkdir -p raspbian-sdk/{prebuilt,sysroot}
{% endhighlight %}

## Download / Install Tools

### clang+llvm

{% highlight bash %}
$ wget http://releases.llvm.org/6.0.0/clang+llvm-6.0.0-x86_64-apple-darwin.tar.xz
$ tar -xzvf clang+llvm-6.0.0-x86_64-apple-darwin.tar.xz -C "raspbian-sdk/prebuilt" --strip-components=1
{% endhighlight %}

### binutils

{% highlight bash %}
$ wget http://ftp.gnu.org/gnu/binutils/binutils-2.30.tar.xz
$ tar -xzvf binutils-2.30.tar.xz
$ cd binutils-2.30
$ brew install coreutils
$ ./configure --prefix="`/usr/local/bin/realpath ../raspbian-sdk/prebuilt`" \
	--target=arm-linux-gnueabihf \
	--enable-gold=yes \
	--enable-ld=yes \
	--enable-targets=arm-linux-gnueabihf \
	--enable-multilib \
	--enable-interwork \
	--disable-werror \
	--quiet
$ make && make install
{% endhighlight %}

Now come back to the initial directory:

{% highlight bash %}
$ cd ..
{% endhighlight %}

## Copy Header/Library Files from Raspberry Pi

Let's copy header and library files from Raspberry Pi over the network with `rsync`:

{% highlight bash %}
$ brew install rsync
$ /usr/local/bin/rsync -rzLR --safe-links \
	pi@raspberry:/usr/lib/arm-linux-gnueabihf \
	pi@raspberry:/usr/lib/gcc/arm-linux-gnueabihf \
	pi@raspberry:/usr/include \
	pi@raspberry:/lib/arm-linux-gnueabihf \
	raspbian-sdk/sysroot
{% endhighlight %}

## Create a Wrapper Shell Script for Cross Compiling

Open a new file,

{% highlight bash %}
$ vi raspbian-sdk/prebuilt/bin/arm-linux-gnueabihf-clang
{% endhighlight %}

and fill it with following lines:

{% highlight bash %}
#!/bin/bash
BASE=$(dirname $0)
SYSROOT="${BASE}/../../sysroot"
TARGET=arm-linux-gnueabihf
COMPILER_PATH="${SYSROOT}/usr/lib/gcc/${TARGET}/6.3.0"
exec env COMPILER_PATH="${COMPILER_PATH}" \
	"${BASE}/clang" --target=${TARGET} \
		--sysroot="${SYSROOT}" \
		-isysroot "${SYSROOT}" \
		-L"${COMPILER_PATH}" \
		--gcc-toolchain="${BASE}" \
		"$@"
{% endhighlight %}

then grant executable permission to it:

{% highlight bash %}
$ chmod +x raspbian-sdk/prebuilt/bin/arm-linux-gnueabihf-clang
{% endhighlight %}

----

## Test Build

Create a `hello.c` file with following content:

{% highlight c %}
#include <stdio.h>

int main(int argc, char** argv) {
	printf("Hello, world\n");
	return 0;
}
{% endhighlight %}

then build it with the shell script:

{% highlight bash %}
$ raspbian-sdk/prebuilt/bin/arm-linux-gnueabihf-clang -o hello hello.c
{% endhighlight %}

If nothing goes wrong, a file named `hello` would be generated.

Upload it to the Raspberry Pi and run with:

{% highlight bash %}
pi@raspberry $ ./hello
{% endhighlight %}

then it will print out:

{% highlight bash %}
Hello, world
{% endhighlight %}

Well done!

If any error occures while building, try again with `-v` option to see what went wrong:

{% highlight bash %}
$ raspbian-sdk/prebuilt/bin/arm-linux-gnueabihf-clang -o hello hello.c -v
{% endhighlight %}

You may have to change the version number of **COMPILER_PATH** in your shell script, or install some packages on Raspberry Pi and copy header/library files again.

## Wrap-Up

Building large projects or binaries on Raspberry Pi is painful; it often goes unresponsive and even crashes sometimes.

With this cross compilation tools, I will try building TensorFlow or Haskell packages like ghc-mod.

Will post about it if I see some progress :-)

