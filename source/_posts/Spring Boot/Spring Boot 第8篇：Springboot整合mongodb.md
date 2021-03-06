---
title: SpringBoot第八篇：springboot整合mongodb
date: 2018-05-29 13:51:27
comments: false #是否可评论
toc: true #是否显示文章目录
categories: "Spring Boot" #分类
type: "tags"
tags:   #标签
    - Spring Boot
---
## 准备工作

- 安装 MongoDB
- jdk 1.8
- maven 3.0
- idea

## 环境依赖

在pom文件引入spring-boot-starter-data-mongodb依赖：

<!-- more -->

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

## 数据源配置

如果mongodb端口是默认端口，并且没有设置密码，可不配置，sprinboot会开启默认的。

```properties
spring.data.mongodb.uri=mongodb://localhost:27017/springboot-db
```

mongodb设置了密码，这样配置：

```properties
spring.data.mongodb.uri=mongodb://name:pass@localhost:27017/dbname
```

## 定义一个简单的实体

mongodb

```java
package com.forezp.entity;

import org.springframework.data.annotation.Id;

public class Customer {

    @Id
    public String id;

    public String firstName;
    public String lastName;

    public Customer() {}

    public Customer(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }

    @Override
    public String toString() {
        return String.format(
                "Customer[id=%s, firstName='%s', lastName='%s']",
                id, firstName, lastName);
    }

}
```

## 数据操作dao层

```java
public interface CustomerRepository extends MongoRepository<Customer, String> {

    public Customer findByFirstName(String firstName);
    public List<Customer> findByLastName(String lastName);

}
```

写一个接口，继承MongoRepository，这个接口有了几本的CURD的功能。如果你想自定义一些查询，比如根据firstName来查询，获取根据lastName来查询，只需要定义一个方法即可。注意firstName严格按照存入的mongodb的字段对应。在典型的java的应用程序，写这样一个接口的方法，需要自己实现，但是在springboot中，你只需要按照格式写一个接口名和对应的参数就可以了，因为springboot已经帮你实现了。

## 测试

```java
@SpringBootApplication
public class SpringbootMongodbApplication  implements CommandLineRunner {

    @Autowired
    private CustomerRepository repository;

    public static void main(String[] args) {
        SpringApplication.run(SpringbootMongodbApplication.class, args);
    }

    @Override
    public void run(String... args) throws Exception {
        repository.deleteAll();

        // save a couple of customers
        repository.save(new Customer("Alice", "Smith"));
        repository.save(new Customer("Bob", "Smith"));

        // fetch all customers
        System.out.println("Customers found with findAll():");
        System.out.println("-------------------------------");
        for (Customer customer : repository.findAll()) {
            System.out.println(customer);
        }
        System.out.println();

        // fetch an individual customer
        System.out.println("Customer found with findByFirstName('Alice'):");
        System.out.println("--------------------------------");
        System.out.println(repository.findByFirstName("Alice"));

        System.out.println("Customers found with findByLastName('Smith'):");
        System.out.println("--------------------------------");
        for (Customer customer : repository.findByLastName("Smith")) {
            System.out.println(customer);
        }
    }
```

在springboot的应用程序，加入测试代码。启动程序，控制台打印了：

## >Customers found with findAll():

```java
 Customer[id=58f880f589ffb696b8a6077e, firstName='Alice', lastName='Smith']
 Customer[id=58f880f589ffb696b8a6077f, firstName='Bob', lastName='Smith']
 Customer found with findByFirstName('Alice'):
 --------------------------------
 Customer[id=58f880f589ffb696b8a6077e, firstName='Alice', lastName='Smith']
 Customers found with findByLastName('Smith'):
 --------------------------------
 Customer[id=58f880f589ffb696b8a6077e, firstName='Alice', lastName='Smith']
 Customer[id=58f880f589ffb696b8a6077f, firstName='Bob', lastName='Smith']
```

测试通过。

源码下载：https://github.com/forezp/SpringBootLearning