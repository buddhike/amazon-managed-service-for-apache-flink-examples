## AVRO One Topic Many Subjects

* Flink version: 1.20
* Flink API: DataStream API
* Language: Java (11)

This example demonstrates how to serialize/deserialize Avro messages in Kafka when one topic stores multiple subject types.

See [this article](https://martin.kleppmann.com/2018/01/18/event-types-in-kafka-topic.html) for more information about why it's sometimes essential to store multiple subjects in the same topic.

This sample is based on a hypothetical use case where two sensors fitted into a room emit telemetry data to a single Kafka topic. We use Confluent Schema Registry to govern the schema of telemetry messages serialized using Avro.

Schema definitions for the messages are in `.avdl` files located in [./src/main/resources/avro](./src/main/resources/avro).

This example uses Avro-generated classes (more details [below](#using-avro-generated-classes)).

A `KafkaSource` produces a stream of Avro data objects (`SpecificRecord`), fetching the writer's schema from AWS Glue Schema Registry. The Avro Kafka message value must have been serialized using AWS Glue Schema Registry.

A `KafkaSink` serializes Avro data objects as Kafka message values, and a `String`, converted to bytes as UTF-8, as Kafka message keys.

## Flink compatibility

**Note:** This project is compatible with Flink 1.15+ and Amazon Managed Service for Apache Flink.

### Flink API compatibility
This example uses the newer `KafkaSource` and `KafkaSink` (as opposed to `FlinkKafkaConsumer` and `FlinkKafkaProducer`, which were deprecated with Flink 1.15).

### Avro-generated classes

This project uses classes generated at build-time as data objects.

As a best practice, only the Avro schema definitions (IDL `.avdl` files in this case) are included in the project source code.

The Avro Maven plugin generates the Java classes (source code) at build-time, during the [`generate-sources`](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html) phase.

The generated classes are written into the `./target/generated-sources/avro` directory and should **not** be committed with the project source.

This way, the only dependency is on the schema definition file(s). If any change is required, the schema file is modified, and the Avro classes are re-generated automatically during the build.

Code generation is supported by all common IDEs like IntelliJ. If your IDE does not see the Avro classes (`AirQuality` and `RoomTemperature`) when you import the project for the first time, you may manually run `mvn generate-sources` once to force source code generation from the IDE.

### Using Avro-generated classes (SpecificRecord) in Apache Flink

Using Avro-generated classes (`SpecificRecord`) within the flow of the Flink application (between operators) or in the Flink state has an additional benefit. Flink will [natively and efficiently serialize and deserialize](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/datastream/fault-tolerance/serialization/types_serialization/#pojos) these objects, without risking falling back to Kryo.

### Running required services locally
Kafka and Confluent Schema Registry configuration is in [./docker-compose.yml](./docker-compose.yml). Start these services by running the `docker compose up` command.

### Running in IntelliJ
To start the Flink job in IntelliJ:
1. Edit the Run/Debug configuration.
2. Enable the option **"Add dependencies with 'provided' scope to the classpath"**.