---
layout: post
title: TensorFlow and Go on Raspberry Pi
tags:
- raspberry pi
- golang
- tensorflow
published: true
---

[TensorFlow 1.0 now supports Golang](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/go), so I gave it a try on Raspberry Pi:

----

# 0. Used Hardware and Software Versions

All steps were taken on my **Raspberry Pi 3 B** model with:

* Minimum GPU memory allocated (16MB)
* **1GB** of swap memory
* External USB HDD (as root partition)

and software versions were:

* tensorflow 1.0.0
* protobuf 3.1.0
* bazel 0.4.4

Before the beginning, I had to install dependencies:

### for python

```bash
$ sudo apt-get install python-pip python-numpy swig python-dev
$ sudo pip install wheel
```

### for protobuf

```bash
$ sudo apt-get install autoconf automake libtool
```

### for bazel

```bash
$ sudo apt-get install pkg-config zip g++ zlib1g-dev unzip oracle-java8-jdk
```

### for compiler optimization and avoiding possible errors

It is said that both protobuf and tensorflow should be built with **gcc-4.8**, so... :

```bash
$ sudo apt-get install gcc-4.8 g++-4.8
$ sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.8 100
$ sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-4.8 100
```

----

# 1. Install Protobuf

I cloned the repository:

```bash
$ git clone https://github.com/google/protobuf.git
```

and built it:

```bash
$ cd protobuf
$ git checkout v3.1.0
$ ./autogen.sh
$ ./configure
$ make CXX=g++-4.8 -j 4
$ sudo make install
$ sudo ldconfig
```

The build took less than 1 hour to finish.

I could see the version of installed protobuf like:

```bash
$ protoc --version
```

```
libprotoc 3.1.0
```

# 2. Install Bazel

## a. download

I got the latest release from [here](https://github.com/bazelbuild/bazel/releases), and unzipped it:

```bash
$ unzip -d bazel bazel-0.4.4-dist.zip
```

## b. edit files

In the unzipped directory, I opened the `scripts/bootstrap/compile.sh` file:

```bash
$ cd bazel
$ vi scripts/bootstrap/compile.sh
```

searched for lines that looked like following:

```
run "${JAVAC}" -classpath "${classpath}" -sourcepath "${sourcepath}" \
      -d "${output}/classes" -source "$JAVA_VERSION" -target "$JAVA_VERSION" \
      -encoding UTF-8 "@${paramfile}"
```

and appended `-J-Xmx500M` to the last line so that the whole lines would look like:

```
run "${JAVAC}" -classpath "${classpath}" -sourcepath "${sourcepath}" \
      -d "${output}/classes" -source "$JAVA_VERSION" -target "$JAVA_VERSION" \
      -encoding UTF-8 "@${paramfile}" -J-Xmx500M
```

It was for enlarging the max heap size of Java.

## c. compile

After that, started building with:

```bash
$ chmod u+w ./* -R
$ ./compile.sh
```

The compilation took about 30 minutes.

## d. install

After the compilation had finished, I could find the compiled binary in `output` directory.

Copied it into `/usr/local/bin` directory:

```bash
$ sudo cp output/bazel /usr/local/bin/
```

# 3. Build libtensorflow.so

(I referenced [this document](https://github.com/samjabrahams/tensorflow-on-raspberry-pi/blob/master/GUIDE.md) for following processes)

## a. download

Got the tensorflow go code with:

```bash
$ go get -d github.com/tensorflow/tensorflow/tensorflow/go
```

## b. edit files

In the downloaded directory, I replaced `lib64` to `lib` in the files with:

```bash
$ cd ${GOPATH}/src/github.com/tensorflow/tensorflow
#$ git fetch --all --tags --prune
#$ git co tags/v1.0.0
$ grep -Rl 'lib64' | xargs sed -i 's/lib64/lib/g'
```

Raspberry Pi still runs on 32bit OS, so it needed to be changed like this.

After that, I commented `#define IS_MOBILE_PLATFORM` in `tensorflow/core/platform/platform.h`:

```c
// Since there's no macro for the Raspberry Pi, assume we're on a mobile
// platform if we're compiling for the ARM CPU.
//#define IS_MOBILE_PLATFORM	// <= commented this line
```

If it is not commented out, bazel will build for mobile platforms like iOS or Android, not Raspberry Pi.

To do this easily, just run:

```bash
$ sed -i "s|#define IS_MOBILE_PLATFORM|//#define IS_MOBILE_PLATFORM|g" tensorflow/core/platform/platform.h
```

Finally, it was time to build tensorflow.

## c. build and install

Started building `libtensorflow.so` with:

```bash
$ ./configure
# (=> I answered to some questions here)
$ bazel build -c opt --copt="-mfpu=neon-vfpv4" --copt="-funsafe-math-optimizations" --copt="-ftree-vectorize" --copt="-fomit-frame-pointer" --jobs 1 --local_resources 1024,1.0,1.0 --verbose_failures --genrule_strategy=standalone --spawn_strategy=standalone //tensorflow:libtensorflow.so
```

I could tweak the **--local_resources** option as [this bazel manual](https://bazel.build/versions/master/docs/bazel-user-manual.html#flag--local_resources),

but if set too agressively, bazel could freeze or even crash with error messages like:

```
Process exited with status 4.
gcc: internal compiler error: Killed (program cc1plus)
```

If this happens, just restart the build. It will resume from the point where it crashed.

My Pi became unresponsive many times, but I kept it going on.

...

After a long time of struggle, (it took over 7 days for me...)

I finally got `libtensorflow.so` in `bazel-bin/tensorflow/`.

So I copied it into `/usr/local/lib/`:

```bash
$ sudo cp ./bazel-bin/tensorflow/libtensorflow.so /usr/local/lib/
$ sudo chmod 644 /usr/local/lib/libtensorflow.so
$ sudo ldconfig
```

All done. Time to test!

# 4. Go Test

I ran a test for validating the installation:

```bash
$ go test github.com/tensorflow/tensorflow/tensorflow/go
```

```
2017-03-15 07:46:27.000618: I tensorflow/cc/saved_model/loader.cc:195] Loading SavedModel from: ../cc/saved_model/testdata/half_plus_two/00000123
2017-03-15 07:46:27.151865: I tensorflow/cc/saved_model/loader.cc:114] Restoring SavedModel bundle.
2017-03-15 07:46:27.162955: I tensorflow/cc/saved_model/loader.cc:149] Running LegacyInitOp on SavedModel bundle.
2017-03-15 07:46:27.166851: I tensorflow/cc/saved_model/loader.cc:239] Loading SavedModel: success. Took 166107 microseconds.
PASS
ok      github.com/tensorflow/tensorflow/tensorflow/go  0.688s
```

Ok, it works!

# 5. Further Test

Wanted to see a simple go program running, so I wrote:

```go
// sample.go
package main

import (
    "fmt"

    tf "github.com/tensorflow/tensorflow/tensorflow/go"
)

// Sorry - I don't have a good example yet :-P
func main() {
    tensor, _ := tf.NewTensor(int64(42))

    if v, ok := tensor.Value().(int64); ok {
        fmt.Printf("The answer is %v\n", v)
    }
}
```

and ran it with `go run sample.go`:

```
The answer is 42
```

See the result?

From now on, I can write tensorflow applications in go, on Raspberry Pi! :-)

----

# 99. Wrap-up

Installing TensorFlow on Raspberry Pi is not easy yet.
([There's a kind project](https://github.com/samjabrahams/tensorflow-on-raspberry-pi) which makes it super easy though!)

Building libtensorflow.so is a lot more difficult, because it takes too much time.

But it is worth trying; managing TensorFlow graphs in golang will be handy for people who don't love python - just like me.

----

# 999. If you need one,

Do you need the compiled file? Good, take it [here](https://github.com/meinside/libtensorflow.so-raspberrypi/releases).

I cannot promise, but will try keeping it up-to-date whenever a newer version of tensorflow comes out.

