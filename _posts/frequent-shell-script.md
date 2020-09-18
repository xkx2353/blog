---
title: 常用的shell脚本和一些好用的命令
date: 2020-08-05 13:45:09
---

##### find相关
```shell
# 使用-exec查找并执行正则匹配
find . -name 'nohup.log-2020-07-22*' -exec grep --colour=auto -n -H -C  5 'SF2020072217505400001' {} \;
```

```shell
# 批量处理文件的一个大概流程:
#!/bin/bash
#IFS=,
#numbers="01,02,03,04,05";
numbers=(01 02 03 04 05);
dir='/Users/xukaixuan/git/Lua-Quick-Start-Guide/';

for ((i=0;i<${#numbers[*]};i++))
do
  path="Chapter${numbers[i]}";
  curPath=$dir${path};
  echo "current path: $curPath";
  find $curPath -name "*.lua" -print0 | while IFS= read -r -d '' file; do 
        sed -i '1i #!/usr/local/bin/lua' $file
        chmod u+x $file        
    done
done
```
##### ps相关
```shell
# 查看的时候把字段名字也展示出来,方便查看
ps aux | grep -E '(nginx|USER)'
```

##### 一次批量删除redis中匹配的key
```shell
redis-cli -h 192.168.4.4 -n 3 -a test --scan --pattern 'xboot:subsernorepeat:*' | xargs redis-cli -h 192.168.4.4 -n 3 -a test -n 3 del
```
##### 使用awk拼接sql
```shell
# test 源文件
# test_result 结果文件
awk -F ',' '{print "UPDATE company_account_type SET `virtual_account` = "$1"  WHERE `account_no` = "$2" AND `account_type` = "$3" AND is_deleted = 0;"}' test > test_result
```

##### maven依赖相关
```shell
# 项目依赖树
mvn dependency:tree >> project.tree
# 分析当前项目的依赖
mvn dependency:analyze >> project.analyze
```

