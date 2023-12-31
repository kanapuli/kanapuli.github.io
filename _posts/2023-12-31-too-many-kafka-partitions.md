## Too Many Kafka Partitions

---

I had lunch with my ex-manager yesterday and was hearing their team's problem with Kafka. They needed more consumer parallelism to increase their throughput and hence his team planned to increase 500 more partitions for their Kafka topic since parallelism in Kafka is tied directly to the topic partitions.
