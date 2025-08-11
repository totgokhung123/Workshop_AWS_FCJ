---
title : "Test Video Processing Stage"
date :  "`r Sys.Date()`" 
weight : 2 
chapter : false
pre : " <b> 4.2 </b> "
---

## Step 1: Upload Video to S3
There are two methods to upload videos to S3:

### Method 1: Upload Directly via S3 Console
1. Open the S3 console from the AWS Management Console.
2. Select the `totgo1` bucket.
3. Go to the `uploads/` folder.
4. Click **Upload** and choose any mp4 file from your computer to upload.
   
![alt text](/images/4.s3/image_42_0.png)
![alt text](/images/4.s3/image_42_1.png)

### Method 2: Use API Gateway
1. Use the following Python code to get the upload URL from API Gateway:
```python
import requests
r = requests.get("https://lx7tj1p0jg.execute-api.ap-southeast-1.amazonaws.com/generate-upload-url?filename=test1234.mp4")
upload_url = r.json()['upload_url']
print(upload_url)
```
2. Then, upload the video to S3 using the received URL:
```python
with open(r"E:\AWS_FCJ\Hugo\video\test1234.mp4", "rb") as f:
    r2 = requests.put(upload_url, data=f, headers={"Content-Type": "video/mp4"})
print(r2.status_code, r2.text)
```

## Step 2: Check SQS Notification After Upload
1. Open the SQS console from the AWS Management Console.
2. Select the `videosense-upload-queue` queue.
3. Go to **Monitoring** to check the metrics and received notifications.

![alt text](/images/4.s3/image_42_2.png)
![alt text](/images/4.s3/image_42_3.png)

## Step 3: Check Lambda Function
1. Open the Lambda console from the AWS Management Console.
2. Select the `videosense-frame-extractor` function.
3. Click **Test**.
4. In the **Event name** section, copy the following JSON code into the Event JSON box, then click **Save** and **Test**:
```json
{
  "Records": [
    {
      "body": "{\"Records\":[{\"s3\":{\"bucket\":{\"name\":\"totgo1\"},\"object\":{\"key\":\"uploads/test1234_2.mp4\"}}}]}"
    }
  ]
}
```

![alt text](/images/4.s3/image_42_4.png)

![alt text](/images/4.s3/image_42_5.png)
## Step 4: Check Results After Execution
1. After execution, check the DynamoDB table:
   - Open the DynamoDB console.
   - Select the `videos` table.
   - Go to **Monitor** to check the metrics in **CloudWatch metrics**.

![alt text](/images/4.s3/image_42_6.png)

2. Check the `totgo1` S3 bucket:
   - Open the `totgo1` bucket.
   - Go to the `frames/` folder.
   - Check all the extracted frames.

![alt text](/images/4.s3/image_42_7.png)
## Step 5: Verify Results
- Ensure that all frames have been successfully extracted and stored in the `frames/` folder on S3.
- Check the information in DynamoDB to confirm that the video metadata has been