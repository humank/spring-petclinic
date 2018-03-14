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

1. Add Maven dependency
2. Add StreamLambdaHandler
3. Run all unit test
4. Write a SAM file to do the deployment
5. Deploy it

```
aws cloudformation package --template-file sam.yaml --output-template-file output-sam.yaml --s3-bucket lambda-jar-upload

aws cloudformation deploy --template-file output-sam.yaml --stack-name spring-petclinic --capabilities CAPABILITY_IAM 

aws cloudformation describe-stacks --stack-name spring-petclinic | jq '.Stacks[0].Outputs[0].OutputValue'

```

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
