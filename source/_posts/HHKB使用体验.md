---
title: HHKB使用体验
date: 2017-11-13 02:40:53
categories: 周边
type: "tags"
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

#### 参考文献

- [HHKB使用体验](http://www.jianshu.com/p/4ddd5de1081e)
- [HHKB入坑指南](http://yannisxu.farbox.com/post/hhkb-chun-xiao-bai-ru-keng-zhi-nan?utm_source=tuicool)
- [配置HHKB](https://www.lanvsblue.top/2016/06/29/hhkb/)
- [黑青茶红绿灰白奶CHERRY七种轴体详解](http://www.pcpop.com/doc/0/977/977721.shtml)