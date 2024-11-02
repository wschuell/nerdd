FROM confluentinc/cp-kafka:7.7.1

# Set environment variables for Kafka broker and topic
ENV KAFKA_BOOTSTRAP_SERVERS=my-cluster-kafka-bootstrap.kafka:9092
ENV KAFKA_TOPIC=system

# Use the console producer directly in the CMD
CMD echo -n "init:{}" | kafka-console-producer --broker-list $KAFKA_BOOTSTRAP_SERVERS --topic $KAFKA_TOPIC --property "parse.key=true" --property "key.separator=:"