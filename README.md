# AWS Cloud Development Kit (AWS CDK)
- This tutorial is based on this AWS documentation `https://docs.aws.amazon.com/cdk/v2/guide/serverless_example.html`. It is further enhanced to add another lambda to the base architecture with API gateway
## Pre-requisites
- Install CDK
```
npm install -g aws-cdk
cdk --version
```
- Install awscli
```
brew install aws-cli
aws version
aws configure
```
- Suggested IAM access
```
ssm:GetParameter
cloudformation:*
ecr:*
iam:*
```
## Building the AWS Resources
### Step 1: Build the Lambda Code
- Initialize the project
```
cdk init --language typescript
```
This will create the following
```
cdk-hello-world
├── .git
├── .gitignore
├── .npmignore
├── README.md
├── bin
│   └── cdk-hello-world.ts
├── cdk.json
├── jest.config.js
├── lib
│   └── cdk-hello-world-stack.ts
├── node_modules
├── package-lock.json
├── package.json
├── test
│   └── cdk-hello-world.test.ts
└── tsconfig.json
- In the `lib\project-stack.ts` file, add an import to the lambda resource
```
import * as lambda from 'aws-cdk-lib/aws-lambda';
```
- Define your constructs in `lib\project-stack.ts`
```
const helloWorldFunction = new lambda.Function(this,'HelloWorldFunction', {
      runtime: lambda.Runtime.NODEJS_20_X, // Choose any supported Node.js runtime
      code: lambda.Code.fromAsset('lambda'), // Points to the lambda directory
      handler: 'hello.handler', // Points to the 'hello' file in the lambda directory
    });
```
Here, `runtime` is the environment the function runs in (ie. Node.js v 20.x). Then, `code` is the function code on your machine; and `handler` is the file that contains the specific code
- Create a lambda directory and a hello.js file
```
mkdir lambda
cd lambda
touch hello.js
```
- Add the following code to it
```
exports.handler = async (event) => {
    return {
        statusCode: 200,
        headers: { "Content-Type": "text/plain" },
        body: JSON.stringify({ message: "Hello, World!" }),
    };
};
```
### Step 2: Define the API Gateway REST API resource
- In `lib\project-stack.ts`, add an import to the API Gateway resource
```
import * as lambda from 'aws-cdk-lib/aws-lambda';
```
- Then add the resource
```
    // Define the API Gateway resource
    const api = new apigateway.LambdaRestApi(this, 'HelloWorldApi', {
      handler: helloWorldFunction,
      proxy: false,
    });
```
- Also, define the /hello resource with a GET method
```
    // Define the '/hello' resource with a GET method
    const helloResource = api.root.addResource('hello');
    helloResource.addMethod('GET');
```
### Step 3: Prepare the application for deployment
- Prepare the application by building the project
```
npm run build
```
- Run `cdk synth` to synthesize an AWS CloudFormation template from your CDK code
```
cdk synth
```
- If successful, the  AWS CDK CLI will show the AWS CloudFormation template in YAML format. A JSON formatted template is saved in `cdk.out` directory
### Step 4: Deploy the application
- Make sure the application is bootstrapped. See `https://docs.aws.amazon.com/cdk/latest/guide/bootstrapping.html`
```
cdk bootstrap
```
- Run `cdk deploy`
## Testing the endpoint
# Running a curl command to test the endpoint
- Run the following curl command
```
curl https://<api-id>.execute-api.<region>.amazonaws.com/prod/hello
{"message":"Hello World!"}% 
curl https://nuijbalhvc.execute-api.us-east-1.amazonaws.com/prod/hello
```
## Cleaning up the resources
# Run a destroy command to remove the resources
- Run `cdk destroy`