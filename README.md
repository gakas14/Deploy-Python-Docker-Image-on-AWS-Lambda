# Deploy-Python-Docker-Image-on-AWS-Lambda
This project aims to get the current Bitcoin price using an API call. We build this project using AWS CDK, Docker, and AWS lambda. 

### Create AWS CDK project 
```
cdk init app --language typescript
```
<img width="589" alt="Screen Shot 2024-02-01 at 12 25 57 PM" src="https://github.com/gakas14/Deploy-Python-Docker-Image-on-AWS-Lambda/assets/74584964/7fba15e9-eea0-424a-b6b4-70e16b5612e2">

### Create a Python hander and a docker file
  - create a folder image and sub-folder src
  - create a Python file inside /src.
  - create a docker file and a requirements file inside the image.

### configure the files

##### Dockerfile
```
# Base image
FROM public.ecr.aws/lambda/python:3.11

# Copy requirements.txt
COPY requirements.txt ${LAMBDA_TASK_ROOT}

# Install the specified packages
RUN pip install -r requirements.txt

# Copy all files in ./src
COPY src/* ${LAMBDA_TASK_ROOT}

#Set the CMD to your handler
CMD ["main.handler"]
```
#### requirements.tx
```
requests
```

#### docker-lambda_stack.ts
```
import * as cdk from "aws-cdk-lib";
import { Construct } from "constructs";
import * as lambda from "aws-cdk-lib/aws-lambda";

export class DockerBitcoinStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const dockerFunc = new lambda.DockerImageFunction(this, "DockerFunc", {
        code:lambda.DockerImageCode.fromImageAsset("./image"),
        memorySize: 1024,
        timeout: cdk.Duration.seconds(10),


    });

    const functionUrl = dockerFunc.addFunctionUrl({
        authType:lambda.FunctionUrlAuthType.NONE,
        cors: {
            allowedMethods: [lambda.HttpMethod.ALL],
            allowedHeaders: ["*"],
            allowedOrigins: ["*"],
        },

    });

    new cdk.CfnOutput(this, "FunctionUrlValue", {
        value: functionUrl.url,

    });


  }
}
```
#### Test the docker image locally
```
docker build -t docker-image:test .
docker run -p 9000:8080 docker-image:test
```

### Deploy the project on AWS
```
cdk bootstrap -region <>
cdk deploy
```

