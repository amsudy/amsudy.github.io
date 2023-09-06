# unicode

unicode 用一套统一的字符集来表示显示符号。

Unicode目前分为17组，即17个平面（Plane), 每个平面拥有65536（2<sup>16</sup>）个代码点。

| 平面 |       始末值        |          名称           |                   name                   |
| :--: | :-----------------: | :---------------------: | :--------------------------------------: |
|  0   |   U+0000 - U+FFFF   |     基本多文种平面      |      BMP(Basic Multilingual Plane)       |
|  1   |  U+10000 - U+1FFFF  |     多文种补充平面      |  SMP(Supplementary Multilingual Plane)   |
|  2   |  U+20000 - U+2FFFF  |    表意文字补充平面     |   SIP(Supplementary Ideographic Plane)   |
|  3   |  U+30000 - U+3FFFF  |    表意文字第三平面     |     TIP(Tertiary Ideographic Plane)      |
| 4~13 |  U+40000 - U+DFFFF  |       （未使用）        |                                          |
|  14  |  U+E0000 - U+EFFFF  |    特殊用途补充平面     | SSP(Supplementary Special-purpose Plane) |
|  15  |  U+F0000 - U+FFFFF  | 保留作为私人使用（A区） |        PUA-A(Private Use Area-A)         |
|  16  | U+100000 - U+10FFFF | 保留作为私人使用（B区） |                  PUA-B                   |

### 编码

将字符集里的码点值转换为一定bit的编码值的过程。

格式有UTF-8
