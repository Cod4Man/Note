# Linux命令

## 1. 路径

## 1.1 当前路径

```shell
pwd
```

## 2. 文件夹

### 2.1 创建文件夹

```shell
mkdir dirName
```

### 2.2 重命名

```shell
例子：将目录A重命名为B

mv A B

例子：将/a目录移动到/b下，并重命名为c

mv /a /b/c
```

## 3. 磁盘容量

- “df  -h” 可以查看系统的分配，已使用和可用情况 
- “du -sh  *” 可以查看当前路径
- 每个文件夹的大小。此举可以快速定位大文件所存在的位置。 

## 4. 端口查看

```shell
systemctl status nginx //查看nginx状态
netstat -tnlp //查看所有运行端口号
netstat -tnlp | grep:98 //查看98端口号状态
```

## 5. 文件搜索-find

- 全盘搜索，也可以指定目录搜bai索。find 搜索目录 -name 目标名字，find / -name file（区分大小写）
- find / -iname file（不区分大小写）
- 我们先使用*通配符来匹配下包含file的相关文件区分大小写的：find / -name *file*。
- find 搜索目录 -size 文件大小。下面我们查找下大于100MB的文件,应该实际是102400KB*2,所有搜索命令为：find / -size +204800。-号是小于，直接写数字就是等于。
- find 搜索目录 -user 用户名。这里是查找属于用户名为user1的文件，linux如何添加删除用户名,可以参考Linux 用户管理命令：find / -user user1。
- find 搜索目录 -type d。查找某个目录下的所有目录：find /tmp -type d。
- find 搜索目录 -cmin -时间(单位分钟)。查找etc下面1小时内被修改的文件,根目录下面太多了,指定一个目录：find /etc -cmin -60。
- 当然find命令是可以多个选项一起添加查询的：-a 是前后条件都要满足，-o 是满足一个条件就好，这样我们可以清除的看到被过滤掉的文件。