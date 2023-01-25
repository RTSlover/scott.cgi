---
layout: post
title: 文件夹加密并隐藏
author: Yang
--- 

此脚本是利用windows自身的attrib把文件夹重命名并设为隐藏文件。[^1]

首先新建一个文本文档

再输入如下内容

![](https://b2.yangtze.in/pic/other/Locker.png)

```

@ECHO OFF ::关掉无关显示

CLS ::清除屏幕闲杂信息

title 文件夹上锁工具 ::命名批处理标题

if EXIST "Control Panel.{21EC2020-3AEA-1069-A2DD-08002B30309D}" goto JIESHUO

if NOT EXIST Locker goto WJJCJ

:YON ::设定输入与确认模块

echo 确认要锁定这个文件夹吗(Y/N)

set/p "cho=>"

if %cho%==Y goto SHUODING
if %cho%==y goto SHUODING

if %cho%==n goto END
if %cho%==N goto END

echo 无效选择 ::显示选择无效

goto YON ::转回到判断模块

:SHUODING ::设定锁定功能的模块

ren Locker "Control Panel.{21EC2020-3AEA-1069-A2DD-08002B30309D}"

attrib +h +s "Control Panel.{21EC2020-3AEA-1069-A2DD-08002B30309D}"

echo 文件夹已被锁定 ::显示锁定信息

goto End ::转到结束模块

: JIESHUO ::设定解锁模块

echo 输入锁定文件夹的密码

set/p "pass=>" ::定义输入变量

if NOT %pass%==0 goto WUXIAO ::判断密码的正确性

attrib -h -s "Control Panel.{21EC2020-3AEA-1069-A2DD-08002B30309D}"

ren "Control Panel.{21EC2020-3AEA-1069-A2DD-08002B30309D}" Locker

echo 文件夹成功解锁！ ::成功解锁信息显示

goto End

:WUXIAO ::设置输入无效提示模块

echo 密码无效 ::密码无效显示

goto end

:WJJCJ ::创建文件夹的模块

md Locker ::创建文件夹的命令

echo 文件夹锁建立成功

goto End

:End ::结束模块

```

再将文件后缀改为**bat**格式，双击运行，会出现一个名为**Locker**的文件夹。再次点击bat文件，输入"y"或"Y"，即可加密文件夹并隐藏。

---

如果需要设置密码，则可以以如下内容替换上面的内容。

![](https://b2.yangtze.in/pic/other/private.png)

```

cls
@ECHO OFF
title Folder Private
if EXIST "HTG Locker" goto UNLOCK
if NOT EXIST Private goto MDLOCKER
:CONFIRM
echo 你确定要加密隐藏Private文件夹吗？(Y/N)
set/p "cho=>"
if %cho%==Y goto LOCK
if %cho%==y goto LOCK
if %cho%==n goto END
if %cho%==N goto END
echo Invalid choice.
goto CONFIRM
:LOCK
ren Private "HTG Locker"
attrib +h +s "HTG Locker"
echo Folder locked
goto End
:UNLOCK
echo 输入密码来解锁文件夹
set/p "pass=>"
if NOT %pass%== 在此处设置密码 goto FAIL
attrib -h -s "HTG Locker"
ren "HTG Locker" Private
echo Folder Unlocked successfully
goto End
:FAIL
echo Invalid password
goto end
:MDLOCKER
md Private
echo Private created successfully
goto End
:End

```

其中**在此处设置密码**指的是可自行设置的文件夹密码。

再将文件后缀改为**bat**格式。双击运行，输入"y"，即可加密并隐藏文件夹。再次点击bat文件，输入你设置的密码，即可加密文件夹打开。

---

[^1]:参考文章：[无需软件加密文件夹](https://jingyan.baidu.com/article/6fb756ec98a317241858fbfe.html)
