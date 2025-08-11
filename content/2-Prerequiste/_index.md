---
title : "Preparation "
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

{{% notice info %}}
You need to pay attention to the steps to link Bucket S3 with Layer, Lambda. 
{{% /notice %}}

To learn how to create IAM, S3, Lambda, SQS, DynamoDB you can refer to the lab:
  - [Amazon IAM](https://000002.awsstudygroup.com/)
  - [Amazon S3](https://000057.awsstudygroup.com/vi)
  - [Amazon Lambda](https://000022.awsstudygroup.com/vi/)
  - [Amazon SQS](https://000077.awsstudygroup.com/vi/)
  - [Amazon DynamoDB](https://000060.awsstudygroup.com/vi/)

To deploy the VideoSense system on AWS, you need to prepare basic resources such as S3 Bucket, Lambda Function, SQS Queue, DynamoDB Table and configure appropriate access rights (IAM Role) for each service. Granting the right rights will ensure that the components in the pipeline can communicate and operate automatically and securely.

In this preparation section, you will gain more knowledge to implement:
- Create the necessary IAM Roles for Lambda, S3, SQS, DynamoDB.
- Configure AWS services to ensure the pipeline operates seamlessly and securely.