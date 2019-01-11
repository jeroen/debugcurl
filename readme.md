# Debug libcurl with llvm

First install openssl and nghttp2 from homebrew:

```
brew install nghttp2
brew remove curl
```

Then clone the curl master branch

```
git clone https://github.com/curl/curl
cd curl
```

Then configure with openssl and nghttp2:

```
./buildconf
./configure \
  --with-ssl=/usr/local/opt/openssl \
  --with-ca-bundle=/usr/local/etc/openssl/cert.pem \
  --with-ca-path=/usr/local/etc/openssl/certs \
  --with-nghttp2=/usr/local/opt/nghttp2 \
  --enable-debug \
  --enable-curldebug \
```

Confirm the config status. Then make and install (you really need to install because otherwise LLDB will probably use the system libcurl).

```
make -j8
make install
```

Now build an application:

```
# Should be /usr/local/lib
pkg-config --cflags --libs libcurl 

# Now build
clang -g ./docs/examples/crawler.c -o crawler.exe -lcurl $(pkg-config --cflags --libs libcurl libxml-2.0)
```

Conform that you link against libcurl in `/usr/local`:

```
otool -L crawler.exe
```

And then run the application in LLDB

```
lldb crawler.exe
```

First check `image list` to make sure we loaded the correct `libcurl.dylib`. Then press `r` to start running and `bt` to get a backtrace!

When done uninstall curl:

```
make uninstall
```