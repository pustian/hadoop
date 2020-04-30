>├── pom.xml
>└── src
>    ├── main
>    │   ├── java
>    │   │   └── tian
>    │   │       └── pusen
>    │   │           ├── Appliaction.java
>    │   │           ├── entity
>    │   │           │   └── Messages.java
>    │   │           └── service
>    │   │               ├── KafkaReceiver.java
>    │   │               └── KafkaSender.java
>    │   └── resources
>    │       └── application.yml
>    └── test
>        └── java




pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>tian.pusen</groupId>
    <artifactId>spring-30-kafka</artifactId>
    <version>1.0-SNAPSHOT</version>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.google.code.gson/gson -->
        <dependency>
            <groupId>com.google.code.gson</groupId>
            <artifactId>gson</artifactId>
            <version>2.8.6</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/commons-lang/commons-lang -->
        <dependency>
            <groupId>commons-lang</groupId>
            <artifactId>commons-lang</artifactId>
            <version>2.6</version>
        </dependency>
    </dependencies>


    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

application.properties

```properties
spring.kafka.bootstrap-servers=211.159.167.180:9092
spring.kafka.consumer.group-id=test-consumer-group
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer
 
 #logging.level.root=debug
```

https://www.cnblogs.com/yx88/p/11013338.html

Application.java

```java
@SpringBootApplication
public class Appliaction {

    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(Appliaction.class, args);

        KafkaSender sender = context.getBean(KafkaSender.class);
        for (int i = 0; i < 1000; i++) {
            sender.send();
            try {
                Thread.sleep(300);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

}
```

entity.Message.java

```java
public class Messages  implements Serializable {
    private Long id;
    private String msg;
    private Date sendTime;
    /**
     * transient 关键字修饰的字段不会被序列化
     */
    private transient String desc;

```

service.KafkaReceiver.java

```bash
@Service
public class KafkaReceiver {

    @KafkaListener(topics = {"newtopic"})
    public void listen(ConsumerRecord<?, ?> record) {
        Optional<?> kafkaMessage = Optional.ofNullable(record.value());
        if (kafkaMessage.isPresent()) {
            Object message = kafkaMessage.get();
            System.out.println("record =" + record);
            System.out.println("message =" + message);

        }
    }
}
```

service.KafkaSender.java

```

@Service
public class KafkaSender {
    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    private Gson gson = new GsonBuilder().create();

    public void send() {
        Messages message = new Messages();
        message.setId(System.currentTimeMillis());
        message.setMsg(RandomStringUtils.randomAlphanumeric(10) );
        message.setSendTime(new Date());
        ListenableFuture<SendResult<String, String>> test0 = kafkaTemplate.send("newtopic", gson.toJson(message));
    }
}
```