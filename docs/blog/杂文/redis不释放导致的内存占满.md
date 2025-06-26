我们有一部分上古时代的 flink 任务是用传统的 DataStream API 写的，最近有tx 去修改的时候，使用 JedisPool 时，没有显示 close，导致在 Redis 连接不释放，代码如下:

```java
DataStream<VehcileLatestState> dataStream =
          tableEnv
              .toDataStream(source_table)
              .flatMap(
                  new RichFlatMapFunction<Row, VehcileLatestState>() {
                      ObjectMapper objectMapper = new ObjectMapper();
                      JedisPool jedisPool = null;

                      @Override
                      public void open(Configuration parameters) throws Exception {
                          jedisPool =
                              new JedisPool(
                                  new GenericObjectPoolConfig<Jedis>(),
                                  "r-xx.redis.rds.xxx.com",
                                  6379,
                                  2000,
                                  "zzzz==");
                      }

                      @Override
                      public void flatMap(Row row, Collector<VehcileLatestState> collector)
                          throws Exception {
                          long partitionId = 0;
                          long kafkaOffset = 0;
                          String kafkaTimestamp = "1970-01-01 00:00:00";
                          xxxx
                          heartbeat.setHeartbeat_v2(1);
                          try (Jedis jedis = jedisPool.getResource()) {
                              String lastTimeKey = String.format("%s_%s", heartbeat.getVehicle_vin(), Constants.VEHICLE_LATEST_TIME_KEY_TAIL);
                              String latestTimestamp = jedis.get(lastTimeKey);
                              if (null != latestTimestamp
                                  && Long.parseLong(latestTimestamp) >= heartbeat.getCurrent_timestamp_seconds()) {
                                  heartbeat.setOut_order(1);
                              } else {
                                  heartbeat.setOut_order(0);
                                  if (!heartbeat.isDirty() || (null == heartbeat.getDirty_reason() || !heartbeat.getDirty_reason().equals(Constants.PARSE_EXCEPTION))) {
                                      String key = String.format("%s_%s", heartbeat.getVehicle_vin(), Constants.VEHICLE_LATEST_STATE_KEY_TAIL);
                                      jedis.setex(key, 7 * 24 * 60 * 60, objectMapper.writeValueAsString(heartbeat));
                                  }
                                  jedis.setex(lastTimeKey, 48 * 60 * 60, heartbeat.getCurrent_timestamp_seconds() + "");
                              }
                          }
                          collector.collect(heartbeat);
                      }
                  });
```

代码中 Jedis 是释放了，但是 JedisPool 没有释放，导致redis 连接不释放。



