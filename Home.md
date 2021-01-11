## RocketMQ-Spring

This project aims to help developers quickly integrate [RocketMQ](http://rocketmq.apache.org/) with [Spring Boot](http://projects.spring.io/spring-boot/). 

## Features

- [x] Send messages synchronously
- [x] Send messages asynchronously
- [x] Send messages in one-way mode
- [x] Send ordered messages
- [x] Send batched messages
- [x] Send transactional messages
- [x] Send scheduled messages with delay level
- [x] Consume messages with concurrently mode (broadcasting/clustering)
- [x] Consume messages with push/pull mode
- [x] Consume ordered messages
- [x] Filter messages using the tag or sql92 expression
- [x] Support message tracing
- [x] Support authentication and authorization
- [x] Support request-reply message exchange pattern

## Prerequisites
- JDK 1.8 and above
- [Maven](http://maven.apache.org/) 3.0 and above
- Spring Boot 2.0 and above

## Usage

Add a dependency using maven:

```xml
<!--add dependency in pom.xml-->
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-spring-boot-starter</artifactId>
    <version>${RELEASE.VERSION}</version>
</dependency>
``` 