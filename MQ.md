# MQ

## RabbitMq

### 引入jar包

```xml
 <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-amqp</artifactId>
 </dependency>
```

### 配置文件

```yml
spring:
  application:
    name: ceshi
  rabbitmq:
    host: 47.97.62.253
    username: guest
    password: guest
    virtual-host: /
    publisher-confirms: true
    publisher-confirm-type: correlated
    publisher-returns: true
```



### 生产者

```java
   @GetMapping("product")
    public void producet() {
        List<Area> areas = areaMapper.selectList(null);
        Area area = areas.get(0);
        RabbitBrokerMessage  rabbitBrokerMessage = new RabbitBrokerMessage();
        rabbitBrokerMessage.setMessage(JSONObject.toJSONString(area));
        rabbitBrokerMessage.setGmtCreate(new Date());
        rabbitBrokerMessage.setStatus("0");
        rabbitBrokerMessage.setTryCount(1);
        rabbitBrokerMessageMapper.insert(rabbitBrokerMessage);
        CorrelationData correlationData = new CorrelationData();
        correlationData.setId(rabbitBrokerMessage.getId().toString());
        rabbitTemplate.convertAndSend("ding.exchange", "ding.routeBy",JSONObject.toJSONString(area),message -> {
//          message.getMessageProperties().setExpiration(1 * 1000 * 60 + "");
          return message;
        },correlationData);
    }
```

### 消费者

```java
@RabbitListener(queues = {"ding.routeBy"})
    public void mqConsumer(String area){

        log.info("area======={}",area);
	}
```



### ack确认机制

```java
  @PostConstruct
    public void init() {
        rabbitTemplate.setMandatory(true);
        rabbitTemplate.setConfirmCallback((correlationData, ack, cause) -> {
            String id = correlationData.getId();
            Area area = new Area();
            if (ack) {
                area.setStatus("1");
                areaMapper.update(area, Wrappers.<Area>lambdaUpdate().eq(Area::getId,id));
                log.info("send message is OK, confirm id: {}", id);
            } else {
                area.setStatus("2");
                areaMapper.update(area, Wrappers.<Area>lambdaUpdate().eq(Area::getId,id));
                log.info("send message is fail, confirm id: {}", id);
            }
        });
    }
```

