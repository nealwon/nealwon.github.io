---
layout: default
title: Linux部分常用快捷键
category: linux
excerpt: Linux 快捷键
tags: ["keymap", "linux"]
---

### {{ page.title }}

***

- CTRL+U    清除光标之前的内容
- CTRL+Y    粘贴上次u的内容
- CTRL+R    增量搜索命令,输入字符串后多次按组合键会继续上翻
- CTRL+P    上一条命令
- CTRL+N    下一条命令
 
- CTRL+L    清屏
- CTRL+A    光标移到行首
- CTRL+E    光标移到行尾
- CTRL+W    清除光标前一个单词
- CTRL+K    清除光标到行尾
- CTRL+T    交换光标前两个字符
- CTRL+V    开始输入控制符
- CTRL+F/B    光标前移/后移一个字符
 
- !!         上一条命令
- !-N        前面第N条命令
- !-N:p    打印前面第N条命令
- !?string?    前面最近包含string的命令
- !-N:gs/str1/str2/    将第N条命令的str1替换为str2执行
