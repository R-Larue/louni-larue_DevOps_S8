# DevOps Larue-Louni S8

## Docker TP

### Partie 1 - 30/01 'Discover Docker'

#### Database

##### Identifiants

- POSTGRES_DB=db
- POSTGRES_USER=usr
- POSTGRES_PASSWORD=pwd

##### Commandes

```
docker build .
docker run --name postgres -p 3306:5432 alpine-14.1
docker start postgres
```

Ainsi on se connecte à la base de données 'db' avec les identifiants fournis.

#### Adminer
##### Network

```
docker network create app-network
```
##### Redemarré Postgres & Lancer Adminer

```
docker stop postgres
docker rm postgres
docker run --name postgres --network app-network -p 5432:5432 -d alpine-14.1
docker run -p "8080:8080" --net=app-network --name=adminer -d adminer
```

Lorsqu'on supprime le conteneur postgres, les données en base de données ne sont plus persistentes.

##### Rendre les données persistentes

```
docker run --name postgres --network app-network -p 5432:5432 -v /home/mutton/DevOps/TP1/data:/var/lib/postgresql/data -d alpine-14.1
```

#### Backend API

##### Basic stage

```
FROM openjdk:11-jre-slim

ENV JAVA_VER 11
ENV JAVA_HOME /usr/lib/jvm/java-11-openjdk-amd64

ADD Main.class .

ENTRYPOINT [ "java", "Main" ]
```
On compile à l'extérieur de l'image puis on la configure pour que le container execute le fichier Main.java au démarrage et affiche "Hello World !" :

```
docker build .
docker image tag 102cd60191ea helloworldstage
docker run helloworldstage
```

##### Multistage

```
FROM openjdk:11

ENV JAVA_VER 11
ENV JAVA_HOME /usr/lib/jvm/java-11-openjdk-amd64

ADD Main.java .
RUN javac Main.java

FROM openjdk:11-jre

# Copy resource from previous stage
COPY --from=0 Main.class .
# Run java code with the JRE

ENTRYPOINT [ "java", "Main" ]
```
On compile le fichier Main.java dans le premier stage (Main.class) puis on l'execute au demarrage du container avec le runtime java(JRE) :

```
docker build .
docker image tag 102cd60191ea helloworldmultistage
docker run helloworldmultistage
```

##### SpringBoot Backend SimpleAPI

Dockerfile:

```
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY pom.xml .
COPY src ./src
RUN mvn package -DskipTests

# Run
FROM amazoncorretto:17
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

ENTRYPOINT java -jar myapp.jar
```
Puis on build et run le docker container sur le port 8080 pour afficher Hello World.

```
docker build -t simpleapi .
docker run --name simpleapp --net=app-network -p 8080:8080 -d simpleapi
```
##### SpringBoot Backend SimpleControllerAPI

Application.yaml

```
spring:
  jpa:
    properties:
      hibernate:
        jdbc:
          lob:
            non_contextual_creation: true
    generate-ddl: false
    open-in-view: true
  datasource:
    url: jdbc:postgresql://postgres:5432/db
    username: usr
    password: pwd
    driver-class-name: org.postgresql.Driver
management:
 server:
   add-application-context-header: false
 endpoints:
   web:
     exposure:
       include: health,info,env,metrics,beans,configprops
```
L'url contient postgres qui est le dns du conteneur docker postgres sur le réseau app-network.

Enfin, on build et run un container puis on appelle l'API /departement/IRC/students pour afficher l'étudiant "Eli Copter".

```
docker build -t simpleapiwbdd .
docker run --name simpleappwbdd --net=app-network -p 8080:8080 -d simpleapiwbdd
```

### HTTP server

```
<VirtualHost *:80>
    ProxyPreserveHost On
    ProxyPass / http://simpleappwbdd:8080/
    ProxyPassReverse / http://simpleappwbdd:8080/
</VirtualHost>
```

### Docker-Compose

```
docker exec -it 687e95097992 cat /usr/local/apache2/conf/httpd.conf > my-httpd.conf
```

```
version: '3.7'

services:
    backend:
        container_name: simpleappwbdd
        build: ../JavaBDD/simple-api-student-main/simple-api-student-main/.
        networks:
         - my-network
        depends_on:
         - database

    database:
        container_name: postgres
        build: ../.
        volumes:
         - ../data:/var/lib/postgresql/data
        networks:
         - my-network

    httpd:
        build: ../HTTPServer/.
        ports:
         - 80:80
        networks:
         - my-network
        depends_on:
         - backend
         - database

networks:
    my-network:
     external:
      name: app-network

```

## Github-CI

### Setup GitHub Actions
```
name: CI devops 2023
on:
  #to begin you want to launch this job in main and develop
  push:
    branches:
     - main
     - develop 
  pull_request:
```

#### Build and test your Application
```

```
#### First steps into the CD World
```

```

## Setup Quality Gate

### Register to SonarCloud
```
 - name: Login to DockerHub
   run: docker login -u ${{ secrets.DOCKER_USERNAME }} -p ${{ secrets.DOCKER_ACCESS_TOKEN }}

 - name: Build and test with Maven
        run: mvn -B verify sonar:sonar -Dsonar.projectKey=R-Larue_louni-larue_DevOps_S8 -Dsonar.organization=r-larue -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{ secrets.SONAR_TOKEN }}  --file ./pom.xml
```

#### Fichier main.yml Version Final

```
name: CI devops 2023
on:
  #to begin you want to launch this job in main and develop
  push:
    branches:
     - main
     - develop 
  pull_request:

jobs:
  test-backend: 
    runs-on: ubuntu-22.04
    steps:
     #checkout your github code using actions/checkout@v2.5.0
      - uses: actions/checkout@v2.5.0

     #do the same with another action (actions/setup-java@v3) that enable to setup jdk 17
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: maven

     #finally build your app with the latest command
      - name: Build and test with Maven
        run: mvn -B verify sonar:sonar -Dsonar.projectKey=R-Larue_louni-larue_DevOps_S8 -Dsonar.organization=r-larue -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{ secrets.SONAR_TOKEN }}  --file ./pom.xml

#define job to build and publish docker image
  build-and-push-docker-image:
     needs: test-backend
     # run only when code is compiling and tests are passing
     runs-on: ubuntu-22.04

     # steps to perform in job
     steps:
       - name: Checkout code
         uses: actions/checkout@v2.5.0
         
         
       - name: Login to DockerHub
         run: docker login -u ${{ secrets.DOCKER_USERNAME }} -p ${{ secrets.DOCKER_ACCESS_TOKEN }}



       - name: Build image and push backend
         uses: docker/build-push-action@v3
         with:
           # relative path to the place where source code with Dockerfile is located
           context: .
           # Note: tags has to be all lower-case
           tags:  ${{secrets.DOCKER_USERNAME}}/simple-api
           push: ${{ github.ref == 'refs/heads/main' }}

       - name: Build image and push database
         uses: docker/build-push-action@v3
         with:
           # relative path to the place where source code with Dockerfile is located
           context: ./DB
           # Note: tags has to be all lower-case
           tags:  ${{secrets.DOCKER_USERNAME}}/db
           push: ${{ github.ref == 'refs/heads/main' }}

       - name: Build image and push httpd
         uses: docker/build-push-action@v3
         with:
           # relative path to the place where source code with Dockerfile is located
           context: ./HTTP
           # Note: tags has to be all lower-case
           tags:  ${{secrets.DOCKER_USERNAME}}/http
           push: ${{ github.ref == 'refs/heads/main' }}
```