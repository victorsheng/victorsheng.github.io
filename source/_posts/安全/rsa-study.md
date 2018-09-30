---
title: rsa-study
abbrlink: 2888652899
date: 2018-08-03 09:22:29
tags:
categories:
---
RSA加解密中必须考虑到的密钥长度、明文长度和密文长度问题。明文长度需要小于密钥长度，而密文长度则等于密钥长度。因此当加密内容长度大于密钥长度时，有效的RSA加解密就需要对内容进行分段。

这是因为，RSA算法本身要求加密内容也就是明文长度m必须0<m<密钥长度n。如果小于这个长度就需要进行padding，因为如果没有padding，就无法确定解密后内容的真实长度，字符串之类的内容问题还不大，以0作为结束符，但对二进制数据就很难，因为不确定后面的0是内容还是内容结束符。而只要用到padding，那么就要占用实际的明文长度，于是实际明文长度需要减去padding字节长度。我们一般使用的padding标准有NoPPadding、OAEPPadding、PKCS1Padding等，其中PKCS#1建议的padding就占用了11个字节。

这样，对于1024长度的密钥。128字节（1024bits）-减去11字节正好是117字节，但对于RSA加密来讲，padding也是参与加密的，所以，依然按照1024bits去理解，但实际的明文只有117字节了。



# 加密/解密分区处理
```
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;
import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;
import javax.crypto.BadPaddingException;
import javax.crypto.Cipher;
import javax.crypto.IllegalBlockSizeException;
import javax.crypto.NoSuchPaddingException;

public class RsaUtils {

  public static final String RSA_ALGORITHM = "RSA";



  public static byte[] publicEncrypt(RSAPublicKey publicKey, byte[] data) {
    try {
      Cipher cipher = Cipher.getInstance(RSA_ALGORITHM);
      cipher.init(Cipher.ENCRYPT_MODE, publicKey);
      return rsaSplitCodec(cipher, Cipher.ENCRYPT_MODE, data, publicKey.getModulus().bitLength());
    } catch (Exception e) {
      throw new RuntimeException("加密字符串[" + data + "]时遇到异常", e);
    }
  }

  public static byte[] privateDecrypt(RSAPrivateKey privateKey, byte[] data) {
    try {
      Cipher cipher = Cipher.getInstance(RSA_ALGORITHM);
      cipher.init(Cipher.DECRYPT_MODE, privateKey);
      return rsaSplitCodec(cipher, Cipher.DECRYPT_MODE, data, privateKey.getModulus().bitLength());
    } catch (Exception e) {
      throw new RuntimeException("解密字符串[" + data + "]时遇到异常", e);
    }
  }

  public static byte[] privateEncrypt(RSAPrivateKey privateKey, byte[] data) {
    try {
      Cipher cipher = Cipher.getInstance(RSA_ALGORITHM);
      cipher.init(Cipher.ENCRYPT_MODE, privateKey);
      return rsaSplitCodec(cipher, Cipher.ENCRYPT_MODE, data, privateKey.getModulus().bitLength());
    } catch (Exception e) {
      throw new RuntimeException("加密字符串[" + data + "]时遇到异常", e);
    }
  }

  public static byte[] publicDecrypt(RSAPublicKey publicKey, byte[] data) {
    try {
      Cipher cipher = Cipher.getInstance(RSA_ALGORITHM);
      cipher.init(Cipher.DECRYPT_MODE, publicKey);
      return rsaSplitCodec(cipher, Cipher.DECRYPT_MODE, data, publicKey.getModulus().bitLength());
    } catch (NoSuchAlgorithmException e) {
      throw new RuntimeException("无此解密算法");
    } catch (NoSuchPaddingException e) {
      e.printStackTrace();
      return null;
    } catch (InvalidKeyException e) {
      throw new RuntimeException("解密公钥非法,请检查");
    }
  }


  public static byte[] rsaSplitCodec(Cipher cipher, int opmode, byte[] datas, int keySize) {
    int maxBlock = 0;
    if (opmode == Cipher.DECRYPT_MODE) {
      maxBlock = keySize / 8;
    } else {
      maxBlock = keySize / 8 - 11;
    }
    ByteArrayOutputStream out = new ByteArrayOutputStream();
    int offSet = 0;
    byte[] buff;
    int i = 0;
    try {
      while (datas.length > offSet) {
        if (datas.length - offSet > maxBlock) {
          buff = cipher.doFinal(datas, offSet, maxBlock);
        } else {
          buff = cipher.doFinal(datas, offSet, datas.length - offSet);
        }
        out.write(buff, 0, buff.length);
        i++;
        offSet = i * maxBlock;
      }
    } catch (IllegalBlockSizeException e) {
      throw new RuntimeException("密文长度非法");
    } catch (BadPaddingException e) {
      throw new RuntimeException("密文数据已损坏");
    }
    byte[] resultDatas = out.toByteArray();
    try {
      out.close();
    } catch (IOException e) {
      throw new RuntimeException("输出流关闭错误");
    }
    return resultDatas;
  }

}


```