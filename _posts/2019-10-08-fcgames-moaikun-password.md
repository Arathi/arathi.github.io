---
layout: post
title:  "FC游戏背后的秘密（一）：摩艾君的续关密码"
date:   2019-10-08 12:30:00 +0800
categories: 6502
tags: 6502
---

# FC游戏背后的秘密（一）：摩艾君的续关密码

## 一. 源码解析

### 工具准备

* FC模拟器/调试器：[FCEUX](https://ci.appveyor.com/project/zeromus/fceux/build/artifacts)
* 十六进制编辑器：[HxD](https://mh-nexus.de/en/downloads.php?product=HxD20)
* 6502仿真器/汇编器：[6502 Macroassembler & Simulator](http://exifpro.com/utils.html)
* 文本编辑器：[VSCode](https://code.visualstudio.com)

### 背景知识

* [6502指令集](http://www.6502.org/tutorials/6502opcodes.html)
* [FC硬件信息](http://fms.komkon.org/EMUL8/NES.html)

### 重要内存地址获取

首先找到存放密码的内存地址，先载入游戏，选择CONTINUE，进入输入密码的界面，打开RAM Search或者直接打开Hex Editor观察，就会发现每次密码改动以后，\$52~\$57也会跟着改动，试着修改这6个地址的值，显示出来的密码也会跟着变化，由此确定\$52~\$57就是存放密码的。

### 密码的解析规则

先添加一个断点：当\$0052~\$0057读取时。此时会发现在`C4EA`这个位置一直读取\$52~\$57，大概是要更新屏幕上显示的数；给断点加个限制条件：`P != #C4EA`，就不会触发了。

然后输入完密码，按下START，程序中断在了`C59A`，简单看看附近的代码：

```asm
; 初始化
C578: LDA #$02
C57A: STA $0008
C57C: LDA #$00
C57E: STA $0009 ; 已循环次数，也就是取$C5DC时的Y，可以理解为数组下标
; 外层循环（$08）开始，共3次
C580: LDA #$04
C582: STA $000A
C584: LDX $0008
C586: LDA #$00
C588: STA $00,X
; 内层循环（$0A）开始，共4次
C58A: LDY $0009
C58C: LDA $C5DC,Y
C58F: AND #$03 ; 取低2位，即后面的右移次数
C591: PHA
C592: LDA $C5DC,Y
C595: LSR
C596: LSR
C597: TAX ; 取高6位，即密码的第几位（0~5）
C598: PLA
C599: TAY
C59A: LDA $52,X ; 获取密码
; 右移（根据$C5DC的数据分析，只会右移1次或者4次），即把b0或者b3放到C
C59C: LSR
C59D: DEY
C59E: BPL $C59C
; 计算$00~$02
C5A0: LDX $0008
C5A2: ROL $00,X ; 循环左移，将C移入
C5A4: INC $0009
C5A6: DEC $000A
C5A8: BNE $C58A
; 内循环结束
C5AA: DEC $0008
C5AC: BPL $C580
; 外循环结束
; 检查$00、$01、$02是否相等
C5AE: LDA $0000
C5B0: CMP $0001
C5B2: BNE $C5DA 
C5B4: CMP $0002
C5B6: BNE $C5DA ; $01、$02任意一个与$00不相等即失败
; 取中间两位，计算校验和
C5B8: LDA #$00
C5BA: STA $0008
C5BC: LDX #$05
C5BE: LDA $52,X
C5C0: LSR ; 去掉最低位
C5C1: AND #$03 ; 保留低2位
C5C3: CLC
C5C4: ADC $0008 ; （不进位）相加
C5C6: AND #$03 ; 保留低2位
C5C8: STA $0008
C5CA: DEX
C5CB: BPL $C5BE
C5CD: LDA $0008
C5CF: BNE $C5DA ; $08不为0x00的时候就是失败
; 正确返回，计算起始关卡 A = $00 * 4 + 1
C5D1: LDA $0000
C5D3: ASL
C5D4: ASL
C5D5: CLC
C5D6: ADC #$01
C5D8: SEC ; 设置C=1表示成功
C5D9: RTS
; 失败返回
C5DA: CLC ; 设置C=0表示失败
C5DB: RTS
```

接下来看看$C5DC的数据：

| 地址   | 十六进制 | $\overline{b_7 b_6 b_5 b_4 b_3 b_2}$ | $ \overline{b_1 b_0} $ |
| ---- | ---- | ------------------------------------ | ---------------------- |
| C5DC | 00   | 000000 (0)                           | 00                     |
| C5DD | 04   | 000001 (1)                           | 00                     |
| C5DE | 08   | 000010 (2)                           | 00                     |
| C5DF | 0C   | 000011 (3)                           | 00                     |
| C5E0 | 0B   | 000010 (2)                           | 11                     |
| C5E1 | 0F   | 000011 (3)                           | 11                     |
| C5E2 | 13   | 000100 (4)                           | 11                     |
| C5E3 | 17   | 000101 (5)                           | 11                     |
| C5E4 | 14   | 000101 (5)                           | 00                     |
| C5E5 | 10   | 000100 (4)                           | 00                     |
| C5E6 | 03   | 000000 (0)                           | 11                     |
| C5E7 | 07   | 000001 (1)                           | 11                     |

由此可见每位密码会被读取两次，分别是读取$b_0$和$b_3$。第一次外循环读取密码的1、2、3、4位的$b_0$，第二次循环读取密码的3、4、5、6位的$b_3$，第三次循环读取密码的5、4位的$b_0$和1、2位的$b_3$。

## 二. 密码规则结论与验证

通过上面代码的分析，每个密码的1~4位的$b_0$为一组，3~6位的$b_3$为一组，5~4位的$b_0$和1~2位的$b_3$为一组，这三组的值就是大关编号（暂时命名为`stage`），当然也要相等；另外每位密码的$\overline{b_2 b_1}$相加必须能被4整除。

接下来以`C91BB8`这个密码（29关，从小记到现在）为例，验证密码规则是否正确：

| 密码位  | 二进制  | $ b_3 $ | $\overline{b_2 b_1}$ | $ b_0 $ |
| ---- | ---- | ------- | -------------------- | ------- |
| C    | 1100 | 1       | 10                   | 0       |
| 9    | 1001 | 1       | 00                   | 1       |
| 1    | 0001 | 0       | 00                   | 1       |
| B    | 1011 | 1       | 01                   | 1       |
| B    | 1011 | 1       | 01                   | 1       |
| 8    | 1000 | 1       | 00                   | 0       |

$(29-1)\div4=7$，即$0111_2$，而三组stage值也都是$0111_2$，而且每位的$\overline{b_2 b_1}$相加的和为$100_2$，确实能被4整除。

然后我们再来手动生成一个密码，并在实际游戏中验证规则是否正确，由于这个游戏总共有56关（虽然理论上最多可以有64关），我们就选最后一个场景的起始关卡第53关，$(53-1) \div 4 = 13 = 1101_2$，也就是说三组`stage`的值是`1101`。就有以下密码：

`0001(1)`, `1001(9)`, `1000(8)`, `1001(9)`, `0001(1)`, `1001(9)`

在这里，由于中间两位随机生成嫌麻烦，就全用0代替（反正0加起来截取低两位也是00）于是就获得了密码`198919`，进入游戏选CONTINUE，输入密码，按START，成功进入53关。

## 三. 密码生成器

就用js随便写一个吧。

```html
<!doctype html>
<html>
  <head>
    <title>摩艾君密码生成器</title>
    <meta charset='utf-8'></meta>
  </head>
  <body>
    请选择关卡：<select id='ddlLevels'></select>
    <button id='btnGenerate' name='btnGenerate'>生成</button>
    <div id="password"></div>
    <script src="https://cdn.bootcss.com/jquery/3.4.1/jquery.slim.min.js"></script>
    <script>
      function convert(passwordArray) {
        var password = ''
        for (var i in passwordArray) {
          var p = passwordArray[i]
          if (p >= 0 && p <= 9) {
            password += String.fromCharCode(p + 0x30)
          }
          else if (p >= 0x0A && p <= 0x0F) {
            password += String.fromCharCode(p + 0x37)
          }
        }
        return password
      }
      
      function generate(level) {
        var password = [0, 0, 0, 0, 0, 0]
        var pswIndexes = [0, 1, 2, 3, 2, 3, 4, 5, 5, 4, 0, 1]
        var rsCounts = [0, 0, 0, 0, 3, 3, 3, 3, 0, 0, 3, 3]
        for (var i = 2; i >= 0; i--) {
          var stage = (level - 1) >> 2
          for (var j = 3; j >= 0; j--) {
            var dataIndex = i*4 + j
            var bit = (stage % 2) << rsCounts[dataIndex]
            stage >>= 1
            password[pswIndexes[dataIndex]] |= bit
          }
        }
        return convert(password)
      }
      
      $('#btnGenerate').click(function(){
        var level = $('#ddlLevels').val()
        $("#password").html("密码为：" + generate(level))
      })
      
      $(document).ready(function(){
        for (var i=0; i<=13; i++) {
          var startLevel = i*4 + 1
          var endLevel = i*4 + 3
          var eleHtml = "<option value='" + startLevel + "'>" + startLevel + '~' + endLevel + "</option>"
          $('#ddlLevels').append(eleHtml)
        }
      })
    </script>
  </body>
</html>
```

