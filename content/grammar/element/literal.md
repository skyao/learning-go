---
date: 2020-07-04T23:50:00+08:00
title: 字面量
weight: 419
menu:
  main:
    parent: "grammar-element"
description : "go语言中的Literal字面量"
---

> 备注：摘录自 golang语言规范 https://golang.org/ref/spec#Integer_literals

#### Integer literals/整型字面量

整数字面量是代表整数常数的数字序列。可选的前缀设置了一个非十进制的基数：0b` 或 `0B代表二进制，0`, `0o`, 或 `0O代表八进制，0x` 或 `0X代表十六进制。单一的0被认为是十进制的0。在十六进制中，字母a到f和A到F代表10到15的数值。

为了便于阅读，下划线字符_可能会出现在基数前缀之后或连续的数字之间；这种下划线不会改变文字的价值。

```go
int_lit        = decimal_lit | binary_lit | octal_lit | hex_lit .
decimal_lit    = "0" | ( "1" … "9" ) [ [ "_" ] decimal_digits ] .
binary_lit     = "0" ( "b" | "B" ) [ "_" ] binary_digits .
octal_lit      = "0" [ "o" | "O" ] [ "_" ] octal_digits .
hex_lit        = "0" ( "x" | "X" ) [ "_" ] hex_digits .

decimal_digits = decimal_digit { [ "_" ] decimal_digit } .
binary_digits  = binary_digit { [ "_" ] binary_digit } .
octal_digits   = octal_digit { [ "_" ] octal_digit } .
hex_digits     = hex_digit { [ "_" ] hex_digit } .
```

```go
42
4_2
0600
0_600
0o600
0O600       // 第二个字是大写字母 'O'
0xBadFace
0xBad_Face
0x_67_7a_2f_cc_40_c6
170141183460469231731687303715884105727
170_141183_460469_231731_687303_715884_105727

_42         // 标识符，不是整型字面量
42_         // invalid: _ 必须隔开连续的数字
4__2        // invalid: 一次只能有一个_
0_xBadFace  // invalid: _ 必须隔开连续的数字
```

#### Floating-point literals/浮点字面量

浮点字面量是浮点常数的十进制或十六进制表示。

十进制浮点字面量由整数部分（小数点）、小数点、分数部分（小数点）和指数部分（e或E后面跟着一个可选的符号和小数点）组成。整数部分或小数部分中的其中一个可以省略；小数点或指数部分中的其中一个可以省略。指数值exp将mantissa（整数和小数部分）的比例为10exp。

十六进制浮点文字由0x或0X前缀、整数部分（十六进制数字）、小数点、小数部分（十六进制数字）和指数部分（p或P，后面跟着一个可选的符号和十进制数字）组成。整数部分或分数部分中的其中一个可以省略；弧度点也可以省略，但指数部分是必须的。(这个语法与IEEE 754-2008 §5.12.3中给出的语法相匹配。)指数值exp将尾数(整数和分数部分)按2exp缩放。

为了便于阅读，下划线字符_可以出现在基数前缀之后或连续的数字之间；这种下划线不会改变字面值。

```go
float_lit         = decimal_float_lit | hex_float_lit .

decimal_float_lit = decimal_digits "." [ decimal_digits ] [ decimal_exponent ] |
                    decimal_digits decimal_exponent |
                    "." decimal_digits [ decimal_exponent ] .
decimal_exponent  = ( "e" | "E" ) [ "+" | "-" ] decimal_digits .

hex_float_lit     = "0" ( "x" | "X" ) hex_mantissa hex_exponent .
hex_mantissa      = [ "_" ] hex_digits "." [ hex_digits ] |
                    [ "_" ] hex_digits |
                    "." hex_digits .
hex_exponent      = ( "p" | "P" ) [ "+" | "-" ] decimal_digits .
```

```go
0.
72.40
072.40       // == 72.40
2.71828
1.e+0
6.67428e-11
1E6
.25
.12345E+5
1_5.         // == 15.0
0.15e+0_2    // == 15.0

0x1p-2       // == 0.25
0x2.p10      // == 2048.0
0x1.Fp+0     // == 1.9375
0X.8p-0      // == 0.5
0X_1FFFP-16  // == 0.1249847412109375
0x15e-2      // == 0x15e - 2 (integer subtraction)

0x.p1        // invalid: mantissa has no digits
1p-2         // invalid: p exponent requires hexadecimal mantissa
0x1.5e-2     // invalid: hexadecimal mantissa requires p exponent
1_.5         // invalid: _ must separate successive digits
1._5         // invalid: _ must separate successive digits
1.5_e1       // invalid: _ must separate successive digits
1.5e_1       // invalid: _ must separate successive digits
1.5e1_       // invalid: _ must separate successive digits
```

#### Imaginary literals/虚数字面量

虚数字面量表示复数常数的虚数部分，它由一个整数或浮点字组成，后面是小写字母i。

`imaginary_lit = (decimal_digits | int_lit | float_lit) "i" .`

为了向后兼容，虚字的整数部分完全由十进制数字（可能还有下划线）组成，即使它以前导0开始，也被认为是一个十进制整数。

```go
0i
0123i         // == 123i for backward-compatibility
0o123i        // == 0o123 * 1i == 83i
0xabci        // == 0xabc * 1i == 2748i
0.i
2.71828i
1.e+0i
6.67428e-11i
1E6i
.25i
.12345E+5i
0x1p-2i       // == 0x1p-2 * 1i == 0.25i
```

#### Rune literals

符文字面量代表一个符文常量，是一个用于识别Unicode code point的整数值。符文字面量表示为一个或多个用单引号括起来的字符，如'x'或'\n'。在引号内，除了换行和未转义的单引号外，任何字符都可以出现。单引号字符代表字符本身的Unicode值，而以反斜杠开头的多字符序列则以各种格式编码值。

最简单的形式代表引号内的单个字符；由于go源文本是以UTF-8编码的Unicode字符，所以多个UTF-8编码的字节可以代表单个整数值。例如，字面量 'a' 持有一个字节，代表字面量 a，Unicode U+0061，值0x61，而 "ä "持有两个字节（0xc3 0xa4），代表文字 "a-dieresis"，U+00E4，值0xe4。

几个反斜杠转义符允许任意值被编码为ASCII文本。有四种方法可以将整数值表示为数字常数。`\x`后面跟着两位十六进制数字；`\u`后面跟着四位十六进制数字；`\U`后面跟着八位十六进制数字，以及后面跟着三位八进制数字的反斜杠`\`。在每一种情况下，字面量的值都是相应基数的数字所代表的值。

虽然这些表示方法都会产生一个整数，但它们的有效范围不同。八进制转义符必须在0到255之间表示一个值。十六进制转义符通过结构满足这个条件。转义符\u和\U代表Unicode码点，所以在它们里面有些值是非法的，特别是那些高于0x10FFFF的值和代用的一半。

在反斜杠之后，某些单字符转义符代表特殊值:

```go
\a   U+0007 alert or bell
\b   U+0008 backspace
\f   U+000C form feed
\n   U+000A line feed or newline
\r   U+000D carriage return
\t   U+0009 horizontal tab
\v   U+000b vertical tab
\\   U+005c backslash
\'   U+0027 single quote  (valid escape only within rune literals)
\"   U+0022 double quote  (valid escape only within string literals)
```

所有其他以反斜杠开头的序列在符文字里面都是非法的。

```go
rune_lit         = "'" ( unicode_value | byte_value ) "'" .
unicode_value    = unicode_char | little_u_value | big_u_value | escaped_char .
byte_value       = octal_byte_value | hex_byte_value .
octal_byte_value = `\` octal_digit octal_digit octal_digit .
hex_byte_value   = `\` "x" hex_digit hex_digit .
little_u_value   = `\` "u" hex_digit hex_digit hex_digit hex_digit .
big_u_value      = `\` "U" hex_digit hex_digit hex_digit hex_digit
                           hex_digit hex_digit hex_digit hex_digit .
escaped_char     = `\` ( "a" | "b" | "f" | "n" | "r" | "t" | "v" | `\` | "'" | `"` ) .
```

```go
'a'
'ä'
'本'
'\t'
'\000'
'\007'
'\377'
'\x07'
'\xff'
'\u12e4'
'\U00101234'
'\''         // rune literal containing single quote character
'aa'         // illegal: too many characters
'\xa'        // illegal: too few hexadecimal digits
'\0'         // illegal: too few octal digits
'\uDFFF'     // illegal: surrogate half
'\U00110000' // illegal: invalid Unicode code point
```

#### String literals/字符串字面量

字符串字面量表示从字符序列的连接中获得的字符串常量。有两种形式：原始(raw)字符串字元和解释(interpreted)字符串字元。

原始字符串字元是后引号之间的字符序列，如 "\`foo\`"。在引号内，除了后引号，任何字符都可以出现。原始字符串字面量的值是由引号之间未解释（隐含UTF-8编码）的字符组成的字符串；特别是，反斜杠没有特殊意义，字符串可能包含换行符。原始字符串字元中的回车字符（'\r'）会从原始字符串值中被丢弃。

被解释的字符串字面量是双引号之间的字符序列，如 "bar"。在引号内，除了换行和未转义的双引号外，任何字符都可以出现。引号之间的文字构成了字面量意义的值，反斜杠转义的解释与符文字面意义的解释相同（除了'/'是非法的，而"/"是合法的），有相同的限制。三位数的八进制(\nnnnn)和两位数的十六进制(\xnn)转义符代表结果字符串的单个字节；所有其他转义符代表单个字符的UTF-8编码（可能是多字节）。因此，在一个字符串中，字面意义中的\377和\xFF代表一个价值0xFF=255的单一字节，而ÿ、\u00FF、\U000000FF和xc3\xbf代表字符U+00FF的UTF-8编码的两个字节0xc3 0xbf。

```go
string_lit             = raw_string_lit | interpreted_string_lit .
raw_string_lit         = "`" { unicode_char | newline } "`" .
interpreted_string_lit = `"` { unicode_value | byte_value } `"` .
```

```go
`abc`                // same as "abc"
`\n
\n`                  // same as "\\n\n\\n"
"\n"
"\""                 // same as `"`
"Hello, world!\n"
"日本語"
"\u65e5本\U00008a9e"
"\xff\u00FF"
"\uD800"             // illegal: surrogate half
"\U00110000"         // illegal: invalid Unicode code point
```

这些例子都代表同一个字符串：

```go
"日本語"                                 // UTF-8 input text
`日本語`                                 // UTF-8 input text as a raw literal
"\u65e5\u672c\u8a9e"                    // the explicit Unicode code points
"\U000065e5\U0000672c\U00008a9e"        // the explicit Unicode code points
"\xe6\x97\xa5\xe6\x9c\xac\xe8\xaa\x9e"  // the explicit UTF-8 bytes
```

如果源码将一个字符表示为两个code point，比如涉及重音和字母的组合形式，如果放在符文rune字面量中，结果将是一个错误（它不是单个的code point），如果放在字符串字面量中，将出现两个code point。