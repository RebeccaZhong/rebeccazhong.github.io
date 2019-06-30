### 问题：
![遇到的问题](https://raw.githubusercontent.com/RebeccaZhong/markdown-photos/master/%E4%B8%93%E4%B8%9A%E6%8A%80%E8%83%BD/db/MongoDB/MongoDB%E4%BD%BF%E7%94%A8%E7%B4%A2%E5%BC%95%E9%81%87%E5%88%B0%E7%9A%84%E9%97%AE%E9%A2%98-1.png)

### 原因：
![原因](https://raw.githubusercontent.com/RebeccaZhong/markdown-photos/master/%E4%B8%93%E4%B8%9A%E6%8A%80%E8%83%BD/db/MongoDB/MongoDB%E4%BD%BF%E7%94%A8%E7%B4%A2%E5%BC%95%E9%81%87%E5%88%B0%E7%9A%84%E9%97%AE%E9%A2%98-2.png)
[参考地址](https://cloud.tencent.com/developer/article/1185064)


### 解决：
mongodb在查询时，自动选择的索引未必是合适的索引，而且还可能在运行过程中更换查询使用的索引，虽然大部分情况下是合适的，但有时也会出现针对某些查询选用到非常不合适的索引情况。

这次正式外网上挖掘结果查询就出现了这个情况，导致查询时超时，因此若在写相关代码时，就能明确知道该查询应该使用哪个索引的时候，应当直接用hint子句进行指定，防止mongodb的查询优化器进行“逆优化”。

特别是针对某些值分布的不是那么均匀的字段的索引，很容易被判定为不是最优的索引。

这次出问题的数据是块状分布，期望使用的索引是mid字段的索引，该字段特点是相同值的记录是连续存储的，即连续多条记录，该字段的值相同，而其他地方找不到相同值的记录。
