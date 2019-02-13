---
title: 字符集与string
abbrlink: 450188657
date: 2018-07-03 07:43:36
tags:
categories:
---
# Java字符串的内部编码
String类内部管理着一个char类型的数组，Java API是这样描述char基本类型的：
char 数据类型（和 Character 对象封装的值）基于原始的 Unicode 规范，将字符定义为固定宽度的 16 位实体。(2倍于字节)


# 字符集对比

```
@Test
  public void chinese() {
    String str = "中国";
    printBytes(str);
  }

  @Test
  public void email() {
    String str = "aa@qq.com";
    printBytes(str);
  }

  private void printBytes(String str) {
    printBytes(str + "的UNICODE编码：", str.getBytes(Charset.forName("unicode")));
    printBytes(str + "的8859编码：", str.getBytes(Charset.forName("ISO-8859-1")));
    printBytes(str + "的BIG5编码：", str.getBytes(Charset.forName("BIG5")));
    printBytes(str + "的GBK编码：", str.getBytes(Charset.forName("GBK")));
    printBytes(str + "的UTF-8编码：", str.getBytes(Charset.forName("UTF-8")));
  }

  public void printBytes(String title, byte[] data) {
    System.out.println(title);
    for (byte b : data) {
      System.out.print("0x" + toHexString(b) + " ");
    }
    System.out.println();
  }

  public String toHexString(byte value) {
    String tmp = Integer.toHexString(value & 0xFF);
    if (tmp.length() == 1) {
      tmp = "0" + tmp;
    }

    return tmp.toUpperCase();
  }
```

## 中国
```
中国的UNICODE编码：
0xFE 0xFF 0x4E 0x2D 0x56 0xFD 
中国的8859编码：
0x3F 0x3F 
中国的BIG5编码：
0xA4 0xA4 0x3F 
中国的GBK编码：
0xD6 0xD0 0xB9 0xFA 
中国的UTF-8编码：
0xE4 0xB8 0xAD 0xE5 0x9B 0xBD 

```


## email
```
aa@qq.com的UNICODE编码：
0xFE 0xFF 0x00 0x61 0x00 0x61 0x00 0x40 0x00 0x71 0x00 0x71 0x00 0x2E 0x00 0x63 0x00 0x6F 0x00 0x6D 
aa@qq.com的8859编码：
0x61 0x61 0x40 0x71 0x71 0x2E 0x63 0x6F 0x6D 
aa@qq.com的BIG5编码：
0x61 0x61 0x40 0x71 0x71 0x2E 0x63 0x6F 0x6D 
aa@qq.com的GBK编码：
0x61 0x61 0x40 0x71 0x71 0x2E 0x63 0x6F 0x6D 
aa@qq.com的UTF-8编码：
0x61 0x61 0x40 0x71 0x71 0x2E 0x63 0x6F 0x6D 

```

## 特殊的字符集（ISO-8859-1）
ISO-8859-1是单字节字符集，是ASCII字符集的补充。通常情况下使用ISO-8859-1字符集进行字符串对象与字节化字符数据的互换操作与前述完全一致。

利用ISO-8859-1字符集，我们可以将任何一个字节数组无损保存到字符串对象中。



# 0x代表的含义:十六进制表示法

```
public class Test {
	
	public static void main(String[] args) {
		
		int a = 0x2f;//小写十六进制（等价于0x002f）
		System.out.println(Integer.toBinaryString(a));
		
		int b = 0x2F;//大写十六进制
		System.out.println(Integer.toBinaryString(b));
		
		int c = 10;//标准十进制
		System.out.println(Integer.toBinaryString(c));
		
		int d = 010;//以零开头，表示八进制
		System.out.println(Integer.toBinaryString(d));
		
		
		
		char e = 0xff;//char为2个字节，16位
		byte f = 0xf;//byte为8位
		short g = 0xff;//short为2个字节，16位
		System.out.println(Integer.toBinaryString(e));
		System.out.println(Integer.toBinaryString(f));
		System.out.println(Integer.toBinaryString(g));

	}
}
/*
  结果如下：(都是按照int型处理)
101111
101111
1010
1000
11111111
1111
11111111
*/
```

# ASCII （American Standard Code for Information Interchange，美国信息交换标准代码）

十进制:0-127

| Bin\(二进制\) | Oct\(八进制\) | Dec\(十进制\) | Hex\(十六进制\) | 缩写/字符 | 解释 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 0000 0000 | 0 | 0 | 00 | NUL\(null\) | 空字符 |
| 0000 0001 | 1 | 1 | 01 | SOH\(start of headline\) | 标题开始 |
| 0000 0010 | 2 | 2 | 02 | STX \(start of text\) | 正文开始 |
| 0000 0011 | 3 | 3 | 03 | ETX \(end of text\) | 正文结束 |
| 0000 0100 | 4 | 4 | 04 | EOT \(end of transmission\) | 传输结束 |
| 0000 0101 | 5 | 5 | 05 | ENQ \(enquiry\) | 请求 |
| 0000 0110 | 6 | 6 | 06 | ACK \(acknowledge\) | 收到通知 |
| 0000 0111 | 7 | 7 | 07 | BEL \(bell\) | 响铃 |
| 0000 1000 | 10 | 8 | 08 | BS \(backspace\) | 退格 |
| 0000 1001 | 11 | 9 | 09 | HT \(horizontal tab\) | 水平制表符 |
| 0000 1010 | 12 | 10 | 0A | LF \(NL line feed, new line\) | 换行键 |
| 0000 1011 | 13 | 11 | 0B | VT \(vertical tab\) | 垂直制表符 |
| 0000 1100 | 14 | 12 | 0C | FF \(NP form feed, new page\) | 换页键 |
| 0000 1101 | 15 | 13 | 0D | CR \(carriage return\) | 回车键 |
| 0000 1110 | 16 | 14 | 0E | SO \(shift out\) | 不用切换 |
| 0000 1111 | 17 | 15 | 0F | SI \(shift in\) | 启用切换 |
| 0001 0000 | 20 | 16 | 10 | DLE \(data link escape\) | 数据链路转义 |
| 0001 0001 | 21 | 17 | 11 | DC1 \(device control 1\) | 设备控制1 |
| 0001 0010 | 22 | 18 | 12 | DC2 \(device control 2\) | 设备控制2 |
| 0001 0011 | 23 | 19 | 13 | DC3 \(device control 3\) | 设备控制3 |
| 0001 0100 | 24 | 20 | 14 | DC4 \(device control 4\) | 设备控制4 |
| 0001 0101 | 25 | 21 | 15 | NAK \(negative acknowledge\) | 拒绝接收 |
| 0001 0110 | 26 | 22 | 16 | SYN \(synchronous idle\) | 同步空闲 |
| 0001 0111 | 27 | 23 | 17 | ETB \(end of trans. block\) | 结束传输块 |
| 0001 1000 | 30 | 24 | 18 | CAN \(cancel\) | 取消 |
| 0001 1001 | 31 | 25 | 19 | EM \(end of medium\) | 媒介结束 |
| 0001 1010 | 32 | 26 | 1A | SUB \(substitute\) | 代替 |
| 0001 1011 | 33 | 27 | 1B | ESC \(escape\) | 换码\(溢出\) |
| 0001 1100 | 34 | 28 | 1C | FS \(file separator\) | 文件分隔符 |
| 0001 1101 | 35 | 29 | 1D | GS \(group separator\) | 分组符 |
| 0001 1110 | 36 | 30 | 1E | RS \(record separator\) | 记录分隔符 |
| 0001 1111 | 37 | 31 | 1F | US \(unit separator\) | 单元分隔符 |
| 0010 0000 | 40 | 32 | 20 | \(space\) | 空格 |
| 0010 0001 | 41 | 33 | 21 | ! | 叹号 |
| 0010 0010 | 42 | 34 | 22 | " | 双引号 |
| 0010 0011 | 43 | 35 | 23 | \# | 井号 |
| 0010 0100 | 44 | 36 | 24 | $ | 美元符 |
| 0010 0101 | 45 | 37 | 25 | % | 百分号 |
| 0010 0110 | 46 | 38 | 26 | & | 和号 |
| 0010 0111 | 47 | 39 | 27 | ' | 闭单引号 |
| 0010 1000 | 50 | 40 | 28 | \( | 开括号 |
| 0010 1001 | 51 | 41 | 29 | \) | 闭括号 |
| 0010 1010 | 52 | 42 | 2A | \* | 星号 |
| 0010 1011 | 53 | 43 | 2B | + | 加号 |
| 0010 1100 | 54 | 44 | 2C | , | 逗号 |
| 0010 1101 | 55 | 45 | 2D | - | 减号/破折号 |
| 0010 1110 | 56 | 46 | 2E | . | 句号 |
| 00101111 | 57 | 47 | 2F | / | 斜杠 |
| 00110000 | 60 | 48 | 30 | 0 | 数字0 |
| 00110001 | 61 | 49 | 31 | 1 | 数字1 |
| 00110010 | 62 | 50 | 32 | 2 | 数字2 |
| 00110011 | 63 | 51 | 33 | 3 | 数字3 |
| 00110100 | 64 | 52 | 34 | 4 | 数字4 |
| 00110101 | 65 | 53 | 35 | 5 | 数字5 |
| 00110110 | 66 | 54 | 36 | 6 | 数字6 |
| 00110111 | 67 | 55 | 37 | 7 | 数字7 |
| 00111000 | 70 | 56 | 38 | 8 | 数字8 |
| 00111001 | 71 | 57 | 39 | 9 | 数字9 |
| 00111010 | 72 | 58 | 3A | : | 冒号 |
| 00111011 | 73 | 59 | 3B | ; | 分号 |
| 00111100 | 74 | 60 | 3C | &lt; | 小于 |
| 00111101 | 75 | 61 | 3D | = | 等号 |
| 00111110 | 76 | 62 | 3E | &gt; | 大于 |
| 00111111 | 77 | 63 | 3F | ? | 问号 |
| 01000000 | 100 | 64 | 40 | @ | 电子邮件符号 |
| 01000001 | 101 | 65 | 41 | A | 大写字母A |
| 01000010 | 102 | 66 | 42 | B | 大写字母B |
| 01000011 | 103 | 67 | 43 | C | 大写字母C |
| 01000100 | 104 | 68 | 44 | D | 大写字母D |
| 01000101 | 105 | 69 | 45 | E | 大写字母E |
| 01000110 | 106 | 70 | 46 | F | 大写字母F |
| 01000111 | 107 | 71 | 47 | G | 大写字母G |
| 01001000 | 110 | 72 | 48 | H | 大写字母H |
| 01001001 | 111 | 73 | 49 | I | 大写字母I |
| 01001010 | 112 | 74 | 4A | J | 大写字母J |
| 01001011 | 113 | 75 | 4B | K | 大写字母K |
| 01001100 | 114 | 76 | 4C | L | 大写字母L |
| 01001101 | 115 | 77 | 4D | M | 大写字母M |
| 01001110 | 116 | 78 | 4E | N | 大写字母N |
| 01001111 | 117 | 79 | 4F | O | 大写字母O |
| 01010000 | 120 | 80 | 50 | P | 大写字母P |
| 01010001 | 121 | 81 | 51 | Q | 大写字母Q |
| 01010010 | 122 | 82 | 52 | R | 大写字母R |
| 01010011 | 123 | 83 | 53 | S | 大写字母S |
| 01010100 | 124 | 84 | 54 | T | 大写字母T |
| 01010101 | 125 | 85 | 55 | U | 大写字母U |
| 01010110 | 126 | 86 | 56 | V | 大写字母V |
| 01010111 | 127 | 87 | 57 | W | 大写字母W |
| 01011000 | 130 | 88 | 58 | X | 大写字母X |
| 01011001 | 131 | 89 | 59 | Y | 大写字母Y |
| 01011010 | 132 | 90 | 5A | Z | 大写字母Z |
| 01011011 | 133 | 91 | 5B | \[ | 开方括号 |
| 01011100 | 134 | 92 | 5C | \ | 反斜杠 |
| 01011101 | 135 | 93 | 5D | \] | 闭方括号 |
| 01011110 | 136 | 94 | 5E | ^ | 脱字符 |
| 01011111 | 137 | 95 | 5F | \_ | 下划线 |
| 01100000 | 140 | 96 | 60 | \` | 开单引号 |
| 01100001 | 141 | 97 | 61 | a | 小写字母a |
| 01100010 | 142 | 98 | 62 | b | 小写字母b |
| 01100011 | 143 | 99 | 63 | c | 小写字母c |
| 01100100 | 144 | 100 | 64 | d | 小写字母d |
| 01100101 | 145 | 101 | 65 | e | 小写字母e |
| 01100110 | 146 | 102 | 66 | f | 小写字母f |
| 01100111 | 147 | 103 | 67 | g | 小写字母g |
| 01101000 | 150 | 104 | 68 | h | 小写字母h |
| 01101001 | 151 | 105 | 69 | i | 小写字母i |
| 01101010 | 152 | 106 | 6A | j | 小写字母j |
| 01101011 | 153 | 107 | 6B | k | 小写字母k |
| 01101100 | 154 | 108 | 6C | l | 小写字母l |
| 01101101 | 155 | 109 | 6D | m | 小写字母m |
| 01101110 | 156 | 110 | 6E | n | 小写字母n |
| 01101111 | 157 | 111 | 6F | o | 小写字母o |
| 01110000 | 160 | 112 | 70 | p | 小写字母p |
| 01110001 | 161 | 113 | 71 | q | 小写字母q |
| 01110010 | 162 | 114 | 72 | r | 小写字母r |
| 01110011 | 163 | 115 | 73 | s | 小写字母s |
| 01110100 | 164 | 116 | 74 | t | 小写字母t |
| 01110101 | 165 | 117 | 75 | u | 小写字母u |
| 01110110 | 166 | 118 | 76 | v | 小写字母v |
| 01110111 | 167 | 119 | 77 | w | 小写字母w |
| 01111000 | 170 | 120 | 78 | x | 小写字母x |
| 01111001 | 171 | 121 | 79 | y | 小写字母y |
| 01111010 | 172 | 122 | 7A | z | 小写字母z |
| 01111011 | 173 | 123 | 7B | { | 开花括号 |
| 01111100 | 174 | 124 | 7C |    | 垂线 |
| 01111101 | 175 | 125 | 7D | } | 闭花括号 |
| 01111110 | 176 | 126 | 7E | ~ | 波浪号 |
| 01111111 | 177 | 127 | 7F | DEL \(delete\) | 删除 |


# 各个字符集对比


## utf16
Unicode的编码空间从U+0000到U+10FFFF，共有1,112,064个码位（code point）可用来映射字符

## utf8
1-6个字节（越往前越常用，一般3个字节）
| First code point | Last code point | Byte 1     | Byte 2     | Byte 3     | Byte 4     | Byte 5     | Byte 6     |
| ---------------- | --------------- | ---------- | ---------- | ---------- | ---------- | ---------- | ---------- |
| U+0000           | U+007F          | `0xxxxxxx` |            |            |            |            |            |
| U+0080           | U+07FF          | `110xxxxx` | `10xxxxxx` |            |            |            |            |
| U+0800           | U+FFFF          | `1110xxxx` | `10xxxxxx` | `10xxxxxx` |            |            |            |
| U+10000          | U+1FFFFF        | `11110xxx` | `10xxxxxx` | `10xxxxxx` | `10xxxxxx` |            |            |
| U+200000         | U+3FFFFFF       | `111110xx` | `10xxxxxx` | `10xxxxxx` | `10xxxxxx` | `10xxxxxx` |            |
| U+4000000        | U+7FFFFFFF      | `1111110x` | `10xxxxxx` | `10xxxxxx` | `10xxxxxx` | `10xxxxxx` | `10xxxxxx` |


所以开始的128个字符（US-ASCII）只需一字节，接下来的1920个字符需要双字节编码，包括带附加符号的拉丁字母，希腊字母，西里尔字母，科普特语字母，亚美尼亚语字母，希伯来文字母和阿拉伯字母的字符。基本多文种平面中其余的字符使用三个字节，剩余字符使用四个字节。


···
对于UTF-8编码中的任意字节B，如果B的第一位为0，则B独立的表示一个字符(ASCII码)；
如果B的第一位为1，第二位为0，则B为一个多字节字符中的一个字节(非ASCII字符)；
如果B的前两位为1，第三位为0，则B为两个字节表示的字符中的第一个字节；
如果B的前三位为1，第四位为0，则B为三个字节表示的字符中的第一个字节；
如果B的前四位为1，第五位为0，则B为四个字节表示的字符中的第一个字节；
···


按四位来计算空间的话：
2^7 + 2^11 + 2^16 + 2^21

**Quote from Wikipedia: "UTF-8 encodes each of the 1,112,064 code points in the Unicode character set using one to four 8-bit bytes (termed "octets" in the Unicode Standard)."**

# 参考
https://blog.csdn.net/darxin/article/details/5079242
https://zh.wikipedia.org/wiki/UTF-8
