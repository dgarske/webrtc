# WebRTC with wolfSSL

Below describes the steps for building WebRTC with wolfSSL.

## wolfSSL

```sh
git clone https://github.com/wolfSSL/wolfssl.git
cd wolfssl
./autogen.sh
./configure --enable-opensslall --enable-keygen --enable-rsapss --enable-aesccm \
    --enable-aesctr --enable-des3 --enable-camellia --enable-curve25519 --enable-ed25519 \
    --enable-dtls --enable-certgen --enable-certreq --enable-certext --enable-tlsv10 \
    CFLAGS="-DWOLFSSL_PUBLIC_MP -DWOLFSSL_DES_ECB"
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

git checkout branch-heads/[branch]
gclient sync # NOTE: this takes a long time
git rebase-update
gclient sync

gn clean out/debug
gn gen out/debug "--args=enable_iterator_debugging=false is_component_build=false rtc_build_wolfssl=true rtc_build_ssl=false rtc_ssl_root=\"/usr/local/include\""
ninja -C out/debug/ -t clean
ninja -C out/debug/ protobuf_lite p2p base jsoncpp

gn clean out/release
gn gen out/release "--args=enable_iterator_debugging=false is_component_build=false is_debug=false rtc_build_wolfssl=true rtc_build_ssl=false rtc_ssl_root=\"/usr/local/include\""
ninja -C out/release/ -t clean
ninja -C out/release/ protobuf_lite p2p base jsoncpp
```

## Testing WebRTC

```sh
cd webrtc/examples/
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
