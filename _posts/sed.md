---

title: 关于sed的使用与理解

date: 2021-01-10 11:26:40

---



![sed_title](https://azou.tech/blog/static/image/sed_title.png)



### sed理解

#### sed 原理
两个空间

#### sed 使用格式

`sed OPTIONS... [SCRIPT] [INPUTFILE...]`

#### sed  options解释

-n
--quite
--silent
		默认情况下，sed脚本执行的时候在每次循环都打印出模式空间中的内容，这个选项会将这个自动打印关闭，并且只有明确的使用`p`		命令的时候才会打印
		打印：输出到控制台；

-e script
--expression=script

​		在脚本中添加一系列的命令，在运行过程中命令将会被执行

#### sed 脚本中的命令

> **这里特指的是上面格式中`SCRIPT`中使用到的命令，不是指`OPTIONS`**

a

p

s


### sed常用

### sed小技巧

sed引用所有匹配到的字符串

使用 & 符号，如：sed ‘s/a.b/-&-/g’，将 a.b 匹配到的内容替换为 -a.b- 。
替换中要加入 & 字符，需要在 & 前添加转义字符，如：\&。
&：引用所有匹配到的字符串

### sed使用案例

`sed -i.bakup -E -e 's/\\"/"/g' -e 's/"\[/\[/g' -e 's/\]"/\]/g' -e 's/"\{/\{/g' -e 's/\}"/\}/g' ${file_name} ;`



convert understore to camelCase，参考：https://unix.stackexchange.com/questions/416656/underscore-to-camelcase

`echo "remote_available_packages" | sed -E 's/_(.)/\U\1/g'`

### sed好玩的




### 参考
- https://www.gnu.org/software/sed/manual/sed.html#Command_002dLine-Options

