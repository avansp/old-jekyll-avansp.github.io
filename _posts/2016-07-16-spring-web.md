---
layout: post
title:  "Web Application with Spring Framework"
date:   2016-01-09
tags: [en]
comments: true
---

I've just encountered [Spring], and I am sold. I can quickly create a Java web app in a blitz.

## Command-Line Interface

Here's the quickest, cleanest and shortest Hello World project for a web application, complete with the web server.

* Make sure you have JDK 1.7+ installed.

* Download the latest RELEASE of [Spring Boot CLI] (command-line interface) binary ZIP file, and extract it.

* Create this this text file somewhere:

```java
@RestController
class ThisWillActuallyRun {
    @RequestMapping("/")
    String home() {
        "Hello World!"
    }
}
```

  save it as `hello.groovy`.

* Run the spring

```bash
$ [EXTRACTED_SPRING_DIR]/bin/spring run hello.groovy
```

* Open your browser and go to [http://localhost:8080/](http://localhost:8080/) .... tada!!

Beautiful.

## Using Maven

The same hello world web app but using Maven:

* Create initial `pom.xml` file for spring-boot:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>myproject</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.3.6.RELEASE</version>
    </parent>

    <!-- Additional lines to be added here... -->

</project>
```

* Add web application dependency

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

* Run initial maven dependency

```bash
$ mvn dependency:tree
```

* Create source directory `src/main/java`

* Create `Example` class

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@EnableAutoConfiguration
public class Example {

	@RequestMapping("/")
	String home() {
		return "Hello, World!";
	}

	public static void main(String[] args) throws Exception {
		SpringApplication.run(Example.class, args);
	}

}
```

* Call maven to run spring

```bash
$ mvn spring-boot:run
```

* Open [http://localhost:8080](http://localhost:8080/)

Tada!!!

## Create Java JAR

We can modify previous example to output `jar` file, which can be deployed and executed using java directly without maven.

* Add the following build scheme in the `pom.xml` after the `dependencies` block:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

* Build the package

```bash
$ mvn package
```

* The jar file is stored in `target` directory. You can run the web server directly:

```bash
$ java -jar target/myproject-0.0.1-SNAPSHOT.jar
```

* Open [http://localhost:8080](http://localhost:8080/)

<!-- URLs -->
[Spring]: (http://spring.io/)
[Spring Boot CLI]: http://repo.spring.io/release/org/springframework/boot/spring-boot-cli/
