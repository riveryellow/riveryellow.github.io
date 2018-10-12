---
title: HBase filter查询数据的正确姿势
cdn: header-off
date: 2018-9-12 16:00:16
header-img: "/img/hbase"
tags:
    - HBase
---
> 由于HBase是一个菲关系型数据库，所以在查询数据时不能像MySQL那样以列值作为条件来查询。但是虽然不推荐使用使用列值来匹配查询结果，HBase还是提供了SingleColumnValueFilter来支持根据列值查询的操作。

## Java
``` java
    Table table = connection.getTable(TableName.valueOf(tableName));
    Scan sc = new Scan();
    sc.setFilter(new SingleColumnValueFilter(col, qualifier, op, new SubstringComparator(value)));
    ResultScanner resultScanner = table.getScanner(sc);
    Iterator<Result> iter = resultScanner.iterator();
    List<Object> result = new ArrayList<>();
    while(iter.hasNext()) {
        result.add(asClass(iter.next(), clazz, qualifier));
    }
    table.close();
```

## Hbase Shell
``` python
import org.apache.hadoop.hbase.filter.CompareFilter
import org.apache.hadoop.hbase.filter.SingleColumnValueFilter
import org.apache.hadoop.hbase.filter.SubstringComparator
scan 'access_token', {COLUMNS => 'account_id:', FILTER => SingleColumnValueFilter.new(Bytes.toBytes('account_id'), Bytes.toBytes(''), CompareFilter::CompareOp.valueOf('EQUAL'), SubstringComparator.new('123456'))}
```

## Summary
对于数据量特别大并且查询频率特别高的表，这的不要使用这样的操作。scan操作是进行全表扫描，在扫描完之前不会释放连接，所以会阻塞新的查询。另外，扫描匹配列也是相当消耗CPU的。
因此，如果有需要根据列值来查询数据的，应该及时修改表结构进行优化。