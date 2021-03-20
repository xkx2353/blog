---

title: 关于etcd

date: 2020-11-23 20:15:17

---
##### 基础特点

1. Go语言实现
2. 高可用分布式键值对数据库
3. raft协议实现的一致性算法
4. http交互
5. 读取：由于集群所有节点数据是强一致性的，读取可以从集群中随便哪个节点进行读取数据
6. 写入：etcd集群有leader，如果写入往leader写入，可以直接写入，然后然后Leader节点会把写入分发给所有Follower，如果往follower写入，？？？？

##### 常见问题

1. 安装过程遇到版本问题，由于v3和v2由比较大的差异，所以一些命令混用了导致一些令人困惑的问题，后续测试全部使用v3



##### Docker 启动

```sh
docker run -d -v /usr/share/ca-certificates/:/etc/ssl/certs -p 4001:4001 -p 2380:2380 -p 2379:2379 \
 --name etcd quay.io/coreos/etcd:v3.0.16 \
 -name etcd0 \
 -advertise-client-urls http://${HostIP}:2379,http://${HostIP}:4001 \
 -listen-client-urls http://0.0.0.0:2379,http://0.0.0.0:4001 \
 -initial-advertise-peer-urls http://${HostIP}:2380 \
 -listen-peer-urls http://0.0.0.0:2380 \
 -initial-cluster-token etcd-cluster-1 \
 -initial-cluster etcd0=http://${HostIP}:2380 \
 -initial-cluster-state new
 
 //	2380 和集群中其他的节点通信
 //	2379/4001 提供HTTP API服务，供客户端交互
```




##### 好玩的
```lua
-- 配合各种Escape Sequence
string.rep('hahaha\t', 10)
```
##### 相关介绍



##### 参考
