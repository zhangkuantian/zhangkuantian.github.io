我们从 2023 年年底引入 DBT，2024 年开始大规模迁入，到2025-06 初，DBT 任务量达到 4000+，都在一个 repo 里

每次 Merge 一个 MergeRequest 的时候，构建 airflow 的 DAG，在 airflow DAG 中，每个 dbt task 执行一个 dbt model，执行命令 ```dbt run --select {model_name}```
自 2025-06-09 开始，出现部份 model 出现 spark-session 长时间无法启动的情况，直到 airflow task timeout

初步排查，可能会出现 Spark Application ID 重复的情况，根据 [Spark Application ID 引发的惨案](Spark%20Application%20ID%20引发的惨案.md) 重新组织了一遍 Spark Application ID, 确认不可能再
出现重复的情况后上线
结果第二天还是出现了相同的问题，只是概率许多。
把日志切换成 info 模式，也没看到打日志的情况。

我们的 Spark 是 Spark on K8S perjob 模式，每个 Job 都是 on aliyun ECI, 跑完之后，eci instance 就清理的。

我们白天少量重跑，无法复现，但是通过观察 ECI 的资源利用率，发现在 DBT 的 compile 阶段，pod 的CPU 利用率已经达到 100%，初步排查大概率是 compile 阶段 CPU 没有及时释放，导致在启动 Spark Driver
的时候，获取不到 CPU，我们给每个 dbt task 的资源是 1核4G，显然如果 compile 阶段资源不释放的话，就会出现 driver 拿不到资源，无法启动的问题

2025-06-19 开始将 dbt submit pod 的规格升级为 2核8G 之后，compile 的 CPU 资源利用率降低到了 70，截止到 2025-06-26， 不再出现 Driver 起不来的情况。

总结：dbt compile 有点坑，可能需要优化，不然后面任务量过多，会导致 CPU利用率飙升，每次都调用 ```show table extended in {database} like '*'```, HMS 扛不住 dbt compile/run 的冲击



