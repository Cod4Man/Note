# Maven

## 1. scope

|                    | compile | test | provided |
| :----------------: | :-----: | :--: | :------: |
|     主程序有效     |    √    |  ×   |    √     |
|    测试程序有效    |    √    |  √   |    √     |
|   参与打包/部署    |    √    |  ×   |    ×     |
| 依赖可传递到子模块 |    √    |  ×   |    ×     |

### 1.1 compile

### 1.2 test

典型：JUnit

### 1.3 provided

典型：servlet-api.jar

## 2. 依赖的原则（jar冲突）

### 2.1 路径最短优先

### 2.2 路径相同，先声明者优先

