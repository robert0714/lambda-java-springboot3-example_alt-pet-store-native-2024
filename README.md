In this sample, you'll build a native GraalVM image for running web workloads in AWS Lambda.
# Serverless Spring Boot 3 example
A basic pet store written with the [Spring Boot 3 framework](https://projects.spring.io/spring-boot/). Unlike older examples, this example is relying on the new 
`SpringDelegatingLambdaContainerHandler`, which you simply need to identify as a   _handler_  of the Lambda function. The main configuration class identified as `MAIN_CLASS`
environment variable or `Start-Class` or `Main-Class` entry in Manifest file. See provided `template.yml` file  for reference. 


The application can be deployed in an AWS account using the [Serverless Application Model](https://github.com/awslabs/serverless-application-model). The `template.yml` file in the root folder contains the application definition.

## Pre-requisites
* [AWS CLI](https://aws.amazon.com/cli/)
* [SAM CLI](https://github.com/awslabs/aws-sam-cli)
* [Gradle](https://gradle.org/) or [Maven](https://maven.apache.org/)

## To build the sample

You first need to build the function, then you will deploy it to AWS Lambda.

Please note that the sample is for `x86` architectures. In case you want to build and run it on ARM, e.g. Apple Mac M1, M2, ... 
you must change the according line in the `Dockerfile` to `ENV ARCHITECTURE aarch64`. 
In addition, uncomment the `arm64` Architectures section in `template.yml`.

### Step 1 - Build the native image

Before starting the build, you must clone or download the code in **pet-store-native**.

1. Change into the project directory: `samples/springboot3/pet-store-native`
2. Run the following to build a Docker container image which will include all the necessary dependencies to build the application 
   ```
   docker build -t al2023-graalvm21:native-web .
   ```
3. Build the application within the previously created build image
   ```
   docker run -it -v `pwd`:`pwd` -w `pwd` -v ~/.m2:/root/.m2 al2023-graalvm21:native-web mvn clean -Pnative package -DskipTests
   ```
4. After the build finishes, you need to deploy the function:
 ```
   sam deploy --guided
 ```

This will deploy your application and will attach an AWS API Gateway
Once the deployment is finished you should see the following:
```
Key                  ServerlessWebNativeApi
Description          URL for application
Value                https://xxxxxxxx.execute-api.us-east-2.amazonaws.com/pets 
```

You can now simply execute GET on this URL and see the listing fo all pets. 


## Reference
* https://github.com/aws/serverless-java-container/wiki/Quick-start---Spring-Boot3
  * Orign source: 
    * https://github.com/aws/serverless-java-container/tree/main/samples/springboot3/pet-store-native
  * related articles 
    * https://aws.amazon.com/cn/blogs/china/re-platforming-java-applications-using-the-updated-aws-serverless-java-container/
    * https://catalog.workshops.aws/java-on-aws-lambda/en-US/02-accelerate/graal-plain-java
    * https://stackoverflow.com/questions/69906369/sam-cli-and-quarkus-var-task-bootstrap-no-such-file-or-directory
    * https://stackoverflow.com/questions/64749387/micronaut-graalvm-native-image-lambda-fails-with-an-error-error-fork-exec-va
* Serverless Java with Spring
  * Video: [Serverless Java with Spring by Maximilian Schellhorn & Dennis Kieselhorst @ Spring I/O 2024](https://youtu.be/AFIHug_HujI)
  * Slide: https://speakerdeck.com/deki/serverless-java-with-spring
    * Method 1: Handling via functions
      * [Tradition](https://docs.aws.amazon.com/zh_tw/lambda/latest/dg/java-handler.html#java-best-practices)
      * Handling via Spring Cloud Functions ( Spring Cloud AWS is designed for non-lambda , other aws services)
      * Multiple functions
    * Method 1: HTTP adapter
      * AWS Serverless Java: [serverless-java-container](https://github.com/aws/serverless-java-container)
    * Summary    
      |Method|Handling via functions|HTTP adapter|
      |------|--------------------------------|----------------------|
      |Approach|Directly deserialize JSON payloads into Java objects without involving HTTP request/response abstractions.|An adapter transforms events into HTTP request/response objects (e.g., Jakarta Servlet API).|
      |Usage|Process content using event-specific code.|Retains compatibility with existing HTTP-based frameworks.|
      |Strengths|1. Well-suited for non-HTTP use cases.<br/>2. Standardized for serverless or event-driven architectures.|Enables reuse of existing code and frameworks without modification|
      |Weaknesses|More challenging to adapt existing applications and frameworks that rely on HTTP constructs.|Introduces performance overhead due to the additional transformation layer.|
* [2022 Reduce Java cold starts by 10x with AWS Lambda](https://youtu.be/Y5b8_KToeDY?t=1163)
* Other implements
  * Spring Cloud AWS 3
    * https://github.com/awspring/spring-cloud-aws
    * Video
      * Spring I/O 2024: [Spring Cloud AWS 3 upgrade and customisation for over 100 teams at Ocado Technology by M. Telepchuk](https://youtu.be/-PgFoRGaa6s)
    * baeldung
       * [github](https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-aws-v3) 
  * Spring Cloud Function
    * LocalStack: https://docs.localstack.cloud/user-guide/integrations/spring-cloud-function/ 
    * Recommended books
      * [Practical Spring Cloud Function: Developing Cloud-Native Functions for Multi-Cloud and Hybrid-Cloud Environments](https://link.springer.com/book/10.1007/978-1-4842-8913-6) ![cover](https://learning.oreilly.com/library/cover/9781484289136/250w/)
# TEARING DOWN RESOURCES
When you run `sam deploy`, it creates or updates a CloudFormation `stack`—a set of resources that has a name, which you’ve seen already with the `--stack-name` parameter of `sam deploy`.

When you want to clean up your AWS account after trying an example, the simplest method is to find the corresponding CloudFormation stack in the AWS Web Console (in the CloudFormation section) and delete the stack using the **Delete** button.

Alternatively, you can tear down the stack from the command line. For example, to tear down the **alt-pet-store** stack, run the following:
```bash
$ PIPELINE_BUCKET="$(aws cloudformation describe-stack-resource --stack-name alt-pet-store --logical-resource-id PipelineStartBucket --query 'StackResourceDetail.PhysicalResourceId' --output text)"
$ aws s3 rm s3://${PIPELINE_BUCKET}/sampledata.json
$ aws s3 rm s3://${PIPELINE_BUCKET} --recursive
$ aws cloudformation delete-stack --stack-name alt-pet-store
```

# SAM LOCAL
## Ready
```bash
vagrant up
vagrant ssh
sudo apt install -y wget unzip


curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version

wget https://github.com/aws/aws-sam-cli/releases/latest/download/aws-sam-cli-linux-x86_64.zip
unzip aws-sam-cli-linux-x86_64.zip -d sam-installation
sudo ./sam-installation/install
sam --version
```
## Test in sam Local
```bash
sam local start-api
```
another terminal
```bash
vagrant@sam:~$ curl --location 'http://127.0.0.1:3000/pets' 

vagrant@sam:~$ curl --location 'http://127.0.0.1:3000/pets' \
--header 'Content-Type: application/json' \
--data '{
  "name": "Robert",
  "breed": "Norwegian Elkhound"
}'
```

## GraalVM Native Image
### Micronaut GraalVM Native Image: Lambda fails with an error "Error: fork/exec /var/task/bootstrap: no such file or directory"
If you ecounter the below error:  
`Error: fork/exec /var/task/bootstrap: no such file or directory Runtime.InvalidEntrypoint`

If this is the issue then you can fix it by using the linux command: `dos2unix bootstrap` on your bootstrap file and then rebuild the native image.

### NativeHints
* NativeHintsConfiguration
  ```java
  @Configuration
  public class NativeHintsConfiguration implements RuntimeHintsRegistrar {
    @Override
    public void registerHints(RuntimeHints hints, ClassLoader classLoader) {

        // Affect compilation
        hints.reflection().registerType(
            com.google.maps.DirectionsApi.Response.class,
            MemberCategory.values());  
        hints.reflection().registerType(
        	 com.google.maps.model.GeocodedWaypoint.class,
             MemberCategory.values());
        hints.reflection().registerType(
        	 com.google.maps.model.DirectionsRoute.class,
             MemberCategory.values());
        hints.reflection().registerType(
        	 com.google.maps.model.Bounds.class,
             MemberCategory.values());	
        hints.reflection().registerType(
        	 com.google.maps.model.DirectionsLeg.class,
             MemberCategory.values());		
        hints.reflection().registerType(
        	 com.google.maps.model.DirectionsStep.class,
             MemberCategory.values());
        hints.reflection().registerType(
        	 com.google.maps.model.EncodedPolyline.class,
             MemberCategory.values());	

         // Affect our result
        hints.reflection().registerType(
        		com.XXXXXXX.distance_calculator.model.RouteResult.class, 
                MemberCategory.values());     
    }
  }
  ```
  * Application
  ```java
  @SpringBootApplication
  @Import({ DistanceController.class })
  @ImportRuntimeHints(NativeHintsConfiguration.class)   // Affect compilation
  public class Application {
  (ommitted...)
  ```  

