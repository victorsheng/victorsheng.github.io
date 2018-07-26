---
title: charset-base64
tags:
categories:
---

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