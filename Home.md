## RocketMQ-Spring

This project aims to help developers quickly integrate [RocketMQ](http://rocketmq.apache.org/) with [Spring Boot](http://projects.spring.io/spring-boot/). 

## Features

- [x] Send messages synchronously
- [x] Send messages asynchronously
- [x] Send messages in one-way mode
- [x] Send messages orderly
- [x] Send messages in batch
- [x] Send transactional messages
- [x] Send scheduled messages with delay level
- [x] Consume messages with concurrently mode (broadcasting/clustering)
- [x] Consume messages with orderly mode
- [x] Support message filter with tag and sql
- [x] Support message tracing capability
- [x] Authentication and authorisation
- [x] Request-reply mode

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