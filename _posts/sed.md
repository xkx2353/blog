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

```shell
# 👇三行意思相同
sed 's/hello/world/' input.txt > output.txt
sed 's/hello/world/' < input.txt > output.txt
cat input.txt | sed 's/hello/world/' - > output.txt
```

#### sed  options解释

-n 
--quite
--silent
		默认情况下，sed脚本执行的时候在每次循环都打印出模式空间中的内容，这个选项会将这个自动打印关闭，并且只有明确的使用`p`		命令的时候才会打印

-e script
--expression=script

​		在脚本中添加一系列的命令，在运行过程中命令将会被执行

```bash
# 关于-e 和-f
sed 's/hello/world/' input.txt > output.txt

sed -e 's/hello/world/' input.txt > output.txt
sed --expression='s/hello/world/' input.txt > output.txt

echo 's/hello/world/' > myscript.sed
sed -f myscript.sed input.txt > output.txt
sed --file=myscript.sed input.txt > output.txt
```

 -i 

​	to edit files in-place instead of printing to standard output

​	`sed -i 's/hello/world/' file.txt` 直接对file文件进行修改

​	这个使用一定要小心，注意备份

 	-i 后面跟着一个可选的参数 所以Because -i takes an optional argument, it should not be followed by other short options	

```bash
sed -Ei '...' FILE
Same as -E -i with no backup suffix - FILE will be edited in-place without creating a backup.

sed -iE '...' FILE
This is equivalent to --in-place=E, creating FILEE as backup of FILE
```

`bash -c "sed -i/Users/xukaixuan/*  "s/A/a/" b.txt"`

bash 下使用* 可以进行前缀或者备份，gnu文档里面都有写 。zsh下执行的时候报错：zsh:no matches found:

-s

​	consider files as separate rather than as a single, continuous long stream

#### sed 脚本中的命令

> **这里特指的是上面格式中`SCRIPT`中使用到的命令，不是指`OPTIONS`**

a

p

​	print specific lines  

​	`sed -n '45p' file.txt`  prints only line 45 of the input file

​	`sed '45p' file.txt`  所有行都会被打印出来，45行会被打印两次

​	`sed -n  '1p ; $p' one.txt two.txt three.txt` 打印one.txt的第一行和three.txt的最后一行
​	`sed -n -s  '1p ; $p' one.txt two.txt three.txt` 打印每个文件的第一行和最后一行

w

​	writing output to other files

y/src/dst/

Transliterate any characters in the pattern space which match any of the source-chars with the corresponding character in dest-chars.

l

Print the pattern space in an unambiguous form



​	
### sed模式空间和保留空间

> 通过 sed --debug 查看详细的流程，不要被网上传来传去的博客误导了

sed一次处理一行内容。处理时，把当前处理的行存储在临时缓冲区中，称为“模式空间”（pattern space），接着用sed命令处理缓冲区(pattern space)中的内容，处理完成后，把缓冲区(pattern space)的内容送往屏幕。接着清空缓冲区(pattern space)，处理下一行，这样不断重复，直到文件末尾。

pattern space（模式空间）相当于车间sed把流内容在这里处理；
hold space（保留空间）相当于仓库，加工的半成品在这里临时储存（当然加工完的成品也在这里存储）。

工作原理：

先读入一行，去掉尾部换行符，存入pattern space，执行编辑命令。
处理完毕，除非加了-n参数，把现在的pattern space打印出来，在后边打印曾去掉的换行符。
把pattern space内容给hold space，把pattern space置空。
接着读下一行，处理下一行。

一种非平凡情况，一个文件仅一行，尾部没换行，sed只打印，不会尾部加换行，但若在尾部又附加了输出，他会再补上那个换行。

通过在pattern space和hold space直接移动数据的一系列命令可以搞出很多花样。

p

​	Print the pattern space.「官网只有这么一句话，通过debug看到是在默认输出之前执行了p命令的输出」

P

​	打印当前模式空间开端至\n的内容，「通过debug看到是在默认输出之前执行了p命令的输出」

n

​	(next) If auto-print is not disabled, print the pattern space, **then, regardless, replace the pattern space with the next line of input**. If there is no more input then `sed` exits without processing any more commands.

N

**Add a newline to the pattern space, then append the next line of input to the pattern space**. If there is no more input then `sed` exits without processing any more commands.

x 

​	交换pattern space和hold space的内容，第一次hold space是空的

d

​	Delete the pattern space; immediately start next cycle.

D

​	If pattern space contains newlines, delete text in the pattern space up to the first newline, and **restart cycle with the resultant pattern space, without reading a new line of input.**

​	If pattern space contains no newline, start a normal new cycle as if the `d` command was issued.

g

Replace the contents of the pattern space with the contents of the hold space.

G

Append a newline to the contents of the pattern space, and then append the contents of the hold space to that of the pattern space.

h

(hold) Replace the contents of the hold space with the contents of the pattern space.

H

Append a newline to the contents of the hold space, and then append the contents of the pattern space to that of the hold space.

###sed跳转

:label 设置一个标记
b label 主要用来条件分支
t label 主要用来循环分支
当然某些情况下两个都可以使用

sed -n ':label s/xkx/kai/p; /xkx/t label' a.xkx
替换xkx成kai，如果还可以匹配到就跳转到label接着执行替换；

```shell
printf '%s\n' a1 a2 a3 | sed -E '/1/bx ; s/a/z/ ; :x ; y/123/456/'
a4
z5
z6

printf '%s\n' a1 a2 a3 | sed -E '/1/!s/a/z/ ; y/123/456/'
a4
z5
z6
```




### sed常用

### sed小技巧

sed引用所有匹配到的字符串

使用 & 符号，如：sed ‘s/a.b/-&-/g’，将 a.b 匹配到的内容替换为 -a.b- 。
替换中要加入 & 字符，需要在 & 前添加转义字符，如：\&。
**&：引用所有匹配到的字符串**

**sed -n '/^$/=' filename，打印出匹配的行号**

### sed使用案例

`sed -i.bakup -E -e 's/\\"/"/g' -e 's/"\[/\[/g' -e 's/\]"/\]/g' -e 's/"\{/\{/g' -e 's/\}"/\}/g' ${file_name} ;`



convert understore to camelCase，参考：https://unix.stackexchange.com/questions/416656/underscore-to-camelcase

`echo "remote_available_packages" | sed -E 's/_(.)/\U\1/g'`

查询匹配的最后一项然后做一些事情：正确处理转义符

`sed -i "$(sed -n '/}/ =' code.xkx | tail -n 1)"' s/}/BY_YOU_END\n&/' code.xkx`



模式空间和保留空间的使用案例

```bash
cat a.txt
1
2
3
4
#-------
sed 'N;p' a.xkx
1
2
1
2
3
4
3
4%
#-------
sed 'N;N;p' a.xkx
1
2
3
1
2
3
4%
```



### sed好玩的

终端直接执行`sed 's/hello/world/'`

If no -e, --expression, -f, or --file option is given, then the first non-option argument is taken as the sed script to interpret.  All
remaining arguments are names of input files; if no input files are specified, then the standard input is read.


### 参考
- https://www.gnu.org/software/sed/manual/sed.html#Command_002dLine-Options
- https://blog.noroot.net/archives/297/
