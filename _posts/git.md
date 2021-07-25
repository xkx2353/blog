---

title: 关于git的使用与理解

date: 2020-08-18 09:26:40

---
##### 终端下的git使用

```latex
# reverting 中想终止
git cherry-pick --abort
# revert merge commit -m 表示保留的分支
git revert -m 1 bd86846
# git push --delete origin tagname 删除远程的tag
git push --delete origin v1.0

```
##### git reset

```latex
git reset --soft 仅仅修改HEAD,work tree 和 index 不会变动, 需要重新commit 和 push
git reset --mixed 修改HEAD,index; work tree 不会变动, 需要重新 add commit 和 push
git reset --hard HEAD,index; work tree 全部都会变动, 目录中的变化会全部丢失,但是只要是提交到git的都是可以找回的 可以通过:git reflog 找一找

```

##### git update-index  工作树中的文件内容注册到索引

```latex

# 用户承诺该文件不会发生变化,然后暂时不会被git管理
git update-index --assume-unchanged xkx.txt
# 恢复上一条命令的修改
git update-index --no-assume-unchanged xkx.txt

```

##### git merge tool

```latex
参考: https://www.scootersoftware.com/support.php?zz=kb_vcs_osx

```


##### git diff 输出的内容的意思

```
diff --git a/_posts/git.md b/_posts/git.md
index 2654564..407cc29 100644
--- a/_posts/git.md
+++ b/_posts/git.md
@@ -4,11 +4,14 @@ date: 2020-08-18 09:26:40
 ---
- fdsfsd
- dfdsf
+ abdc
+ dfdf
```

进行比较的是，a版本的/_posts/git.md（即变动前）和b版本的/_posts/git.md（即变动后）。 

index 2654564..407cc29 100644

表示两个版本的git哈希值，（index区域的2654564对象，与工作目录区域的407cc29对象进行比较），最后的六位数字是对象的模式（100代表普通文件，644代表权限）。

--- a/_posts/git.md   "---"表示变动前的版本

+++ b/_posts/git.md  "+++"表示变动后的版本

\ No newline at end of file 最后一行没有换行

@@ -4,11 +4,14 @@

代表的意思是源文件的第4行开始，向后偏移11行与目标文件的第4行开始，向后偏移14行有差异,下面才是具体的差异信息;

-红色部分表示减少的部分

+绿色部分表示增加的部分


##### git底层原理
参考:https://git-scm.com/book/zh/v2/Git-%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86-%E5%BA%95%E5%B1%82%E5%91%BD%E4%BB%A4%E4%B8%8E%E4%B8%8A%E5%B1%82%E5%91%BD%E4%BB%A4

内部如何实现的



##### 参考
