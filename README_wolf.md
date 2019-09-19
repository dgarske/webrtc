# WebRTC with wolfSSL

Below describes the steps for building WebRTC with wolfSSL.

## wolfSSL

Support was added in the following wolfSSL pull requests:
* ref 57: https://github.com/wolfSSL/wolfssl/pull/2462
* ref m79: https://github.com/wolfSSL/wolfssl/pull/2585

```sh
git clone https://github.com/wolfSSL/wolfssl.git
cd wolfssl
./autogen.sh
./configure --enable-opensslall --enable-dtls
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
fetch --nohooks webrtc
cd src

# See releases
git branch -r

# wolfssl work is baed on m79
git checkout branch-heads/m79
gclient sync # NOTE: this takes a long time

# get wolfssl branch
git remote add wolf https://github.com/dgarske/webrtc.git
git fetch wolf
git checkout wolf_m79

git rebase-update
gclient sync

gn clean out/debug
gn gen out/debug "--args=rtc_build_wolfssl=true rtc_build_ssl=false rtc_ssl_root=\"/usr/local/include\""
ninja -C out/debug/ -t clean
ninja -C out/debug/ protobuf_lite p2p base jsoncpp

gn clean out/release
gn gen out/release "--args=is_debug=false rtc_build_wolfssl=true rtc_build_ssl=false rtc_ssl_root=\"/usr/local/include\""
ninja -C out/release/ -t clean
ninja -C out/release/ protobuf_lite p2p base jsoncpp
```

## Testing WebRTC

```sh
cd examples/
gn gen out/debug
ninja -C out/debug/ -t clean
ninja -C out/debug/

# NOTE: Must have webcam, otherwise will get assert in debug mode
cd out/debug
./peerconnection_server &
./peerconnection_client
```

## Support

For questions or issue please email support@wolfssl.com
