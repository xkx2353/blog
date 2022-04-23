---

title: 两台Mac上同步应用配置和脚本

date: 2022-04-23 14:53:43

---

### 思路和处理过程中使用到的工具

1. 解决家用Mac mini 和公司用Mac Book Pro 两台机器部分应用配置和常用脚本的同步问题
2. 如何实现
   1. 主要通过dropbox来进行同步
      1. 有些应用提供同步功能，直接将配置存放到dropbox目录中即可，例如AIfred
      2. **有的应用没有提供同步功能，这里主要想说的就是这种**
      3. 一般文件就通过软链形式来处理
   2. Mac 应用的配置文件构成
   3. [fswatch 文件变动监听处理](https://github.com/emcrisostomo/fswatch)
   4. 关于Plist
      1. [What is a Plist?](https://discussions.apple.com/thread/1869002)
      2. [Putil使用](https://blog.csdn.net/cneducation/article/details/54729106#%E5%89%8D%E8%A8%80)

### Mac 应用的配置文件构成

1. todo



### 一些想法

1. 处理过程中对于shell语法，grep，awk 再次熟悉
2. 解决问题时，有了想法后，将想法用流程大概画一下，可以将一些问题提前暴露出来

