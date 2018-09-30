title: charset-base64
abbrlink: 2481242048
date: 2018-09-30 10:08:59
tags:
categories:
---


# Base64的编码转换方式

所谓Base64，就是说选出64个字符----小写字母a-z、大写字母A-Z、数字0-9、符号"+"、"/"（再加上作为垫字的"="，实际上是65个字符）----作为一个基本字符集。然后，其他所有符号都转换成这个字符集中的字符。

具体来说，转换方式可以分为四步。

第一步，将每三个字节作为一组，一共是24个二进制位。

第二步，将这24个二进制位分为四组，每个组有6个二进制位。

第三步，在每组前面加两个00，扩展成32个二进制位，即四个字节。

第四步，根据下表，得到扩展后的每个字节的对应符号，这就是Base64的编码值。

　　0　A　　17　R　　　34　i　　　51　z

　　1　B　　18　S　　　35　j　　　52　0

　　2　C　　19　T　　　36　k　　　53　1

　　3　D　　20　U　　　37　l　　　54　2

　　4　E　　21　V　　　38　m　　　55　3

　　5　F　　22　W　　　39　n　　　56　4

　　6　G　　23　X　　　40　o　　　57　5

　　7　H　　24　Y　　　41　p　　　58　6

　　8　I　　　25　Z　　　42　q　　　59　7

　　9　J　　26　a　　　43　r　　　60　8

　　10　K　　27　b　　　44　s　　　61　9

　　11　L　　28　c　　　45　t　　　62　+

　　12　M　　29　d　　　46　u　　　63　/

　　13　N　　30　e　　　47　v

　　14　O　　31　f　　　48　w　　　

　　15　P　　32　g　　　49　x

　　16　Q　　33　h　　　50　y

**因为，Base64将三个字节转化成四个字节，因此Base64编码后的文本，会比原文本大出三分之一左右**。


# 参考
http://www.ruanyifeng.com/blog/2008/06/base64.html

# base64和base64UrlSafe的区别
和标准版区别有三

+ 被 - 代替，加号对应减号
/ 被 _ 代替
没有 = 这个 padding 字符

其中,62,63有所变化

```
   Value Encoding  Value Encoding  Value Encoding  Value Encoding
       0 A            17 R            34 i            51 z
       1 B            18 S            35 j            52 0
       2 C            19 T            36 k            53 1
       3 D            20 U            37 l            54 2
       4 E            21 V            38 m            55 3
       5 F            22 W            39 n            56 4
       6 G            23 X            40 o            57 5
       7 H            24 Y            41 p            58 6
       8 I            25 Z            42 q            59 7
       9 J            26 a            43 r            60 8
      10 K            27 b            44 s            61 9
      11 L            28 c            45 t            62 - (minus)
      12 M            29 d            46 u            63 _
      13 N            30 e            47 v           (underline)
      14 O            31 f            48 w
      15 P            32 g            49 x
      16 Q            33 h            50 y         (pad) =
C
```

# 工作中遇到的:
```
import java.util.Base64;


Base64.getUrlEncoder().encodeToString(content);
```
结果字符串还是带有=的padding

https://bugs.openjdk.java.net/browse/JDK-8026330