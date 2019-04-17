# JT/T808-2011协议解析规则白话文详解（JAVA版本）
## 1.包详解
解析之前复习下java的数据结构，以及对应JT/L808-2011里面的数据类型。

JT/L808| JAVA数据类型 | JT/L808描述 | JAVA描述 
---|---|---|---|
BYTE  | Byte |无符号单字节整型(字节， 8 位)|以二进制补码表示的整数|
WORD  | Short|无符号双字节整型(字节，16位)|short数据类型是16位、有符号的以二进制补码表示的整数|
DWORD |Intger|无符号四字节整型(双字，32位)|数据类型是32位、有符号的以二进制补码表示的整数|
BYTE[n] |Byte[n]|n字节|n字节|
BCD[n]  |BCD[n] |8421码，n字节|2个字符标识1个Byte(个人理解)|

关于BCD码，可以自行百度，搜索关键字：java bcd码。


### 1.1 消息结构

标识位 | 消息头 | 消息体 | 校验码 |  标识位  |
---|---|---|---|---|
标识位：固定用7E标识

消息头：**1.2介绍**

消息体：承载消息内容（个人理解）

校验码：校验码指从消息头开始，同后一字节异或，直到校验码前一个字节，占用一个字节。

标识位：固定用7E标识

### 1.2 消息头

起始字节 | 字段 | 数据类型 | 描述及要求 |
---|--- | ---|---|
0 | 消息ID|WORD|-|
2 | 消息体属性|WORD|消息体属性格式结构图|
4 |终端手机号 |BCD[6] |根据安装后终端自身的手机号转换|
10|消息流水号|WORD|按发送顺序从 0 开始循环累加
12|消息包封顶项||如果消息体属性中相关标识位确定消息分包处理，则该项有内容，否则无该项

#### 个人理解：（1~3不包含3，为byte数据下标，与JDK截取规则保持一致）
消息ID：占用2个字节，解析byte位从1~3,byte[0]为标识符。

消息体属性：占2个字节，解析byte位为从3~5。

终端手机号：占6个字节，解析byte位为5~11。

终端流水号：占2个字节，解析byte为位为11~13。

消息包封顶项：由消息体属性决定(第13bit为1，有分包)。

#### 消息体属性格式结构图如图
<table>
  <tr>
    <td>15</td>
    <td>14</td>
    <td>13</td>
    <td>12</td>
    <td>11</td>
    <td>10</td>
    <td>9</td>
    <td>9</td>
    <td>7</td>
    <td>6</td>
    <td>5</td>
    <td>4</td>
    <td>3</td>
    <td>2</td>
    <td>1</td>
    <td>0</td>
  </tr>
  <tr>
    <td colspan="2">保留</td>
    <td>分包</td>
    <td colspan="3">数据加密方式</td>
    <td colspan="10">消息体长度</td>
  </tr>
</table>

#### 消息包封顶项内容表
起始字节| 字段 | 数据类型 | 描述及要求|
-- | -- | -- | -- | -- |
0 |消息包总数| WORD |该消息分包的总包数|
2 |序号      | WORD | 从1开始 |

## 2.消息包举例
### 2.1终端注册
#### 消息包
```
7e 0100 0021 014785236900 0046 000B04575331303030533130303030303031323334353637327ca4425039344a35 ef 7e
```
详细解释
```
7e                  --标识符 
01 00               --消息头（消息ID）
00 21               --消息头（消息体属性） （二进制串： 0000 0000 0010 0001） 此消息不分包、不加密
01 47 85 23 69 00   --消息头（终端手机号）
00 46               --消息头（消息流水号） 
00 0B               --省域ID
04 57               --市县域ID
53 31 30 30 30      --制造商ID
53 31 30 30 30 30 30 30 --终端型号
31 32 33 34 35 36 37    --终端ID
32                  --车牌颜色（不确定是否用01 02标识，曾经解析的硬件是GBK编码的byte表示）（JT/T 415-2006 的 5.4.12）
7c a4 42 50 39 34 4a 35 --车牌GBK编码（公安交通管理部门颁发的机动车号牌）
ef                  --校验码 
7e                  --标识符
```

### 2.2终端鉴权
#### 消息包
```
7e010200060147852369000045616161616161ef7e
```
详细解释
```
7e                  --标识符
01 02               --消息头（消息ID）
00 06               --消息头（消息体属性）           
01 47 85 23 69 00   --消息头（终端手机号） 
00 45               --消息头（消息流水号）  
61 61 61 61 61 61   --鉴权码
ef                  --校验码 
7e                  --标识符
```

### 2.3终端心跳
#### 消息包
```
7e0002000001453017984000BF067e
```
详细解释
```
7e                  --标识符
00 02               --消息头（消息ID）
00 00               --消息头（消息体属性）           
01 45 30 17 98 40   --消息头（终端手机号） 
00 BF               --消息头（消息流水号）  
06                  --校验码 
7e                  --标识符
```

### 2.4终端注销
#### 消息包
```
7e000300000147852369000046ef7e
```
详细解释
```
7e                  --标识符
00 03               --消息头（消息ID）
00 00               --消息头（消息体属性）           
01 47 85 23 69 00   --消息头（终端手机号） 
00 46               --消息头（消息流水号）  
ef                  --校验码 
7e                  --标识符
```


### 2.5位置信息汇报
#### 消息包
```
7E0200005B014141138693224E00000100000000000157E6DE06CBEC600000000000001703090019200104000026F5EB3700060089FFFFFFFD000700B400FFFFFFFF002400A901CC000627BD0FABCC27910000B727911287BF27BD1159C327BD0000BB27910ED1B5C97E
```
详细解释
```
7E                  --标识符
02 00               --消息头（消息ID）
00 5B               --消息头（消息体属性）  
01 41 41 13 86 93   --消息头（终端手机号）
22 4E               --消息头（消息流水号）
00 00 01 00         --报警标志
00 00 00 00         --状态
01 57 E6 DE         --纬度
06 CB EC 60         --经度
00 00               --高程
00 00               --速度
00 00               --方向
17 03 09 00 19 20   --时间（GTM+8）   
01                  --附加消息ID(0x01)
04                  --附加消息长度(DWORD)
00 00 26 F5         --附加消息（里程）
EB                  --附加消息(0xEB,这里是厂商自定义的)
37                  --附加消息长度(DWORD,16进制的37,10进制为55,剩下的消息55个长度)
0006 0089 FFFFFFFD  --附加消息（厂商自定）
0007 00B4 00 FFFF FFFF --附加消息（厂商自定）
0024 00A9 01CC 00 06   --附加消息（厂商自定）
27BD 0FAB CC           --附加消息（厂商自定）  
2791 0000 B7           --附加消息（厂商自定）
2791 1287 BF           --附加消息（厂商自定）
27BD 1159 C3           --附加消息（厂商自定）
27BD 0000 BB           --附加消息（厂商自定）
2791 0ED1 B5           --附加消息（厂商自定）
C9                  --校验码 
7E                                    --标识符
```

## 3.最后的话
同时也可以为有需要的朋友定制开发jt-808协议开发。
有好的建议或者不明白请联系我，hncdyj123#163.com (#换成@) 

## 4.关于作者
```
var info = {
    nickName  : "杰锅锅",
    introduce : "五年以上码农，算不上资深",
    site : "http://blog.csdn.net/hncdyj"
  }
```
转载请注明：
