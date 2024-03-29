#    Phonebook Application
A Rest API developed with Spring Boot in Kotlin that allows contact to be saved, searched and deleted.
This project was created as a way to solidify and build on existing Kotlin knowledge in response to a job application task.
While building this project I encountered a number of problems which I've to a large extent been able to overcome.
I couldn't solve one in particular - a problem I encounter in my test class for my Controller layer.
I'm still continuing search for the solution to this problem. I'm a TDD guy and not been able to write test cases for my
controller class bugs me. However enjoy reading through.

### Languages: Kotlin
### Framework: Spring, Hibernate

### Technologies
Java 17
Spring Boot(rest api)   
Maven(build automation tool)  
Lombok(eliminates boilerplate code)    
Hibernate(ORM/code-first approach)      
MySql (relational database)   
Junit Jupiter(Spring Boot test)
H2 (relational in app database used for testing)   
Mockito (unit testing/mocking layers)  
Mockk (unit testing/mocking layers)
Docker(containerization)    
Mockaroo (fake data api)  

### Endpoints
The base URL for our application is - "/api/v1/phonebook/".
There are in total 4 APIs to reach the service layer of our REST application
1. "/save-contact": saves a single contact; requires a request object of type Entry and returns a ResponseEntity object
   containing the newly saved contact.
2. "/": gets all the contact in a phonebook app; takes no arguments, returns ResponseEntity object containing List of all contacts.
3. "/search-contact": searches for contacts that has matching name field of our contact; needs a request parameter
   of type String and returns a List of matching contacts.
4. "delete-contact": deletes a single contact per call, needs a request parameter of type String.


### To run the app locally
Pull the project.
Create a database named "contacts" in MySql.  
Create a .env file at the root level of the project directory.  
Open src>main>resources>"application-prod.properties" file.
Note the following:
```
spring.jpa.hibernate.ddl-auto=create  
pring.datasource.username=${MYSQL_USER}
spring.datasource.password=${MYSQL_PASSWORD}
spring.datasource.url=jdbc:mysql://${DOCKER_HOST}:3306/${MYSQL_DATABASE}?createDatabaseIfNotExist=true
```
Go back to your .env file and create variables to mirror the following:
```
MYSQL_USER=[put your MySql username here(probably root)]
MYSQL_PASSWORD=[put your MySql password here]
MYSQL_DATABASE=contacts
```
Now we can run our our app on localhost. Just start the main method in our main class.  

### To run as a container
Since I didn't use docker compose:  
First, we'll need to define a network to run our 2 services (containers)  
Second, provide an image for our db  
Third, convert our app to a docker image
Fourth, run (or containerize) both image in separate containers but on same network: one for phonebook app and the other for mysql.

Please follow the following instructions in order carefully:
1. cd to the directory of the folder
2. Create a network to run our containers on using the command below
```
docker network create [specify network name] (e.g. docker network create phonebook-mysql-net)
```
3. Pull MySql image from public repository
```
docker pull [specify image]:[specify tag] (e.g. docker pull mysql:8.0)
```
4. Next let's edit our .env file. We need to so that we can pass the env vars in .env file to containerize our image  
  a. Add field MYSQL_ROOT_PASSWORD key and provide a value as your root user password.  
  b. Next edit the MY_SQL_USER key value. If the value is root, change it to - say your name. Docker will complain if it is root.
```
MYSQL_ROOT_PASSWORD={specify password]
MYSQL_USER=[specify a name other than root]
```
5. Run the next command to containerize and start mysql service. Confirm using the "docker ps" command.
```
docker run --name [sprecify a name for your container] --network [specify your network name] --env-file=.env -d [image name:image tag]
(e.g. docker run --name contacts_db --network phonebook-mysql-net --env-file=.env -d mysql:5.7
```
6. Then we'll build our application image using the following command
```
docker build -t [your image name here] here.
(e.g. docker build -t phonebook . )
```
7. Next we need to containerize our phonebook app. We'll do this using specifications inside our Dockerfile.   
  a. Create a Dockerfile at the root of the project directory.  
  b. This project uses Java 17  
  c.This next step is has an option. You either choose to do what's in this step. Or 
    you jump over to step d. If you have data and space to spare on you PC, 
    add the following to the Dockerfile:
```
From alpine/latest
WORKDIR /app
COPY target/Phonebook-1.jar /app/phonebook-1.jar
MAINTAINER "Your_name_here"
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app/phonebook-1.jar"]
```
  d. This next step is optional.
  If you're like me that likes and seeks a better way you can instead go this route.
  The step in c above will likely download into repository an image size around 500mb.
  I just didn't like that one application I'm never gonna run always eats that much of my data
  and especially takes that much space on my local machine. So I looked for a better way 
  on the internet and I found on Wolt blog. 
  For explanation, check out post out [here](https://careers.wolt.com/en/blog/tech/how-to-reduce-jvm-docker-image-size).
```dockerfile
# base image to build a JRE
FROM amazoncorretto:17.0.3-alpine as corretto-jdk

# required for strip-debug to work
RUN apk add --no-cache binutils

# Build small JRE image
RUN $JAVA_HOME/bin/jlink \
         --verbose \
         --add-modules ALL-MODULE-PATH \
         --strip-debug \
         --no-man-pages \
         --no-header-files \
         --compress=2 \
         --output /customjre

# main app image
FROM alpine:latest
ENV JAVA_HOME=/jre
ENV PATH="${JAVA_HOME}/bin:${PATH}"

# copy JRE from the base image
COPY --from=corretto-jdk /customjre $JAVA_HOME

#FROM openjdk:8-jdk-alpine
WORKDIR /app
COPY target/Phonebook-1.jar /app/phonebook-1.jar
MAINTAINER "Your_name_here"
EXPOSE 8080
ENTRYPOINT ["/jre/bin/java", "-jar", "/app/phonebook-1.jar"]
```
8. Finally, run the following command to containerize our application.
```
 docker run --network [specify network name] --name [specify container name] --env-file=.env -p 8080:8080 -d [application name]
 (e.g. docker run --network springboot-mysql-net --name phonebook_app --env-file=.env -p 8080:8080 -d phonebook)
```

### Issues
While testing my Controller layer, I had a persistent problem with my autowired mockMvc object.
The default value for the Header fields "Content-Type" and "Accept" for the mockMvc(and mockkMvc) instance is set to 
inputStream object while I set both fields value to Application.JSON. This makes the test to fail repeatedly.
However, if same endpoints are tested on Postman, it return 200 http status as expected.
