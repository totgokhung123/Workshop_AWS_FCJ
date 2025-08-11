---
title : "Video & Processing"
date :  "`r Sys.Date()`" 
weight : 4 
chapter : false
pre : " <b> 4. </b> "
---

Sau khi video được upload lên hệ thống, giai đoạn tiếp theo là xử lý video để trích xuất các frame quan trọng phục vụ cho việc phân tích và tìm kiếm. Quá trình này sử dụng AWS Lambda để tự động phát hiện cảnh (scene detection), trích xuất frame định kỳ, nén ảnh (WebP) và lưu trữ vào S3. Metadata của video và các frame sẽ được lưu vào DynamoDB để phục vụ cho các bước tìm kiếm sau này.

![alt text](/images/4.s3/arc-4.png)

Trong phần này, bạn sẽ thực hành:
- Tạo S3 bucket để lưu trữ frame.
- Cấu hình Lambda để tự động trích xuất frame từ video.
- Lưu metadata vào DynamoDB.

### Nội dung:

  - [Quy trình xử lý video thành frame](./4.1-Create-S3-Lambda-DynamoDB/)
  - [Testing trích xuất video](./4.2-Testing-processing/)

