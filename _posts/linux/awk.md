---

title: 关于awk的使用与理解

date: 2021-01-10 11:26:40

---

# awk理解
# awk日常使用

``` bash
// 打印出所有拆分后变量(多个分隔符)
awk -F '[, :]+' '{for(i=1;i<=NF;++i) print $i}' chat_timeout_one_line.xkx

```

``` bash 
// 生成sql语句
// 使用awk的一些函数处理字符串
awk -F '[, :]+' '{print "INSERT INTO `tmp_chat_time_cost` (`channel`, `open_time`, `user_cancel_time`, `close_time`, `complete_time`, `first_resp_time`, `first_exception_time`, `second_exception_time`, `third_exception_time`, `first_timeout_time`, `second_timeout_time`, `third_timeout_time`) \
VALUES (\""$4"\", \""substr($8,1,length($8)-2)"\", \""substr($10,1,length($10)-2)"\", \""substr($12,1,length($12)-2)"\", \""substr($14,1,length($14)-2)"\", \""substr($16,1,length($16)-2)"\", \""substr($18,1,length($18)-2)"\", \""substr($20,1,length($20)-2)"\", \""substr($22,1,length($22)-2)"\", \""substr($24,1,length($24)-2)"\", \""substr($26,1,length($26)-2)"\", \""substr($28,1,length($28)-3)"\");"}' chat_timeout_one_line.xkx

```
# 参考
