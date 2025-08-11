---
title : "Kiểm Tra Giai Đoạn Xử Lý Video"
date :  "`r Sys.Date()`" 
weight : 2 
chapter : false
pre : " <b> 4.2 </b> "
---

## Bước 1: Tải Video Lên S3
Có hai phương thức để tải video lên S3:

### Phương Thức 1: Tải Trực Tiếp Qua Giao Diện S3
1. Mở bảng điều khiển S3 từ AWS Management Console.
2. Chọn bucket `totgo1`.
3. Vào thư mục `uploads/`.
4. Nhấn **Upload** và chọn tệp mp4 bất kỳ từ máy tính của bạn để tải lên.
   
![alt text](/images/4.s3/image_42_0.png)
![alt text](/images/4.s3/image_42_1.png)

### Phương Thức 2: Sử Dụng API Gateway
1. Sử dụng đoạn mã Python sau để lấy URL tải lên từ API Gateway:
```python
import requests
r = requests.get("https://lx7tj1p0jg.execute-api.ap-southeast-1.amazonaws.com/generate-upload-url?filename=test1234.mp4")
upload_url = r.json()['upload_url']
print(upload_url)
```
2. Sau đó, tải video lên S3 bằng URL đã nhận:
```python
with open(r"E:\AWS_FCJ\Hugo\video\test1234.mp4", "rb") as f:
    r2 = requests.put(upload_url, data=f, headers={"Content-Type": "video/mp4"})
print(r2.status_code, r2.text)
```

## Bước 2: Kiểm Tra Thông Báo SQS Sau Khi Tải Lên
1. Mở bảng điều khiển SQS từ AWS Management Console.
2. Chọn hàng đợi `videosense-upload-queue`.
3. Chọn **Monitoring** để kiểm tra các thông số và thông báo đã nhận.

![alt text](/images/4.s3/image_42_2.png)
![alt text](/images/4.s3/image_42_3.png)

## Bước 3: Kiểm Tra Hàm Lambda
1. Mở bảng điều khiển Lambda từ AWS Management Console.
2. Chọn hàm `videosense-frame-extractor`.
3. Nhấn **Test**.
4. Trong phần **Event name**, sao chép mã JSON sau vào ô Event JSON, sau đó nhấn **Save** và **Test**:
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
## Bước 4: Kiểm Tra Kết Quả Sau Khi Chạy
1. Sau khi chạy xong, kiểm tra bảng DynamoDB:
   - Mở bảng điều khiển DynamoDB.
   - Chọn bảng `videos`.
   - Chọn **Monitor** để kiểm tra các chỉ số trong **CloudWatch metrics**.

![alt text](/images/4.s3/image_42_6.png)

2. Kiểm tra bucket S3 `totgo1`:
   - Mở bucket `totgo1`.
   - Chọn thư mục `frames/`.
   - Kiểm tra tất cả các khung hình đã được trích xuất.

![alt text](/images/4.s3/image_42_7.png)
## Bước 5: Kiểm Tra Kết Quả
- Đảm bảo rằng tất cả các khung hình đã được trích xuất thành công và lưu trữ trong thư mục `frames/` trên S3.
- Kiểm tra các thông tin trong DynamoDB để xác nhận rằng metadata video đã được lưu trữ chính xác.

