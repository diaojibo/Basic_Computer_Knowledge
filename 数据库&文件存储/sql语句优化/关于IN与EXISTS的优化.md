## IN和EXISTS
SQL查询中in和exists的区别分析：

``` sql
select * from A where id in (select id from B);

select * from A where exists (select 1 from B where A.id=B.id);
```

对于以上两种情况，in是在内存里遍历比较，而exists需要查询数据库，所以**当B表数据量较大时，exists效率优于in**。

IN()只执行一次，它查出B表中的所有id字段并缓存起来,因此它会在B表进行全表扫描，并且缓存起来。

exists()会执行A.length次，它并不缓存exists()结果集，因为exists()结果集的内容并不重要，重要的是其内查询语句的结果集空或者非空，空则返回false，非空则返回true。

所以，**通常情况下采用exists要比in效率高，因为IN不走索引**。但要看实际情况具体使用：

 - 当A表数据与B表数据一样大时，in与exists效率差不多，可任选一个使用。
 - IN适合于外表大而内表小的情况，内表小，则放进内存处理效率更高
 - EXISTS适合于外表小而内表大的情况。内表大，所以用索引每次进行EXISTS检索就好。
