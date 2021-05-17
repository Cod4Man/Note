# MyBatisPlus

## 1. 整合Druid

### 1.1 异常java.sql.SQLFeatureNotSupportedException

- 原因：版本冲突
- 解决方案： 
  - Druid 1.1.21修复了对MyBatisPlus3.2以上版本的支持
  - 将mybatis-plus的版本降至3.2.0或以下 
  - 修改LocalDateTime为Date