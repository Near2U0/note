[toc]



# 





```json
cat /tmp/identity.2.log|clickhouse-client --query="INSERT INTO tutorial.identity   FORMAT  JSONEachRow" --input_format_skip_unknown_fields=1 --max_insert_block_size=100000  --password



input_format_skip_unknown_fields
启用或禁用跳过多余数据的插入。

写入数据时，如果输入数据包含目标表中不存在的列，则ClickHouse会引发异常。如果启用了跳过，则ClickHouse不会插入额外的数据，也不会引发异常。

支持的格式：

JSONEachRow
CSVWithNames
TabSeparatedWithNames
TSKV
可能的值：

0-禁用。
1-启用。

```





