# rdm_repo
Save from Anvndev.com
Building microservices with Dropwizard, MongoDB & Docker
#
java
#
programming
#
mongodb
#
tutorial
Dropwizard making magic. 😎
In this tutorial I will create a microservice using Dropwizard, this framework provides all kind of libraries to build a web application. Some of the libraries that include are:

Jetty Server: this is an open-source web server, lightweight and easy to embed in any application.

Jersey: implementation for creating RESTful web services.

Jackson: allows you manipulation from JSON objects to POJOs and vice versa.

Metrics: provides information about the state of HTTP requests, database connections, queues, etc.

Guava: support with many classes to deal with collections, validations, strings etc.

Hibernate Validator: validations as constrains for java objects.

JDBI: wraps JDBC and provides a flexible way to manipulate relational databases.

Liquibase: perfect to migrations and DDL changes in a database.

MongoDB a NoSQL database and much more.🌳
This NoSQL database oriented to documents (by documents like JSON) combines some of the features from relational databases, easy to use and the multi-platform is the best option for scale up and have fault tolerance, load balancing, map reduce, etc.

Technologies used in this microservice 📝
OpenJDK 8
Docker 2.0.0.0-mac81 (version for Mac)
MongoDB 4.0
Maven 3.5
Mac OS mojave (or any other like windows or linux)
Nginx 1.15
IntelliJ IDEA 2018
Ok let's start to create the project, Dropwizard provide us a maven archetype that we can use from its web site, here a example of this command:
mvn archetype:generate -DarchetypeGroupId=io.dropwizard.archetypes -DarchetypeArtifactId=java-simple -DarchetypeVersion=1.3.5 -DgroupId=com.demo -DartifactId=dropwizard-mongodb-ms -Dversion=1.0.0-SNAPSHOT -Dname=DropwizardMongoDBMicroservice
Open a terminal and paste the before the command, make sure you have installed and configurated maven and java.

The final structure looks like this:
dropwizard-mongodb-ms/
├── README.md
├── config.yml
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── demo
    │   │           ├── DropwizardMongoDBMicroserviceApplication.java
    │   │           ├── DropwizardMongoDBMicroserviceConfiguration.java
    │   │           ├── api
    │   │           ├── cli
    │   │           ├── client
    │   │           ├── core
    │   │           ├── db
    │   │           ├── health
    │   │           └── resources
    │   └── resources
    │       ├── assets
    │       └── banner.txt
    └── test
        ├── java
        │   └── com
        │       └── demo
        │           ├── api
        │           ├── client
        │           ├── core
        │           ├── db
        │           └── resources
        └── resources
            └── fixtures
Once the project was created, add the dependencies in the pom.xml file:
. . .
<properties>
        <dropwizard.version>1.3.5</dropwizard.version>
        <mainClass>com.demo.DropwizardMongoDBMicroserviceApplication</mainClass>
        <mongodb.version>3.8.2</mongodb.version>
        <jdk.version>1.8</jdk.version>
        <dropwizard.swagger.version>1.0.6-1</dropwizard.swagger.version>
        <mockito.core.version>2.23.0</mockito.core.version>
</properties>
. . .
<dependency>
    <groupId>io.dropwizard</groupId>
    <artifactId>dropwizard-core</artifactId>
</dependency>
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongodb-driver-sync</artifactId>
    <version>${mongodb.version}</version>
</dependency>

<dependency>
    <groupId>com.smoketurner</groupId>
    <artifactId>dropwizard-swagger</artifactId>
    <version>${dropwizard.swagger.version}</version>
</dependency>

<!-- Testing -->
<dependency>
    <groupId>io.dropwizard</groupId>
    <artifactId>dropwizard-testing</artifactId>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>${mockito.core.version}</version>
    <scope>test</scope>
</dependency>
<!-- Testing -->
. . .
I added some configurations to the file configuration.yaml, looks like this:
server:
  maxThreads: 512
  applicationContextPath: /dropwizard-mongodb-ms
  applicationConnectors:
    - type: http
      port: 8080
  adminConnectors:
    - type: http
      port: 8081

logging:
  level: INFO
  loggers:
    com.demo: INFO

#You can choose the user and password what you want.
mongoDBConnection:
  credentials:
    username: "user_donuts" 
    password: "pAsw0Rd"
  seeds:
    - host: "mongodb"
      port: 27017
  database: "donuts"

swagger:
  basePath: /dropwizard-mongodb-ms
  resourcePackage: com.demo.resources
  scan: true
  info:
    version: "1.0.0"
    title: "Donuts API CRUD"
    description: "A simple API used for expose CRUD operation on MongoDB collection"
    termsOfService: "http://swagger.io/terms/"
    contact:
      name: "Donuts API "
    license:
      name: "Rich Lopez"
The configuration.yml maps to a class that extends of io.dropwizard.Configuration and looks like this:
package com.demo;

import com.demo.db.configuration.MongoDBConnection;
import com.fasterxml.jackson.annotation.JsonProperty;

import io.dropwizard.Configuration;
import io.federecio.dropwizard.swagger.SwaggerBundleConfiguration;

public class DropwizardMongoDBMicroserviceConfiguration extends Configuration {

    /**
     * The data configuration for MongoDB.
     */

    private MongoDBConnection mongoDBConnection;

    @JsonProperty("swagger")
    private SwaggerBundleConfiguration swaggerBundleConfiguration;

    //Getters and setters
}
This microservice will expose an API REST base on a CRUD (Create, Read, Update, Delete) I've chosen a product like donuts (🍩 🤤), some parts of the code you can find in my Github repository at the end of this tutorial.

I'm using the com.mongodb.client.MongoCollection interface for access to the collection called donuts to have more granularity in the operations so I created a DAO (Data Access Object) to manipulate the CRUD operations.
package com.demo.db.daos;
//imports ... 
public class DonutDAO {

    /** The collection of Donuts */
    final MongoCollection<Document> donutCollection;

    /**
     * Constructor.
     *
     * @param donutCollection the collection of donuts.
     */
    public DonutDAO(final MongoCollection<Document> donutCollection) {
        this.donutCollection = donutCollection;
    }

    /**
     * Find all donuts.
     *
     * @return the donuts.
     */
    public List<Donut> getAll() {
        final MongoCursor<Document> donuts = donutCollection.find().iterator();
        final List<Donut> donutsFind = new ArrayList<>();
        try {
            while (donuts.hasNext()) {
                final Document donut = donuts.next();
                donutsFind.add(DonutMapper.map(donut));
            }
        } finally {
            donuts.close();
        }
        return donutsFind;
    }

    /**
     * Get one document find in other case return null.
     *
     * @param id the identifier for find.
     * @return the Donut find.
     */
    public Donut getOne(final ObjectId id) {
        final Optional<Document> donutFind = Optional.ofNullable(donutCollection.find(new Document("_id", id)).first());
        return donutFind.isPresent() ? DonutMapper.map(donutFind.get()) : null;
    }

    public void save(final Donut donut){
        final Document saveDonut = new Document("price", donut.getPrice())
                                      .append("flavor", donut.getFlavor());
        donutCollection.insertOne(saveDonut);
    }

    /**
     * Update a register.
     *
     * @param id the identifier.
     * @param donut the object to update.
     */
    public void update(final ObjectId id, final Donut donut) {
        donutCollection.updateOne(new Document("_id", id),
                new Document("$set", new Document("price", donut.getPrice())
                        .append("flavor", donut.getFlavor()))
        );
    }

    /**
     * Delete a register.
     * @param id    the identifier.
     */
    public void delete(final ObjectId id){
        donutCollection.deleteOne(new Document("_id", id));
    }
For map the information that comes from the database I use a utility class that filters the fields from MongoDB to a POJO.
package com.demo.util;

import com.demo.api.Donut;
import org.bson.Document;

public class DonutMapper {

    /**
     * Map objects {@link Document} to {@link Donut}.
     *
     * @param donutDocument the information document.
     * @return A object {@link Donut}.
     */
    public static Donut map(final Document donutDocument) {
        final Donut donut = new Donut();
        donut.setId(donutDocument.getObjectId("_id"));
        donut.setFlavor(donutDocument.getString("flavor"));
        donut.setPrice(donutDocument.getDouble("price"));
        return donut;
    }
}
Then create a POJO to manipulate the information on MongoDB.
package com.demo.api;
//imports ...
public class Donut implements Serializable {

    /** The id.*/
    @JsonSerialize(using = ObjectIdSerializer.class)
    private ObjectId id;

    /** The price. */
    @NotNull
    private double price;

    /** The principal flavor.*/
    @NotNull
    private String flavor;

    /**Constructor.*/
    public Donut() {
    }

//getters & setters
//hashcode, equals and toString methods
This class is only used in response to the endpoint.
package com.demo.api;
//imports ...
public class Response {
    /** The message.*/
    private String message;

    /** Constructor.*/
    public Response() {
    }
//getters & setters
//hashcode, equals and toString methods
Here more utility classes for convert org.bson.types.ObjectId to String objects using Jackson.
package com.demo.util;

import java.io.IOException;

import org.bson.types.ObjectId;

import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.databind.JsonSerializer;
import com.fasterxml.jackson.databind.SerializerProvider;

public class ObjectIdSerializer extends JsonSerializer<ObjectId> {
    @Override
    public void serialize(final ObjectId objectId, final JsonGenerator jsonGenerator,
                          final SerializerProvider serializerProvider) throws IOException {
        jsonGenerator.writeString(objectId.toString());
    }
}
For convert String object to array of char.
package com.demo.util;

import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.databind.JsonSerializer;
import com.fasterxml.jackson.databind.SerializerProvider;

import java.io.IOException;

public class PasswordSerializer extends JsonSerializer<String> {
    @Override
    public void serialize(final String input, final JsonGenerator jsonGenerator,
                          final SerializerProvider serializerProvider) throws IOException {
        jsonGenerator.writeString(input.toCharArray(), 0, input.toCharArray().length);
    }

    @Override
    public Class<String> handledType() {
        return String.class;
    }
}
Use these POJOs that map information with the configuration.yml.
package com.demo.db.configuration;

import java.util.Arrays;
import java.util.Objects;

import com.demo.util.PasswordSerializer;
import com.fasterxml.jackson.databind.annotation.JsonSerialize;

/**
 * This class is used for credentials.
 * @version 1.0.0
 * @since 1.0.0
 * @author Rich Lopez
 */
public class Credentials {

    /** The user name.*/
    private String username;

    /** The password.*/
    @JsonSerialize(using = PasswordSerializer.class)
    private char[] password;

    //getters, setters, hashcode and equals methods.
}
package com.demo.db.configuration;

import java.util.Objects;

public class Seed {

    /** The host.*/
    private String host;

    /** The port.*/
    private int port;
    //getters, setters, hashcode and equals methods.
}
package com.demo.db.configuration;

import java.util.List;

public class MongoDBConnection {

    /**
     * The credentials user and password.
     */
    private Credentials credentials;

    /**
     * The lis of seeds.
     */
    private List<Seed> seeds;

    /**
     * The db.
     */
    private String database;
}
For manage the connection with MongoDB we need map the information come from the yaml file configuration to this classes com.demo.db.configuration.Credentials , com.demo.db.configuration.MongoDBConnection, com.demo.db.configuration.Seed.

The class com.demo.db.MongoDBFactoryConnection creates the client for MongoDB with the options that provided in yaml file, the method for create the instance look like:
package com.demo.db;
//imports ...
public class MongoDBFactoryConnection {

    /** The configuration for connect to MongoDB Server.*/
    private MongoDBConnection mongoDBConnection;

    /**
     * Constructor.
     *
     * @param mongoDBConnection the mongoDB connection data.
     */
    public MongoDBFactoryConnection(final MongoDBConnection mongoDBConnection) {
        this.mongoDBConnection = mongoDBConnection;
    }

    /**
     * Gets the connection to MongoDB.
     *
     * @return the mongo Client.
     */
    public MongoClient getClient() {
        LOGGER.info("Creating mongoDB client.");
        final Credentials configCredentials = mongoDBConnection.getCredentials();

        final MongoCredential credentials = MongoCredential.createCredential(
                configCredentials.getUsername(),
                mongoDBConnection.getDatabase(),
                configCredentials.getPassword());

        final MongoClient client = MongoClients.create(
                MongoClientSettings.builder()
                        .credential(credentials)
                        .applyToClusterSettings(builder -> builder.hosts(getServers())).build()
        );

        return client;
    }

    /**
     * Map the object {@link Seed} to objects {@link ServerAddress} that contain the information of servers.
     *
     * @return the list of servers.
     */
    private List<ServerAddress> getServers() {
        final List<Seed> seeds = mongoDBConnection.getSeeds();
        return seeds.stream()
                .map(seed -> {
                    final ServerAddress serverAddress = new ServerAddress(seed.getHost(), seed.getPort());
                    return serverAddress;
                })
                .collect(Collectors.toList());
    }
In Dropwizard life cycle the object can be manage by the interface io.dropwizard.lifecycle.Managed and the class io.dropwizard.lifecycle.MongoDBManaged.
package com.demo.db;
//imports ...
public class MongoDBManaged implements Managed {

    /** The mongoDB client.*/
    private MongoClient mongoClient;

    /**
     * Constructor.
     * @param mongoClient   the mongoDB client.
     */
    public MongoDBManaged(final MongoClient mongoClient) {
        this.mongoClient = mongoClient;
    }

    @Override
    public void start() throws Exception {
    }

    @Override
    public void stop() throws Exception {
        mongoClient.close();
    }
Another important class is the Healthcheck, for creating this class is only required to extend from com.codahale.metrics.health.HealthCheck and implement the method check.
package com.demo.health;
//imports ...
public class DropwizardMongoDBHealthCheck extends HealthCheck {
     /** A client of MongoDB.*/
    private MongoClient mongoClient;

    /**
     * Constructor.
     *
     * @param mongoClient the mongo client.
     */
    public DropwizardMongoDBHealthCheck(final MongoClient mongoClient) {
        this.mongoClient = mongoClient;
    }
    @Override
    protected Result check() {
        try {
            final Document document = mongoClient.getDatabase("donuts").runCommand(new Document("buildInfo", 1));
            if (document == null) {
                return Result.unhealthy("Can not perform operation buildInfo in Database.");
            }
        } catch (final Exception e) {
            return Result.unhealthy("Can not get the information from database.");
        }
        return Result.healthy();
    }
The entry point of this microservice is the class com.demo.DropwizardMongoDBMicroserviceApplication loads all bundles configurations, initialization classes, etc. This class must extend from io.dropwizard.Application and is parametrized by a class that extends of io.dropwizard.Configuration.
package com.demo;

import com.demo.db.MongoDBFactoryConnection;
import com.demo.db.daos.DonutDAO;
import com.demo.db.MongoDBManaged;
import com.demo.health.DropwizardMongoDBHealthCheck;
import com.demo.resources.DonutResource;

import io.dropwizard.Application;
import io.dropwizard.setup.Bootstrap;
import io.dropwizard.setup.Environment;
import io.federecio.dropwizard.swagger.SwaggerBundle;
import io.federecio.dropwizard.swagger.SwaggerBundleConfiguration;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class DropwizardMongoDBMicroserviceApplication extends Application<DropwizardMongoDBMicroserviceConfiguration> {

    /**
     * Logger class.
     */
    private static final Logger LOGGER = LoggerFactory.getLogger(DropwizardMongoDBMicroserviceApplication.class);

    /**
     * Entry point for start Application.
     *
     * @param args the args.
     * @throws Exception when the app can not start.
     */
    public static void main(final String[] args) throws Exception {
        LOGGER.info("Start application.");
        new DropwizardMongoDBMicroserviceApplication().run(args);
    }

    @Override
    public String getName() {
        return "DropwizardMongoDBMicroservice";
    }

    @Override
    public void initialize(final Bootstrap<DropwizardMongoDBMicroserviceConfiguration> bootstrap) {
        bootstrap.addBundle(new SwaggerBundle<DropwizardMongoDBMicroserviceConfiguration>() {
            @Override
            protected SwaggerBundleConfiguration getSwaggerBundleConfiguration(
                        final DropwizardMongoDBMicroserviceConfiguration dropwizardMongoDBMicroserviceConfiguration) {
                return dropwizardMongoDBMicroserviceConfiguration.getSwaggerBundleConfiguration();
            }
        });
    }

    @Override
    public void run(final DropwizardMongoDBMicroserviceConfiguration configuration,
                    final Environment environment) {

        final MongoDBFactoryConnection mongoDBManagerConn = new MongoDBFactoryConnection(configuration.getMongoDBConnection());

        final MongoDBManaged mongoDBManaged = new MongoDBManaged(mongoDBManagerConn.getClient());

        final DonutDAO donutDAO = new DonutDAO(mongoDBManagerConn.getClient()
                .getDatabase(configuration.getMongoDBConnection().getDatabase())
                .getCollection("donuts"));

        environment.lifecycle().manage(mongoDBManaged);
        environment.jersey().register(new DonutResource(donutDAO));
        environment.healthChecks().register("DropwizardMongoDBHealthCheck",
                new DropwizardMongoDBHealthCheck(mongoDBManagerConn.getClient()));
    }
}
For building the project using the command:

$ mvn clean package

Before to start the application is required to have the database, with MongoDB could you download from the website or using a docker image in DockerHub.

Once download the image of MongoDB, start a new container:

$ docker run --name mongodb -d -p 27017:27017 mongo

NOTE: Ensure the creation of a volume to maintain the persistence of information.

enter the container and type the next commands:

$ docker exec -it [container name or id] /bin/bash
$ mongo
$ use donuts
> db.createUser({ user: "user_donuts", pwd: "pAsw0Rd", roles: [ { role: "readWrite", db: "donuts"} ]});

By default, MongoDB sets the authentication mechanism to SCRAM.

And now start the application with the command:

$ java -jar target/dropwizard-mongodb-ms-1.0.0-SNAPSHOT.jar server configuration.yml

Once we have created the user and the database, we can enter the swagger documentation in the following URL:

http://localhost:8080/dropwizard-mongodb-ms/swagger

Swagger Documentation

Using docker compose 🐳
One of the reasons for use docker every day is because provide us with a unique environment and all developers keep the dependencies, databases, web servers, etc synchronized. The easy way to build and deploy artifacts reduce delivery time.

I’m using docker for start the application and the MongoDB server, besides I add Nginx proxy for receiving the request, this is an example of docker compose file:
version: '3'
services:
  mongodb:
    image: mongo
    restart: always
    container_name: mongodb
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: admin
    ports:
      - 27017:27017
    networks:
      - dropw-mongodb-ntw

  nginx:
    image: nginx
    container_name: nginx
    volumes:
    - ./nginx.conf:/etc/nginx/nginx.conf
    ports:
    - "8080:80"
    - "443:443"
    networks:
    - dropw-mongodb-ntw

  dropw-ms:
    image: openjdk:8-jre
    container_name: dropw-ms
    volumes:
    - ./target/dropwizard-mongodb-ms-1.0.0-SNAPSHOT.jar:/microservice/dropwizard-mongodb-ms-1.0.0-SNAPSHOT.jar
    - ./configuration.yml:/microservice/configuration.yml
    working_dir: /microservice
    command: ["java", "-jar", "dropwizard-mongodb-ms-1.0.0-SNAPSHOT.jar", "server", "configuration.yml"]
    ports:
    - "8090:8080"
    - "8091:8081"
    networks:
    - dropw-mongodb-ntw

networks:
  dropw-mongodb-ntw:
    external:
      name: dropw-mongodb-ntw
We create the docker network with the command :

docker network create dropw-mongodb-ntw

build the project using the command:

$ mvn clean package

Start docker compose:

$ docker-compose up

once we have created all the services we need to create the user for the database, we will access the container with the following commands.

$ docker exec -it [container name or id] /bin/bash
$ mongo
> use donuts
> db.createUser({ user: "user_donuts", pwd: "pAsw0Rd", roles: [ { role: "readWrite", db: "donuts"} ]});

http://localhost:8080/dropwizard-mongodb-ms/swagger

Conclusion
Dropwizard provides us with several tools to create Microservices in a way easy. You can integrate with other technologies like MongoDB, Docker, Nginx by example. Remember that all depends on the requirements and the problem to solve, but for me, this is good for start and share experiences.

Github repository: https://github.com/ricdev2/dropwizard-mongodb-ms
  ``````````````````
  #Cre: Ricardo Dev
