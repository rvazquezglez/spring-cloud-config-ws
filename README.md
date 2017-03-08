# Spring Cloud Config Workshop
The purpose of workshop is to review some of the features of the library and use cases

## Exercise 1 Config Server Setup

In the first exercise we will get the config server up and running.

### Generate project 

* Go to https://start.spring.io/
* Select gradle project and the latests version of Spring boot
* Assign a group and artifact, for example: com.ns.configws:config-server
* Add dependencies for: Config Server
* Download and unpack

### Enable Config Server
* Fork the properties repository https://github.com/blzb/jvm-mx-server-configurations
* Add the following configurations to the application.yml
```
server.port=8888
spring.cloud.config.server.git.uri=[your fork url]
spring.cloud.config.server.git.searchPaths=simple
```
* Add the `@EnableConfigServer` annotation to the main class.
```
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConfigServerApplication.class, args);
	}
}

```
* Run the application
```
gradle bootRun
```
* Request a simple properties file like : http://localhost:8888/writer-dev.properties

### Try differente file structures
* Change the searchPath property to `searchPaths=dir-per-env/{profile}`
* Reboot
* Check how altough the endpoint is the same the location of the files is different
* change the searchPath property to `searchPaths=dir-per-app`
* Check how altough the endpoint is the same the location of the files is different

### Push changes to the git repo
* Change or add a property to one of the properties file
* Commit and push the changes
* Request the file using the config server endpoint
* Check how the value is updated

## Exercise 2 Setup a Client

### Generate project 

* Go to https://start.spring.io/
* Select gradle project and the latests version of Spring boot
* Assign a group and artifact, for example: com.ns.configws:reader
* Add dependencies for: Config Client, Web
* Download and unpack

### Client configuration
* Add a bootstrap.yml file next to the application.yml
* Add the following properties to the bootstrap.yml
```
spring.application.name=reader
spring.cloud.config.uri=http://localhost:8888
```
* Add a Controller that uses `@Value` annotation
```
@Controller
@RequestMapping("app")
public class RepeatController {
    @Value("${app.repeat: 1}")
    Long repeat;

    @RequestMapping(value = "/repeat", method = RequestMethod.GET)
    @ResponseBody String getName() {
        StringBuilder stringBuilder = new StringBuilder();
        for(int i=0;i<repeat; i++){
          stringBuilder.append('*********\n')
        }
        return stringBuilder.toString();
    }
}
```
* Run the app
* Do a get request to /app/repeat

### Update a property
* Change a properties file for the reader, commit and push the change.
* Do a get request to the config server and verify it has changed.
* Check the reader endpoint, it should have change .... but it __hasn't__
* To unable auto update we need to add another anotation to the controller, add `@RefreshScope` annotation.
* Restart the reader app and verify the property value is set.
* Change the property, commit and push the change. 
* Do a POST to http://localhost:8080/refresh
* Verify the value is updated.

## Exercise 3 Async Refresh
* Add spring-cloud-config-monitor to the config server dependencies
* Add spring-cloud-starter-bus-amqp to the reader service 
* Restart both apps
* Start an ngrok process pointing to the config server 
* Add a webhook on githu to push changes to the ngrok url
* Change a property in the reader properties file
* Push and commit the changes.
* Go a get to the reader endpoint, it should have the new value

## Excercise 4 Updating multiple services with the same config server
* Change the rabbitmq configuration on the reader for point to the cloudamqp service.
* Restart the reader app.
* The instructor will start the the config server and push a change to the repo.
* What how all the services update the property.

## Excercise 5 Encrypt values

### Install JCE 
* Download JCE from http://www.oracle.com/technetwork/java/javase/downloads/index.html
* Make a back up of $JAVA_HOME/jre/lib/security files.
* Copy local_policy.jar and US_export_policy.jar into $JAVA_HOME/jre/lib/security
* Create a keystore with the following command
```
keytool -genkeypair -alias mytestkey -keyalg RSA \
  -dname "CN=Web Server,OU=Unit,O=Organization,L=City,S=State,C=US" \
  -keypass changeme -keystore server.jks -storepass letmein
```
### Add encrypt properties to the config server
* Add the following properties to the application.yml
```
encrypt:
  key-store:
    location: file://${user.home}/server.jks
    password: letmein
    alias: mytestkey
    secret: changeme
```
* Or use System variables
```
ENCRYPT_KEY_STORE_PASSWORD=letmein
ENCRYPT_KEY_STORE_ALIAS=mytestkey
ENCRYPT_KEY_STORE_SECRET=changeme</pre>
```
* Start the config server
* Test the encription with a curl
```
curl localhost:8888/encrypt -d 'Hello Spring Boot!'
```
### Use decrypted properties
* Add the following dependency to th reader app
```
org.springframework.security:spring-security-rsa
```
* Use the encrypt endpoint to add a cypher value to the properties file
* Restart the reader app


