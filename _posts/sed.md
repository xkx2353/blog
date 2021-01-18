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

​	`sed -n  '1p ; $p' one.txt two.txt three.txt` 打印one.txt的第一行和three.txt的最后一行
​	`sed -n -s  '1p ; $p' one.txt two.txt three.txt` 打印每个文件的第一行和最后一行

s

​	

w

​	writing output to other files

d

​	


### sed常用

### sed小技巧

sed引用所有匹配到的字符串

使用 & 符号，如：sed ‘s/a.b/-&-/g’，将 a.b 匹配到的内容替换为 -a.b- 。
替换中要加入 & 字符，需要在 & 前添加转义字符，如：\&。
&：引用所有匹配到的字符串

sed -n '/^$/=' filename，打印出匹配的行号

### sed使用案例

`sed -i.bakup -E -e 's/\\"/"/g' -e 's/"\[/\[/g' -e 's/\]"/\]/g' -e 's/"\{/\{/g' -e 's/\}"/\}/g' ${file_name} ;`



convert understore to camelCase，参考：https://unix.stackexchange.com/questions/416656/underscore-to-camelcase

`echo "remote_available_packages" | sed -E 's/_(.)/\U\1/g'`

查询匹配的最后一项然后做一些事情：正确处理转义符

`sed -i "$(sed -n '/}/ =' code.xkx | tail -n 1)"' s/}/BY_YOU_END\n&/' code.xkx`

### sed好玩的

终端直接执行`sed 's/hello/world/'`

If no -e, --expression, -f, or --file option is given, then the first non-option argument is taken as the sed script to interpret.  All
remaining arguments are names of input files; if no input files are specified, then the standard input is read.


### 参考
- https://www.gnu.org/software/sed/manual/sed.html#Command_002dLine-Options

