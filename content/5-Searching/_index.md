---
title: "Create and Configure Search Services"
date: "`r Sys.Date()`"
weight: 5
chapter: false
pre: "<b> 5. </b>"
---

![alt text](/images/5.fwd/image.png)

## Step 1: Create S3 "embeddings-cache"
1. Open the AWS console and select **S3**.
2. Select the `totgo1` bucket.
3. Create a folder/prefix `embeddings-cache/` (just a key prefix).
4. In the bucket, go to **Management**.
5. Click **Lifecycle rules** and select **Create lifecycle rule**.
   - **Rule name**: embeddings-cache-7d.
   - **Scope**: Limit prefix `embeddings-cache/`.
   ![alt text](/images/5.fwd/image-1.png)
6. Add expiration action:
   - **Expire current versions of objects** — 7 days.
7. Click **Create rule** to finish.

**Storage note**: Store embeddings as raw float16 bytes (compressed) to reduce cost and egress. Suggested file key: `embeddings-cache/{video_id}/{frame_id}.npy` or `.bin`.

## Step 2: Create 3 IAM Roles for Lambda
### Role for Lambda Search
1. Open the IAM console from AWS Management Console.
2. Click **Roles** and select **Create role**.
   ![alt text](/images/5.fwd/image-2.png)
3. Choose **Lambda** as the service type.
   ![alt text](/images/5.fwd/image-3.png)
4. Enter the following IAM policy:
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

### Role for Lambda CLIP and Lambda Batch
1. Create a similar role with the following IAM policy:
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

## Step 3: Create ECR "lambda-clip-inference"
1. Open the ECR console from AWS Management Console.
2. Click **Create repository** and enter the name `lambda-clip-inference`.
3. Click **Create repository** to finish.

## Step 4: Create Dockerfile
1. Create a `Dockerfile` in your project folder and paste the following code:
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

## Step 5: Create app.py File
1. Create a file named `app.py` in the same folder and paste the following code:
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

## Step 6: Run CMD Commands to Build Docker Image
1. Open CMD and run the following commands:
(a) Log in to ECR from Docker:
```bash
aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin <acount_id>.dkr.ecr.ap-southeast-1.amazonaws.com
```
(b) Build the image:
```bash
docker build -t lambda-clip-inference .
```

![alt text](/images/5.fwd/image-6.png)

![alt text](/images/5.fwd/image-7.png)
(c) Tag the image with the ECR URI:
```bash
docker tag lambda-clip-inference:latest <acount_id>.dkr.ecr.ap-southeast-1.amazonaws.com/lambda-clip-inference:latest
```
(d) Push the image to ECR:
```bash
docker push <acount_id>.dkr.ecr.ap-southeast-1.amazonaws.com/lambda-clip-inference:latest
```
2. Check if the docker build has been pushed to ECR
![alt text](/images/5.fwd/image-8.png)

![alt text](/images/5.fwd/image-9.png)

## Step 7: Test Query Using CLIP Model
1. Go to AWS Lambda → Create function.
2. Select Container image.
   ![alt text](/images/5.fwd/image-10.png)
3. Enter Function name = clip-inference.
4. Container image URI = the ECR URI you just pushed.
   ![alt text](/images/5.fwd/image-11.png)

   ![alt text](/images/5.fwd/image-12.png)
5. Select the Role for Lambda ```LambdaCLIPRole```
   
   ![alt text](/images/5.fwd/image-13.png)
6. Test Lambda:

    - Click **Test** and enter the following JSON payload:
  ```json
  {
  "mode": "text",
  "items": ["a man riding a bicycle"]
  }
  ```
    - Click **Test**.
    - The returned result will be the embeddings of the given text strings.

![alt text](/images/5.fwd/image-14.png)

![alt text](/images/5.fwd/image-15.png)

![alt text](/images/5.fwd/image-16.png)

This content has been detailed step-by-step for creating and configuring the search services. If you need more information or further edits, please let me.