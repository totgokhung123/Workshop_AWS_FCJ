---
title : "Kiểm tra Video Upload Process"
date :  "`r Sys.Date()`" 
weight : 2 
chapter : false
pre : " <b> 3.2. </b> "
---

![SSMPublicinstance](/images/arc-02.png)


1. Tạo enviroment cho lambda lấy S3. Truy cập vào [giao diện quản trị của dịch vụ ](https://ap-southeast-1.console.aws.amazon.com/lambda/home?region=ap-southeast-1#/).
  + Click chọn **Create function**
  + Click chọn **Author from scratch**
  + Đặt tên ``` Presigner ```
  + Chọn **Python 3.9**.
  + Mục **Architecture** chọn "x86_64"

![Connect](/images/3.connect/001-connect-2.png)

  + Chọn **Create function**

2. Thêm KEY-VALUE cho Lambda
  + Trong ``` Presigner ``` vừa mới tạo chọn mục **Configuration**
  + Chọn **Environment variables** 
  + Chọn **Edit**
  
![alt text](/images/3.connect/002-connect-2.png)

   + Click **Add environment variable**
   + Nhập KEY là ``` BUCKET_NAME ```, VALUE là tên Bucket S3 bạn đã tạo ở bước trước.
![alt text](/images/3.connect/003-connect-2.png)

   + Click **Save**
3. Thêm code Lamda

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

    # Lấy filename an toàn
    filename = None
    try:
        # HTTP API (apigatewayv2) truyền query params vào event['queryStringParameters']
        q = event.get('queryStringParameters') if isinstance(event, dict) else None
        if q and isinstance(q, dict):
            filename = q.get('filename')
    except Exception as e:
        logger.warning("Cannot read queryStringParameters: %s", e)

    # Nếu không có filename, tạo tên mặc định
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
  + Click **Deploy** hoặc tổ hợp phím "Ctrl+shift+U"
  + Chọn mục **Test**
  + Paste code test:

```JSON
{
  "queryStringParameters": {"filename": "test123.mp4"}
}
```
![alt text](/images/3.connect/004-connect-2.png)

   + Click **Test** và xem kết quả trả về có chứa đường dẫn upload video.
   + Kiểm tra **Executing function**.
  
![alt text](/images/3.connect/005-connect-2.png)

{{% notice note %}}
Chú ý phần output ```"body": "{\"upload_url\"....``` đây là mẫu API để bạn có thể upload file mp4 từ local qua API vào S3 ```totgo1``` 
{{% /notice %}}

4. Tạo API Gateway để kết nối với Lambda
  + Truy cập vào [giao diện API Gateway](https://ap-southeast-1.console.aws.amazon.com/apigateway/home?region=ap-southeast-1#/apis)
  + Click **Create API**
  + Chọn **HTTP API**
  + Click **Build**
  + Đặt API name ```/generate-upload-url```

![alt text](/images/3.connect/006-connect-2.png)

  + Chọn **Review and create**
  + Tạo phương thức GET, POST link tới Lambda ```Presigner```, Chọn API vừa mới tạo ```/``` -> Vào **Routes** -> Click **Create** -> Chọn **GET** và nhập ```/generate-upload-url```
![alt text](/images/3.connect/007-connect-2.png)

  + Click **Attach integration** 
  + Click **Create and attach an integration**
  + Mục **Integration target** chọn **Lambda function**
  + Ở **Lambda function** chọn ```Presigner``` bạn đã tạo trước đó.
  + Chọn **Create**

![alt text](/images/3.connect/008-connect-2.png)

{{% notice note %}}
Phương thức GET và POST làm tương tự nhau
{{% /notice %}}

5. Tạo policy cho Lambda cho phép truy cập với API Gateway, S3, SQS

   + Truy cập vào [giao diện quản trị của dịch vụ IAM](https://ap-southeast-1.console.aws.amazon.com/iamv2/home?region=ap-southeast-1#/)
   + Chọn **Role**
   + Tìm  ```Presigner-role-vq5i02ad``` là Role Default lúc tạo Lambda ```Presigner```

![alt text](/images/3.connect/009-connect-2.png)
   + Trong **Permissions** chọn **Create inline policy**
   + Chọn **JSON** và paste code sau:
   
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

6. Thêm trigger API Gateway cho Lambda

![alt text](/images/3.connect/011-connect-2.png)

   + Trong **Configuration** của Lambda ```Presigner``` chọn **Triggers**
   + Click **Add trigger**
   + Chọn **API Gateway**
   + Chọn API vừa tạo ```/generate-upload-url```
   + Chọn phương thức là **GET** và **POST**
   + Click **Add**
7. Test chạy gửi file mp4 từ local qua API
  + Trong máy tính mở **IDE Code** lên thực thi scripts sau:
  ```python
   import requests
   r = requests.get("https://<API_ID>.execute-api.ap-southeast-1.amazonaws.com/generate-upload-url?filename=test1234.mp4")
   upload_url = r.json()['upload_url']
   print(upload_url)

   with open(r"E:\AWS_FCJ\Hugo\video\test1234.mp4","rb") as f:
      r2 = requests.put(upload_url, data=f, headers={"Content-Type":"video/mp4"})
   print(r2.status_code, r2.text)
  ```
   + Thay thế ```<API_ID>``` bằng ID của API Gateway bạn đã tạo ```/generate-upload-url``` (Nếu báo 200 thì thành công)
   + Kiểm tra S3 và SQS sau khi mp4 được upload lên:
   ```cmd
   aws s3api head-object --bucket <bucket_id> --key uploads/test1234.mp4 --region ap-southeast-1
   aws sqs receive-message --queue-url "https://sqs.ap-southeast-1.amazonaws.com/<Acount_id>/videosense-upload-queue" --max-number-of-messages 1 --region ap-southeast-1
   ```
   + Thay thế ```<bucket_id>``` và ```<Acount_id>``` bằng ID của Bucket S3 và Account AWS của bạn.

![alt text](/images/3.connect/012-connect-2.png)

![alt text](/images/3.connect/013-connect-2.png)
