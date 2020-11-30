# Maven

## 1. dependencyManagement和dependencies的区别 

- dependencyManagement一般用在父层，统一**声明（只是声明）**一些依赖版本。
- 父层声明后，子层就无需规定version/scope, 如果子类想自定义版本，可以在子层自己定义
- 因此只有子层用到，才会去导包，父层只是声明