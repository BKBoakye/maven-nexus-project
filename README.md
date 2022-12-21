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
