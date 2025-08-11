---
title : "tạo S3 và liên kết SQS"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b> 3.1. </b> "
---
![SSMPublicinstance](/images/arc-02.png)

1. Truy cập vào [giao diện quản trị của dịch vụ SQS](https://ap-southeast-1.console.aws.amazon.com/sqs/v3/home?region=ap-southeast-1#/queues).
  + Click chọn **Create queue**
  + Đặt tên ``` videosense-upload-queue ```.
  + Tùy chọn **Visibility timeout** đặt 1 phút (Option).

![Connect](/images/3.connect/001-connect-1.png)


  + Các Option khác để giữ nguyên.
  + Click **Create queue**.
2. Thêm policy cho SQS.
  + Trong **Queues** chọn **videosense-upload-queue** vừa mới tạo
  + Mục **Access policy** Click **Edit**
  + Thêm policy sau:

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
Chú ý "<id-account>" là ID của Account AWS của bạn, "<bucket-name>" là tên Bucket S3 bạn tạo trong bước tiếp theo
{{% /notice %}}
1. Truy cập vào [giao diện tạo S3](https://ap-southeast-1.console.aws.amazon.com/s3/home?region=ap-southeast-1)

  + Click **Create bucket**.
  + Đặt tên Bucket S3 ``` totgo1 ```.

![Connect](/images/3.connect/002-connect-1.png)

  + Thông tin khác giữu nguyên, Click **Save**.
  
4. Sau đó vào S3 **totgo1** vừa tạo
  + Chọn **Properties**
  + Lướt xuống mục **Event notifications** , chọn **Create event notification**  

![Connect](/images/3.connect/003-connect-1.png)

  + Đặt tên ``` NewVideoUploaded ```.
  + Chọn **All object create events**.
  + Prefix: ```uploads/```
![Connect](/images/3.connect/004-connect-1.png)

  + Chọn **Send to** là **SQS Queue**.
  + Chọn **videosense-upload-queue**.
![Connect](/images/3.connect/005-connect-1.png)

  + Click **Save changes**. 


{{% notice note %}}
Bạn đã hoàn thành xong bước tạo nguồ lưu trữ video cũng như frame sau này, SQS để note lại lưu lượng được lưu.
 {{% /notice %}}

Bước tiếp theo sẽ testing xem quá trình Upload video thành công hay không.

