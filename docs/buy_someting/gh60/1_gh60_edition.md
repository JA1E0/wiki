---
title: 0x01 GH60的各种版本
date: 2020-02-16
---

2020-02-16	


> [https://medium.com/@xream/gh60-%E5%90%84%E7%89%88%E6%9C%AC%E8%AF%B4%E6%98%8E-b7c1301b38b3](https://medium.com/@xream/gh60-各版本说明-b7c1301b38b3)
>
> 作者: 小一
>
> 补充:aWEi

### GH60 各版本说明

### GH60 Rev.A

- 无灯
- 最早的方案

### GH60 Rev.B, GH60 Rev.C

- 11 灯
- GH60 Rev.C 修复了 bug, 增强了稳定性
- B5, B6, B7 都用作行列矩阵了, 背光控制需要 software PWM, 比如 https://github.com/kairyu/tmk_keyboard_custom/blob/master/keyboard/gh60/backlight.c
- 咸鱼有别人做好的,可能会便宜点70到100左右或者某宝.

### EEPW 版 GH60, TU60

- 基于 GH60 Rev.B
- 全灯
- B5, B6, B7 都用作行列矩阵了, 背光控制需要 software PWM, 比如 https://github.com/kairyu/tmk_keyboard_custom/blob/master/keyboard/gh60/backlight.c
- [某宝](https://item.taobao.com/item.htm?spm=a230r.1.14.18.5c5a53fbqgMBtL&id=543703427453&ns=1&abbucket=2#detail)

### GH60 Rev.CHN/CNY(Satan)

- 基于 GH60 Rev.B
- 全灯
- 留有 PWM: B6

### AMJ60

- 行列的 pinout 跟 GH60 Rev.B 不同
- 全灯
- 留有 PWM: B6

### XD60

- 第四/五行支持更多的配列
- 2.0 版本带 RGB 底灯
- 全灯
- B5, B6, B7 都用作行列矩阵了, 背光控制需要 software PWM, 比如 https://github.com/kairyu/tmk_keyboard_custom/blob/master/keyboard/gh60/backlight.c
- [某宝](https://item.taobao.com/item.htm?spm=a230r.1.14.60.1d443d1cvcXNho&id=546047283701&ns=1&abbucket=2#detail)