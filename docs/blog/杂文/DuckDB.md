# DuckDB load plugin 使用不当导致的任务集体失败

## 背景
  我们有一堆任务使用了 duckdb 来拉取 http 接口的数据，每个任务之前都需要下载一遍 http_extension 安装报，最近为了降本增效，我们部分可用区的公网带宽被降级了，导致download 失败，后续一堆任务全挂了

## 过程
  在每次通过 duckdb 来拉取http接口数据时，都需要下载 http_extension，以前任务都是几分钟就跑完了，我们设置的任务超时时间是15分钟，最近几天任务都出现超时情况，超时之前任务都没有出现异常日志，
重新clear airflow 任务也不会成功，但是在接口机执行任务时，任务很快就跑完了，怀疑是出问题的可用区机器有问题，重启 airflow 任务，将任务的可用区改为跟接口机相同的可用区，发现任务很快就跑完了。
  重新提交任务，在任务的每个步骤都打印日志，发现任务主要卡死在 duckdb 拉取 http_extension 这一步。进入到 出问题任务的 Pod 上执行 curl -O xxxx.gz 也无法执行，ping google 和 github 都没有反应，确定该可用区
已经没有公网网络。

## Fix 思路
  确定无法访问公网之后，接下来的问题就是怎么加载 http_extension 的问题了。目前我们的办法是直接把 http_extension 打到 docker image 中，通过 ```LOAD '/opt/external-tools/httpfs.duckdb_extension';``` 加载

