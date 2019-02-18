## odoo中性能提升
1.最关键的地方是 索引 ， 在odoo中每张表存在的外键 可以说是不计其数，所以通常情况下的效率问题是索引，
当然.，如果用法不合理的话，也会导致严重的效率问题，最值得一提的是 odoo中的 .

2.domain的解析方法很笨，对 
```python
[('product_id', '=', 1234)，('order_line.state', '=', 'draft')]
```
这类domain的解析会把('order_id.state', '=', 'draft') 这个先进行过滤 取出order 为draft 然后 再进行拼接
```sql
select * from sale_order_line where product_id = "" and order_id in ('state 为draft‘的id 列表)
```
如果('order_id.state', '=', 'draft')  

这个条件组成的是个大表的情况下就会导致sql及其的慢。这是写法上必须注意的，当数据量小的时候没有感觉的，
3.适当的时候运用 sql 代替odoo ORM 中的domain 提取.
