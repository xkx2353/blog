---

title: 关于git的使用与理解

date: 2020-08-18 09:26:40

---
##### 终端下的git使用技巧

```latex
# reverting 中想终止
git cherry-pick --abort
# revert merge commit -m 表示保留的分支
git revert -m 1 bd86846
# git push --delete origin tagname 删除远程的tag
git push --delete origin v1.0

```



##### git diff 输出的内容的意思

```
diff --git a/_posts/git.md b/_posts/git.md
index 2654564..407cc29 100644
--- a/_posts/git.md
+++ b/_posts/git.md
@@ -4,11 +4,14 @@ date: 2020-08-18 09:26:40
 ---
 ##### 终端下的git使用技巧

-```
-#reverting 中想终止
+```latex
+# reverting 中想终止
 git cherry-pick --abort
 # revert merge commit -m 表示保留的分支
 git revert -m 1 bd86846
+# git push --delete origin tagname 删除远程的tag
+git push --delete origin v1.0
+
 ```
```

进行比较的是，a版本的/_posts/git.md（即变动前）和b版本的/_posts/git.md（即变动后）。 

index 2654564..407cc29 100644

表示两个版本的git哈希值，（index区域的2654564对象，与工作目录区域的407cc29对象进行比较），最后的六位数字是对象的模式（100代表普通文件，644代表权限）。

--- a/_posts/git.md   "---"表示变动前的版本

+++ b/_posts/git.md  "+++"表示变动后的版本

\ No newline at end of file 最后一行没有换行

@@ -4,11 +4,14 @@

代表的意思是源文件的4-11行与目标文件的4-14行有差异,下面才是具体的差异信息;

-红色部分表示减少的部分

+绿色部分表示增加的部分


##### git理解

##### 



##### git好玩的
```

```





##### 参考
- git book

