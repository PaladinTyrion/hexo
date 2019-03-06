---
title: HHKB使用体验
date: 2017-11-13 02:40:53
categories: 周边
tags:
  - 周边
---

### HHKB Pro2 Type-S

HHKB非常重视手感，所以入Type-S才是最爽的，双十一打折价格￥1899，目测平时很难便宜了。不要想跟耳机一样突然搞重量级折扣。

#### Mode

Dip的模式根据使用的操作系统做相应设置。

SW1 | SW2 | SW3 | SW4 | SW5 | SW6
--- | --- | --- | --- | --- | ---
&SW2决定模式|&SW1决定模式|Del/BS(on)|left◇=>Fn(on)|Exchange ◇&&Fn(on)|Wake up(on)

根据操作系统做设置，建议如下：
- MacOSX: 011000
- Linux/Win: 101110

<!-- more -->

#### 键位&&基本快捷键

![键盘键位](keymap.jpeg)

- HHKB将Caps集成到了Tabs键上，通过Fn+Tabs触发
- 左右移动:

```
Fn + [;'/
```

- 移动光标快捷键 
    - Ctrl-F 光标前进一个字符，相当于右键（F = Forward）
    - Ctrl-B 光标后退一个字符，相当于左键（B = Backward）
    - Ctrl-P 上移一行，相当于上键（P = Previous）
    - Ctrl-N 下移一行，相当于下键（N = Next）
    - Ctrl-A 移动到一行的开头（A = Ahead）
    - Ctrl-E 移动到一行的结尾（E = End）
    - Option-B/F 为向左/右移动一个单词
- 文字删除快捷键
    - Ctrl-H BS的功能
    - Ctrl-D Del的功能
    - Ctrl-K 从光标处剪切一行
    - Option-D 删除光标后面至本单词结束
- 文字选择快捷键
    - Ctrl-Shift-A 选中从光标开始，到一行开头的所有文字
    - Ctrl-Shift-E 选中从光标开始，到一行结尾的所有文字

#### 推荐软件

- CheatSheet: 快捷键是长按command
- Karabiner: 修改keymap，设置新的组合键。
- Alfred: 快捷键设为 **Option+空格** 较好
- Chrome VIM: 用惯了Vimium，别人推荐CVim更牛逼，看个人习惯

#### 推荐Karabiner修改

- internal keyboard: Disable
- Option_R 设置为右击
- 上下左右设置为Option_L + IKJL
- Vim Mode
    - Option-V启动『Complete Vim』模式，Esc退出

#### 推荐键盘

- HHKB Pro2 Type-S: 静电电容键盘敲击感和声音最佳, 60键
- Filco Minila Air: 推荐红轴，67键。参数对比如下：

![Cherry键盘对比](cherry.jpeg)

#### 键盘轴详解

##### 名词说明

- cN：厘牛，也叫厘牛顿、克力；1N=102cN
- +-,-：当一个数值后跟随着加减号，或加号，或减号，表示在设计数值下允许有如下误差。比如：2 ± 0.6 mm，表示设计数值是2毫米，但允许有正负0.6毫米的设计误差。
- 触发：开关闭合，计算机接收到键盘电子信号。总行程：开关的轴芯(十字柱)完成整个运动过程的位移长度。
- 触发行程：轴芯部分完成触发的位移长度。
- 初始压力：外物下按开关在轴芯即将发生位移时受到的压力。
- 触发压力：外物下按开关到触发时受到的压力。
- 段落压力：外物下按开关到段落点时受到的压力。
- 触底压力：外物下按开关到底部时受到的压力。
- 其他:
    - 对于无段落特点的开关，如黑、灰、红是用触发压力来描述这个开关的特性。
    - 对于有段落特点的开关，如青、茶、绿、白、奶等是用段落压力来描述这个开关的特性。

##### 各类轴参数

1. 黑轴
    - 总行程：4-0.4mm
    - 触发行程：2±0.6 mm
    - 初始压力：30 cN min
    - 触发压力：60 ± 20 cN
    - 段落压力：无
    - 段落行程：无
    - 触底压力：90 cN max
2. 灰轴
    - 总行程：4-0.4mm
    - 触发行程：2±0.6 mm
    - 初始压力：30 cN min
    - 触发压力：80 ± 25 cN
    - 段落压力：无
    - 段落行程：无
    - 触底压力：110-120cN
3. 红轴
    - 总行程：4-0.4mm
    - 触发行程：2±0.6 mm
    - 初始压力：30 cN min
    - 触发压力：45 ± 15 cN
    - 段落压力：无
    - 段落行程：无
    - 触底压力：60 cN min
4. 青轴
    - 总行程：4-0.5mm
    - 触发行程：2.2±0.6 mm
    - 初始压力：25 cN min
    - 触发压力：50± 15 cN
    - 段落压力：60± 15 cN
    - 段落行程：1.75 mm
    - 触底压力：60 cN min
5. 绿轴
    - 总行程：4-0.5mm
    - 触发行程：2.2±0.6 mm
    - 初始压力：25 cN min
    - 触发压力：70± 15 cN
    - 段落压力：80± 15 cN
    - 段落行程：1.75 mm
    - 触底压力：90 cN min
6. 茶轴
    - 总行程：4-0.4mm
    - 触发行程：2±0.6 mm
    - 初始压力：30 cN min
    - 触发压力：45± 15 cN
    - 段落压力：55± 15 cN
    - 段落行程：1.25 mm
    - 触底压力：60 cN min
7. 奶轴
    - 总行程：4-0.5mm
    - 触发行程：2.2±0.6 mm
    - 初始压力：30 cN min
    - 触发压力：70± 20 cN
    - 段落压力：80± 20 cN
    - 段落行程：2 mm
    - 触底压力：90 cN max
8. 白轴
    - 总行程：4-0.5mm
    - 触发行程：2±0.6 mm
    - 初始压力：40± 15 cN
    - 触发压力：55± 20 cN
    - 段落压力：65± 20 cN
    - 段落行程：1.25 mm
    - 触底压力：100 cN min

![键轴参数](compare.jpeg)

#### 参考文献

- [HHKB使用体验](http://www.jianshu.com/p/4ddd5de1081e)
- [HHKB入坑指南](http://yannisxu.farbox.com/post/hhkb-chun-xiao-bai-ru-keng-zhi-nan?utm_source=tuicool)
- [配置HHKB](https://www.lanvsblue.top/2016/06/29/hhkb/)
- [黑青茶红绿灰白奶CHERRY七种轴体详解](http://www.pcpop.com/doc/0/977/977721.shtml)