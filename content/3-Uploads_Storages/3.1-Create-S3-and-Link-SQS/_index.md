---
title : "Create S3 and Link SQS"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b> 3.1. </b> "
---
![SSMPublicinstance](/images/arc-02.png)

1. Go to the [SQS service management console](https://ap-southeast-1.console.aws.amazon.com/sqs/v3/home?region=ap-southeast-1#/queues).
  + Click **Create queue**
  + Name it ``` videosense-upload-queue ```.
  + Optionally set **Visibility timeout** to 1 minute.

![Connect](/images/3.connect/001-connect-1.png)

  + Leave other options as default.
  + Click **Create queue**.
2. Add a policy to SQS.
  + In **Queues**, select the **videosense-upload-queue** you just created.
  + In the **Access policy** section, click **Edit**.
  + Add the following policy:
  
```JSON
{
  "Version": "2012-10-17",
  "Id": "__default_policy_ID",
  "Statement": [
    {
      "Sid": "__owner_statement",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::<id-account>:root"
      },
      "Action": "SQS:*",
      "Resource": "arn:aws:sqs:ap-southeast-1:<id-account>:videosense-upload-queue"
    },
    {
      "Sid": "AllowS3SendMessage",
      "Effect": "Allow",
      "Principal": {
        "Service": "s3.amazonaws.com"
      },
      "Action": "SQS:SendMessage",
      "Resource": "arn:aws:sqs:ap-southeast-1:<id-account>:videosense-upload-queue",
      "Condition": {
        "ArnEquals": {
          "aws:SourceArn": "arn:aws:s3:::<bucket-name>"
        }
      }
    }
  ]
}
```
  + Click **Save**
{{% notice note %}}
Note: "<id-account>" is your AWS Account ID, "<bucket-name>" is the name of the S3 Bucket you will create in the next step.
{{% /notice %}}
1. Go to the [S3 creation console](https://ap-southeast-1.console.aws.amazon.com/s3/home?region=ap-southeast-1)

  + Click **Create bucket**.
  + Name the S3 Bucket ``` totgo1 ```.

![Connect](/images/3.connect/002-connect-1.png)

  + Leave other settings as default, then click **Save**.
  
4. Then go to the newly created S3 bucket **totgo1**
  + Select **Properties**
  + Scroll down to **Event notifications**, select **Create event notification**  

![Connect](/images/3.connect/003-connect-1.png)

  + Name it ``` NewVideoUploaded ```.
  + Select **All object create events**.
  + Prefix: ```uploads/```
![Connect](/images/3.connect/004-connect-1.png)

  + For **Send to**, select **SQS Queue**.
  + Choose **videosense-upload-queue**.
![Connect](/images/3.connect/005-connect-1.png)

  + Click **Save changes**. 

{{% notice note %}}
You have completed the step of creating the storage source for videos and frames, and SQS to record the upload traffic.
{{% /notice %}}

The next step is to test whether the video upload