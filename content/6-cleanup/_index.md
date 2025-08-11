+++
title = "Resource Cleanup"
date = 2025
weight = 6
chapter = false
pre = "<b>6. </b>"
+++

We will follow these steps to delete the resources created during this workshop.

#### Delete IAM Roles

1. Go to the [IAM Management Console](https://console.aws.amazon.com/iamv2/home#/home)
  + Click **Roles**.
  + In the search box, find the roles used for Lambda.
  ![alt text](/images/6.clean/image-3.png)
  + Click **Delete**, then enter the role name and click **Delete** to remove the role.
  
![alt text](/images/6.clean/image-4.png)

![alt text](/images/6.clean/image-5.png)

#### Delete S3 Buckets

1. Go to the [S3 Management Console](https://s3.console.aws.amazon.com/s3/home)
  + Click on the S3 buckets you created for the workshop (e.g., totgo1, scenedetect-layer).
  ![alt text](/images/6.clean/image.png)
  + Click **Empty**.
  + Enter **permanently delete**, then click **Empty** to delete all objects in the bucket.
  + Click **Exit**.

2. After deleting all objects in the bucket, click **Delete**.

![alt text](/images/6.clean/image-1.png)

3. Enter the name of the S3 bucket, then click **Delete bucket** to remove the S3 bucket.

![alt text](/images/6.clean/image-2.png)

#### Delete SQS
1. Go to the [SQS Management Console](https://console.aws.amazon.com/sqs/v2/home)
   + Click **Queues**.
   + Select the queue you created for the workshop.
   ![alt text](/images/6.clean/image-6.png)

   + Click **Delete** to remove the queue.

![alt text](/images/6.clean/image-7.png)
#### Delete Lambda Functions
1. Go to the [Lambda Management Console](https://console.aws.amazon.com/lambda/home)
   + Click **Functions**.
   + Select the Lambda functions you created for the workshop.
   ![alt text](/images/6.clean/image-8.png)

   + Click **Delete** to remove the Lambda function.

![alt text](/images/6.clean/image-9.png)
#### Delete API Gateway
1. Go to the [API Gateway Management Console](https://console.aws.amazon.com/apigateway/home)
   + Click **APIs**.
   + Select the API you created for the workshop.
   ![alt text](/images/6.clean/image-10.png)

   + Click **Actions** and select **Delete API** to remove the API.

![alt text](/images/6.clean/image-11.png)
#### Delete DynamoDB
1. Go to the [DynamoDB Management Console](https://console.aws.amazon.com/dynamodbv2/home)
   + Click **Tables**.
   + Select the table you created for the workshop.
   ![alt text](/images/6.clean/image-12.png)
   + Click **Delete** to remove the table.
 
![alt text](/images/6.clean/image-13.png) 
#### Delete ECR
1. Go to the [ECR Management Console](https://console.aws.amazon.com/ecr/repositories)
   + Click **Repositories**.
   + Select the repository you created for the workshop.
   ![alt text](/images/6.clean/image-14.png)
   + Click **Delete** to remove the table.

![alt text](/images/6.clean/image-15.png)

