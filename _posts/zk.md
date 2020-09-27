---

title: 关于zk

date: 2020-09-25 16:47:17

---
##### 基础概念
##### 选举机制
##### 源码
##### 工具包
##### CLI

- 节点信息

```
ctime : 节点创建时间戳 , 将鼠标放到右侧的value上会提示成易读时间(yyyy-MM-dd HH:mm:ss SSS)
mtime : 节点最后一次被修改的时间戳, 将鼠标放到右侧的value上会提示成易读时间(yyyy-MM-dd HH:mm:ss SSS)
dataversion :  数据节点的版本号
cZxid : 节点被创建时的事务 ID
mZxid : 节点最后一次被修改时的事务 ID
dataLength : 数据长度
ephemeralOwner : 如果值为0表示永久节点 , 否则表示创建该临时节点时的 , 会话 sessionID
numChildren : 子节点数
aclversion : 节点ACL版本号
pZxid : 节点的子节点列表最后一次被修改时的事务 ID, 只有子节点列表变更才会更新 pZxid , 子节点内容变更不会更新
cversion : 子节点的版本号
```

##### 常见问题



##### 好玩的
```lua
-- 配合各种Escape Sequence
string.rep('hahaha\t', 10)
```
##### 相关介绍
	


##### 参考
