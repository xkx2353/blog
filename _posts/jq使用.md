---
title: jq使用
date: 2024-01-09 11:44
tags:
  - json
---
# 日常使用

```
// 提取数组中的元素
// 主要的使用场景是从curl的结果中解析一些数据,这里的result是一个数组

echo '{"result":[{"msg":"hello"},{"msg":"world"}]}' | jq '.result|.[]|.msg'

{
  "result": [
    {
      "msg": "hello"
    },
    {
      "msg": "world"
    }
  ]
}

curl *******
 --compressed | jq '.result|.[] |.msg' >> chat_timeout.xkx


```