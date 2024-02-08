# Spring PetClinic Sample Application [![Build Status](https://github.com/spring-projects/spring-petclinic/actions/workflows/maven-build.yml/badge.svg)](https://github.com/spring-projects/spring-petclinic/actions/workflows/maven-build.yml)

## Understanding the Spring Petclinic application with a few diagrams

[See the presentation here](https://speakerdeck.com/michaelisvy/spring-petclinic-sample-application)

## Run Petclinic locally

Spring Petclinic is a [Spring Boot](https://spring.io/guides/gs/spring-boot) application built [Gradle](https://spring.io/guides/gs/gradle/). You can build a jar file and run it from the command line:

```bash
git clone https://github.com/spring-projects/spring-petclinic.git
cd spring-petclinic
./gradldew bootJar
java -jar build/libs/*.jar
```

You can then access the Petclinic at <http://localhost:8080/>.

<img width="1042" alt="petclinic-screenshot" src="https://cloud.githubusercontent.com/assets/838318/19727082/2aee6d6c-9b8e-11e6-81fe-e889a5ddfded.png">

Or you can run it from Gradle directly using the Spring Boot Maven plugin. If you do this, it will pick up changes that you make in the project immediately (changes to Java source files require a compile as well - most people use an IDE for this):

```bash
./gradlew bootRun
```


## Database configuration

In its default configuration, Petclinic uses an in-memory database (H2) which
gets populated at startup with data. The h2 console is exposed at `http://localhost:8080/h2-console`,
and it is possible to inspect the content of the database using the `jdbc:h2:mem:<uuid>` URL. The UUID is printed at startup to the console.

A similar setup is provided for MySQL and PostgreSQL if a persistent database configuration is needed. Note that whenever the database type changes, the app needs to run with a different profile: `spring.profiles.active=mysql` for MySQL or `spring.profiles.active=postgres` for PostgreSQL.

You can start MySQL or PostgreSQL locally with whatever installer works for your OS or use docker:

```bash
docker run -e MYSQL_USER=petclinic -e MYSQL_PASSWORD=petclinic -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=petclinic -p 3306:3306 mysql:8.2
```

or

```bash
docker run -e POSTGRES_USER=petclinic -e POSTGRES_PASSWORD=petclinic -e POSTGRES_DB=petclinic -p 5432:5432 postgres:16.1
```

Further documentation is provided for [MySQL](https://github.com/spring-projects/spring-petclinic/blob/main/src/main/resources/db/mysql/petclinic_db_setup_mysql.txt)
and [PostgreSQL](https://github.com/spring-projects/spring-petclinic/blob/main/src/main/resources/db/postgres/petclinic_db_setup_postgres.txt).

Instead of vanilla `docker` you can also use the provided `docker-compose-localdev.yml` file to start the database containers. Each one has a profile just like the Spring profile:

```bash
docker-compose -f docker-compose-localdev.yml --profile mysql up
```

or

```bash
docker-compose -f docker-compose-localdev.yml --profile postgres up
```

## Building a Container

### There are multiple `Dockerfile`'s in this project to demonstrate different strategies you may use to build an image.

```bash
# The jar file needs to be prebuilt in /build/libs/*.jar
./gradlew bootJar
docker build -t petclinic-app -f Dockerfile-basic .
```

```bash
# The source code gets built during docker image creation
docker build -t petclinic-app -f Dockerfile-build .
```

```bash
# The source code gets built during docker image creation and then gets removed from the generated image. -> Has the output size of Dockerfile-basic' result
docker build -t petclinic-app -f Dockerfile-multistage .
```

### You can also build a container image using the Spring Boot build plugin, without needing any Dockerfile:

```bash
./gradlew bootBuildImage --imageName=petclinic-app
```

### There are multiple `docker-compose file`s in this project to demonstrate different strategies you may use to build an composed environment.
> All of the examples use --build to keep it updated in case of changing the data and -d to detach from the running shell

```bash
# Run only the one of the databases and the app locally
docker compose -f docker-compose-localdev.yml --profile {mysql|postgres} up --build -d
```

```bash
# Run the application and postgres db 
docker compose -f docker-compose-postgres.yml up --build -d
```

```bash
# Run the application and mysql db
docker compose -f docker-compose-mysql.yml up --build -d
```

> The main docker-compose.yml enforces using profiles to always select configuration.
> As the name is 'docker-compose.yml' we can omit specifying the file.
 
```bash
# Run the application and postgres db
docker compose --profile postgres --profile app-postgres up --build -d
```

```bash
# Run the application and mysql db
docker compose --profile mysql --profile app-mysql up --build -d
```

```bash
# Run only db
docker compose --profile {mysql|postgres} up --build -d
```

```bash
# Run only the app (why not)
docker compose --profile {app-mysql|app-postgres} up --build -d
```

> You can also pass the profiles using an environment variable infront of the command
> `COMPOSE_PROFILES=app-mysql,mysql docker compose up --build -d`
