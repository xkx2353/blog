---

title: vim使用

date: 2020-08-09 19:45:05

sticky: 9998

---

![ADM-3A](https://azou.tech/blog/static/image/ADM-3A.png)

#### 思维模式

> Just a text editor, not a IDE

对于大部分(可以说是百分之九十九)人来说,对于文本编辑就是:噼里啪啦输入,中间夹杂着一些删除,最后一个回车,这看着很普通的一系列操作,但是对于经常要文本编辑的工作者来说,可能就不会很好(不是不好,看个人习惯,仅个人看法).需要将这个过程细化,才可以更加高效;我们面对的其实是:单个字符,单个字符串,单个符号(`()`,`{}`,`[]`,`''`,`""`,`<>`)范围内,单行,单个段落,文本开始,文本末尾等的选择,匹配,替换,移动,编辑,复制,删除等操作.只要将思维想明白了,也就对vim没有那么迷惑了.
#### Unix-like Editor History
> 熟悉了计算机文本编辑的历史发展,会对vim有一个更好的理解

**ed** is a line editor for Unix and Unix-like  operating systems. It was one of the first parts of the Unix operating system that was developed, in August 1969.

ed text editor was one of the first three key elements of the Unix operating system—assembler, editor, and shell—developed by Ken Thompson in August 1969 on a PDP-7 at AT&T Bell Labs.

![ed_is_the_standard_text_editor](https://azou.tech/blog/static/image/ed_is_the_standard_text_editor.png)

![Rob Pike twitter bio](https://azou.tech/blog/static/image/RobPikeTwitter.png)

![Rob Pike](https://azou.tech/blog/static/image/RobPikePhoto.png)

`ed--->ex`

**ex**, short for EXtended,is a line editor for Unix systems originally written by Bill Joy in 1976. **The original Unix editor, distributed with the Bell Labs versions of the operating system in the 1970s, was the rather user-unfriendly ed.** George Coulouris of Queen Mary College, London, which had installed Unix in 1973, developed an improved version called em in 1975 that could take advantage of video terminals. While visiting Berkeley, Coulouris presented his program to **Bill Joy, who modified it to be less demanding on the processor; Joy's version became exand got included in the Berkeley Software Distribution.**

![bill joy](https://azou.tech/blog/static/image/vi_creator_bill_joy.png)

vi是ex行编辑器的可视模式,ex是vi的底层行编辑器

**ex** was eventually given a **full-screen visual interface** (adding to its command line oriented operation), thereby becoming the vi text editor. In recent times, ex is implemented as a personality of the vi program; most variants of vi still have an "ex mode", which is invoked using the command ex, or from within vi for one command by typing the : (colon) character. **Although there is overlap between ex and vi functionality, some things can only be done with ex commands, so it remains useful when using vi**.

![keyboard_layout](https://azou.tech/blog/static/image/keyboard_layout.png)

Bruce Englar encouraged Joy to redesign the editor, he did June through October 1977 adding a full-screen visual mode to ex—which came to be vi

**vi** and ex share their code; vi is the ex binary launching with the capability to **render the text being edited onto a computer terminal**—it is ex's visual mode. **The name vi comes from the abbreviated ex command (vi) to enter the visual mode from within it**. 

------

**vim**

Vim (a contraction of Vi IMproved) is a free and open-source, screen-based text editor program. **It is an improved clone of Bill Joy's vi. Vim's author, Bram Moolenaar, derived Vim from a port of the Stevie editor for Amiga and released a version to the public in 1991.** Vim is designed for use both from a command-line interface and as a standalone application in a graphical user interface. 

**Since its release for the Amiga, cross-platform development has made it available on many other systems**. In 2006, it was voted the most popular editor amongst Linux Journal readers; in 2015 the Stack Overflow developer survey found it to be the third most popular text editor, and in 2019 the fifth most popular development environment.

Stevie, ST Editor for VI Enthusiasts, is a discontinued clone of Bill Joy's vi text editor. Stevie was written by Tim Thompson for the Atari ST in 1987. **It later became the basis for Vim, which was released in 1991.**

Amiga is a family of **personal computers** introduced by Commodore in 1985. 

**vi enhancements**

Some of Vim's enhancements include **completion**, comparison and merging of files (known as **vimdiff**), a comprehensive integrated help system, **extended regular expressions**, **scripting languages** (both native and through alternative scripting interpreters such as Perl, Python, Ruby, Tcl, etc.) including **support for plugins**, a graphical user interface (known as gvim), limited integrated development environment-like features, mouse interaction (both with and without the GUI), **folding**, editing of **compressed or archived files** in gzip, bzip2, zip, and tar format and files over network protocols such as SSH, FTP, and HTTP, **session state preservation**, **spell checking**, **split (horizontal and vertical) and tabbed windows**, Unicode and other multi-language support, **syntax highlighting**, trans-session command, search and cursor position histories, **multiple level and branching undo/redo history** which can persist across editing sessions, and visual mode.

#### 必须会

> 某些场景完成一些必须的操作,比如登录服务器做一些事情

##### 打开移动编辑退出

------

#### vim中的模式

常用的模式:NORMAL, INSERT,  VISUAL, VISUAL BLOCK, REPLACE, COMMAND_LINE

还是看online doc 吧，:h vim-modes

在vim实用技巧中,有一段话写的特好:**一个画家虽然直接在画布上画画的时间很多,但是他们做的最多的工作研究主题,调整光线,把颜料混合成新的色彩等等.所以说画家在休息的时候不把画笔放在画布上.和画家一样,程序员,也只会花一部分时间来编写代码,绝大部分的时间都是在思考,阅读,以及在代码中穿梭浏览**. 仔细揣摩,就会明白不同模式存在的意义.

#### 必备技能

##### 查文档

> ## Always use the built-in help

`h help`

`h 02.8` `h help-summary`

#### 基本操作

##### 稍微快点

###### 移动

移动范围从小到大

1. 单行内
   1. 字符
   2. 词
   3. 行首
   4. 行尾
   5. 行中
   6. 查找跳转
      1. 向前
      2. 向后

2. 跨行
   1. 跳转到固定行
      1. 固定行数
      2. 按照百分比来跳转

   2. 文件开始
   3. 文件结尾
   4. 调整当前行
      1. 到屏幕视图头部
      2. 到屏幕视图中间
      3. 到屏幕视图尾部
   5. 翻屏
      1. 光标到屏幕的位置
         1. 上方
         2. 中间
         3. 下方

      2. 屏幕移动
         1. 下一屏
         2. 上一屏
         3. 下半屏
         4. 上半屏
         5. 屏幕慢慢上下滚动

   6. 括号的妙用
      1. 句子
      2. 段落
   7. 搜索查找

3. 跨文件
   1. 标记


###### 开始编辑

1. 字符前
2. 字符后
3. 行开始
4. 行结尾

###### 文本选择

> motion这个还是挺重要的

`h text-objects`

###### 删除

###### 批量操作

使用visual block 模式

###### 重复操作

##### 工具技能

###### 查找匹配

> 说一下最基本的,其他的需要另外在开一篇,好好讲一讲正则

###### 替换

> substitute

#### 稍微复杂点

##### 宏操作

> 这个挺好

##### 命令行模式

> h ex-cmd-index

##### 编辑

###### buffer

###### tab

###### window

##### 寄存器

#### 其他功能

##### 配置

> 根据自己的使用习惯，使用一些插件，或者写脚本，vim-script,lua,perl等
>
> 选项 例 set nu

##### 映射

##### 折叠

#### 无处不在的vim

##### IdeaVim

[Github Repo](https://github.com/JetBrains/ideavim)

> IdeaVim is a Vim engine for JetBrains IDEs.
>
> save ourselves a few hand movements and be even more productive
>
> 这个插件非常强大

##### Surfingkeys

[Github Repo](https://github.com/brookhong/Surfingkeys)

> Surfingkeys和现有的一些插件一样，让你尽可能的通过键盘来使用Chrome/Firefox浏览器，比如跳转网页，上下左右滚屏。但不只是给vim用户使用

##### 其他

> Vscode, Sublime 等等

------

# **下面的还没有整理,但是也可以看**

------

#### 字符的含义

- `a:append,ascii`
- `b:backward`
- `c:change,close`
- `d:delete`
- `e:edit,end`
- `f:find,forward,fold`
- `g:global,go` 
- `h:left` 
- `i:insert,ident`
- `j:down,join `
- `k:up`
- `l:right`
- `m:mark,mode,motion`
- `n:repeat search`
- `o:`
- `p:put,paste`
- `q:quit`
- `r:read,repeat,replace`
- `s:substitute`
- `t:till,tab,tag`
- `u:undo`
- `v:visual`
- `w:write,window,word`
- `x:delete`
- `y:yank`
- `z:fold`
#### 几个列表
- `jumplist`
- `changelist`
#### 命令行模式的强大

#### 总是记不住的一些指令
> do & undo

```
normal mode
u: undo last change (can be repeated to undo preceding commands)

Ctrl-r: Redo changes which were undone (undo the undos).
Compare to . to repeat a previous change, at the current cursor position.
Ctrl-r (hold down Ctrl and press r) will redo a previously undone change, wherever the change occurred.
A related command is:
U: return the last line which was modified to its original state (reverse all changes in last modified line)
U is not actually a true "undo" command as it does not actually navigate undo history like u and Ctrl-r. This means that (somewhat confusingly) U is itself undo-able with u; it creates a new change to reverse previous changes.
U is seldom useful in practice, but is often accidentally pressed instead of u, so it is good to know about.

```
> 每一行进行重复操作
> 将 :normal 命令和 `.` 或者 宏 结合起来  就非常强大了

```
语法: `:{range}norm[al][!] {commands}`
normal命令使得在命令行模式执行普通模式命令成为可能。撤销操作会撤销所有的命令。
如果某行执行命令时发生错误，不会影响其他行的命令执行，即vim normal命令是在所有目标行上 并行 执行。这里的的 并行 意在类比并联电路的健壮性，从本质上来说，vim仍然是顺序地执行宏，而不会真正地并发执行多处修改。
normal命令中的可选参数 ! 用于指示vim在当前命令中不使用任何vim映射；如果没有显式使用 ! 选项，即便是执行一个非递归映射 (noremap) 命令，它的参数仍有可能被重新映射。
例如，假设已经设置了vim映射 :nnoremap G dd，则在vim普通模式按下 G 将执行命令 dd，即会删除一整行；此时，若在vim命令行模式下执行命令 :normal G 同样将删除当前行而不会跳转到当前文件的末行。
为了在即便 G 命令已经被设置了映射的条件下也能在vim normal命令中不改变 G 命令原始的含义，需要使用 :normal! G。通过 ! 选项显式指示Vim在当前命令中不使用任何vim映射。
所以，在任何时候写Vim脚本时，都建议总是使用 normal!，永远不要使用 normal 而给自己埋下不确定性的问题。

```

> [How do I reload the current file?](https://vi.stackexchange.com/questions/444/how-do-i-reload-the-current-file)

The command you want is :e (short for :edit). If you use :edit! it will discard local changes and reload.

Another way  You can actually invoke this prompt using the `:checktime` command.

> 下划线替换成驼峰 a_b---> aB

`%s/_\(\w\)/\=toupper(submatch(1))/cg`

> vim 编辑完成的时候需要root权限保存

`:w !sudo tee %`

> vim 批量操作宏

使用qe开启录制，然后记录操作，再按q录制完毕（注意换行j的位置）
normal模式下使用@e执行一次，使用10@e执行10次

> vim 复制

先使用yaw等命令复制内容(可以放到你习惯的寄存器中),在插入模式下，点击CTRL-R然后输入寄存器的标识符，可以粘贴相应寄存器中的内容到当前位置

#### vim中的正则
The Vim editor uses regular expressions to specify what to search for.
Regular expressions are an extremely powerful and compact way to specify a search pattern. Unfortunately, this power comes at a price, because regular expressions are a bit tricky to specify.
So, do it.

Tricks:

- 1.The ^ character matches the beginning of a line.
- 2.The $ character matches the end of a line.
- 3.The . (dot) character matches any existing character;If you really want to match a dot, you must avoid its special meaning by putting a backslash before it.If you search for "ter.", searching for "ter\." can get you want.
- 4.set ignorecase smartcase.If you have a pattern with at least one uppercase character, the search becomes case sensitive;If you want to ignore case for one specific pattern, you can do this by prepending the "\c" string.  Using "\C" will make the pattern to match case.This overrules the 'ignorecase' and 'smartcase' options, when "\c" or "\C" is used their value doesn't matter.**{only Vim supports \c and \C}**
- 5.To turn off search wrapping, use the following command:set nowrapscan
- 6.'/const/e[num]':The "e" offset indicates an offset from the end of the match;'/const/b[num]': mean that the cursor moves to the beginning of the pattern.
- 7.The "*" item specifies that the item before it can match any number of times.Thus:` /a*`matches "a", "aa", "aaa", etc.  But also "" (the empty string), because zero times is included.
- 8.To match a whole string multiple times, it must be grouped into one item.  This is done by putting "\(" before it and"\)" after it.  Thus this command:
`/\(ab\)*` Matches: "ab", "abab", "ababab", etc.  And also "".

- 9.To avoid matching the empty string, use "\+".  This makes the previous item match one or more times `/ab\+`.
- 10.To match an optional item, use "\=".  Example: `/folders\=` Matches "folder" and "folders".
- 11.To match a specific number of items use the form "\{n,m}".  "n" and "m" are numbers.  The item before it will be matched "n" to "m" times inclusive.
Example:`/ab\{3,5}` matches "abbb", "abbbb" and "abbbbb".

- 12.To match as few as possible, use "\{-n,m}". For example, use:
`/ab\{-1,3}` Will match "ab" in "abbb".  Actually, it will never match more than one b;
"\{-}".  This matches the item before it zero or more times, as few as possible.  The item by itself always matches zero times.
It would try to match as many characters as possible with ".*".

- 13.The "or" operator in a pattern is "\|".
- 14.A related item is "\&".  This requires that both alternatives match in the
same place.  The resulting match uses the last alternative.  Example:
`/forever\&...` This matches "for" in "forever".  It will not match "fortuin",
attention the three dots;

- 15.To match "a", "b" or "c" you could use "/a\|b\|c".  When you want to match all letters from "a" to "z" this gets very long. There is a shorter method:/[a-z],The [] construct matches a single character.  Inside you specify which characters to match.  You can include a list of characters, like this:/[0123456789abcdef] This will match any of the characters included.  For consecutive characters you can specify the range.  "0-3" stands for "0123".  "w-z" stands for "wxyz".Thus the same command as above can be shortened to:/[0-9a-f]
To match the "-" character itself make it the first or last one in the range.

- 16.Predefined:

```
        \s      whitespace character: <Space> and <Tab>         /\s
        \S      non-whitespace character; opposite of \s        /\S
        \d      digit:                          [0-9]           /\d
        \D      non-digit:                      [^0-9]          /\D
        \x      hex digit:                      [0-9A-Fa-f]     /\x
        \X      non-hex digit:                  [^0-9A-Fa-f]    /\X
        \o      octal digit:                    [0-7]           /\o
        \O      non-octal digit:                [^0-7]          /\O
        \w      word character:                 [0-9A-Za-z_]    /\w
        \W      non-word character:             [^0-9A-Za-z_]   /\W
        \h      head of word character:         [A-Za-z_]       /\h
        \H      non-head of word character:     [^A-Za-z_]      /\H
        \a      alphabetic character:           [A-Za-z]        /\a
        \A      non-alphabetic character:       [^A-Za-z]       /\A
        \l      lowercase character:            [a-z]           /\l
        \L      non-lowercase character:        [^a-z]          /\L
        \u      uppercase character:            [A-Z]           /\u
        \U      non-uppercase character:        [^A-Z]          /\U
        \e      matches <Esc>                                   /\e
        \t      matches <Tab>                                   /\t
        \r      matches <CR>                                    /\r
        \b      matches <BS>                                    /\b
        \n      matches an end-of-line                          /\n
```

- 17.The "\f" items stands for file name characters.  Thus this matches a sequence of characters that can be a file name.
 Which characters can be part of a file name depends on the system you are using. Actually, Unix allows using just about any character in a file name, including white space.

- 18."\s" matches white space, "\_s" matches white space or a line break.
Similarly, "\a" matches an alphabetic character, and "\_a" matches an
alphabetic character or a line break.The other character classes and ranges
can be modified in the same way by inserting a "_".
Many other items can be made to match a line break by prepending "\_".  For
example: "\_." matches any character or a line break.

- 19."\<" and "\>" are used to find only whole words.
- 20.`\&`:"foobeep\&..." matches "foo" in "foobeep";`".*Peter\&.*Bob" `matches in a line containing both "Peter" and "Bob".
- 21.

```
        /star   *       \*      0 or more       as many as possible
        /\+     \+      \+      1 or more       as many as possible
        /\=     \=      \=      0 or 1          as many as possible
        /\?     \?      \?      0 or 1          as many as possible

        /\{     \{n,m}  \{n,m}  n to m          as many as possible
                 \{n}    \{n}   n               exactly
                 \{n,}   \{n,}  at least n      as many as possible
                 \{,m}   \{,m}  0 to m          as many as possible
                 \{}     \{}    0 or more       as many as possible (same as *)

        /\{-    \{-n,m} \{-n,m} n to m          as few as possible
                \{-n}   \{-n}   n               exactly
                \{-n,}  \{-n,}  at least n      as few as possible
                \{-,m}  \{-,m}  0 to m          as few as possible
                \{-}    \{-}    0 or more       as few as possible
```

- 22.zero-width::

```
        /\@>    \@>     \@>     1, like matching a whole pattern
        /\@=    \@=     \@=     nothing, requires a match /zero-width
        /\@!    \@!     \@!     nothing, requires NO match /zero-width
        /\@<=   \@<=    \@<=    nothing, requires a match behind /zero-width
        /\@<!   \@<!    \@<!    nothing, requires NO match behind /zero-width
```

- 23.magic/nomagic::
Use of "\m" makes the pattern after it be interpreted as if 'magic' is set,ignoring the actual value of the 'magic' option.
Use of "\M" makes the pattern after it be interpreted as if 'nomagic' is used.
Use of "\v" means that after it, all ASCII characters except '0'-'9', 'a'-'z',
'A'-'Z' and '_' have special meaning: "very magic".
Use of "\V" means that after it, only a backslash and terminating character
(usually / or ?) have special meaning: "very nomagic".S
**only Vim supports \m, \M, \v and \V**

- 24.`\@>` Observe this difference: "a*b" and "a*ab" both match "aaab", but in the second case the "a*" matches only the first two "a"s.  "\(a*\)\@>ab" will not match "aaab", because the "a*" matches the "aaa" (as many "a"s as possible), thus the "ab" can't match.

- 25.

 ``` 
    \%V  Match inside the Visual area. `/\%Vxuan\%V`
    \%#  Example, to highlight the word under the cursor.`/\k*\%#\k*`
    \%'m    'Matches with the position of mark m.
    \%<'m   'Matches before the position of mark m.
    \%>'m   'Matches after the position of mark m.
    \%23l   Matches in a specific line.
    \%<23l  Matches above a specific line (lower line number).
    \%>23l  Matches below a specific line (higher line number).
    \%23c   Matches in a specific column.
    \%<23c  Matches before a specific column.
    \%>23c  Matches after a specific column.
    \%23v   Matches in a specific virtual column.
    \%<23v  Matches before a specific virtual column.
    \%>23v  Matches after a specific virtual column.
    
 ```

- 26.
```
    \1      Matches the same string that was matched by the first sub-expression in \( and \)
    \2      Like "\1", but uses second sub-expression.
    \9      Like "\1", but uses ninth sub-expression.
```

- 27.\%[]:A sequence of optionally matched atoms. This always matches. It matches as much of the list of atoms it contains as possible.  Thus it stops at the first atom that doesn't match. For example: /r\%[ead] matches "r", "re", "rea" or "read".  The longest that matches is used.


#### 参考
- https://github.com/mhinz/vi-editor-history
- ed
  - https://www.gnu.org/software/ed/
  - https://www.chedan.io/posts/ed-is-the-standard-text-editor/
  - https://levelup.gitconnected.com/why-ed-is-the-standard-text-editor-bf45f8f21a3a  上面的英文版
- https://ex-vi.sourceforge.net/
- [Here is why vim uses hjkl keys as arrow keys](https://catonmat.net/why-vim-uses-hjkl-as-arrow-keys)
- https://vimhelp.org/
- https://yianwillis.github.io/vimcdoc/doc/help.html
- https://vimjc.com
- https://vim.fandom.com/wiki/Vim_Tips_Wiki
- https://book.douban.com/subject/25869486/ [英文版:https://www.oreilly.com/library/view/practical-vim-2nd/9781680501629/]
- https://vimawesome.com/ 
