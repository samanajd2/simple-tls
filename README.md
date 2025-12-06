# simple-tls

Simple and easy to use TCP Connect repeater。Can add a layer to the original data stream TLS。Support through gRPC transmission。

---

## Parameters

```text
      Client listening address               Server listening address
           |                            |
|client|-->|simple-tls client|--TLS1.3-->|simple-tls Server|-->|final destination|
                                        |                     |   
                                   client destination address     Server destination address  

# Common parameters
  -b string
      [Host:Port] (required) listening address。
  -d string
      [Host:Port] (required) destination address。
  -grpc
      Use gRPC Agreement。The client and server must be consistent。
  -grpc-path string
      (Optional) gRPC service path。The client and server must be consistent。

# client parameters
# e.g. simple-tls -b 127.0.0.1:1080 -d your_server_ip:1080 -n your.server.name

  -n string
      Server certificate name。Used to verify the validity of the server's certificate。also used as SNI。
  -no-verify
      The client will not verify the validity of the server's certificate。(Certificate chain verification)
  -ca string
      Used to verify the server's certificate CA certificate file。(Use system certificate pool by default)
  -cert-hash string
      server certificate hash。(Server certificate lock)
      tips: Use -hash-cert The command can generate the certificate hash

# Server parameters
# e.g. simple-tls -b :1080 -d 127.0.0.1:12345 -s -key /path/to/your/key -cert /path/to/your/cert
# Certificate format must be PEM (base64).
# -cert and -key can be left blank at the same time, and a temporary certificate will be generated in memory. The domain name of the certificate is random by default, but can also be taken from the `-n` parameter.
# e.g. simple-tls -b :1080 -d 127.0.0.1:12345 -s -n my.test.domain

  -s    
      (required) Run as server。
  -cert string
      Certificate path。
  -key string
      key path。

# Other general parameters

  -t int
      Connection idle timeout，Unit second (Default300)。
  -outbound-buf int
      Set up outbound tcp rw socket buf。
  -inbound-buf    
      Set up inbound tcp rw socket buf。

# command

  -gen-cert
      Generate a key length of 256 的 ECC Certificate to current directory。
      certificate dns name available `-n` Settings。The default is a random string。
      available `-template` Specify template certificate。In addition to key parameters such as keys，All other parameters will be copied from the template certificate。
      available `-cert` 和 `-key` Specify certificate output location。(The default is the current directory and the file name is the certificate. dns name)
      e.g. simple-tls -gen-cert -n my.domain
      A certificate will be generated my.domain.cert and key my.domain.key Two files to the current directory。
  -hash-cert
      showing certificate hash 值。(for client -cert-hash)
      e.g. simple-tls -hash-cert ./my.cert
  -v
      Show current program version
```

## How to use it quickly when the server does not have a valid certificate 

Server uses temporary certificate，The client does not do any verification。This solution can be used when the underlying connection has security measures。

```shell
# If -cert and -key on the server are left blank at the same time, a temporary certificate will be generated in memory.
simple-tls -b :1080 -d 127.0.0.1:12345 -s -n my.cert.domain
# The client disables certificate chain verification.
simple-tls -b :1080 -d your.server.address:1080 -n my.cert.domain -no-verify
```

Server uses fixed certificate，Client use hash Verify server certificate (Certificate pinning)。

```shell
# The server generates a certificate.
simple-tls -gen-cert -n my.cert.domain
# Then display the hash of the certificate. e.g. 8910fe28d2fb40398a...
simple-tls -hash-cert ./my.cert.domain.cert
# Use this certificate to start the server
simple-tls -b :1080 -d 127.0.0.1:12345 -s -key ./my.cert.domain.key -cert ./my.cert.domain.cert
# The client disables certificate chain verification but enables certificate hash verification.
simple-tls -b :1080 -d your.server.address:1080 -n my.cert.domain -no-verify -cert-hash 8910fe28d2fb40398a...
```

## Used as SIP003 plug-in

support shadowsocks 的 [SIP003](https://shadowsocks.org/en/wiki/Plugin.html) plug-in protocol. The shadowsocks main program will automatically set the listening address `-b` and the destination address `-d`.

以 [shadowsocks-rust](https://github.com/shadowsocks/shadowsocks-rust) for example:

```shell
ssserver -c config.json --plugin simple-tls --plugin-opts "s;key=/path/to/your/key;cert=/path/to/your/cert"
sslocal -c config.json --plugin simple-tls --plugin-opts "n=your.server.certificates.dnsname"
```

### Android SIP003 plug-in

simple-tls-android 是 [shadowsocks-android](https://github.com/shadowsocks/shadowsocks-android) plugin with GUI. Currently shipped with simple-tls. You can download the apk common to all platforms from the release interface.

simple-tls-android The source code is in [here](https://github.com/IrineSistiana/simple-tls-android) 。

### Beta version

simple-tls Compatibility between versions is currently not guaranteed。
