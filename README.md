## 1. Source Code

**Order.java**

```java
public class Order {
    private String customerName;
    private String product;
    private int quantity;

    public String getCustomerName(){
        return customerName;
    }

    public void setCustomerName(String customerName) {
        this.customerName = customerName;
    }

    public String getProduct() {
        return product;
    }

    public void setProduct(String product){
        this.product = product;
    }

    public int getQuantity() {
        return quantity;
    }

    public void setQuantity(int quantity) {
        this.quantity = quantity;
    }
}
```

**OrderDeserializer.java**

```java
import org.apache.kafka.common.serialization.Deserializer;

import com.fasterxml.jackson.databind.ObjectMapper;

public class OrderDeserializer implements Deserializer<Order> {

    @Override
    public Order deserialize(String topic, byte[] data) {
        ObjectMapper objectMapper = new ObjectMapper();
        try {
            return objectMapper.readValue(data, Order.class);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
}
```

**OrderConsumer.java**

```java
import java.time.Duration;
import java.util.Collections;
import java.util.Properties;

import org.apache.kafka.clients.consumer.Consumer;
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.serialization.StringDeserializer;

public class OrderConsumer {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.setProperty(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.setProperty(ConsumerConfig.GROUP_ID_CONFIG, "order-consumer-group");
        props.setProperty(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        props.setProperty(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, OrderDeserializer.class.getName());

        Consumer<String, Order> consumer = new KafkaConsumer<>(props);
        consumer.subscribe(Collections.singletonList("OrderPartitionedTopic"));

        try {
            while (true) {
                ConsumerRecords<String, Order> records = consumer.poll(Duration.ofMillis(100));
                records.forEach(record -> {
                    Order order = record.value();
                    System.out.println("Received Order - Customer: " + order.getCustomerName() +
                            ", Product: " + order.getProduct() +
                            ", Quantity: " + order.getQuantity());
                });
            }
        } finally {
            consumer.close();
        }
    }
}
```

## 2. How to compile and run the application

### 2.1. Run the following commands to execute zookeeper and kafka-server

Run the following command to start zookeeper:

```
zookeeper-server-start C:\kafka_2.13-3.6.0\config\zookeeper.properties
```

Run the following command to start kafka-server

```
kafka-server-start C:\kafka_2.13-3.6.0\config\server.properties
```

### 2.2. Compiling the application

```
javac -cp ".;lib/*" src/Order.java src/OrderConsumer.java src/OrderDeserializer.java
```

### 2.3. Run the command for executing the application

```
java -cp ".;lib/*;src" OrderConsumer
```

## 3. Application output

![image](https://github.com/luiscoco/Kafka_Java_Consumer_Serialize_message_to_JSON_sample2/assets/32194879/b84cfed5-1545-4345-8a61-9db74aaa8bc6)
