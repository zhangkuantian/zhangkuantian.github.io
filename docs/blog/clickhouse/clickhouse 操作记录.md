# ClickHouse 操作记录
- 2024-10-30

## 问题原因
业务数据激增，前期建表时，没有设置合理的 TTL，历史数据全留在 CK 中，导致 CK 磁盘满了

## 处理记录
- 查看磁盘占用率高的表
```sql
SELECT 
    name, 
    sum(total_bytes) AS size 
FROM system.tables 
WHERE database = 'data_pipeline' 
GROUP BY name
ORDER BY size 
```
- 根据业务需求，选择需要设置 TTL 的表及 TTL 时长
- 通过 DataX 将需要设置 TTL 的表历史数据备份到数据湖中
- 设置 TTL
```sql
-- 单 column
ALTER TABLE `xxx_db`.`xxx_table` MODIFY TTL toDate(`xxx_column`) + toIntervalDay(7);

-- 多 column 以逗号 分割
ALTER TABLE `xxx_db`.`xxx_table` MODIFY TTL toDate(`xxx_column_a`) + toIntervalDay(7), toDate(`xxx_column_b`) + toIntervalDay(7),;

```
