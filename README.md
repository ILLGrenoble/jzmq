What is JZMQ?
-------------

This is the Java language binding for libzmq (aka ZeroMQ, 0MQ).

[![Build Status](https://travis-ci.org/zeromq/jzmq.png?branch=master)](https://travis-ci.org/zeromq/jzmq)

The latest [javadocs](http://zeromq.github.com/jzmq/javadocs/).

Building and Installing JZMQ
----------------------------

To build you need to have the libzmq library already installed, then you run:

```bash
cd jzmq-jni/
./autogen.sh
./configure
make
make install
cd ..
mvn package
```

If you hope to install to your local maven, then you should run:

```
mvn install -Dgpg.skip=true
```

Building Windows 64bit with CMake & vcpkg
-----------------------------------------

It is recommended to follow these steps with *vcpkg* and a *Visual C++* compiler.

Set up *vcpkg* as described in [Tutorial: Install and use packages with CMake](https://learn.microsoft.com/en-us/vcpkg/get_started/get-started)
then install the *ZeroMQ* vcpkg package:

```
vcpkg install zeromq
```

Clone this repository and go to the jzmq-jni directory:

```
cd jmzq-jni
```

Configure the build with CMake using the toolchain provided by vcpkg after finding its path:

```
cmake -B build -S . "-DCMAKE_TOOLCHAIN_FILE=\path\to\vcpkg\vcpkg\scripts\buildsystems\vcpkg.cmake"
```

Build the library:

```
cmake --build build --config Release
```

The file jzmq.dll should be present in *build\lib\Release* along with the ZeroMQ dll provided by vcpkg.
To run a Java program using JZMQ, set the *%Path%* so that the JZMQ and ZeroMQ dlls are found.


Avoiding JNI
------------

JZMQ uses JNI to wrap libzmq for the best performance. If performance isn't your primary goal, look at the [JeroMQ](https://github.com/zeromq/jeromq) project, which is a pure Java implementation that provides an identical API to JZMQ, and uses the same protocol.

Building Packages
-----------------

To build a Debian package, run:

```bash
$ dpkg-buildpackage -rfakeroot
```

To build an RPM package, run:

```bash
$ rpmbuild -tb jzmq-X.Y.Z.tar.gz
```

Where X.Y.Z is replaced with the version that you've downloaded.

If configure can't find your libzmq installation, you can tell it where to look, using e.g. `--with-zeromq=/usr/local`.

You may want to take a look at http://www.zeromq.org/docs:tuning-zeromq for additional hints.

For more information, refer to the ØMQ website at http://www.zeromq.org/.

On Mac OS X you may need to compile and make install pkg-config if configure fails with "syntax error near unexpected token newline".   
See http://stackoverflow.com/questions/3522248/how-do-i-compile-jzmq-for-zeromq-on-osx for details.   

You may also need to symlink the header files of your standard Java installation (e.g. `/Developer/SDKs/MacOSX10.6.sdk/System/Library/Frameworks/JavaVM.framework/Versions/CurrentJDK/Headers/*.h`) into a suitable directory (e.g. `/usr/local/include`) and point the `JAVA_HOME` environment variable to the parent directory (e.g.`/usr/local`).

## Acknowledgements

YourKit is kindly supporting ZeroMQ project with its full-featured [Java Profiler](http://www.yourkit.com/java/profiler/index.jsp).

Copying
-------

Free use of this software is granted under the terms of the GNU Lesser General
Public License (LGPL). For details see the files `COPYING` and `COPYING.LESSER`
included with the Java binding for ØMQ.
