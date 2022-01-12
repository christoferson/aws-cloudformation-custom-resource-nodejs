# aws-cloudformation-custom-resource-nodejs
Aws Cloudformation Custom Resource (NodeJS Lambda) 

This project provides example on how to provision a cloudformation custom resource implemented using Lambda NodeJs.
The following instructions will launch 2 cloudformation stacks that will show how to create and use the custom resource.
The first stack named custom-resource will provision the custom resource lambda.
Next, the second stack named custom-resource-example shows how to reference the custom resource.

## Prerequisite
1. AWS Account
2. Configured AWS CLI


## Create the lambda package and upload to S3 bucket
zip package/custom-resource-nodejs.zip resource.js package.json 

## Launch the stack to create the custom resource.
aws cloudformation create-stack \
  --stack-name custom-resource \
  --template-body file://cfn-stacks/custom-resource.yaml \
  --capabilities CAPABILITY_IAM \
  --disable-rollback \
  --parameters \
    ParameterKey="CodeBucket",ParameterValue="my-cloudformation-bucket" \
    ParameterKey="CodeKey",ParameterValue="custom-resource-nodejs.zip" \
    ParameterKey="LambdaHandler",ParameterValue="resource.handler" \
    ParameterKey="LambdaRuntime",ParameterValue="nodejs14.x"

## Get the Custom Resource Lambda Arn
aws cloudformation describe-stacks --stack-name custom-resource --query Stacks[*].Outputs[?OutputKey=='CustomResourceLambdaArn'].OutputValue --output text
Output: arn:aws:lambda:us-west-1:1234567890:function:custom-resource-CustomResourceLambda-D6LJZxuXaY5w

## Launch a stack that uses the custom resource. Set the CustomResourceLambdaArn parameter value with output from previous step.
aws cloudformation create-stack \
  --stack-name custom-resource-example \
  --template-body file://cfn-stacks/custom-resource-example.yaml \
  --disable-rollback \
  --parameters \
    ParameterKey="CustomResourceLambdaArn",ParameterValue=""

## Cleanup, delete both stacks
aws cloudformation delete-stack --stack-name custom-resource-example
aws cloudformation delete-stack --stack-name custom-resource
