# Apache Kafka - Source

The Apache Kafka Event source enables Knative Eventing integration with Apache
Kafka. When a message is produced to Apache Kafka, the Apache Kafka Event Source
will consume the produced message and post that message to the corresponding
event sink.

## Deployment steps

1. Setup [Knative Eventing](../../DEVELOPMENT.md)
1. If not done already, install an Apache Kafka cluster!

   - For Kubernetes a simple installation is done using the
     [Strimzi Kafka Operator](http://strimzi.io). Its installation
     [guides](http://strimzi.io/quickstarts/) provide content for Kubernetes and
     Openshift.

   > Note: The `KafkaSource` is not limited to Apache Kafka installations on
   > Kubernetes. It is also possible to use an off-cluster Apache Kafka
   > installation.

1. Now that Apache Kafka is installed, apply the `KafkaSource` config:

   ```
   ko apply -f config/
   ```

1. Create the `KafkaSource` custom objects, by configuring the required
   `consumerGroup`, `bootstrapServers` and `topics` values on the CR file of
   your source. Below is an example:

   ```yaml
   apiVersion: sources.eventing.knative.dev/v1alpha1
   kind: KafkaSource
   metadata:
     name: kafka-source
   spec:
     binding: 
       name: kafka-binding
     consumerGroup: knative-group
     # Broker URL. Replace this with the URLs for your kafka cluster,
     # which is in the format of my-cluster-kafka-bootstrap.my-kafka-namespace:9092.
     bootstrapServers: REPLACE_WITH_CLUSTER_URL
     topics: knative-demo-topic
     sink:
       ref:
         apiVersion: serving.knative.dev/v1alpha1
         kind: Service
         name: event-display
   ```

If there is already existing Kafka configuration you can use it to provide default values that can be overriden in yaml file above and additional properties specific to Kafka. For example:

```bash
$ cat kafka.properties
sasl.username="T7f3xuwnKG0Q5V71" 
sasl.password="Xwup9VRp9diu4CD04vHLBIe9cLlAy3QP";
security.protocol=SASL_SSL
sasl.mechanism=PLAIN
ssl.protocol=TLSv1.2
ssl.enabled.protocols=TLSv1.2
ssl.endpoint.identification.algorithm=HTTPS
```

```bash
$ kubectl create secret generic kafka-binding --from-file=kafka.properties
```

```bash
$ kubectl get secret kafka-binding -o yaml
apiVersion: v1
data:
  kafka.properties: c2FzbC5qYWFzLmNvb...==
kind: Secret
metadata:
  creationTimestamp: "2019-11-12T00:11:46Z"
  name: kafka-secret
  namespace: default
  resourceVersion: "638930"
  selfLink: /api/v1/namespaces/default/secrets/kafka-secret
  uid: 03c39ae8-04e1-11ea-9c5d-2e8421deef0d
type: Opaque
```

For compatibility with Apache Kafka properties configuraiton `sasl.jaas.config` is also supported to specify SASL username and password, for example:

```
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="T7f3xuwnKG0Q5V71" password="Xwup9VRp9diu4CD04vHLBIe9cLlAy3QP";
```

List of supported property names (based on Apache Kafka and librdkafka): TBW

## Example

A more detailed example of the `KafkaSource` can be found in the
[Knative documentation](https://knative.dev/docs/eventing/samples/).
