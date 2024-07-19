# Spark Application ID 引发的惨案
我们于22年开始，将任务迁移到 Spark on k8s, 23年年底，将 Spark 任务全部转换成 DBT 任务，通过 dbt run 执行任务，2024年年中，任务量开始膨胀到几千个，
任务达到几千个之后，并发任务开始变多，出现同一时间点，有多个任务同时跑的情况。

Spark 3.3 之前，Spark Application ID 的默认生成格式: ```"spark-application-" + System.currentTimeMillis```

当出现两个 Spark 任务同一时刻开始启动时，就会出现共用 APP ID的情况，Spark On K8S 模式下，任务跑完之后，会根据 app id 去删除用完的资源文件，
如果两个任务共用 app id，就会出现一个任务跑完后，立即删除所有的资源文件，导致未跑完的任务，一直拿不到资源的情况。describe pod 的话，会出现 ```Error: MountVolume.SetUp failed for volume "spark-conf-volume-exec"``` 情况。

解决方案:
- 将 image 升级到 3.3 版本（比较困难，需要做很多验证工作）
- 在 3.2 上打 patch, 将 app_id 的格式替换成: ```"spark-app-" + System.currentTimeMillis + "-" + UUID.randomUUID().toString()```
- 在 dbt 测自定义 app_id, spark.app.id = ```"spark-app-" + System.currentTimeMillis + "-" + UUID.randomUUID().toString()```

目前的这三个方案中，我们选择了第三个，改起来简单，个人更倾向于第二种方案，成本低且在一段时间内能够一劳永逸


