# jenkins

## 0. 文档

### 0.1 kubernetes中部署Jenkins并简单使用

[kubernetes中部署Jenkins并简单使用 ](https://www.cnblogs.com/coolops/p/13129955.html)

### 0.2 


## 1. 快速开始

## 2. 问题

### 2.1 安装过程

#### 2.1.1 权限 permissions

```shell
touch: cannot touch ‘/var/jenkins_home/copy_reference_file.log’: Permission denied Can not write to /var/jenkins_home/copy_reference_file.log. Wrong volume permissions?
```

在启动时，权限问题无法写入文件，这是因为jenkins无法在root用户下写入，切换到当前为挂载目录分配权限即可。

* `chown -R 1000:1000 /nfs/data/jenkins`
