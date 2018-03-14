# The application is forked from Spring-boot sample project

REST version of Spring PetClinic Sample Application (spring-framework-petclinic extend ) [![Build Status](https://travis-ci.org/spring-petclinic/spring-petclinic-rest.png?branch=master)](https://travis-ci.org/spring-petclinic/spring-petclinic-rest/)

This backend version of the Spring Petclinic application only provides a REST API. **There is no UI**.
The [spring-petclinic-angular project](https://github.com/spring-petclinic/spring-petclinic-angular) is a Angular 5 front-end application witch consumes the REST API.

## Integrate with aws-serverless-java-container

![image](src/main/resources/images/SpringBoot-lambda-wrapper.png)

By using the aws-serverless-java-container wrapper library, you can easily intercept all the incoming web traffic in to API Gateway.
And the best practice is bind a lambda function with proxy integration mode onto API Gateway. 

All the incoming traffic will first be received by APIG and then delegate to your legacy spring controller.

### Integration Steps 

1.  Add Maven dependency
    ```
    <dependency>
        <groupId>com.amazonaws.serverless</groupId>
        <artifactId>aws-serverless-java-container-spring</artifactId>
        <version>[0.8,)</version>
    </dependency>
    
    For current latest version is 1.0, you can just specify the version to 1.0
    ```
2.  Add StreamLambdaHandler
    ```
    public class WebLambdaHandler implements RequestStreamHandler {
        private static SpringBootLambdaContainerHandler<AwsProxyRequest, AwsProxyResponse> handler;
        static {
            try {
                handler = SpringBootLambdaContainerHandler.getAwsProxyHandler(ServletInitializer.class);
            } catch (ContainerInitializationException e) {
                // if we fail here. We re-throw the exception to force another cold start
                e.printStackTrace();
                throw new RuntimeException("Could not initialize Spring Boot application", e);
            }
        }
    
        public WebLambdaHandler() {
            // we enable the timer for debugging. This SHOULD NOT be enabled in production.
            Timer.enable();
        }
    
        @Override
        public void handleRequest(InputStream inputStream, OutputStream outputStream, Context context)
            throws IOException {
            handler.proxyStream(inputStream, outputStream, context);
    
            // just in case it wasn't closed by the mapper
            outputStream.close();
        }
    }

    ```

3.  Run all unit test
    ```
    mvn test
    ```
    > Nothing special, you can just run the following command to check all the changes are well-tested.
    In order words, You still use the Junit + Mockito + SpringMock related tools to maintain the quality

4.  Write a SAM file to do the deployment
    ```
    AWSTemplateFormatVersion: '2010-09-09'
    Transform: AWS::Serverless-2016-10-31
    Description: Example Pet Store API written with SpringBoot with the aws-serverless-java-container library
    Resources:
      PetStoreFunction:
        Type: AWS::Serverless::Function
        Properties:
          Handler: org.springframework.samples.petclinic.WebLambdaHandler::handleRequest
          Runtime: java8
          CodeUri: target/spring-petclinic-1.5.2.jar
          MemorySize: 1512
          Policies: AWSLambdaFullAccess
          Timeout: 60
          Events:
            GetResource:
              Type: Api
              Properties:
                Path: /{proxy+}
                Method: any
    
    Outputs:
      SpringBootPetStoreApi:
        Description: URL for application
        Value: !Sub 'https://${ServerlessRestApi}.execute-api.${AWS::Region}.amazonaws.com/Prod/api/owners'
        Export:
          Name: SpringBootPetClinicApi
    ```
5.  Deploy it
    ```
    aws cloudformation package --template-file sam.yaml --output-template-file output-sam.yaml --s3-bucket lambda-jar-upload
    
    aws cloudformation deploy --template-file output-sam.yaml --stack-name spring-petclinic --capabilities CAPABILITY_IAM 
    
    aws cloudformation describe-stacks --stack-name spring-petclinic | jq '.Stacks[0].Outputs[0].OutputValue'
    
    ```
    > Then you can hit the provided api gateway url to access the owners list in json payload.
## Screenshot of the Angular 5 client

The application is backed to serve spring-petclinic-angular5 client, so if you want to have a integrated ui experiense, then you would like to clone it and run it.
Partially modify the service api path to the integrated API Gateway.

Migrate the front-end web site to s3-static web hosting site and then Run full integration test

<img width="1427" alt="spring-petclinic-angular2" src="https://cloud.githubusercontent.com/assets/838318/23263243/f4509c4a-f9dd-11e6-951b-69d0ef72d8bd.png">


## Database configuration

In its default configuration, Petclinic uses an in-memory database (HSQLDB) which
gets populated at startup with data.
A similar setups is provided for MySql and PostgreSQL in case a persistent database configuration is needed.
To run petclinic locally using persistent database, it is needed to change profile defined in application.properties file.
