---
layout: post
title: TensorFlow and Go on Raspberry Pi
tags:
- raspberry pi
- golang
- tensorflow
published: true
---

This is a guide which I ran through for building `libtensorflow` on Raspberry Pi.

----

# 0. Used Hardwares and Softwares

All steps were taken on my **Raspberry Pi 3 B** model with:

* Minimum GPU memory allocated (16MB)
* **1GB** of swap memory
* External USB HDD (as root partition)

and software versions were:

* Raspbian (Stretch) / gcc 6.3.0
* Tensorflow 1.3.0
* Protobuf 3.1.0
* Bazel 0.5.1

Before the beginning, I had to install dependencies:

### for protobuf

```bash
$ sudo apt-get install autoconf automake libtool
```

### for bazel

```bash
$ sudo apt-get install pkg-config zip g++ zlib1g-dev unzip oracle-java8-jdk
```

----

# 1. Install Protobuf

I cloned the protobuf's repository:

```bash
$ git clone https://github.com/google/protobuf.git
```

and started building:

```bash
$ cd protobuf
$ git checkout v3.1.0
$ ./autogen.sh
$ ./configure
$ make -j 4
$ sudo make install
$ sudo ldconfig
```

It took less than an hour to finish.

I could see the version of installed protobuf with:

```bash
$ protoc --version
```

```
libprotoc 3.1.0
```

# 2. Install Bazel

## a. download

I got a zip file of bazel from [here](https://github.com/bazelbuild/bazel/releases) and unzipped it:

```bash
$ wget https://github.com/bazelbuild/bazel/releases/download/0.5.1/bazel-0.5.1-dist.zip
$ unzip -d bazel bazel-0.5.1-dist.zip
```

## b. edit bootstrap files

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

It also took about an hour.

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

In the downloaded directory, I checked out the latest tag and replaced `lib64` to `lib` in the files with:

```bash
$ cd ${GOPATH}/src/github.com/tensorflow/tensorflow
$ git fetch --all --tags --prune
$ git checkout tags/v1.3.0
$ grep -Rl 'lib64' | xargs sed -i 's/lib64/lib/g'
```

Raspberry Pi still runs on 32bit OS, so they had to be changed like this.

After that, I commented `#define IS_MOBILE_PLATFORM` out in `tensorflow/core/platform/platform.h`:

```c
// Since there's no macro for the Raspberry Pi, assume we're on a mobile
// platform if we're compiling for the ARM CPU.
//#define IS_MOBILE_PLATFORM	// <= commented this line
```

If it is not commented out, bazel will build for mobile platforms like `iOS` or `Android`, not Raspberry Pi.

To do this easily, just run:

```bash
$ sed -i "s|#define IS_MOBILE_PLATFORM|//#define IS_MOBILE_PLATFORM|g" tensorflow/core/platform/platform.h
```

Finally, it was time to configure and build tensorflow.

## c. configure and build

```bash
$ ./configure
```

I had to answer to some questions here.

Then I started building `libtensorflow.so` with:

```bash
$ bazel build -c opt --copt="-mfpu=neon-vfpv4" --copt="-funsafe-math-optimizations" --copt="-ftree-vectorize" --copt="-fomit-frame-pointer" --jobs 1 --local_resources 1024,1.0,1.0 --verbose_failures --genrule_strategy=standalone --spawn_strategy=standalone //tensorflow:libtensorflow.so
```

My Pi became unresponsive many times during this process, but I kept it going on.

## d. install

After a long time of struggle, (it took nearly 7 hours for me!)

I finally got `libtensorflow.so` compiled in `bazel-bin/tensorflow/`.

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

then I could see:

```
ok      github.com/tensorflow/tensorflow/tensorflow/go  0.350s
```

Ok, it works!

**Edit**: As [this instruction](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/go#generate-wrapper-functions-for-ops) says, I had to regenerate operations before the test:

```bash
$ go generate github.com/tensorflow/tensorflow/tensorflow/go/op
```

# 5. Further Test

I wanted to see a simple go program running, so I wrote this code:

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
		fmt.Printf("The answer to the life, universe, and everything: %v\n", v)
	}
}
```

and ran it with `go run sample.go`:

```
The answer to the life, universe, and everything: 42
```

See the result?

From now on, I can write tensorflow applications in go, on Raspberry Pi! :-)

----

# 98. Trouble shooting

## Build failure due to a problem with Eigen

Back in the day with [Tensorflow 1.2.0](https://github.com/tensorflow/tensorflow/releases/tag/v1.2.0), I encountered [this issue](https://github.com/tensorflow/tensorflow/issues/9697) while building, but it's still not fixed yet in [1.3.0](https://github.com/tensorflow/tensorflow/releases/tag/v1.3.0).

So I had to work around this problem again by editing `tensorflow/workspace.bzl` from:

```
native.new_http_archive(
	name = "eigen_archive",
	urls = [
		"http://mirror.bazel.build/bitbucket.org/eigen/eigen/get/f3a22f35b044.tar.gz",
		"https://bitbucket.org/eigen/eigen/get/f3a22f35b044.tar.gz",
	],
	sha256 = "ca7beac153d4059c02c8fc59816c82d54ea47fe58365e8aded4082ded0b820c4",
	strip_prefix = "eigen-eigen-f3a22f35b044",
	build_file = str(Label("//third_party:eigen.BUILD")),
)
```

to:

```
native.new_http_archive(
	name = "eigen_archive",
	urls = [
		"http://mirror.bazel.build/bitbucket.org/eigen/eigen/get/d781c1de9834.tar.gz",
		"https://bitbucket.org/eigen/eigen/get/d781c1de9834.tar.gz",
	],
	sha256 = "a34b208da6ec18fa8da963369e166e4a368612c14d956dd2f9d7072904675d9b",
	strip_prefix = "eigen-eigen-d781c1de9834",
	build_file = str(Label("//third_party:eigen.BUILD")),
)
```

and starting again from the beginning:

```bash
$ bazel clean
$ ./configure
$ bazel build -c opt --copt="-mfpu=neon-vfpv4" --copt="-funsafe-math-optimizations" --copt="-ftree-vectorize" --copt="-fomit-frame-pointer" --jobs 1 --local_resources 1024,1.0,1.0 --verbose_failures --genrule_strategy=standalone --spawn_strategy=standalone //tensorflow:libtensorflow.so

...
```

Then I could build it without further problems.

I hope it would be fixed on future releases.

----

# 99. Wrap-up

Installing TensorFlow on Raspberry Pi is not easy yet.
(There's [a kind project](https://github.com/samjabrahams/tensorflow-on-raspberry-pi) which makes it super easy though!)

Installing `libtensorflow.so` is a lot more difficult, because it takes too much time to build it.

But it is worth trying; managing TensorFlow graphs in golang will be handy for people who don't love python - just like me.

----

# 999. If you need one,

You don't have time to build it yourself, but still need the compiled file?

Good, take it [here](https://github.com/meinside/libtensorflow.so-raspberrypi/releases).

I cannot promise, but will try keeping it up-to-date whenever a newer version of tensorflow comes out.

