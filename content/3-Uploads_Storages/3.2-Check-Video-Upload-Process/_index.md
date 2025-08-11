---
title : "Check Video Upload Process"
date :  "`r Sys.Date()`" 
weight : 2 
chapter : false
pre : " <b> 3.2. </b> "
---

![SSMPublicinstance](/images/arc-02.png)

1. Create an environment for the Lambda to access S3. Go to the [Lambda management console](https://ap-southeast-1.console.aws.amazon.com/lambda/home?region=ap-southeast-1#/).
  + Click **Create function**
  + Select **Author from scratch**
  + Name it ``` Presigner ```
  + Choose **Python 3.9**.
  + For **Architecture**, select "x86_64"

![Connect](/images/3.connect/001-connect-2.png)

  + Click **Create function**

2. Add KEY-VALUE for Lambda
  + In the newly created ``` Presigner ```, select **Configuration**
  + Choose **Environment variables**
  + Click **Edit**
  
![alt text](/images/3.connect/002-connect-2.png)

   + Click **Add environment variable**
   + Enter KEY as ``` BUCKET_NAME ```, VALUE as the name of the S3 Bucket you created earlier.
![alt text](/images/3.connect/003-connect-2.png)

   + Click **Save**
3. Add Lambda code

```python
# lambda_presigner_safe.py
import json
import os
import time
import logging
import boto3

logger = logging.getLogger()
logger.setLevel(logging.INFO)

s3 = boto3.client('s3')

def lambda_handler(event, context):
    logger.info("Event received: %s", event)

    bucket = os.environ.get('BUCKET_NAME')
    if not bucket or not isinstance(bucket, str) or bucket.strip() == "":
        logger.error("Missing or invalid BUCKET_NAME environment variable.")
        return {
            "statusCode": 500,
            "body": json.dumps({"error": "Missing BUCKET_NAME env var", "detail": "Set environment variable BUCKET_NAME to your S3 bucket name."})
        }

    # Get safe filename
    filename = None
    try:
        # HTTP API (apigatewayv2) passes query params in event['queryStringParameters']
        q = event.get('queryStringParameters') if isinstance(event, dict) else None
        if q and isinstance(q, dict):
            filename = q.get('filename')
    except Exception as e:
        logger.warning("Cannot read queryStringParameters: %s", e)

    # If no filename, create a default name
    if not filename or not isinstance(filename, str):
        filename = f"uploads/{int(time.time())}.mp4"
    else:
        filename = filename if filename.startswith('uploads/') else f"uploads/{filename}"

    logger.info("Using bucket=%s key=%s", bucket, filename)

    try:
        url = s3.generate_presigned_url(
            ClientMethod='put_object',
            Params={'Bucket': bucket, 'Key': filename, 'ContentType': 'video/mp4'},
            ExpiresIn=3600
        )
    except Exception as e:
        logger.exception("Failed to create presigned URL")
        return {
            "statusCode": 500,
            "body": json.dumps({"error": "Failed to create presigned URL", "detail": str(e)})
        }

    return {
        "statusCode": 200,
        "headers": {"Content-Type": "application/json"},
        "body": json.dumps({"upload_url": url, "key": filename})
    }

```
  + Click **Deploy** or use "Ctrl+Shift+U"
  + Select **Test**
  + Paste the test code:

```JSON
{
  "queryStringParameters": {"filename": "test123.mp4"}
}
```
![alt text](/images/3.connect/004-connect-2.png)

   + Click **Test** and check if the result contains the video upload URL.
   + Check **Executing function**.
  
![alt text](/images/3.connect/005-connect-2.png)

{{% notice note %}}
Note: The output part ```"body": "{\"upload_url\"....``` is the API sample for you to upload an mp4 file from local to S3 ```totgo1``` via API.
{{% /notice %}}

4. Create API Gateway to connect with Lambda
  + Go to the [API Gateway console](https://ap-southeast-1.console.aws.amazon.com/apigateway/home?region=ap-southeast-1#/apis)
  + Click **Create API**
  + Select **HTTP API**
  + Click **Build**
  + Name the API ```/generate-upload-url```

![alt text](/images/3.connect/006-connect-2.png)

  + Click **Review and create**
  + Create GET, POST methods linked to Lambda ```Presigner```. Select the API you just created ```/``` -> Go to **Routes** -> Click **Create** -> Select **GET** and enter ```/generate-upload-url```
![alt text](/images/3.connect/007-connect-2.png)

  + Click **Attach integration** 
  + Click **Create and attach an integration**
  + For **Integration target**, select **Lambda function**
  + In **Lambda function**, select the ```Presigner``` you created earlier.
  + Click **Create**

![alt text](/images/3.connect/008-connect-2.png)

{{% notice note %}}
GET and POST methods are configured similarly.
{{% /notice %}}

5. Create a policy for Lambda to allow access to API Gateway, S3, SQS

   + Go to the [IAM management console](https://ap-southeast-1.console.aws.amazon.com/iamv2/home?region=ap-southeast-1#/)
   + Select **Role**
   + Find  ```Presigner-role-vq5i02ad``` which is the default Role when creating Lambda ```Presigner```

![alt text](/images/3.connect/009-connect-2.png)
   + In **Permissions**, select **Create inline policy**
   + Choose **JSON** and paste the following code:
   
```JSON
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowPutUploadPrefix",
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:PutObjectAcl"
            ],
            "Resource": "arn:aws:s3:::totgo1/uploads/*"
        }
    ]
}
```

![alt text](/images/3.connect/010-connect-2.png)

6. Add API Gateway trigger for Lambda

![alt text](/images/3.connect/011-connect-2.png)

   + In **Configuration** of Lambda ```Presigner```, select **Triggers**
   + Click **Add trigger**
   + Select **API Gateway**
   + Choose the API you just created ```/generate-upload-url```
   + Select methods **GET** and **POST**
   + Click **Add**
7. Test uploading mp4 file from local via API
  + On your computer, open your **IDE Code** and run the following script:
  ```python
   import requests
   r = requests.get("https://<API_ID>.execute-api.ap-southeast-1.amazonaws.com/generate-upload-url?filename=test1234.mp4")
   upload_url = r.json()['upload_url']
   print(upload_url)

   with open(r"E:\AWS_FCJ\Hugo\video\test1234.mp4","rb") as f:
      r2 = requests.put(upload_url, data=f, headers={"Content-Type":"video/mp4"})
   print(r2.status_code, r2.text)
  ```
   + Replace ```<API_ID>``` with the ID of the API Gateway you created for ```/generate-upload-url``` (If you get 200, it's successful)
   + Check S3 and SQS after uploading the mp4:
   ```cmd
   aws s3api head-object --bucket <bucket_id> --key uploads/test1234.mp4 --region ap-southeast-1
   aws sqs receive-message --queue-url "https://sqs.ap-southeast-1.amazonaws.com/<Acount_id>/videosense-upload-queue" --max-number-of-messages 1 --region ap-southeast-1
   ```
   + Replace ```<bucket_id>``` and ```<Acount_id>``` with your S3 Bucket ID and AWS Account ID.

![alt text](/images/3.connect/012-connect-2.png)

![alt text](/images/3.connect/013-connect-2.png)