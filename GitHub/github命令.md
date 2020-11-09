# github命令

## 1.  克隆代码

`git clone   xxx.git[仓库url.git]`

## 2. 分支

### 2.1 创建分支

```shell
git branch branch2 创建一个分支
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
git merge resourceBranch 合并resourceBranch分支到本分支（此时只是本地合并）
git push  (会将本地合并后的分支提交到远程仓库)
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



