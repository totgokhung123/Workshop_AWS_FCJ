---
title : "Các bước chuẩn bị"
date :  "`r Sys.Date()`" 
weight : 2 
chapter : false
pre : " <b> 2. </b> "
---

{{% notice info %}}
Bạn cần chú ý các bước thao tác liên kết Bucket S3 với Layer , Lambda. 
{{% /notice %}}

Để tìm hiểu cách tạo các IAM, S3, Lambda, SQS, DynamoDB các bạn có thể tham khảo bài lab :
  - [Amazon IAM](https://000002.awsstudygroup.com/)
  - [Amazon S3](https://000057.awsstudygroup.com/vi)
  - [Amazon Lambda](https://000022.awsstudygroup.com/vi/)
  - [Amazon SQS](https://000077.awsstudygroup.com/vi/)
  - [Amazon DynamoDB](https://000060.awsstudygroup.com/vi/)

Để triển khai hệ thống VideoSense trên AWS, bạn cần chuẩn bị các tài nguyên cơ bản như S3 Bucket, Lambda Function, SQS Queue, DynamoDB Table và cấu hình các quyền truy cập (IAM Role) phù hợp cho từng dịch vụ. Việc cấp quyền đúng sẽ đảm bảo các thành phần trong pipeline có thể giao tiếp và hoạt động tự động hóa, bảo mật.

Trong phần chuẩn bị này, bạn sẽ tiếp thêm kiến thức thực hiện:
- Tạo các IAM Role cần thiết cho Lambda, S3, SQS, DynamoDB.
- Cấu hình các dịch vụ AWS để đảm bảo pipeline hoạt động liền mạch và an toàn.



  
