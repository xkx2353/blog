---

title: 关于SSH

date: 2020-10-28 18:11:19

---
##### ssh知识相关
- SSH只是一种协议，存在多种实现，既有商业实现，也有开源实现；OpenSSH，PuTTY「Windows」
-  的

##### ssh服务端配置

##### ssh端口转发

- 本地端口转发（Local Port Forwarding）

	ssh -N -f -g  -L localhost:20000:localhost:49160 azou@azou.tech
	
	**访问本地的20000端口，通过ssh隧道，访问的是azou.tech的49160端口**

- 远程端口转发（Remote Port Forwarding）

	ssh -N -g -f -R 18098:localhost:8098 azou.tech
	
	**访问的是azou.tech的18098端口，通过ssh隧道，访问本地的8098端口**
	
- 动态端口转发（Dynamic Port Forwarding）



##### 好玩的
```lua	
-- 什么是灿烂
```



##### 参考
- http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html

