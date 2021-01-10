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



#### sed 脚本中的命令

p 


### sed常用

### sed小技巧

### sed使用案例



`sed -i.bakup -E -e 's/\\"/"/g' -e 's/"\[/\[/g' -e 's/\]"/\]/g' -e 's/"\{/\{/g' -e 's/\}"/\}/g' ${file_name} ;`

### sed好玩的




### 参考
- https://www.gnu.org/software/sed/manual/sed.html#Command_002dLine-Options

