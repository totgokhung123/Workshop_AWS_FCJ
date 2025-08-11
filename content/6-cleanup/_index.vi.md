+++
title = "Dọn dẹp tài nguyên "
date = 2025
weight = 6
chapter = false
pre = "<b>6. </b>"
+++

Chúng ta sẽ tiến hành các bước sau để xóa các tài nguyên chúng ta đã tạo trong bài thực hành này.

#### Xóa IAM Role 

1. Truy cập [giao diện quản trị dịch vụ IAM](https://console.aws.amazon.com/iamv2/home#/home)
  + Click **Roles**.
  + Tại ô tìm kiếm , tìm các role dùng cho Lambda.
  ![alt text](/images/6.clean/image-3.png)
  + Chon **Delete** sau đó điền tên role và click **Delete** để xóa role.
  
![alt text](/images/6.clean/image-4.png)

![alt text](/images/6.clean/image-5.png)

#### Xóa S3 bucket

1. Truy cập [giao diện quản trị dịch vụ S3](https://s3.console.aws.amazon.com/s3/home)
  + Click chọn S3 bucket chúng ta đã tạo cho bài thực hành. (Ví dụ : totgo1, scenedetect-layer)
  ![alt text](/images/6.clean/image.png)
  + Click **Empty**.
  + Điền **permanently delete**, sau đó click **Empty** để tiến hành xóa object trong bucket.
  + Click **Exit**.

2. Sau khi xóa hết object trong bucket, click **Delete**

![alt text](/images/6.clean/image-1.png)

3. Điền tên S3 bucket, sau đó click **Delete bucket** để tiến hành xóa S3 bucket.

![alt text](/images/6.clean/image-2.png)

#### Xóa SQS 
1. Truy cập [giao diện quản trị dịch vụ SQS](https://console.aws.amazon.com/sqs/v2/home)
   + Click **Queues**.
   + Chọn hàng đợi chúng ta đã tạo cho bài thực hành.
   ![alt text](/images/6.clean/image-6.png)

   + Click **Delete** để xóa hàng đợi.

![alt text](/images/6.clean/image-7.png)
#### Xóa Lambda 
1. Truy cập [giao diện quản trị dịch vụ Lambda](https://console.aws.amazon.com/lambda/home)
   + Click **Functions**.
   + Chọn các hàm Lambda chúng ta đã tạo cho bài thực hành.
   ![alt text](/images/6.clean/image-8.png)

   + Click **Delete** để xóa hàm Lambda.

![alt text](/images/6.clean/image-9.png)
#### Xóa API Gateway 
1. Truy cập [giao diện quản trị dịch vụ API Gateway](https://console.aws.amazon.com/apigateway/home)
   + Click **APIs**.
   + Chọn API chúng ta đã tạo cho bài thực hành.
   ![alt text](/images/6.clean/image-10.png)

   + Click **Actions** và chọn **Delete API** để xóa API.

![alt text](/images/6.clean/image-11.png)
#### Xóa DynamoDB 
1. Truy cập [giao diện quản trị dịch vụ DynamoDB](https://console.aws.amazon.com/dynamodbv2/home)
   + Click **Tables**.
   + Chọn bảng chúng ta đã tạo cho bài thực hành.
   ![alt text](/images/6.clean/image-12.png)
   + Click **Delete** để xóa bảng.
 
![alt text](/images/6.clean/image-13.png) 
#### Xóa ECR
1. Truy cập [giao diện quản trị dịch vụ ECR](https://console.aws.amazon.com/ecr/repositories)
   + Click **Repositories**.
   + Chọn kho chứa chúng ta đã tạo cho bài thực hành.
   ![alt text](/images/6.clean/image-14.png)
   + Click **Delete** để xóa kho chứa.

![alt text](/images/6.clean/image-15.png)