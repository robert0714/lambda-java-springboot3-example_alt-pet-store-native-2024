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
  * https://aws.amazon.com/cn/blogs/china/re-platforming-java-applications-using-the-updated-aws-serverless-java-container/
  * https://catalog.workshops.aws/java-on-aws-lambda/en-US/02-accelerate/graal-plain-java
  * https://stackoverflow.com/questions/69906369/sam-cli-and-quarkus-var-task-bootstrap-no-such-file-or-directory
  * https://stackoverflow.com/questions/64749387/micronaut-graalvm-native-image-lambda-fails-with-an-error-error-fork-exec-va

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
### Micronaut GraalVM Native Image: Lambda fails with an error "Error: fork/exec /var/task/bootstrap: no such file or directory"
If you ecounter the below error:  
`Error: fork/exec /var/task/bootstrap: no such file or directory Runtime.InvalidEntrypoint`

If this is the issue then you can fix it by using the linux command: `dos2unix bootstrap` on your bootstrap file and then rebuild the native image.


