# Spring Boot Docker


## 环境准备
- JDK 1.8
- Maven 3.2+
- Intellij IDEA editor
- Docker


### Install and run Docker for Mac
**Docker** 需要安装在 Linux 环境中使用，如果开发环境是 Windows,则需要安装 VirtualBox,配置 Linux 环境使用 Docker;如果使用的是 mac，则在 macOS 10.11+以上都可以直接下载安装 Docker。
下面是按照在 mac pro 10.13.6 版本上进行的 Docker 部署配置。
* [Docker 下载地址](https://docs.docker.com/docker-for-mac/install/)
* 双击下载的 Docker.dmg 打开安装程序，然后将鲸鱼拖到 Applications 中；
* 然后在 Applications 中双击 Docker.app 启动运行 Docker,在任务栏中可以看见鲸鱼图标，然后有 Docker Desktop is running 的提示，则表明已经在 mac 启动成功了。

**查看正在运行的 Docker：**
```bash
$ docker ps
```
***

## 准备 Spring Boot 程序
可以使用官网给提供的程序：

- ``` git clone https://github.com/spring-guides/gs-spring-boot-docker.git ```
- cd into ```gs-spring-boot-docker/initial```
- 创建一个简单的 Application. ```src/main/java/hello/Application.java```

或者使用 IDEA 创建一个 Spring Boot maven 程序，目前该 demo 使用的是 IDEA 创建的项目。
```java
package com.galaxy;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class SpringBootDockerApplication {

	@RequestMapping("/")
	public String home() {
		return "Hello Docker World";
	}


	public static void main(String[] args) {
		SpringApplication.run(SpringBootDockerApplication.class, args);
	}

}

```
我们可以在 Docker 容器外运行应用程序：
```bash
$ ./mvnw package && java -jar target/spring-boot-docker-0.1.0.jar
```
接着打开 **localhost:8080** 就可以看见 *“Hello Docker World”* 信息。

***
## 将项目容器化
Docker has a simple Dockerfile file format that it uses to specify the "layers" of an image. So let’s go ahead and create a Dockerfile in our Spring Boot project:
### 创建 ```Dockerfile``` 文件
```docker
FROM openjdk:8-jdk-alpine
VOLUME /tmp
ARG JAR_FILE
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```
为了利用 spring boot fat jar 文件中依赖项和应用程序资源之间的干净分离, 我们将使用稍微不同的 Dockerfile 实现:
```docker
FROM openjdk:8-jdk-alpine
VOLUME /tmp
ARG DEPENDENCY=target/dependency
COPY ${DEPENDENCY}/BOOT-INF/lib /app/lib
COPY ${DEPENDENCY}/META-INF /app/META-INF
COPY ${DEPENDENCY}/BOOT-INF/classes /app
ENTRYPOINT ["java","-cp","app:app/lib/*","com.galaxy.SpringBootDockerApplication"]
```

### Build a Docker Image with Maven
在 Maven pom.xml 中添加新的插件配置：
```xml
<properties>
   <docker.image.prefix>peterluo2010</docker.image.prefix>
</properties>
<build>
    <plugins>
        <plugin>
            <groupId>com.spotify</groupId>
            <artifactId>dockerfile-maven-plugin</artifactId>
            <version>1.4.9</version>
            <configuration>
                <repository>${docker.image.prefix}/${project.artifactId}</repository>
            </configuration>
        </plugin>
    </plugins>
</build>
```
为了确保在创建 docker 映像之前解压缩 jar, 我们为依赖项插件添加了一些配置:
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <executions>
        <execution>
            <id>unpack</id>
            <phase>package</phase>
            <goals>
                <goal>unpack</goal>
            </goals>
            <configuration>
                <artifactItems>
                    <artifactItem>
                        <groupId>${project.groupId}</groupId>
                        <artifactId>${project.artifactId}</artifactId>
                        <version>${project.version}</version>
                    </artifactItem>
                </artifactItems>
            </configuration>
        </execution>
    </executions>
</plugin>

```

可以使用命令行生成标记的 Docker image:
```bash
$ ./mvnw install dockerfile:build
```
 在本地运行：
 ```bash
$ docker run -p 8080:8080 -t peterluo2010/spring-boot-docker
```
这是访问 **localhost:8080**能看到 *“Hello Docker World”*。

***

## 推送 Docker image 到 dockerhub
* 首先需要在 dockerhub 上注册一个账号；
* 然后在本地环境登录；
* 使用 push 命令上传 docker image 到 hub。

```bash
$ docker login
```

```bash
$ docker push peterluo2010/spring-boot-docker
```
这时候会发现在dockerhub 账号下有该镜像了。

***

## 在服务器上使用该 docker image
在服务上安装 docker ce：
参考：[Get Docker CE for CentOS](https://docs.docker.com/install/linux/docker-ce/centos/)

在服务器上从 DockerHub 下载 Docker image:
```bash
$ docker pull peterluo2010/spring-boot-docker
```
在服务器上启动运行 Spring Boot 程序：
```bash
$ docker run -p 8080:8080 -t peterluo2010/spring-boot-docker
```
然后通过 ip 访问服务器上的web：106.14.31.165:8080，正常打开能看到 *“Hello Docker World”* 表示部署成功。

***
## 参考资料
* [Why Docker?](https://yeasy.gitbooks.io/docker_practice/introduction/why.html)
* [DockerHub](https://hub.docker.com/)
* [Spring Boot with Docker](https://spring.io/guides/gs/spring-boot-docker/#initial)
* [Get Docker CE for CentOS](https://docs.docker.com/install/linux/docker-ce/centos/)
* [Official Images](https://hub.docker.com/search/?type=image&image_filter=official)






