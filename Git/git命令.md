# git命令

## 0. 创建工程并关联到远程

```shell
## 进入本地项目
git init
git add .
git commit -m "first commit"
[git branch -M main]
git remote add origin git@github.com:Cod4Man/closeframework.git
git push -u origin main
```



## 1.  克隆代码

`git clone   xxx.git[仓库url.git]`

## 2. 分支

### 2.1 创建分支

```shell
git branch branch2 创建一个分支
git checkout branch2 切换到创建好的分支

# 一步到位，创建并切换分支
git checkout -b branchname 创建branchname分支并切换至
```

### 2.2 查看分支

```shell
git branch 查看所有分支
git branch -r 查看所有远程分支
git branch -a 查看所有本地分支和远程分支
```

### 2.3 切换分支

```shell
git checkout dev(分支名)
git checkout - 切换至上一个分支
```

### 2.4 分支合并

```shell
git checkout master
git merge dev 合并dev分支到本地当前分支（此时只是本地合并）
git push  (会将本地合并后的分支提交到远程仓库)
```

### 2.5 删除分支

```shell
# 先切换到主干
git checkout master
# 删除
git branch -D <分支名>
```



## 3. 拉取代码

```shell
git pull [remote] [branch]
eg.  git pull origin master (dev) // 拉取远程master分支到本地dev分支
```

## 4. 查看代码修改状态

```shell
git status
```

## 5. 添加文件并提交

```shell
git add . （添加所有，也可指定路径）
git commit -m '提交描述'
```

## 6. 推送提交

```shell
git push 
```

## 7. 获取远程状态

```shell
git fetch 
```

## 8. 代码冲突

### 8.1 pull时提示需要提交本地冲突文件先 

```shell
$ git push origin master
From https://github.com/Cod4Man/Note
 * branch            master     -> FETCH_HEAD
Updating a08cf2c..5f09fd6
error: Your local changes to the following files would be overwritten by merge:
        Git/test
Please commit your changes or stash them before you can merge. // 提示先commit(到本地)后merge
Aborting

ASUS@LAPTOP-S6B6C46G MINGW64 /e/SoftwareNote (master)
```

#### 8.1.1 先commit冲突文件到本地仓库

```shell
$ git status
$ git add xxx
$ git commit -m ''
```

#### 8.1.2 不能push/merge，而是要做pull 

```shell
$ git pull
// git merge
$ git merge origin/master
Updating a08cf2c..5f09fd6
error: Your local changes to the following files would be overwritten by merge:
        Git/test
Please commit your changes or stash them before you can merge.
```

#### 8.1.3 pull从远程拉代码下来，会尝试合并冲突文件，合并失败则需要手动合并

#### 8.1.4 需要再次commit文件，然后push到远程

```shell
$ git status
$ git add xxx
$ git commit -m ''
$ git push origin master
```

#### 8.1.5 远程文件不再改动，则以本地文件覆盖远程文件



## 9. 本地已Commit文件回滚(本地仓库) git revert

```shell
$ git revert head a74de7a3cf9b82973a724be1dd3c50f8e54cab37 // 回到到该commit-id的版本
[master e420ee2] Revert "commit git命令.md"
 5 files changed, 3 insertions(+), 77 deletions(-)
 rename {GitHub => Git}/GitHub.md (100%)
 delete mode 100644 "Git/git\345\221\275\344\273\244.md"
 create mode 100644 Git/test
 delete mode 100644 "\351\235\242\350\257\225\345\207\206\345\244\207/JAVASE/JVM
[master 38179d5] Revert "xx"
 1 file changed, 2 insertions(+), 1 deletion(-)

ASUS@LAPTOP-S6B6C46G MINGW64 /e/SoftwareNote (master)
```

## 10. 文件对比

```shell
git diff xx
```



## 11. git控制台中文显示unicode  \u547d\u4ee4

```shell
在使用git的时候，经常会碰到有一些中文文件名或者路径被转义成\xx\xx\xx之类的，此时可以通过git的配置来改变默认转义
具体命令如下：
git config core.quotepath false
```



## 12. Git工作流

- master：主干分支
- develop：开发主分支
- feature_goldstyle/fuature_game：开发功能模块分支

1. 场景1：开发到一半，master的releaseV1.0发现严重生产bug

   - 开发分支暂停开发/或者抽出一个人解决bug
   - 由masterV1.0版本新建一个分支hotfix，**修复完成后合并到master发布V1.1版本**，**此时hotfix还不能删除，需要同步到开发主分支develop**
   - 当feature_goldstyle/fuature_game开发功能模块分支开发完成后，**各自与开发主分支master合并**(此时develop已经带有hotfix分支内容)
   - 然后在develop分支测试，测试发现bug，可以和master发现bug一样，在develop的基础上开一个新分支修复bug，然后再合并到develop分支

   

![1621994901333](E:\SoftwareNote\Git\img\Git工作流.png)