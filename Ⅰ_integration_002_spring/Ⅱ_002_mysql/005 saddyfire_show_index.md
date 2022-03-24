`SHOW INDEX FROM <表名> [ FROM <数据库名>]`

```sql
mysql> SHOW INDEX FROM tb_stu_info2\G
*************************** 1. row ***************************
        Table: tb_stu_info2
   Non_unique: 0
     Key_name: height
 Seq_in_index: 1
  Column_name: height
    Collation: A
  Cardinality: 0
     Sub_part: NULL
       Packed: NULL
         Null: YES
   Index_type: BTREE
      Comment:
Index_comment:
1 row in set (0.03 sec)
```

其中各主要参数说明如下：

![[Pasted image 20220324110403.png]]

- 索引类型
	- normal：表示普通索引
	- unique：表示唯一的，不允许重复的索引，如果该字段信息保证不会重复例如身份证号用作索引时，可设置为unique
	- full textl: 表示 全文索引。 FULLTEXT 用于搜索很长一篇文章的时候，效果最好。用在比较短的文本，如果就一两行字的，普通的 INDEX 也可以。
	- 总结，索引的类别由建立索引的字段内容特性来决定，通常normal最常见。


- 创建索引, 删除索引
```sql
-- ALTER TABLE用来创建普通索引、UNIQUE索引或PRIMARY KEY索引。
ALTER TABLE table_name ADD INDEX index_name (column_list)
ALTER TABLE table_name ADD UNIQUE (column_list)
ALTER TABLE table_name ADD PRIMARY KEY (column_list)

-- CREATE INDEX可对表增加普通索引或UNIQUE索引。
CREATE INDEX index_name ON table_name (column_list)
CREATE UNIQUE INDEX index_name ON table_name (column_list)

-- ALTER TABLE或DROP INDEX语句来删除索引
DROP INDEX index_name ON talbe_name
ALTER TABLE table_name DROP INDEX index_name
ALTER TABLE table_name DROP PRIMARY KEY


```