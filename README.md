
# Overview
This is an example code that should be modified per use case.  

I have a lambda for ServiceNow App Alert. Basically, this will send the ServiceNow notification through email. This lambda consist of 2 packages. One for generating the notification. Another for sending the email by 
 attaching the notification as email body.

This lambda can be triggered from another lambda in the target account. From this invoker lambda, we can pass the parameters, so that the ServiceNow lambda will generate the notification from the parameters and send
email notifications.

## 1. Infrastructure setup
We have 4 cloudformation stack for this lambda setup. We will see all the stacks one by one. 
 1. ServiceNow Alerting lambda should be deployed in Account A. For this, we are going to use the file from S3 Bucket. So this needs 2 cloudformation templates.
	- S3Callee CloudFormation (This will create the S3 bucket required to upload the lambda files)
	- Callee CloudFormation (In this, we will create lambda by referring the file that we uploaded in the S3Calle template)

 2. ServiceNow-Invoker Lambda is to invoke our lambda in Account A, and this Invoker lambda should be deployed in Account B. As like ServiceNow Lambda, for Invoker lambda, we are using 2 stacks, one for S3 Bucket and another for Lambda. 
	- S3Caller CloudFormation
	- Caller CloudFormation 

 - **S3Callee  and S3Caller Cloudformation stack**

	The main purpose of this 2 cloudformation stack is to create the S3 bucket, that we can use to upload the lambda zip file. Because in lambda stack, we are not hard coding the lambda code directly into the stack. We are using S3 method as a source. So there must be one zip file with the lambda code present in S3 bucket. And this 2 lambda will help us to create those buckets in account A and B respectively.

	*Services that are provisioned under this stack*
	1. KMS key for Bucket encryption
	2. Alias Name for the KMS key for ease of use.
	3. S3 Bucket with Encryption enabled.
	4. Bucket Policy for denying UnEncrypted uploads, allowing secure transport, prevent further policy edit and object permission for the user/role.

 - **Callee CloudFormation**

	As we discussed before, this stack is to create Lambda and its permissions and dependencies. This Lambda is to sending ServiceNow email notifications.

	*Services that are provisioned under this stack*
	1. Lambda (Service Now Alerting Script)
	2. IAM Role (For lambda)
	3. Policies (For the lambda's Role, this will provide permissions to CloudWatch Logs, Invoke Lambda, Ec2-VPC for accessing the Network Interface and Private IP Addresses.)
	4. Cross Account IAM Role (To be able to assume by Target lambda, inorder to trigger Service Now Alerting Lambda.
	5. Cross Account Policy (This will give permission to the cross account role to invoke the lambda)
	6. Lambda Invoke Permission
	7. LogGroup (To save lambda logs)
	8. Security Group (To enable port 2525 to gain the access from Lambda to VPC)

 - **Caller CloudFormation**

	This is to create the Caller lambda function and its main task is to trigger/invoke the ServiceNow alerting lambda.  This lambda will assume the Cross Account Role in Account A, and then it will trigger the ServiceNow alerting lambda.

	*Services that are provisioned under this stack*
	1. Lambda (Service Now Invoker Script)
	2. IAM Role (For lambda)
	3. Policies (For the lambda's Role, this will provide permissions to CloudWatch Logs, Invoke Lambda)
	4. Lambda Invoke Permission.
	5. LogGroup (To save lambda logs)



















# Architecture Components

Underlying architecture consists of following major components:

1- **Caller Lambda Function** residing in *Account A*
2- **Caller S3 bucket** containing python code for Caller Lambda function residing in *Account A*
3- **Callee Lambda Function** residing in *Account B*
4- **Callee S3 bucket** containing python code for Callee Lambda function residing in *Account B*

The purpose of Caller Lambda function is to invoke Callee Lambda function through IAM roles and trust relationship. 

Above mentioned architecture is being deployed using cloud formation templates with following details:

Caller S3 bucket template for Account A (s3caller.yaml)
-------------------------------------------------------------------------------------------
Following key functions are being performed in s3caller cloud-formation template

 - [KMS Key creation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-kms-key.html) with permissions to manage and access keys
 - [S3 bucket creation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-s3-bucket.html) encrypted with referenced KMS key

Caller Lambda template for Account A (callercloudformation.yaml)
-------------------------------------------------------------------------------------------
Following key functions are being performed in callercloudformation cloud-formation template

 - [IAM role creation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-iam-role.html) with permissions for cloud watch group logs, VPC   
   execution, Function execution and Assume role.  
 - [Cloud Watch Log group creation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-logs-loggroup.html) for lambda function's logs ingestion
 - [Security group creation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-security-group.html) with outbound rule
 - [Lambda function creation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-lambda-function.html) with referenced
   VPC

Callee S3 bucket template for Account A (s3callee.yaml)
-------------------------------------------------------------------------------------------
Following key functions are being performed in s3callee cloud-formation template

 - [KMS Key creation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-kms-key.html) with permissions to manage and access keys
 - [S3 bucket creation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-s3-bucket.html) encrypted with referenced KMS key

Callee Lambda template for Account B(calleecloudformation.yaml)
-------------------------------------------------------------------------------------------
Following key functions are being performed in calleecloudformation cloud-formation template

 - [IAM role creation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-iam-role.html) with permissions for cloud watch group logs, VPC   
   execution, Function execution and Assume role
 - [Cloud Watch Log group creation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-logs-loggroup.html) for lambda function's logs ingestion
 - [Security group creation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-security-group.html) with outbound rule
 - [Lambda function creation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-lambda-function.html) with referenced VPC
