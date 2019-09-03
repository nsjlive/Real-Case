### MySQL 慢查询优化

##### 开启 MySQL 慢查询日志

一个起步就不简单的原因是，我们如何才能定位到那些真正形成瓶颈的慢查询。一个普通项目中的 SQL 可能就有大几十甚至上百个，而「凶手们」就藏匿其中。

一个**朴素**的想法是在项目中每一个 SQL 执行前后打上时间戳来估计执行时间，暂且不论由于各种因素的影响这种估算可能不准确，更让人不可接受的是这对**原始代码**造成的极大的**侵入**。

好在 MySQL 提供了慢查询日志。这个日志会记录所有执行时间超过 long_query_time（默认是 10s）的 SQL 及相关的信息。

在 MySQL 的 Console 中：

```mssql
mysql> show variables like 'long_query_time';
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+
1 row in set

mysql> show variables like 'slow_query%';
+---------------------+--------------------------------------+
| Variable_name       | Value                                |
+---------------------+--------------------------------------+
| slow_query_log      | OFF                                  |
| slow_query_log_file | /var/log/mysql/log-slow-queries.log  |
+---------------------+--------------------------------------+
2 rows in set
```

在这里，slow_query_log 指的是慢查询日志是否开启，slow_query_log_file 指明了日志所在的位置。

在 MySQL 的配置文件 my.cnf 的 [mysqld] 项下可以配置慢查询日志开启，一般来讲如下配置即可：

```sql
[mysqld]
slow_query_log=1
slow_query_log_file=/var/log/mysql/log-slow-queries.log
long_query_time=2
```

需要注意的是，应该赋予 slow_query_log_file 指向的目录 mysql 用户的写入权限：chown mysql.mysql -R /var/log/mysql，一般用默认的目录就好。详细的文档可以看这里：[6.4.5 The Slow Query Log](https://link.zhihu.com/?target=https%3A//dev.mysql.com/doc/refman/5.7/en/slow-query-log.html)。

之后需重载 MySQL 配置生效：/etc/init.d/mysql reload。

##### 分析慢查询日志

在开启了 MySQL 慢查询日志一段时间之后，日志中就会把所有超过 long_query_time 的 SQL 记录下来。另一个有用的相关 MySQL 命令是 mysqldumpslow：由于慢查询日志可能很大或者很难分析，使用它可以获得 MySQL 对慢查询日志的一个总结报告，直接获得我们想要的统计分析后的结果。详细的文档可以看这里：[5.6.8 mysqldumpslow — Summarize Slow Query Log Files](https://link.zhihu.com/?target=https%3A//dev.mysql.com/doc/refman/5.7/en/mysqldumpslow.html)。

当然，你也可以打开日志文件自己来查询、分析。

##### 优化 SQL

在得知哪些 SQL 是慢查询之后，我们就可以定位到具体的业务接口并针对性的进行优化了。

首先，你要看是**否能在不改变现有业务逻辑的前提下改进查询的速度**。一个典型的场景是，你需要查询数据库中是否存在符合某个条件的记录，返回一个布尔值来表示有或者没有，一般用于通知提醒。如果程序员在撰写接口时没把性能放在心上，那么他就有可能写出 **SELECT count(*) FROM tbl_xxx WHERE XXXX** 这样的查询，当数据量一大时（而且索引不恰当或没有索引）这个**查询会相当之慢**，但如果改成 SELECT id FROM tbl_xxx WHERE XXXX LIMIT 1 这样来查询，对速度的提升则是巨大的。这个例子并不是我凭空捏造的，最近在实际项目中我就看到了跟这个例子一模一样的场景。

能够找到上述的通过改变查询方式而又不改变业务逻辑的慢查询是幸运的，因为这些场景往往意味着只需重写 SQL 语句就能带来显著的性能提升，而且稍有经验的程序员在一开始就不会写出能够明显改良的查询语句。**在绝大多数情况下，SQL 足够复杂而且难以做任何有价值的改动，这时就需要通过优化索引来提升效率了**。

如何更好的创建数据库索引绝对是一门技术活，我也并不觉得简简单单就能厘得很清楚，很多时候还是得具体 SQL 具体分析，甚至多条 SQL 一起来分析。可以先读一读这篇美团点评技术团队的文章：[MySQL索引原理及慢查询优化](https://link.zhihu.com/?target=http%3A//tech.meituan.com/mysql-index.html)，更深入的了解则可以阅读[《高性能MySQL》](https://link.zhihu.com/?target=https%3A//book.douban.com/subject/23008813/)一书。引用一下美团点评技术团队文章中提到的几个原则：

> 1. 最左前缀匹配原则，非常重要的原则，mysql会一直向右匹配直到遇到范围查询(>、<、between、like)就停止匹配，比如a = 1 and b = 2 and c > 3 and d = 4 如果建立(a,b,c,d)顺序的索引，d是用不到索引的，如果建立(a,b,d,c)的索引则都可以用到，a,b,d的顺序可以任意调整；
> 2. =和in可以乱序，比如a = 1 and b = 2 and c = 3 建立(a,b,c)索引可以任意顺序，mysql的查询优化器会帮你优化成索引可以识别的形式；
> 3. **尽量选择区分度高的列作为索引**,区分度的公式是count(distinct col)/count(*)，表示字段不重复的比例，比例越大我们扫描的记录数越少，唯一键的区分度是1，而一些状态、性别字段可能在大数据面前区分度就是0，那可能有人会问，这个比例有什么经验值吗？使用场景不同，这个值也很难确定，一般需要join的字段我们都要求是0.1以上，即平均1条扫描10条记录；
> 4. **索引列不能参与计算**，保持列“干净”，比如from_unixtime(create_time) = ’2014-05-29’就不能使用到索引，原因很简单，b+树中存的都是数据表中的字段值，但进行检索时，需要把所有元素都应用函数才能比较，显然成本太大。所以语句应该写成create_time = unix_timestamp(’2014-05-29’);
> 5. **尽量的扩展索引**，不要新建索引。比如表中已经有a的索引，现在要加(a,b)的索引，那么只需要修改原来的索引即可。

常用的套路是**在定位到慢查询语句**之后，使用 EXPLAIN + SQL 来了解 MySQL 在执行这条数据时的一些细节，比如是否进行了优化、是否使用了索引等等。基于 Explain 的返回结果我们就可以根据 MySQL 的执行细节进一步分析是否应该优化搜索、怎样优化索引。

就在这几天，美团技术团队开源了一款用于分析如何优化 SQL 的工具——[SQLAdvisor](https://link.zhihu.com/?target=https%3A//github.com/Meituan-Dianping/SQLAdvisor)，有兴趣可以试试。

优化索引也并不是万能的，这种情况下只能考虑通过其他方式来缓和性能上的瓶颈了。

> 查询容易，优化不易，且写且珍惜！



转载自：[MySQL 慢查询优化](http://maples7.com/2017/03/08/mysql-run-faster/)



