
## HTTPS是如何加密的？

[彻底搞懂HTTPS的加密原理](https://zhuanlan.zhihu.com/p/43789231)

---

## LetsEncrypt证书生成的4个文件都是什么用途？

| 文件名 | 文件作用 |
| --- | --- |
| cert.pem | 服务端证书 |
| chain.pem | 浏览器需要的所有证书但不包括服务端证书，比如根证书和中间证书 |
| fullchain.pem | 包括了cert.pem和chain.pem的内容 |
| privkey.pem | 证书的私钥 |

---

## crt、pem、pfx、cer、key 作用及区别

X.509 证书，可能有不同的编码格式，目前有以下两种编码格式：

 - PEM - Privacy Enhanced Mail：文本格式，以“-----BEGIN...”开头，“-----END...”结尾，内容是 BASE64 编码。
 - DER - Distinguished Encoding Rules：二进制格式，不可读。

虽然我们已经知道有 PEM 和 DER 这两种编码格式，但文件扩展名并不一定就叫“PEM”或者“DER”，常见的扩展名除了 PEM 和 DER 还有以下这些，它们除了编码格式可能不同之外，内容也有差别，但大多数都能相互转换编码格式。

 - CRT：CRT 应该是 certificate 的三个字母，其实还是证书的意思。常见于 UNIX 系统，有可能是 PEM 编码，也有可能是 DER 编码，大多数应该是 PEM 编码。
 - CER：还是 certificate，还是证书。常见于 Windows 系统，同样的可能是 PEM 编码，也可能是 DER 编码，大多数应该是 DER 编码。
 - KEY：通常用来存放一个公钥或者私钥，并非 X.509 证书。编码同样的，可能是 PEM，也可能是 DER。
 - CSR：Certificate，Signing Request，即证书签名请求。这个并不是证书，而是向权威证书颁发机构获得签名证书的申请，其核心内容是一个公钥（当然还附带了一些别的信息）。在生成这个申请的时候，同时也会生成一个私钥，私钥要自己保管好。做过 iOS APP 的朋友都应该知道是，怎么向苹果申请开发者证书的吧。