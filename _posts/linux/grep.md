---

title: grep

date: 2021-12-04 23:54:02

---

# grep理解

一个有意思的事情(昨天看到的),grep的这个命令从哪里来的呢?
 Its name comes from the ed command `g/re/p` (global / regular expression search / and print), which has the same effect.



# 常用的脚本

``` shell
// 
 egrep 'openTime:\d*ms' chat_timeout.xkx

// Prints only the matching part of the lines.
egrep -o 'openTime:\d*ms' chat_timeout.xkx

//  Only a count of selected lines is written to standard output
egrep -o -c 'openTime:\d*ms' chat_timeout.xkx

// Only the names of files containing selected lines are written to standard output.
egrep -o -l 'openTime:\d*ms' chat_timeout.xkx


// 在多个文件中查找匹配

egrep -o  'openTime:\d*ms' *.xkx

egrep -o  'openTime:\d*ms' chat_timeout.xkx chat_timeout_bak.xkx


```



# 总结
1. 使用命令的时候多看man
