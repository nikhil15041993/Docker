## Run SpringBoot-App Container with MySql Container and PhpMyAdmin Container together using Docker-Compose

Steps:

 * Clone/Download/Create Spring Boot Application
 * Create Dockerfile and Add this with Spring Boot Application
 * Create a Docker-Compose file and Add this with Spring Boot Application
 * Update application.properties file
 * Build Project using Maven
 * Run command for Docker Compose
 * Explore


### 1. Clone/Download/Create Spring Boot Application

Clone this application: customer service.

### 2. Create Dockerfile and Add this with Spring Boot Application

```
FROM openjdk:8-jre-alpine
EXPOSE 8084
WORKDIR /app
COPY target/customer-service-0.0.1-SNAPSHOT.jar .
ENTRYPOINT [ "java", "-jar", "customer-service-0.0.1-SNAPSHOT.jar" ]
```

### 3. Create a Docker-Compose file and Add this with Spring Boot Application

```
version: '3.3'

services:
    #service 1: definition of mysql database
    db:
      image: mysql:latest
      container_name: mysql-db2   
      environment:
        - MYSQL_ROOT_PASSWORD=mukit
        - MYSQL_USER=root
      ports:
        - "3306:3306"
      restart: always
      
    
    #service 2: definition of phpMyAdmin
    phpmyadmin:
      image: phpmyadmin/phpmyadmin:latest
      container_name: my-php-myadmin
      ports:
        - "8082:80"
      restart: always
        
      depends_on:
        - db
      environment:
        SPRING_DATASOURCE_USERNAME: root
        SPRING_DATASOURCE_PASSWORD: mukit
    
    
    
    #service 3: definition of your spring-boot app 
    customerservice:                        #it is just a name, which will be used only in this file.
      image: customer-service               #name of the image after dockerfile executes
      container_name: customer-service-app  #name of the container created from docker image
      build:
        context: .                          #docker file path (. means root directory)
        dockerfile: Dockerfile              #docker file name
      ports:
        - "8084:8084"                       #docker containter port with your os port
      restart: always
        
      depends_on:                           #define dependencies of this app
        - db                                #dependency name (which is defined with this name 'db' in this file earlier)
      environment:
        SPRING_DATASOURCE_URL: jdbc:mysql://mysql-db2:3306/customer?createDatabaseIfNotExist=true
        SPRING_DATASOURCE_USERNAME: root
        SPRING_DATASOURCE_PASSWORD: mukit
```

###  4. Update the application.properties file

If you are building your Spring-boot application then update the data-source URL. Put the database container name and listening port number instead of the localhost.

```
spring.datasource.url=jdbc:mysql://mysql-db2:3306/customer?createDatabaseIfNotExist=true
```

### 5. Build Project using Maven

Run the below command from the terminal by opening the terminal on the source code root directory:

```
mvn install -DskipTests
```

Successful execution of this command will create a jar file in the ‘Target’ folder of the application.

### 6. Run command for Docker Compose

Run the below command from the terminal inside your source code root directory

```
docker-compose up --build -d
```

### 7. Explore

After successful execution of all the above steps now writes the ‘docker ps’ command into terminal/PowerShell, and you will see running containers you just made like below:

Running Containers Now visit http://localhost:8082/ for phpMyAdmin.

Username: root Password: mukit The username and password for MySQL are set in the docker-compose file.
