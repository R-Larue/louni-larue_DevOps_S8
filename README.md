<<<<<<< HEAD
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
=======
# simple-api-devops

>>>>>>> Init Java App
