---
title: "Tạo và Cấu Hình Các Dịch Vụ Tìm Kiếm"
date: "`r Sys.Date()`"
weight: 5
chapter: false
pre: "<b> 5. </b>"
---

![alt text](/images/5.fwd/image.png)

## Bước 1: Tạo S3 "embeddings-cache"
1. Mở bảng điều khiển AWS và chọn **S3**.
2. Chọn bucket `totgo1`.
3. Tạo thư mục/prefix `embeddings-cache/` (chỉ là key prefix).
4. Trong bucket, chọn **Management**.
5. Nhấn **Lifecycle rules** và chọn **Create lifecycle rule**.
   - **Rule name**: embeddings-cache-7d.
   - **Scope**: Limit prefix `embeddings-cache/`.
   ![alt text](/images/5.fwd/image-1.png)
6. Thêm hành động hết hạn: 
   - **Expire current versions of objects** — 7 days.
7. Nhấn **Create rule** để hoàn tất.

**Ghi chú lưu trữ**: Lưu embeddings dưới dạng raw bytes float16 (nén) để giảm chi phí và egress. File key đề xuất: `embeddings-cache/{video_id}/{frame_id}.npy` hoặc `.bin`.

## Bước 2: Tạo 3 Role IAM cho Lambda
### Role cho Lambda Search
1. Mở bảng điều khiển IAM từ AWS Management Console.
2. Nhấn **Roles** và chọn **Create role**.
   ![alt text](/images/5.fwd/image-2.png)
3. Chọn **Lambda** làm loại dịch vụ.
   ![alt text](/images/5.fwd/image-3.png)
4. Nhập chính sách IAM sau:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": "arn:aws:s3:::totgo1/embeddings-cache/*"
    },
    {
      "Effect": "Allow",
      "Action": ["dynamodb:GetItem", "dynamodb:Query", "dynamodb:Scan"],
      "Resource": "arn:aws:dynamodb:ap-southeast-1:YOUR_ACCOUNT_ID:table/videos"
    },
    {
      "Effect": "Allow",
      "Action": ["lambda:InvokeFunction"],
      "Resource": "arn:aws:lambda:ap-southeast-1:YOUR_ACCOUNT_ID:function:lambda-clip-inference"
    },
    {
      "Effect": "Allow",
      "Action": ["logs:CreateLogGroup", "logs:CreateLogStream", "logs:PutLogEvents"],
      "Resource": "arn:aws:logs:ap-southeast-1:YOUR_ACCOUNT_ID:*"
    }
  ]
}
```

### Role cho Lambda CLIP và Lambda Batch
1. Tạo role tương tự như trên với chính sách IAM sau:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": "arn:aws:s3:::totgo1/frames/*"
    },
    {
      "Effect": "Allow",
      "Action": ["s3:PutObject", "s3:PutObjectAcl"],
      "Resource": "arn:aws:s3:::totgo1/embeddings-cache/*"
    },
    {
      "Effect": "Allow",
      "Action": ["logs:CreateLogGroup", "logs:CreateLogStream", "logs:PutLogEvents"],
      "Resource": "arn:aws:logs:ap-southeast-1:YOUR_ACCOUNT_ID:*"
    }
  ]
}
```

![alt text](/images/5.fwd/image-4.png)

## Bước 3: Tạo ECR "lambda-clip-inference"
1. Mở bảng điều khiển ECR từ AWS Management Console.
2. Nhấn **Create repository** và nhập tên là `lambda-clip-inference`.
3. Nhấn **Create repository** để hoàn tất.

## Bước 4: Tạo Dockerfile
1. Tạo tệp `Dockerfile` trong thư mục dự án và dán mã sau vào:
```dockerfile
# Dockerfile
FROM public.ecr.aws/lambda/python:3.9

# Install system deps (Pillow may need libjpeg)
RUN yum -y install libjpeg-turbo-devel gcc make git && yum clean all

# Copy handler
COPY app.py ${LAMBDA_TASK_ROOT}/
# Install Python packages (CPU wheels for torch)
RUN python3 -m pip install --upgrade pip setuptools wheel \
 && python3 -m pip install --no-cache-dir \
    "torch" "torchvision" --extra-index-url https://download.pytorch.org/whl/cpu \
    transformers \
    pillow \
    boto3 \
    numpy

# For Lambda container image, aws-lambda-ric entrypoint is already provided in base image.
# Set the handler (module.function)
CMD ["app.handler"]
```
![alt text](/images/5.fwd/image-5.png)

## Bước 5: Tạo Tệp app.py
1. Tạo tệp `app.py` trong cùng thư mục và dán mã sau vào:
```python
import json
import io
import os
import boto3
import torch
import numpy as np
from PIL import Image
from transformers import CLIPProcessor, CLIPModel

s3 = boto3.client("s3")

# Global model (load once per container)
MODEL_NAME = "openai/clip-vit-base-patch32"
_model = None
_processor = None
_device = torch.device("cpu")

def load_model():
    global _model, _processor
    if _model is None:
        _model = CLIPModel.from_pretrained(MODEL_NAME).to(_device)
        _processor = CLIPProcessor.from_pretrained(MODEL_NAME)
        _model.eval()
    return _model, _processor

def read_s3_image(s3uri):
    # s3uri: "s3://bucket/key"
    assert s3uri.startswith("s3://")
    path = s3uri[5:]
    bucket, key = path.split("/", 1)
    obj = s3.get_object(Bucket=bucket, Key=key)
    return Image.open(io.BytesIO(obj['Body'].read())).convert("RGB")

def handler(event, context):
    # event can be dict already or JSON string
    if isinstance(event, str):
        try:
            body = json.loads(event)
        except:
            body = {}
    else:
        body = event if isinstance(event, dict) else {}
    mode = body.get("mode", "text")
    items = body.get("items", [])

    model, processor = load_model()

    if mode == "text":
        # items: list of strings
        inputs = processor(text=items, return_tensors="pt", padding=True)
        for k,v in inputs.items():
            inputs[k] = v.to(_device)
        with torch.no_grad():
            feats = model.get_text_features(**inputs)
    else:  # image mode, items are s3:// URIs or base64 bytes (here we handle s3)
        imgs = []
        for it in items:
            if isinstance(it, str) and it.startswith("s3://"):
                imgs.append(read_s3_image(it))
            else:
                # unsupported type -> skip
                continue
        inputs = processor(images=imgs, return_tensors="pt", padding=True)
        for k,v in inputs.items():
            inputs[k] = v.to(_device)
        with torch.no_grad():
            feats = model.get_image_features(**inputs)

    # normalize
    feats = torch.nn.functional.normalize(feats, p=2, dim=1).cpu().numpy()
    # return list of lists (float32)
    embeddings = feats.astype(float).tolist()
    return {
        "statusCode": 200,
        "body": json.dumps({"embeddings": embeddings, "dim": feats.shape[1]})
    }
```

## Bước 6: Chạy Lệnh CMD Để Xây Dựng Docker Image
1. Mở CMD và chạy các lệnh sau:
(a) Đăng nhập vào ECR từ Docker:
```bash
aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin <acount_id>.dkr.ecr.ap-southeast-1.amazonaws.com
```
(b) Xây dựng image:
```bash
docker build -t lambda-clip-inference .
```

![alt text](/images/5.fwd/image-6.png)

![alt text](/images/5.fwd/image-7.png)
(c) Gán thẻ cho image theo URI ECR:
```bash
docker tag lambda-clip-inference:latest <acount_id>.dkr.ecr.ap-southeast-1.amazonaws.com/lambda-clip-inference:latest
```
(d) Đẩy image lên ECR:
```bash
docker push <acount_id>.dkr.ecr.ap-southeast-1.amazonaws.com/lambda-clip-inference:latest
```
2. Kiểm tra docker build đã được push lên ECR hay chưa
![alt text](/images/5.fwd/image-8.png)

![alt text](/images/5.fwd/image-9.png)

## Bước 7: Chạy test truy vấn sử dụng mô hình CLIP
1. Vào AWS Lambda → Create function.
2. Chọn Container image.
   ![alt text](/images/5.fwd/image-10.png)
3. Nhập Function name = clip-inference.
4. Container image URI = URI ECR vừa push.
   ![alt text](/images/5.fwd/image-11.png)

   ![alt text](/images/5.fwd/image-12.png)
5. Chọn Role cho Lambda ```LambdaCLIPRole```
   
   ![alt text](/images/5.fwd/image-13.png)
6. Test Lambda :

    - Chọn **Test** và nhập payload JSON sau:
  ```json
  {
  "mode": "text",
  "items": ["a man riding a bicycle"]
  }
  ```
    - Nhấn **Test**.
    - Kết quả trả về sẽ là embeddings của các chuỗi văn bản đã cho.

![alt text](/images/5.fwd/image-14.png)

![alt text](/images/5.fwd/image-15.png)

![alt text](/images/5.fwd/image-16.png)

Nội dung này đã được trình bày chi tiết cho từng bước trong quá trình tạo và cấu hình các dịch vụ tìm kiếm. Nếu bạn cần thêm thông tin hoặc chỉnh sửa nào khác, hãy cho tôi biết!