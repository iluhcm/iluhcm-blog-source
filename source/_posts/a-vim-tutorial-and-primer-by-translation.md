title: 【译】VIM初级与进阶教程
date: 2016-02-23 16:45:11
tags: [VIM,tutorial]
categories: Course
---

原文链接：<https://danielmiessler.com/study/vim/>

---
![](http://7xl6ic.com1.z0.glb.clouddn.com/blog_vim_splash_opt.jpg)

# 介绍

网上有许多Vim教程，但大多数要么枯燥乏味(go ninja straight away)，要么讲得太基础，没有深入地介绍，不适合新手学习。

**本教程旨在带你进入Vim编辑器这个神圣之门**，从理解Vim哲学(在使用Vim过程中将伴随你一生)开始，到Vim能够胜任并且超过当前你使用的编辑器，最后成为Vim高级使用者之一。

简而言之，我们将开始以一种伴随你一生的方式来学习Vim。

下面开始正式学习Vim。

## 为什么选择Vim?

我相信人们使用`vim`有下面这些缘由：

1. (vim教程)无处不在。你不必担心学习新的编辑器会存在各种各样的困难。
2. 可扩展。你可以仅仅用它来编辑配置文件，或者它可以成为你一生的书写平台
3. 强大。vim像[一门语言](https://danielmiessler.com/study/vim/#language)，帮助你非常快速地从沮丧的编辑状态突然为之精神一震而跃跃欲试。

总之，我相信你应该考虑使用`vim`像使用你的母语，或者基本的数学运算等等一样运筹帷幄。下面从技术角度开始认识你的未来编辑器吧。

## Vim学习阶段

[Kana the Wizard](http://whileimautomaton.net/2008/11/vimm3/operator)说过，vim进阶分为5个等级：

![](http://7xl6ic.com1.z0.glb.clouddn.com/blog_vim_kana_the_wizard_opt.jpg)

- Level 0：不认识vim
- Level 1：知道基本的vim操作
- Level 2：知道visual模式
- Level 3：知道各种各样的motions
- Level 4：不再需要visual模式

我并不了解这个，但我认为阶段等级机制值得被提及，毕竟`Kana`是个巫师。我的Vim学习方法主要基于以下四个方面：

- 入门/基础（Intro/Basics）：介绍基本概念，带你入门并熟悉基本操作，通过正确的方式来使用和思考`Vim`。
- 专业进阶（Getting Stuff Done）：这个阶段就像一块肉，需要使用叉子来吃，也许还需要一张卫生纸来擦嘴。在这个阶段的学习你似乎会感到困惑。
- 高级进阶（Advanced）：这个阶段，我将带你领略并进阶高级`Vim`使用者阶段。
- 频繁需求（Frequent Requests）：此阶段你讲能够根据需求来掌握使用`Vim`的任何技巧。

总之，如果你已经可以简单地使用Vim，你可以从`Getting Stuff Done`阶段开始并逐步学习。如果你对专业进阶已经具有扎实的基础，那就直接从高级进阶阶段开始学习。如果你是为了解决一种特殊的问题而学习`Vim`，譬如“忘记你是如何完成一件事的”，那直接从频繁需求阶段开始学习吧。

因此，Vim起步，打好基础，学习专业知识，然后开始频繁地更变需求——这四步将能够满足你使用`Vim`的需求。

## Vim配置

正如我所说，我不会把这一部分看成是最好的Vim配置。Vim有许多种配置方案。本教程目的是加强你对Vim概念的理解，并将Vim作为一个强有力的工具来使用。但我们将会讲解部分基本的配置。

首先，我推荐你通过系统自身的软件管理工具安装Vim。我通常喜欢用[Janus](https://github.com/carlhuda/janus)（Vim的一个发行版本）作为我的Vim工具，但不喜欢我不能确认Janus能否准确执行命令的事实。我最喜欢的Vim配置既简单又优雅，就跟使用Vim一样的感觉。

最后，我直接使用`~/.vim`，以及`~/.vimrc`来作为我的Vim配置目录和文件。

### 小部分快捷键变化（~/.vimrc）

首先，我认为使用`<Esc>`键离开insert模式相对过时了。Vim是效率工具，如果离开了主键程区（即使用左手的尾指按`<Esc>`键），那效率必然会大大打折。因此不要这么做。

	inoremap jk <ESC>

[NOTE：有些人喜欢把`<ESC>`键改成`jj`，但我认为手指从`j`滚动到`k`显得更自然。]

#### 修改Leader键（leader key）

Leader是一个按键，用来配合其他按键来激活快捷键，它非常强大。例如，如果你准备通过字母`c`来设置一些快捷键，那么你需要按下leader+`c`。

默认的leader键(`\`)似乎也已经过时了，所以我喜欢把leader键设置为空格键(Space)

	let mapleader = "\<Space>"
	
现在，当你正准备学习使用俏皮的快捷键时，你可以使用大拇指来完成，因为大拇指总是在空格键上。

[NOTE：感谢Adam Stankiewicz推荐我设置空格键为leader键]

#### 重映射CAPSLOCK键

这个配置不是在你的conf文件中完成，但却是一个与默认配置有偏差的具有重要影响的设置。`CAPSLOCK`键通过对我毫无用处，所以我从操作系统级别把`CAPSLOCK`键重定向为`Ctrl`键。这样，我的左尾指可以轻易地按到`CAPSLOCK`键来使用`Ctrl`功能。

接下来有一些基本配置被大多数人推荐，并使得使用起来变得更简单。

	filetype plugin indent on
	syntax on
	set encoding=utf-8

记住，你可以花费毕生的经历去优化你的`~/.vimrc`配置文件；以上这些配置仅仅是作为带你入门的简单基础教程。全部详细的配置可以参考[我的配置文件](https://github.com/danielmiessler/vim)或者看一下[参考链接](#参考链接)。

### 通过Pathogen管理插件

[NOTE：如果对Vim还没有熟悉或者觉得使用插件麻烦，跳过这一章节，待你熟悉以后再回来浏览。]

#### 摆脱Janus

对于Janus，我曾经最喜欢它管理插件的方式，但后来我转向了[Pathogen](https://github.com/tpope/vim-pathogen)。对于Pathogen，你只需要做以下的配置：

1. 安装[Pathogen](https://github.com/tpope/vim-pathogen)。
2. `git clone`插件到`~/.vim/bundle`
3. 添加`execute pathogen#infect()`到`~/.vimrc`文件里

通过以上步骤，便可以安装任何插件，并且不需要担心它们是怎么被加载的。

### 考虑使用GitHub备份配置

在Vim安装配置过程中，我将我的整个`~/.vim`目录放到了一个`git`仓库下，并托管在[我的GitHub](https://github.com/danielmiessler/vim)上。这样，我便可以随时随地地通过命令行：

	git clone https://github.com/danielmiessler/vim
	
获取我的整个vim环境。

你也可以这样做。将整个工程拷贝下来，放在用户根目录(`~/`)下，简历一个从`~/.vimrc`到`~/.vim/vimrc`的软连接就可以了。

## 将Vim作为一门语言学习

可以说，Vim让人感觉明智的地方在于你在使用它的同时，你会用它来*思考*。Vim像一门普通的语言一样，一条实现某种功能的命令由名词、动词和副词组成。

记住接下来我要用到的术语从技术角度来说并非完全正确，但能帮助你更好地理解vim如何工作。再次声明，这篇教程不能替代一本完整的vim教程或者完整的帮助文档——它旨在帮助你从vim的各种教程资源中获取不容易得到的东西。

### 动词（Verbs）

动词是我们要触发的动作，动词后面跟名词。这里有一些例子：

`d`: delete——删除  
`c`: change——改变  
`y`: yank (copy)——复制  
`v`: visually select (V for line vs. character)——进入选择模式  

### 修饰符（Modifiers）

修饰符使用在名词的前面，用来描述即将做某事的方式。例如：

`i`: inside——在……里面  
`a`: around——在……周围  
`NUM`: number (e.g.:1,2,10)——数字  
`t`: searches for something and stops before it——查找某些字符或字符串并将光标停留在字符串前面  
`f`: searches for that thing and lands on it——查找某些字符或字符串并将光标停留在字符串之后  
`/`: find a string (literal or regex)——查找字符串（文字或正则表达式）  

### 名词（Nouns）

英语中，名词就是你即将要触发某种动作的对象。在vim里面也一样，vim中的名词有：

`w`: word——名词  
`s`: sentence——句子  
`)`: sentence (another way of doing it)——句子  
`p`: paragraph——段落  
`}`: paragraph (another way of doing it)——段落  
`t`: tag (think HTML/XML)——标签（类似html/xml语法中的<TAG>）  
`b`: block (think programming)——块  

### 名词当做动词（Nouns as motion）

你也可以把名词当做动词来使用，意味着你可以使用名词来移动内容。我们将会在移动内容这个章节看到相关例子。

### 使用语法构造句子（命令）

Vim有各种各样词类型，怎样去使用它们构造句子呢？很简单，就像英语一样，用一种直观的方式把<font color="red">动词</font>、<font color="green">修饰符</font>和<font color="blue">名词</font>结合起来就可以了。

对于下面的命令，仅需要记住<font color="red">红</font><font color="green">绿</font><font color="blue">蓝</font>分别对应<font color="red">动词</font>、<font color="green">修饰符</font>和<font color="blue">名词</font>。

#### 删除两个词语

Delete two words

---

<font color="red">d</font><font color="green">2</font><font color="blue">w</font>

---

#### 删除句子并进入编辑模式

Change inside sentence (delete the current one and enter insert mode)

---

<font color="red">c</font><font color="green">i</font><font color="blue">s</font>

---

#### 复制选择的句子

Yank inside paragraph (copy the paragraph you’re in)

---

<font color="red">y</font><font color="green">i</font><font color="blue">p</font>

---

#### 删除'<'之前的所有字符

Change to open bracket (change the text from where you are to the next open bracket)

---

<font color="red">c</font><font color="green">t</font><font color="blue"><</font>

---

记住，这里的名词实际上是一个左中括号，但实际上可以是任何字符。作用在名词上的修饰符是`t`，所以可以执行命令：`dt.`（删除当前光标到下一个句号`.`之前的所有字符）或者`yt;`（复制当前光标到下一个分号`;`之前的所有字符）

是不是很美妙？使用这种思维过程可以使你编辑文字变得更直观优雅，并且像任何其他语言一样，使用得越多，就会对它爱不释手。

`<!--未完待续-->`



















