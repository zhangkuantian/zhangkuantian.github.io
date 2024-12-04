## unified-catalog 元数据缓存引发的惨案
我们在5月份引入了 StarRocks 3.x 存算分离版本的时候，同时引入了 unified-catalog, 通过 unified-catalog 我们构建的 lakehouse 在查询的时候就不用再区分底层是 parquet、hudi、iceberg 还是 paimon，catalog 统一为 lakehouse

在创建 catalog 的时候，没有仔细查看参数设置，运行了半年之后，随着 iceberg 表不断新增，有些物化视图基于 iceberg 表创建的，物化视图的更新是通过 airflow 调度去刷新的，airflow 会在 iceberg 表任务完成后，立即去刷新物化视图。

前几天突然发现，物化视图的数据没有更新，查看日志发现 刷新任务已经执行过，iceberg 表也是只写入一遍，而且是在物化视图刷新前就已经写入了。

怀疑是元数据缓存了，导致物化视图在刷新的时候，没有及时读到 iceberg 刚刚写入的数据。

通过一个实时写 iceberg 的任务，用 spark 和 starrocks 分别实时去查询的时候发现，starrocks 会有部分查询没有及时查到数据，延迟大概在1分钟内，基本可以判断是 starrocks 里 iceberg 相关的元数据没有及时刷新。

接下来的操作就是如何在 catalog 中修改元数据同步策略。

我们打算将元数据同步缓存关闭，但是查看文档发现没有 alter catalog 的功能，只能删除重建，需要在不影响业务的前提下操作，考虑到我们的 catalog 只涉及到读，都是报表去查询的，分钟内操作完对用户的影响不大。

我们直接删除 catalog，然后重建重名 catalog。

重建后里面可以正常访问，实时任务通过 spark 和 starrocks 查询基本上没有差别了。

OK，以为到这里就结束了，没想到。。。

大概半小时后，用户反馈，报表查不到数据了。。。。。

查看日志发现，报表查询 starrocks 的用户没有权限查询新创建的 catalog 了。

擦看代码发现在删除 catalog 的时候，权限也清理了。。。。

好吧，重新给 用户授权，然后开始写事故报告。。。。。。。。

😮‍💨 没力气了




