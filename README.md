https://bitcointalk.org/index.php?topic=149479.0 

Making a copy if the world stops bitcointalk 

	
Building headless Bitcoin and Bitcoin-qt on Windows
March 05, 2013, 10:26:41 PM
Merited by neiros (1), cinnamon_carter (1), fisheater (1), ItaCraft (1)
 #1
Hi all, I recently went through the process of building bitcoind on windows.
I found the official build-msw.txt to be a bit lacking, so I thought that documenting the steps here on the forums could save some time to people wanting to compile their own windows binary. Wink

Please note this is mostly for testing purposes. Always use official executables on production environments.
The following instructions are intended for use with version 0.9.4. See additional notes if compiling 0.10rc4 or an older 0.8.6 branch.

1. Prepare your build system.
I strongly suggest setting up a clean windows virtual machine via Virtualbox or similar.

1.1 Install msys shell:
http://sourceforge.net/projects/mingw/files/Installer/mingw-get-setup.exe/download
From MinGW installation manager -> All packages -> MSYS
mark the following for installation:

msys-base-bin
msys-autoconf-bin
msys-automake-bin
msys-libtool-bin

then click on Installation -> Apply changes

Make sure no mingw packages are checked for installation or present from a previous install. Only the above msys packages should be installed. Also make sure that msys-gcc and msys-w32api packages are not installed.

1.2 Install MinGW-builds project toolchain:
Download http://sourceforge.net/projects/mingw-w64/files/Toolchains%20targetting%20Win32/Personal%20Builds/mingw-builds/4.9.2/threads-posix/dwarf/i686-4.9.2-release-posix-dwarf-rt_v3-rev1.7z/download
and unpack it to C:\

1.3. Ensure that mingw-builds bin folder is set in your PATH environment variable. On Windows 7 your path should look something like:
Code:
C:\mingw32\bin;%SystemRoot%\system32;%SystemRoot%;%SystemRoot%\System32\Wbem;%SYSTEMROOT%\System32\WindowsPowerShell\v1.0\

1.4 Additional checks:
C:\MinGW\bin should contain nothing but mingw-get.exe.
Your gcc -v output should be:
Code:
$ gcc -v
Using built-in specs.
COLLECT_GCC=c:\mingw32\bin\gcc.exe
COLLECT_LTO_WRAPPER=c:/mingw32/bin/../libexec/gcc/i686-w64-mingw32/4.9.2/lto-wrapper.exe
Target: i686-w64-mingw32
Configured with: ../../../src/gcc-4.9.2/configure --host=i686-w64-mingw32 --build=i686-w64-mingw32 --target=i686-w64-mingw32 --prefix=/mingw32 --with-sysroot=/c/mingw492/i686-492-posix-dwarf-rt_v3-rev1/mingw32 --with-gxx-include-dir=/mingw32/i686-w64-mingw32/include/c++ --enable-shared --enable-static --disable-multilib --enable-languages=ada,c,c++,fortran,objc,obj-c++,lto --enable-libstdcxx-time=yes --enable-threads=posix --enable-libgomp --enable-libatomic --enable-lto --enable-graphite --enable-checking=release --enable-fully-dynamic-string --enable-version-specific-runtime-libs --disable-sjlj-exceptions --with-dwarf2 --disable-isl-version-check --disable-cloog-version-check --disable-libstdcxx-pch --disable-libstdcxx-debug --enable-bootstrap --disable-rpath --disable-win32-registry --disable-nls --disable-werror --disable-symvers --with-gnu-as --with-gnu-ld --with-arch=i686 --with-tune=generic --with-libiconv --with-system-zlib --with-gmp=/c/mingw492/prerequisites/i686-w64-mingw32-static --with-mpfr=/c/mingw492/prerequisites/i686-w64-mingw32-static --with-mpc=/c/mingw492/prerequisites/i686-w64-mingw32-static --with-isl=/c/mingw492/prerequisites/i686-w64-mingw32-static --with-cloog=/c/mingw492/prerequisites/i686-w64-mingw32-static --enable-cloog-backend=isl --with-pkgversion='i686-posix-dwarf-rev1, Built by MinGW-W64 project' --with-bugurl=http://sourceforge.net/projects/mingw-w64 CFLAGS='-O2 -pipe -I/c/mingw492/i686-492-posix-dwarf-rt_v3-rev1/mingw32/opt/include -I/c/mingw492/prerequisites/i686-zlib-static/include -I/c/mingw492/prerequisites/i686-w64-mingw32-static/include' CXXFLAGS='-O2 -pipe -I/c/mingw492/i686-492-posix-dwarf-rt_v3-rev1/mingw32/opt/include -I/c/mingw492/prerequisites/i686-zlib-static/include -I/c/mingw492/prerequisites/i686-w64-mingw32-static/include' CPPFLAGS= LDFLAGS='-pipe -L/c/mingw492/i686-492-posix-dwarf-rt_v3-rev1/mingw32/opt/lib -L/c/mingw492/prerequisites/i686-zlib-static/lib -L/c/mingw492/prerequisites/i686-w64-mingw32-static/lib -Wl,--large-address-aware'
Thread model: posix
gcc version 4.9.2 (i686-posix-dwarf-rev1, Built by MinGW-W64 project)


2. Download, unpack and build required dependencies.
I'll save them in c:\deps folder.

2.1 OpenSSL: http://www.openssl.org/source/openssl-1.0.1l.tar.gz
From a MinGw shell (C:\MinGW\msys\1.0\msys.bat), unpack the source archive with tar (this will avoid symlink issues) then configure and make:
Code:
cd /c/deps/
tar xvfz openssl-1.0.1l.tar.gz
cd openssl-1.0.1l
./Configure no-zlib no-shared no-dso no-krb5 no-camellia no-capieng no-cast no-cms no-dtls1 no-gost no-gmp no-heartbeats no-idea no-jpake no-md2 no-mdc2 no-rc5 no-rdrand no-rfc3779 no-rsax no-sctp no-seed no-sha0 no-static_engine no-whirlpool no-rc2 no-rc4 no-ssl2 no-ssl3 mingw
make

2.2 Berkeley DB: http://download.oracle.com/berkeley-db/db-4.8.30.NC.tar.gz
We'll use version 4.8 to preserve binary wallet compatibility.
From a MinGW shell unpack the source archive, configure and make:
Code:
cd /c/deps/
tar xvfz db-4.8.30.NC.tar.gz
cd db-4.8.30.NC/build_unix
../dist/configure --enable-mingw --enable-cxx --disable-shared --disable-replication
make

2.3 Boost: http://sourceforge.net/projects/boost/files/boost/1.57.0/
Download either the zip or the 7z archive, unpack boost inside your C:\deps folder, then bootstrap and compile from a Windows command prompt:
Code:
cd C:\deps\boost_1_57_0\
bootstrap.bat mingw
b2 --build-type=complete --with-chrono --with-filesystem --with-program_options --with-system --with-thread toolset=gcc variant=release link=static threading=multi runtime-link=static stage

This will compile the required boost libraries and put them into the stage folder (C:\deps\boost_1_57_0\stage).
Note: make sure you don't use tarballs, as unix EOL markers can break batch files.

2.4 Miniupnpc: http://miniupnp.free.fr/files/download.php?file=miniupnpc-1.9.20150206.tar.gz
Unpack Miniupnpc to C:\deps, rename containing folder from "miniupnpc-1.9.20150206" to "miniupnpc" then from a Windows command prompt:
Code:
cd C:\deps\miniupnpc
mingw32-make -f Makefile.mingw init upnpc-static

2.5 protoc and libprotobuf:
Download https://github.com/google/protobuf/releases/download/v2.6.1/protobuf-2.6.1.tar.gz
Then from msys shell
Code:
tar xvfz protobuf-2.6.1.tar.gz
cd /c/deps/protobuf-2.6.1
configure --disable-shared
make

2.6 qrencode:
Download and unpack http://download.sourceforge.net/libpng/libpng-1.6.16.tar.gz inside your deps folder then configure and make:
Code:
cd /c/deps/libpng-1.6.16
configure --disable-shared
make
cp .libs/libpng16.a .libs/libpng.a

Download and unpack http://fukuchi.org/works/qrencode/qrencode-3.4.4.tar.gz inside your deps folder then configure and make:
Code:
cd /c/deps/qrencode-3.4.4

LIBS="../libpng-1.6.16/.libs/libpng.a ../../mingw32/i686-w64-mingw32/lib/libz.a" \
png_CFLAGS="-I../libpng-1.6.16" \
png_LIBS="-L../libpng-1.6.16/.libs" \
configure --enable-static --disable-shared --without-tools

make

2.7 Qt 5 libraries:
Qt must be configured with ssl and zlib support.
Download and unpack Qt base and tools sources:
http://download.qt-project.org/official_releases/qt/5.3/5.3.2/submodules/qtbase-opensource-src-5.3.2.7z
http://download.qt-project.org/official_releases/qt/5.3/5.3.2/submodules/qttools-opensource-src-5.3.2.7z
Then from a windows command prompt (note that the following assumes qtbase has been unpacked to C:\Qt\5.3.2 and qttools have been unpacked to C:\Qt\qttools-opensource-src-5.3.2):
Code:
set INCLUDE=C:\deps\libpng-1.6.16;C:\deps\openssl-1.0.1l\include
set LIB=C:\deps\libpng-1.6.16\.libs;C:\deps\openssl-1.0.1l

cd C:\Qt\5.3.2
configure.bat -release -opensource -confirm-license -static -make libs -no-sql-sqlite -no-opengl -system-zlib -qt-pcre -no-icu -no-gif -system-libpng -no-libjpeg -no-freetype -no-angle -no-vcproj -openssl -no-dbus -no-audio-backend -no-wmf-backend -no-qml-debug

mingw32-make

set PATH=%PATH%;C:\Qt\5.3.2\bin

cd C:\Qt\qttools-opensource-src-5.3.2
qmake qttools.pro
mingw32-make

Note: consider using -j switch with mingw32-make to speed up compilation process. On a quad core -j4 or -j5 should give the best results.


3. Download and unpack Bitcoin 0.9.4 from git https://github.com/bitcoin/bitcoin/archive/v0.9.4.zip

From msys shell configure and make bitcoin:
Code:
cd /c/bitcoin-0.9.4

./autogen.sh

CPPFLAGS="-I/c/deps/db-4.8.30.NC/build_unix \
-I/c/deps/openssl-1.0.1l/include \
-I/c/deps \
-I/c/deps/protobuf-2.6.1/src \
-I/c/deps/libpng-1.6.16 \
-I/c/deps/qrencode-3.4.4" \
LDFLAGS="-L/c/deps/db-4.8.30.NC/build_unix \
-L/c/deps/openssl-1.0.1l \
-L/c/deps/miniupnpc \
-L/c/deps/protobuf-2.6.1/src/.libs \
-L/c/deps/libpng-1.6.16/.libs \
-L/c/deps/qrencode-3.4.4/.libs" \
BOOST_ROOT=/c/deps/boost_1_57_0 \
./configure \
--disable-upnp-default \
--disable-tests \
--with-qt-incdir=/c/Qt/5.3.2/include \
--with-qt-libdir=/c/Qt/5.3.2/lib \
--with-qt-plugindir=/c/Qt/5.3.2/plugins \
--with-qt-bindir=/c/Qt/5.3.2/bin \
--with-protoc-bindir=/c/deps/protobuf-2.6.1/src

make

strip src/bitcoin-cli.exe
strip src/bitcoind.exe
strip src/qt/bitcoin-qt.exe



Additional notes:
64 bit binaries
64 bit binaries can be compiled by using the following toolchain:
http://sourceforge.net/projects/mingw-w64/files/Toolchains%20targetting%20Win64/Personal%20Builds/mingw-builds/4.9.2/threads-posix/seh/x86_64-4.9.2-release-posix-seh-rt_v3-rev1.7z/download

All dependencies must be rebuilt with the above toolchain.
Openssl should be configured for:
Code:
mingw64
instead of:
Code:
mingw


Bitcoin v0.10rc4
Since v0.10 Bitcoin depends on gmp: https://gmplib.org/download/gmp/gmp-6.0.0a.tar.xz
Compile it in a msys shell
Code:
tar xvf gmp-6.0.0a.tar.xz
cd /c/deps/gmp-6.0.0
./configure --disable-shared
make

Then add relevant CPPFLAGS and LDFLAGS when configuring Bitcoin:
Code:
CPPFLAGS="-I/c/deps/db-4.8.30.NC/build_unix \
-I/c/deps/openssl-1.0.1l/include \
-I/c/deps \
-I/c/deps/protobuf-2.6.1/src \
-I/c/deps/libpng-1.6.16 \
-I/c/deps/qrencode-3.4.4 \
-I/c/deps/gmp-6.0.0" \
LDFLAGS="-L/c/deps/db-4.8.30.NC/build_unix \
-L/c/deps/openssl-1.0.1l \
-L/c/deps/miniupnpc \
-L/c/deps/protobuf-2.6.1/src/.libs \
-L/c/deps/libpng-1.6.16/.libs \
-L/c/deps/qrencode-3.4.4/.libs \
-L/c/deps/gmp-6.0.0/.libs" \
BOOST_ROOT=/c/deps/boost_1_57_0 \
./configure \
--disable-upnp-default \
--disable-tests \
--with-qt-incdir=/c/Qt/5.3.2/include \
--with-qt-libdir=/c/Qt/5.3.2/lib \
--with-qt-plugindir=/c/Qt/5.3.2/plugins \
--with-qt-bindir=/c/Qt/5.3.2/bin \
--with-protoc-bindir=/c/deps/protobuf-2.6.1/src


Compiling with unit tests
In order to build unit tests you will need the following:
-boost
Boost must be configured with --with-test option also.

-hexdump
To generate test data an hexdump like program is needed.
Download and unpack https://github.com/wahern/hexdump then compile hexdump.exe by running: 
Code:
gcc -std=gnu99 -g -O2 -Wall -Wextra -Werror -Wno-unused-variable -Wno-unused-parameter hexdump.c -DHEXDUMP_MAIN -o hexdump.exe
Do not forget to add hexdump folder to your PATH environment variable.

-bitcoin
In order to work with the previously compiled hexdump version Makefile must be patched (src/Makefile.test.include if v0.10, src/Makefile.include if v0.9):
Code:
--- src/Makefile.test.include	Tue Dec 23 20:14:37 2014
+++ src/Makefile.test.include	Sat Jan 10 16:53:56 2015
@@ -111,7 +111,7 @@
 	@$(MKDIR_P) $(@D)
 	@echo "namespace json_tests{" > $@
 	@echo "static unsigned const char $(*F)[] = {" >> $@
-	@$(HEXDUMP) -v -e '8/1 "0x%02x, "' -e '"\n"' $< | $(SED) -e 's/0x  ,//g' >> $@
+	@$(HEXDUMP) -e '8/1 "0x%02x, "' $< | $(SED) -e 's/0x  ,//g' >> $@
 	@echo "};};" >> $@
 	@echo "Generated $@"
 
@@ -119,6 +119,6 @@
 	@$(MKDIR_P) $(@D)
 	@echo "namespace alert_tests{" > $@
 	@echo "static unsigned const char $(*F)[] = {" >> $@
-	@$(HEXDUMP) -v -e '8/1 "0x%02x, "' -e '"\n"' $< | $(SED) -e 's/0x  ,//g' >> $@
+	@$(HEXDUMP) -e '8/1 "0x%02x, "' $< | $(SED) -e 's/0x  ,//g' >> $@
 	@echo "};};" >> $@
 	@echo "Generated $@"

bitcoin must be configured without --disable-tests option.






Additional notes for older Bitcoin 0.8.6 (may be useful for Bitcoin based altcoins)
msys-autoconf, msys-automake and msys-libtool at step 1.1 are not needed. You can skip steps 2.5 and 2.7.
Note that openssl v1.0.1k and later may lead to consesus forks:
http://sourceforge.net/p/bitcoin/mailman/message/33221963/

Compile bitcoind 0.8.6: https://github.com/bitcoin/bitcoin/archive/v0.8.6.zip

With a texteditor edit BOOST_SUFFIX, INCLUDEPATHS and LIBPATHS in your C:\bitcoin-0.8.6\src\makefile.mingw according to your dependencies location:
Code:
BOOST_SUFFIX?=-mgw49-mt-s-1_57

INCLUDEPATHS= \
 -I"$(CURDIR)" \
 -I"/c/deps/boost_1_57_0" \
 -I"/c/deps/db-4.8.30.NC/build_unix" \
 -I"/c/deps/openssl-1.0.1j/include"
 
LIBPATHS= \
 -L"$(CURDIR)/leveldb" \
 -L"/c/deps/boost_1_57_0/stage/lib" \
 -L"/c/deps/db-4.8.30.NC/build_unix" \
 -L"/c/deps/openssl-1.0.1j"

and add -static option to LDFLAGS in makefile.mingw to compile a statically linked executable.
Code:
LDFLAGS=-Wl,--dynamicbase -Wl,--nxcompat -Wl,--large-address-aware -static

makefile.mingw patch:
Code:
--- makefile.mingw	Thu Dec 05 14:11:26 2013
+++ makefile.mingw	Thu Jun 19 20:00:00 2014
@@ -21,15 +21,19 @@
 USE_IPV6:=1
 
 DEPSDIR?=/usr/local
-BOOST_SUFFIX?=-mgw46-mt-sd-1_52
+BOOST_SUFFIX?=-mgw49-mt-s-1_57
 
 INCLUDEPATHS= \
  -I"$(CURDIR)" \
- -I"$(DEPSDIR)/include"
-
+ -I"/c/deps/boost_1_57_0" \
+ -I"/c/deps/db-4.8.30.NC/build_unix" \
+ -I"/c/deps/openssl-1.0.1j/include"
+ 
 LIBPATHS= \
  -L"$(CURDIR)/leveldb" \
- -L"$(DEPSDIR)/lib"
+ -L"/c/deps/boost_1_57_0/stage/lib" \
+ -L"/c/deps/db-4.8.30.NC/build_unix" \
+ -L"/c/deps/openssl-1.0.1j"
 
 LIBS= \
  -l leveldb \
@@ -47,7 +51,7 @@
 DEBUGFLAGS=-g
 CFLAGS=-mthreads -O2 -w -Wall -Wextra -Wformat -Wformat-security -Wno-unused-parameter $(DEBUGFLAGS) $(DEFS) $(INCLUDEPATHS)
 # enable: ASLR, DEP and large address aware
-LDFLAGS=-Wl,--dynamicbase -Wl,--nxcompat -Wl,--large-address-aware
+LDFLAGS=-Wl,--dynamicbase -Wl,--nxcompat -Wl,--large-address-aware -static
 
 TESTDEFS = -DTEST_DATA_DIR=$(abspath test/data)
 

Upnp support is disabled by default. If you want to compile with upnp support set
Code:
USE_UPNP:=1
and add miniupnpc path to INCLUDEPATHS and LIBPATHS:
Code:
INCLUDEPATHS= \
 -I"$(CURDIR)" \
 -I"/c/deps/boost_1_57_0" \
 -I"/c/deps" \
 -I"/c/deps/db-4.8.30.NC/build_unix" \
 -I"/c/deps/openssl-1.0.1j/include"
 
LIBPATHS= \
 -L"$(CURDIR)/leveldb" \
 -L"/c/deps/boost_1_57_0/stage/lib" \
 -L"/c/deps/miniupnpc" \
 -L"/c/deps/db-4.8.30.NC/build_unix" \
 -L"/c/deps/openssl-1.0.1j"

From Msys shell compile bitcoind:
Code:
cd /c/bitcoin-0.8.6/src
make -f makefile.mingw
strip bitcoind.exe


Compile bitcoin-qt 0.8.6 with Qt 4.8: 
Download and unpack http://download.qt-project.org/official_releases/qt/4.8/4.8.6/qt-everywhere-opensource-src-4.8.6.zip
Note that due to a bug in qt 4.8.6 you will need to explicitly enable windows styles: https://bugreports.qt-project.org/browse/QTBUG-38706
Assuming qt sources have been unpacked to C:\Qt\4.8.6, from a windows command prompt:
Code:
cd C:\Qt\4.8.6
configure -release -opensource -confirm-license -static -no-sql-sqlite -no-qt3support -no-opengl -qt-zlib -no-gif -qt-libpng -qt-libmng -no-libtiff -qt-libjpeg -no-dsp -no-vcproj -no-openssl -no-dbus -no-phonon -no-phonon-backend -no-multimedia -no-audio-backend -no-webkit -no-script -no-scripttools -no-declarative -no-declarative-debug -qt-style-windows -qt-style-windowsxp -qt-style-windowsvista -no-style-plastique -no-style-cleanlooks -no-style-motif -no-style-cde -nomake demos -nomake examples
mingw32-make

Note that if you skipped bitcoind compilation or if you have cleaned up your source folder you will need to compile libleveldb.a and libmemenv.a libraries before proceeding.
From msys shell:
Code:
cd /C/bitcoin-0.8.6/src/leveldb
TARGET_OS=NATIVE_WINDOWS make libleveldb.a libmemenv.a


Edit C:\bitcoin-0.8.6\bitcoin-qt.pro with your favourite text editor and add 

dependency library locations:
Code:
# Dependency library locations can be customized with:
#    BOOST_INCLUDE_PATH, BOOST_LIB_PATH, BDB_INCLUDE_PATH,
#    BDB_LIB_PATH, OPENSSL_INCLUDE_PATH and OPENSSL_LIB_PATH respectively

BOOST_LIB_SUFFIX=-mgw49-mt-s-1_57
BOOST_INCLUDE_PATH=C:/deps/boost_1_57_0
BOOST_LIB_PATH=C:/deps/boost_1_57_0/stage/lib
BDB_INCLUDE_PATH=C:/deps/db-4.8.30.NC/build_unix
BDB_LIB_PATH=C:/deps/db-4.8.30.NC/build_unix
OPENSSL_INCLUDE_PATH=C:/deps/openssl-1.0.1j/include
OPENSSL_LIB_PATH=C:/deps/openssl-1.0.1j
MINIUPNPC_INCLUDE_PATH=C:/deps/
MINIUPNPC_LIB_PATH=C:/deps/miniupnpc
QRENCODE_INCLUDE_PATH=C:/deps/qrencode-3.4.4
QRENCODE_LIB_PATH=C:/deps/qrencode-3.4.4/.libs

Comment out genleveldb.commands for win32
Code:
    LIBS += -lshlwapi
    #genleveldb.commands = cd $$PWD/src/leveldb && CC=$$QMAKE_CC CXX=$$QMAKE_CXX TARGET_OS=OS_WINDOWS_CROSSCOMPILE $(MAKE) OPT=\"$$QMAKE_CXXFLAGS $$QMAKE_CXXFLAGS_RELEASE\" libleveldb.a libmemenv.a && $$QMAKE_RANLIB $$PWD/src/leveldb/libleveldb.a && $$QMAKE_RANLIB $$PWD/src/leveldb/libmemenv.a
}

flags for static build:
Code:
CONFIG += static

Code:
win32:QMAKE_LFLAGS *= -Wl,--large-address-aware -static

bitcoin-qt.pro patch:
Code:
--- bitcoin-qt.pro	Thu Dec 05 14:11:26 2013
+++ bitcoin-qt.pro	Thu Jun 19 20:00:00 2014
@@ -7,6 +7,7 @@
 DEFINES += QT_GUI BOOST_THREAD_USE_LIB BOOST_SPIRIT_THREADSAFE
 CONFIG += no_include_pwd
 CONFIG += thread
+CONFIG += static
 
 # for boost 1.37, add -mt to the boost libraries
 # use: qmake BOOST_LIB_SUFFIX=-mt
@@ -17,6 +18,17 @@
 # Dependency library locations can be customized with:
 #    BOOST_INCLUDE_PATH, BOOST_LIB_PATH, BDB_INCLUDE_PATH,
 #    BDB_LIB_PATH, OPENSSL_INCLUDE_PATH and OPENSSL_LIB_PATH respectively
+BOOST_LIB_SUFFIX=-mgw49-mt-s-1_57
+BOOST_INCLUDE_PATH=C:/deps/boost_1_57_0
+BOOST_LIB_PATH=C:/deps/boost_1_57_0/stage/lib
+BDB_INCLUDE_PATH=C:/deps/db-4.8.30.NC/build_unix
+BDB_LIB_PATH=C:/deps/db-4.8.30.NC/build_unix
+OPENSSL_INCLUDE_PATH=C:/deps/openssl-1.0.1j/include
+OPENSSL_LIB_PATH=C:/deps/openssl-1.0.1j
+MINIUPNPC_INCLUDE_PATH=C:/deps/
+MINIUPNPC_LIB_PATH=C:/deps/miniupnpc
+QRENCODE_INCLUDE_PATH=C:/deps/qrencode-3.4.4
+QRENCODE_LIB_PATH=C:/deps/qrencode-3.4.4/.libs
 
 OBJECTS_DIR = build
 MOC_DIR = build
@@ -47,7 +59,7 @@
 # for extra security on Windows: enable ASLR and DEP via GCC linker flags
 win32:QMAKE_LFLAGS *= -Wl,--dynamicbase -Wl,--nxcompat
 # on Windows: enable GCC large address aware linker flag
-win32:QMAKE_LFLAGS *= -Wl,--large-address-aware
+win32:QMAKE_LFLAGS *= -Wl,--large-address-aware -static
 
 # use: qmake "USE_QRCODE=1"
 # libqrencode (http://fukuchi.org/works/qrencode/index.en.html) must be installed for support
@@ -109,7 +121,7 @@
         QMAKE_RANLIB = $$replace(QMAKE_STRIP, strip, ranlib)
     }
     LIBS += -lshlwapi
-    genleveldb.commands = cd $$PWD/src/leveldb && CC=$$QMAKE_CC CXX=$$QMAKE_CXX TARGET_OS=OS_WINDOWS_CROSSCOMPILE $(MAKE) OPT=\"$$QMAKE_CXXFLAGS $$QMAKE_CXXFLAGS_RELEASE\" libleveldb.a libmemenv.a && $$QMAKE_RANLIB $$PWD/src/leveldb/libleveldb.a && $$QMAKE_RANLIB $$PWD/src/leveldb/libmemenv.a
+    #genleveldb.commands = cd $$PWD/src/leveldb && CC=$$QMAKE_CC CXX=$$QMAKE_CXX TARGET_OS=OS_WINDOWS_CROSSCOMPILE $(MAKE) OPT=\"$$QMAKE_CXXFLAGS $$QMAKE_CXXFLAGS_RELEASE\" libleveldb.a libmemenv.a && $$QMAKE_RANLIB $$PWD/src/leveldb/libleveldb.a && $$QMAKE_RANLIB $$PWD/src/leveldb/libmemenv.a
 }
 genleveldb.target = $$PWD/src/leveldb/libleveldb.a
 genleveldb.depends = FORCE

From a windows command prompt configure and make:
Code:
set PATH=%PATH%;C:\Qt\4.8.6\bin
cd C:\bitcoin-0.8.6\
qmake "USE_QRCODE=1" "USE_UPNP=1" "USE_IPV6=1" bitcoin-qt.pro
mingw32-make -f Makefile.Release










Last updated on 08/02/2015
bitcoin 0.9.4
toolchain: i686-4.9.2-release-posix-dwarf-rt_v3-rev1
openssl-1.0.1l
libpng-1.6.16
miniupnpc-1.9.20150206.tar.gz

10/01/2015
compiling unit tests

30/11/2014
toolchain: i686-4.9.2-release-posix-dwarf-rt_v3-rev0
boost_1_57_0
protobuf-2.6.1
libpng-1.6.15
miniupnpc-1.9.20141128
gmp-6.0.0a for current master

28/10/2014
openssl-1.0.1j
libpng-1.6.14
do not build unnecessary openssl ciphers and protocols
use BOOST_ROOT when configuring bitcoin

28/09/2014
toolchain: i686-4.9.1-release-posix-dwarf-rt_v3-rev1
bitcoin 0.9.3
miniupnpc-1.9.20140911 https://github.com/bitcoin/bitcoin/commit/9f7f504efca27c7d390f121410846b45c1732761
qt5.3.2

10/08/2014
update toolchain i686-4.9.1-release-posix-dwarf-rt_v3-rev0
update dependencies openssl-1.0.1i qrencode-3.4.4 qt4.8.6

02/7/2014
qt5.3.1

20/6/2014
bitcoin 0.9.2.1
mingw-builds 4.9.0 rev2
openssl-1.0.1h
qt5.3
libpng-1.6.12
removed older instructions about bitcoin 0.8.6 with qt5

25/5/2014:
update toolchain
added makefile patch commit

23/4/2014:
removed python and activestate perl as they are not needed with current qt config options,
use -openssl-linked instead of -openssl (see src\network\ssl\qsslsocket_openssl_symbols.cpp for more info),
do not compile shared libraries when building dependencies,
libpng 1.6.10.
