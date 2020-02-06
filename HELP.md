# Read Me First
The following was discovered as part of building this project:

* The original package name 'com.in28minutes.springboot.microservice.example.forex.spring-boot-microservice-forex-service' is invalid and this project uses 'com.in28minutes.springboot.microservice.example.forex.springbootmicroserviceforexservice' instead.

# Getting Started

### Reference Documentation
For further reference, please consider the following sections:

* [Official Apache Maven documentation](https://maven.apache.org/guides/index.html)
* [Spring Boot Maven Plugin Reference Guide](https://docs.spring.io/spring-boot/docs/2.2.4.RELEASE/maven-plugin/)
* [Spring Web](https://docs.spring.io/spring-boot/docs/2.2.4.RELEASE/reference/htmlsingle/#boot-features-developing-web-applications)
* [Spring Boot DevTools](https://docs.spring.io/spring-boot/docs/2.2.4.RELEASE/reference/htmlsingle/#using-boot-devtools)
* [Spring Data JPA](https://docs.spring.io/spring-boot/docs/2.2.4.RELEASE/reference/htmlsingle/#boot-features-jpa-and-spring-data)

### Guides
The following guides illustrate how to use some features concretely:

* [Building a RESTful Web Service](https://spring.io/guides/gs/rest-service/)
* [Serving Web Content with Spring MVC](https://spring.io/guides/gs/serving-web-content/)
* [Building REST services with Spring](https://spring.io/guides/tutorials/bookmarks/)
* [Accessing Data with JPA](https://spring.io/guides/gs/accessing-data-jpa/)

# Using the environment
## Commands used
To build the target using maven: mvn clean install
Then the artifact is build on target/ directory
To build the docker image: first, create the Dockerfile. Then: docker build -t ccc/forex-service:latest .
The tag is in the form: name[:tag]
To launch the container: docker run -d --name forex-service -p 8000:8000 ccc/forex-service:latest
The service is then available in the 8000 port of the local host.

## To allow communications
docker network create mynet
docker run --network mynet --name container-1 ...
docker run --network mynet --name container-2 ...

## For the conversion-service
docker build -t ccc/conversion-service .
docker run --rm -it --network mynet --name conversion-service -p 8100:8100 ccc/conversion-service:latest

## Launching services
- From ForexService
	mvn clean package
	docker build -t ccc/forex-service:latest .
	docker run -d --network mynet --name forex-service -p 8000:8000 ccc/forex-service:latest
- From CurrencyConversionService
	docker build -t ccc/conversion-service .
	docker run --rm -it --network mynet --name conversion-service -p 8100:8100 ccc/conversion-service:latest
	
	
# Automating efficiently Maven builds using Docker
## Docker file
FROM openjdk:8-jdk-alpine
### ----
### Install Maven
RUN apk add --no-cache curl tar bash
ARG MAVEN_VERSION=3.3.9
ARG USER_HOME_DIR="/root"
RUN mkdir -p /usr/share/maven && \
curl -fsSL http://apache.osuosl.org/maven/maven-3/$MAVEN_VERSION/binaries/apache-maven-$MAVEN_VERSION-bin.tar.gz | tar -xzC /usr/share/maven --strip-components=1 && \
ln -s /usr/share/maven/bin/mvn /usr/bin/mvn
ENV MAVEN_HOME /usr/share/maven
ENV MAVEN_CONFIG "$USER_HOME_DIR/.m2"
### speed up Maven JVM a bit
ENV MAVEN_OPTS="-XX:+TieredCompilation -XX:TieredStopAtLevel=1"
ENTRYPOINT ["/usr/bin/mvn"]
### ----
### Install project dependencies and keep sources
### make source folder
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app
### install maven dependency packages (keep in image)
COPY pom.xml /usr/src/app
RUN mvn -T 1C install && rm -rf target
### copy other source files (keep in image)
COPY src /usr/src/app/src

## Using the Dockerfile: building the builder
$ # pull latest (or specific version) builder image
$ docker pull myrep/mvn-builder:latest
$ # build new builder
$ docker build -t myrep/mvn-builder:latest --cache-from myrep/mvn-builder:latest .
## Use builder container to run tests
$ # run tests - test results are saved into $PWD/target/surefire-reports
$ docker run -it --rm -v "$PWD"/target:/usr/src/app/target myrep/mvn-builder -T 1C -o test
## Use builder container to createa pplication WAR
$ # create application WAR file (skip tests) - $PWD/target/spring-boot-rest-example-0.3.0.war
$ docker run -it --rm -v $(shell pwd)/target:/usr/src/app/target myrep/mvn-builder package -T 1C -o -Dmaven.test.skip=true

# Playing with Kubernetes
minikube start
minikube dashboard&

$ export SERVICE_IP=$( --template "{{ range (index .status.loadBalancer.ingress 0) }}{{ . }}{{ end }}")
$ echo http://$SERVICE_IP:8080/login


Using the local repository is possible if you build in k8s. Before building with "docker build .", the command "eval $(minikube docker-env)" must be runned.
