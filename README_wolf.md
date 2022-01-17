# WebRTC with wolfSSL

Below describes the steps for building WebRTC with wolfSSL.

## wolfSSL

Support was added in the following wolfSSL pull requests:
* ref 57: https://github.com/wolfSSL/wolfssl/pull/2462
* ref m79: https://github.com/wolfSSL/wolfssl/pull/2585
* DTLS SRTP: 

```sh
git clone https://github.com/wolfSSL/wolfssl.git
cd wolfssl
./autogen.sh
./configure --enable-opensslall --enable-dtls --enable-srtp --enable-keying-material
make
make check
sudo make install
```

## Build WebRTC with wolfSSL

Based on instructions:
* https://webrtc.org/native-code/development/

```sh
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
export PATH=`pwd`/depot_tools:"$PATH"
mkdir webrtc-checkout
cd webrtc-checkout

# NOTE: this takes a long time
fetch --nohooks webrtc
cd src

# See releases
git branch -r

# wolfssl work is based on m84
git checkout branch-heads/4147

# NOTE: this takes a long time
gclient sync

# get wolfssl branch
git remote add wolf https://github.com/dgarske/webrtc.git
git fetch wolf
git checkout wolf_m84

# debug build
gn clean out/debug
gn gen out/debug "--args=rtc_build_wolfssl=true rtc_build_ssl=false rtc_ssl_root=\"/usr/local/include\""
ninja -C out/debug/ -t clean
ninja -C out/debug/ protobuf_lite p2p base jsoncpp

# release build
gn clean out/release
gn gen out/release "--args=is_debug=false rtc_build_wolfssl=true rtc_build_ssl=false rtc_ssl_root=\"/usr/local/include\""
ninja -C out/release/ -t clean
ninja -C out/release/ protobuf_lite p2p base jsoncpp
```

## Testing WebRTC

```sh
cd examples/
gn clean out/debug
gn gen out/debug
ninja -C out/debug/ -t clean
ninja -C out/debug/
```

### Unit Tests

```
./out/debug/peerconnection_unittests
...
[==========] 1994 tests from 84 test suites ran. (279776 ms total)
[  PASSED  ] 1994 tests.
```

### Peer Test

NOTE: Must have webcam, otherwise might get assert in debug mode

```
cd out/debug
./peerconnection_server &
./peerconnection_client
```

## Tips

### Pre requisites

```
cd webrtc-checkout/src
./build/install-build-deps.sh

ERROR: lsb_release not found in $PATH
try: sudo apt-get install lsb-release
```

### GLIBC 

`/lib64/libc.so.6: version 'GLIBC_2.18' not found`

Build GLIBC 2.18:

```
wget http://ftp.gnu.org/gnu/glibc/glibc-2.18.tar.gz
tar zxvf glibc-2.18.tar.gz
cd glibc-2.18/
mkdir build
cd build
../configure --prefix=/opt/glibc-2.18
make -j4
sudo make install
```

In build console: `export LD_LIBRARY_PATH=/opt/glibc-2.14/lib`

## Static Build

```sh
# wolfSSL
./configure --enable-opensslall --enable-dtls --enable-srtp --enable-keying-material --enable-static --disable-shared --prefix=/home/`whoami`/wolfssl-install
make
make install

# WebRTC
gn clean out/debug
gn gen out/debug "--args=rtc_build_wolfssl=true rtc_build_ssl=false rtc_ssl_root=\"/home/`whoami`/wolfssl-install/include\""
ninja -C out/debug/ -t clean
ninja -C out/debug/ protobuf_lite p2p base jsoncpp
```

## Support

For questions or issue please email support@wolfssl.com
