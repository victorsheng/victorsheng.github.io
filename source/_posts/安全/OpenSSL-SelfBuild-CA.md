title: 使用OpenSSL自建CA并颁发SSL证书
date: 2018-07-10 14:58:32
tags:
    - 证书
    - 安全
categories:
    - 安全
---
# 申请 X.509 CA 证书的流程
1. 用户先在本地通过 RSA 生成一对公钥 Kp 和密钥 Ks，
2. 然后，用户在本地生成一张 CSR 证书，既 Certificate Signing Request
    - 需要填写诸如你的域名，公司名，部门名称，城市名，地区名，国家名，电子邮件等等证明你身份的信息，
    - 添加用户公钥 Kp
    - 将上述信息通过 Ks 进行签名得到 Scsr
    - 将上述的签名 Scsr、Kp 以及身份信息合并成为一张 CSR 证书 ;
    - TODO: 需要合并 Scsr 吗？这个还需要进一步求证……
3. 申请者通过向 CA 中心或者 RA 中心提交 CSR 证书，申请 X.509 证书；
4. CA 中心用它的密钥 Ks(ca) 对用户提交的 CSR 证书进行签名，将签名和 CSR 合并生成一张 X.509 证书；详细的签名过程，
    - CA 中心通过签名算法(比如 MD5)对 CSR 证书进行签名；
    - 然后通过 Ks(ca) 对上述的签名进行加密，得到加密后的签名；
    - 最后，将加密后的签名和用户提交的 CSR 合并成为一张 X.509 证书；
5. 最后将该 X.509 证书 颁发给用户；


# 操作


## 建立 CA


## 配置openssl.cnf
```
sudo cp /etc/ssl/openssl.cnf ~/ssl/openssl.cnf
```
理论上是可以拷贝过来的.可是mac中的配置过于简单且不完成,所以我们从github中进行下载

https://github.com/openssl/openssl/blob/master/apps/openssl.cnf

修改变量:
```
dir         = /Users/victor/demoCA           # Where everything is kept
```

## 为用户颁发证书
### 密钥生成
```
openssl genrsa -out nginx.key 2048
```

### 生成证书签署请求
```
openssl req -new -key nginx.key -out nginx.csr -config=demoCA/openssl.cnf

输入信息
...
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:GD
Locality Name (eg, city) []:SZ
Organization Name (eg, company) [Internet Widgits Pty Ltd]:COMPANY
Organizational Unit Name (eg, section) []:IT_SECTION
Common Name (e.g. server FQDN or YOUR name) []:your.domain.com
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
...
输入至少4位的密码



```

### 签署证书

```
openssl ca -in nginx.csr -out nginx.crt -config=demoCA/openssl.cnf

确认:
Using configuration from demoCA/openssl.cnf
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number: 1 (0x1)
        Validity
            Not Before: Jul 10 06:43:19 2018 GMT
            Not After : Jul 10 06:43:19 2019 GMT
        Subject:
            countryName               = cn
            stateOrProvinceName       = cn
            organizationName          = cn
            organizationalUnitName    = cn
            commonName                = www.td.com
            emailAddress              = td@qq.com
        X509v3 extensions:
            X509v3 Basic Constraints:
                CA:FALSE
            Netscape Comment:
                OpenSSL Generated Certificate
            X509v3 Subject Key Identifier:
                A0:FF:94:8C:85:19:2C:1F:B1:21:6F:41:72:D5:2F:BC:93:F7:A1:72
            X509v3 Authority Key Identifier:
                DirName:/C=cn/ST=cn/L=cn/O=cn/OU=cn/CN=cn/emailAddress=cn
                serial:F1:EC:A3:82:A6:C1:21:92

Certificate is to be certified until Jul 10 06:43:19 2019 GMT (365 days)
Sign the certificate? [y/n]
```

## 查看证书

![upload successful](/images/pasted-212.png)


 ## 把RSA私钥转换成PKCS8格式(pem和der格式)

`openssl pkcs8 -topk8 -inform PEM -in nginx.key -outform PEM -nocrypt -out nginx_private_key.pem` 
`openssl pkcs8 -topk8 -inform PEM -outform DER -in nginx.key -out nginx_private_key.der -nocrypt`

## 生成RSA公钥(pem和der格式)

`openssl rsa -in nginx.key -pubout -out nginx_public_key.pem`
`openssl rsa -in nginx.key -pubout -outform DER -out nginx_public_key.der`


## PEM转DER格式(PEM是DER格式进行base64编码的格式)
```
# 私钥：PEM --(convert)--> DER
 openssl rsa -inform PEM -in Key0.pem -outform DER -out Key0.der

# 公钥：PEM --(convert)> DER
 openssl rsa -inform PEM -in Key0_pub.pem -pubin -outform DER -out Key0_pub.der
```


# 其他证书操作
- 查看证书的内容：` openssl x509 -in cert.pem -text -noout`
- 吊销证书：` openssl ca -revoke cert.pem -config openssl.cnf`
- 证书吊销列表：` openssl ca -gencrl -out cacert.crl -config openssl.cnf`
- 查看列表内容：` openssl crl -in cacert.crl -text -noout`

# 密钥/公钥格式

## PEM
DER 是密钥的二进制表述格式；

http://fileformats.archiveteam.org/wiki/DER


>Distinguished Encoding Rules (DER) is a binary serialization of ASN.1 format. It is often used for cryptographic data such as certificates, but has other uses.

很明显，DER 就是 ASN.1 的二进制格式；

## DER
PEM 格式既是对 DER 编码转码为 Base64 字符格式；通过解码，将会还原为 DER 格式；

>A PEM file is plain text. It contain one or more objects, such as certificates or keys, which may not all be the same type. Each object is delimited by lines similar to “—–BEGIN …—–” and “—–END …—–”. Data that is not between such lines is ignored, and is sometimes used for comments, or for a human-readable dump of the encoded data.
>
>Following the “BEGIN” and “END” keywords is a name (such as “CERTIFICATE”) that can be used as an identifier for the type of object.
>
>The data between the delimiter lines starts with an optional email-like header section, followed by base64-encoded payload data. After decoding, the payload data is in DER format.

总体而言，PEM 是明文格式，可以包含证书或者是密钥；其内容通常是以类似 “—–BEGIN …—–” 开头 “—–END …—–” 为结尾的这样的格式进行展示的；后续内容也描述到，PEM 格式的内容是 Base64 格式；通过解码，转换为 DER 格式，也就是说，PEM 是建立在 DER 编码之上的；


# PKCS
## PKCS 1 公钥和密钥的格式标准（通过 ASN.1 的格式标准来定义并明文展示）
- 以及为 RSA 进行加密、解密，生成和验证签名等操作定义了基本的算法和编码/补零(padding)的方案
- PCKS #1 定义的都是明文的格式

ASN.1 是一种接口描述性语言，该语言定义了能够进行跨平台、序列化和反序列化的数据格式；它被广泛的用于电子通讯以及计算机网络中，特别是用在密码学的领域；由此可知，ASN.1 定义了一种专用于密码学领域的一种可以进行序列化和反序列化的数据格式；

### 公钥
```
      RSAPublicKey ::= SEQUENCE {
          modulus           INTEGER,  -- n
          publicExponent    INTEGER   -- e
      }

```

### 私钥
```
      RSAPrivateKey ::= SEQUENCE {
          version           Version,
          modulus           INTEGER,  -- n
          publicExponent    INTEGER,  -- e
          privateExponent   INTEGER,  -- d
          prime1            INTEGER,  -- p
          prime2            INTEGER,  -- q
          exponent1         INTEGER,  -- d mod (p-1)
          exponent2         INTEGER,  -- d mod (q-1)
          coefficient       INTEGER,  -- (inverse of q) mod p
          otherPrimeInfos   OtherPrimeInfos OPTIONAL
      }
```

## PKCS 7 被加密消息的格式标准
- PKCS 7 通常在一个 PKI 中用来签名或者加密信息，也通常用于证书的传递
- PKCS 7 主要定义了消息的加密语法标准

CMS 用来描述数据加密的一种封装语法；CMS 可以用于支持多种多样的证书管理实现，比如 PKIX (X.509 中的公钥管理的内部实现)；

```
   ContentInfo ::= SEQUENCE {
     contentType ContentType,
     content [0] EXPLICIT ANY DEFINED BY contentType }

   ContentType ::= OBJECT IDENTIFIER


   SignedData ::= SEQUENCE {
     version CMSVersion,
     digestAlgorithms DigestAlgorithmIdentifiers,
     encapContentInfo EncapsulatedContentInfo,
     certificates [0] IMPLICIT CertificateSet OPTIONAL,
     crls [1] IMPLICIT RevocationInfoChoices OPTIONAL,
     signerInfos SignerInfos }

   DigestAlgorithmIdentifiers ::= SET OF DigestAlgorithmIdentifier

   SignerInfos ::= SET OF SignerInfo

   EncapsulatedContentInfo ::= SEQUENCE {
     eContentType ContentType,
     eContent [0] EXPLICIT OCTET STRING OPTIONAL }

   SignerInfo ::= SEQUENCE {
     version CMSVersion,
     sid SignerIdentifier,
     digestAlgorithm DigestAlgorithmIdentifier,
     signedAttrs [0] IMPLICIT SignedAttributes OPTIONAL,
     signatureAlgorithm SignatureAlgorithmIdentifier,
     signature SignatureValue,
     unsignedAttrs [1] IMPLICIT UnsignedAttributes OPTIONAL }

   SignerIdentifier ::= CHOICE {
     issuerAndSerialNumber IssuerAndSerialNumber,
     subjectKeyIdentifier [0] SubjectKeyIdentifier }

   SignedAttributes ::= SET SIZE (1..MAX) OF Attribute

   UnsignedAttributes ::= SET SIZE (1..MAX) OF Attribute

   Attribute ::= SEQUENCE {
     attrType OBJECT IDENTIFIER,
     attrValues SET OF AttributeValue }

   AttributeValue ::= ANY

   SignatureValue ::= OCTET STRING

   EnvelopedData ::= SEQUENCE {
     version CMSVersion,
     originatorInfo [0] IMPLICIT OriginatorInfo OPTIONAL,
     recipientInfos RecipientInfos,
     encryptedContentInfo EncryptedContentInfo,
     unprotectedAttrs [1] IMPLICIT UnprotectedAttributes OPTIONAL }

```
可见，PKCS#7 定义完整的一整套的用于加密数据，签名，签名者，加密算法等等一系列信息；由此，奠定了其作为 PKI 的基础；


## PKCS 8 私钥信息格式标准
包含了两个部分，private key 的原始格式 private key info 和 private key 的加密格式 encrypted private key info；

可以看到，PKCS #8 在原来私钥的格式上做了一层抽象封装，这样使得它可以兼容任何的私钥格式；使得 PKCS 的私钥标准可以使用到任何加密算法，这个同 PKCS#1 中定义的 RSA 私钥语法是不同的，PKCS #1 定义的只是特定的 RSA 私钥的语法格式；


# 参考
https://segmentfault.com/a/1190000002569859
https://deepzz.com/post/based-on-openssl-privateCA-issuer-cert.html
https://yq.aliyun.com/articles/47314
https://docs.azure.cn/zh-cn/articles/azure-operations-guide/application-gateway/aog-application-gateway-howto-create-self-signed-cert-via-openssl

http://www.shangyang.me/categories/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A7%91%E5%AD%A6%E4%B8%8E%E6%8A%80%E6%9C%AF/%E5%8A%A0%E5%AF%86%E8%A7%A3%E5%AF%86/RSA/


https://blog.csdn.net/guyongqiangx/article/details/74331892