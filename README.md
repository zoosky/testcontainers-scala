[![Build Status](https://travis-ci.org/dimafeng/testcontainers-scala.svg?branch=master)](https://travis-ci.org/dimafeng/testcontainers-scala)

# Testcontainers-scala


Scala wrapper for [testcontainers-java](https://github.com/testcontainers/testcontainers-java) that
allows using docker containers for functional/integration/~~unit~~ testing.

> TestContainers is a Java 8 library that supports JUnit tests, providing lightweight, throwaway instances of common databases, Selenium web browsers, or anything else that can run in a Docker container.

## Why can't I use testcontainers-java in my scala project?

*testcontainers-java* is awesome and yes, you can use it in scala project but:

* It's written to be used in JUnit tests
* `DockerComposeContainer<SELF extends DockerComposeContainer<SELF>>` - it's not convinient to use its api with
this 'recursive generic' from scala

Plus

* This wrapper provides with scala interfaces, approaches, types
* This wrapper is integrated with [scalatest](http://www.scalatest.org/)

## Setup

*Maven*

```xml
<dependency>
    <groupId>com.dimafeng</groupId>
    <artifactId>testcontainers-scala</artifactId>
    <version>0.2.0</version>
    <scope>test</scope>
</dependency>

```

*Gradle*

```groovy
testCompile("com.dimafeng:testcontainers-scala:0.2.0")
```

*SBT*

```scala
libraryDependencies += "com.dimafeng" % "testcontainers-scala" % "0.2.0" % "test"
```

## Requirements

* JDK >= 1.8
* [See 'Compatibility' section](http://testcontainers.viewdocs.io/testcontainers-java/)

## Quick Start

There are two modes of container launching: `ForEachTestContainer` and `ForAllTestContainer`.
The first one starts a new container before each test case and then stops and removes it. The second one
 starts and stops a container only once.

To start using it, you just need to extend one of those traits and override a `container` val as follows:

```scala
import com.dimafeng.testcontainers.{ForAllTestContainer, MySQLContainer}

class MysqlSpec extends FlatSpec with ForAllTestContainer {

  override val container = MySQLContainer()

  it should "do something" in {
    Class.forName(container.driverClassName)
    val connection = DriverManager.getConnection(container.jdbcUrl, container.username, container.password)
    ...
  }
}
```

This spec has a clean mysql database instance for each of its test cases.

```scala
import org.testcontainers.containers.MySQLContainer

class MysqlSpec extends FlatSpec with ForAllTestContainer {

    override val container = MySQLContainer()

    it should "do something" in {
      ...
    }

    it should "do something 2" in {
      ...
    }
}
```

This spec starts one container and both tests share the container's state.


## Container types

### Generic Container

The most flexible but less convinient containtainer type is `GenericContainer`. This container allows to launch any docker image
with custom configuration.

```scala
class GenericContainerSpec extends FlatSpec with ForAllTestContainer {
  override val container = GenericContainer("nginx:latest",
    exposedPorts = Seq(80),
    waitStrategy = Wait.forHttp("/")
  )

  "GenericContainer" should "start nginx and expose 80 port" in {
    assert(Source.fromInputStream(
      new URL(s"http://${container.containerIpAddress}:${container.mappedPort(80)}/").openConnection().getInputStream
    ).mkString.contains("If you see this page, the nginx web server is successfully installed"))
  }
}
```

### Docker Compose

```scala
class ComposeSpec extends FlatSpec with ForAllTestContainer {
  override val container = DockerComposeContainer(new File("src/test/resources/docker-compose.yml"), exposedService = Map("redis_1" -> 6379))

  "DockerComposeContainer" should "retrieve non-0 port for any of services" in {
    assert(container.getServicePort("redis_1", 6379) > 0)
  }
}
```

### Selenium

Requires you to add [this dependency](http://mvnrepository.com/artifact/org.testcontainers/selenium) to your build script.


```
class SeleniumSpec extends FlatSpec with SeleniumTestContainerSuite with WebBrowser {
  override def desiredCapabilities = DesiredCapabilities.chrome()

  "Browser" should "show google" in {
      go to "http://google.com"
  }
}

```

In this case, you'll obtain a clean instance of browser (firefox/chrome) within container to which
a test will connect via remote-driver. See [Webdriver Containers](http://testcontainers.viewdocs.io/testcontainers-java/usage/webdriver_containers/)
for more details.

### Mysql

Requires you to add [this dependency](http://mvnrepository.com/artifact/org.testcontainers/mysql)

```scala
class MysqlSpec extends FlatSpec with ForAllTestContainer {

  override val container = MySQLContainer()

  "Mysql container" should "be started" in {
    Class.forName(container.driverClassName)
    val connection = DriverManager.getConnection(container.jdbcUrl, container.username, container.password)
      ...
  }
}
```

### Multiple Containers

```scala
...
val container = MultipleContainers(MySQLContainer(), GenericContainer(...))

// access to containers
containers.containers._1.containerId // container id of the first container
...

```

## Release notes

* **0.2.0**
    * TestContainers `1.0.5` -> `1.1.0`
    * Code refactoring
    * Scala wrappers for major container types


## License

The MIT License (MIT)

Copyright (c) 2016 Dmitry Fedosov

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated
documentation files (the "Software"), to deal in the Software without restriction, including without limitation the
rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit
persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the
Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE
WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR
COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR
OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
