# 数据库系统

## 数据库理论

### 1. 库表是基于 bag 模型的,而非 set 模型

数据结构 | 重复性 | 顺序性
-----|-----|----
list | 可重复 | 有顺序
set | 不可重复 | 无顺序
bag(mutiset) | 可重复 | 无顺序


## aggregates and group by

aggregate 函数只能用于 **select** 的 **output list**.

- avg(col)
- min(col)
- max(col)
- sum(col)
- count(col)

### multiple aggregates

## 数据库

```
// 构建数据库
sqlite3 bike_sharing.db < setup.sql
```

