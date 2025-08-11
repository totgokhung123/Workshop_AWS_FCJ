---
title: "Tạo S3, Lambda và DynamoDB cho Xử Lý Video"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 4.1. </b>"
---

## Bước 1: Tạo Thư Mục Chứa Thư Viện Cần Thiết Trong AWS CloudShell
```bash
mkdir -p ~/layer_build && cd ~/layer_build
rm -rf python
mkdir python
python3 -m pip install --no-cache-dir scenedetect[opencv-headless] -t python/
zip -r9 scenedetect-layer.zip python
ls -lh scenedetect-layer.zip
```
![alt text](/images/4.s3/image.png)

## Bước 2: Tạo Bucket S3 Cho Thư Viện
1. Mở bảng điều khiển S3 từ AWS Management Console.
2. Nhấn **Create bucket**.
3. Nhập tên bucket là `scenedetect-layer` và chọn khu vực (region) phù hợp.
4. Nhấn **Create bucket** để hoàn tất.
5. Tải tệp zip lên S3 bằng lệnh sau:
```bash
aws s3 cp scenedetect-layer.zip s3://scenedetect-layer/scenedetect-layer.zip
```
![alt text](/images/4.s3/image-1.png)

## Bước 3: Tạo Layer Lambda
1. Trong bảng điều khiển AWS Lambda, chọn **Layers** từ menu bên trái.
2. Nhấn **Create layer**.
3. Nhập tên là `scenedetect-layer`.
4. Chọn **Upload a file from Amazon S3** và dán liên kết S3:
   - `https://scenedetect-layer.s3.ap-southeast-1.amazonaws.com/scenedetect-layer.zip`
5. Chọn **Compatible runtimes** là `Python 3.9` và **Compatible architectures** là `x86_64`.
   ![alt text](/images/4.s3/image-2.png)
6. Nhấn **Create** để hoàn tất.
   ![alt text](/images/4.s3/image-3.png)

## Bước 4: Tạo Hàm Lambda Để Trích Xuất Khung Hình
1. Trong bảng điều khiển AWS Lambda, nhấn **Create function**.
2. Chọn **Author from scratch**.
3. Nhập tên hàm là `videosense-frame-extractor`, chọn **Python 3.9** làm runtime.
   ![alt text](/images/4.s3/image-4.png)
4. Nhấn **Create function**.
5. Cập nhật chính sách IAM cho hàm Lambda với các quyền sau:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowReadUploads",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:GetObjectAcl"
            ],
            "Resource": "arn:aws:s3:::totgo1/uploads/*"
        },
        {
            "Sid": "AllowWriteFrames",
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:PutObjectAcl"
            ],
            "Resource": "arn:aws:s3:::totgo1/frames/*"
        },
        {
            "Sid": "DynamoDBVideosMetadata",
            "Effect": "Allow",
            "Action": [
                "dynamodb:GetItem",
                "dynamodb:PutItem",
                "dynamodb:UpdateItem"
            ],
            "Resource": "arn:aws:dynamodb:ap-southeast-1:905418067993:table/videos"
        },
        {
            "Sid": "CWLogs",
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "arn:aws:logs:ap-southeast-1:905418067993:*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "sqs:ReceiveMessage",
                "sqs:DeleteMessage",
                "sqs:GetQueueAttributes"
            ],
            "Resource": "arn:aws:sqs:ap-southeast-1:905418067993:videosense-upload-queue"
        }
    ]
}
```
![alt text](/images/4.s3/image-7.png)

![alt text](/images/4.s3/image-8.png)
{{% notice note %}}
Chú ý sửa lại bucket name S3 cho phù hợp, Account ID
{{% /notice %}}
6. Nhấn **Save** để lưu các thay đổi.

## Bước 5: Tạo bảng DynamoDB
1. Mở bảng điều khiển DynamoDB từ AWS Management Console.
2. Nhấn **Create table**.
3. Nhập tên bảng là `videos` và thiết lập khóa phân vùng:
   - Tên khóa: `video_id`
   - Loại: String
4. Nhấn **Create table**.
5. Khi bảng được tạo xong, bạn sẽ thấy `videos` trong danh sách bảng.

## Bước 6: Cấu Hình Hàm Lambda
1. Mở hàm `videosense-frame-extractor` và đi đến **Configuration**.
2. Trong phần **Environment variables**, thêm các biến sau:
   - `BUCKET_NAME = totgo1`
   - `VIDEOS_TABLE = videos`
   - `MAX_FRAMES = 100` (có thể tùy chỉnh)
   - `EXTRACT_FPS = 30`
   - `TMP_DIR = /tmp`
   - `LOG_LEVEL = INFO`

![alt text](/images/4.s3/image-9.png)

3. Đảm bảo rằng trigger SQS đã được bật (kích thước lô = 1).
   ![alt text](/images/4.s3/image-10.png)
4. Thêm layer đã tạo trước đó:
   - Chọn **Add layer**.
   - Chọn **Custom layer** và chọn layer `scenedetect-layer`.
   ![alt text](/images/4.s3/image-5.png)

## Bước 7: Triển Khai Mã Lambda Để Xử Lý Video

![alt text](/images/4.s3/image-6.png)

1. Trong phần **Code source**, chọn **app.py**.
- Sử dụng mã sau trong `app.py` để xử lý video với SceneDetect:
```python
# app.py
import os
import decimal
import json
import time
import hashlib
import logging
import tempfile
import shutil
from urllib.parse import unquote_plus

import boto3
import botocore
import cv2
from scenedetect import VideoManager, SceneManager
from scenedetect.detectors import ContentDetector

# Setup
LOG_LEVEL = os.getenv("LOG_LEVEL", "INFO").upper()
logging.basicConfig(level=LOG_LEVEL)
logger = logging.getLogger("videosense-frame-extractor")

s3 = boto3.client("s3")
dynamodb = boto3.resource("dynamodb")

BUCKET = os.getenv("BUCKET_NAME", "totgo1")
VIDEOS_TABLE_NAME = os.getenv("VIDEOS_TABLE", "videos")
MAX_FRAMES = int(os.getenv("MAX_FRAMES", "100"))
EXTRACT_FPS = int(os.getenv("EXTRACT_FPS", "30"))
TMP_DIR = os.getenv("TMP_DIR", "/tmp")

table = dynamodb.Table(VIDEOS_TABLE_NAME)

def parse_sqs_record(record):
    """
    Parse an SQS record and return (bucket, key).
    Supports body being S3 event JSON (string) or direct JSON with 'bucket'/'key'.
    """
    body = record.get("body")
    if not body:
        raise ValueError("Empty SQS record body")

    # body might be JSON string
    try:
        payload = json.loads(body)
    except Exception:
        payload = body

    # If payload is S3 event structure
    if isinstance(payload, dict) and payload.get("Records"):
        try:
            s3info = payload["Records"][0]["s3"]
            bucket = s3info["bucket"]["name"]
            key = unquote_plus(s3info["object"]["key"])
            return bucket, key
        except Exception:
            pass

    # If payload is dict with bucket/key
    if isinstance(payload, dict) and payload.get("bucket") and payload.get("key"):
        return payload["bucket"], unquote_plus(payload["key"])

    raise ValueError("Cannot parse SQS body to get S3 bucket/key")

def make_deterministic_video_id(bucket, key):
    """
    Use S3 object's ETag if available to produce deterministic id.
    """
    try:
        head = s3.head_object(Bucket=bucket, Key=key)
        etag = head.get("ETag", "").strip('"')
    except botocore.exceptions.ClientError as e:
        logger.warning("head_object failed: %s", e)
        etag = None

    if etag:
        raw = f"{bucket}|{key}|{etag}"
    else:
        # fallback: use timestamp so not strictly deterministic but unique
        raw = f"{bucket}|{key}|{int(time.time())}"
    vid_hash = hashlib.sha256(raw.encode("utf-8")).hexdigest()
    return vid_hash

def write_status(video_id, status, extra=None):
    item = {
        "video_id": video_id,
        "status": status,
        "updated_at": int(time.time())
    }
    if extra and isinstance(extra, dict):
        item.update(extra)
    table.put_item(Item=item)
    logger.info("Wrote status=%s for video_id=%s", status, video_id)

def detect_scenes_local(video_path, downscale_factor=4, threshold=30.0):
    """
    Detect scenes and return scene_list.
    This function is robust to different PySceneDetect API signatures.
    """
    vm = VideoManager([video_path])
    sm = SceneManager()
    sm.add_detector(ContentDetector(threshold=threshold))

    try:
        # optional speed up
        try:
            vm.set_downscale_factor(downscale_factor)
        except Exception:
            # older/newer API might not have this method; ignore if not present
            pass

        # Start the video manager (required)
        vm.start()

        # Try calling detect_scenes using different possible signatures.
        # 1) common: sm.detect_scenes(frame_source=vm)
        # 2) fallback: sm.detect_scenes(vm)
        # 3) final fallback: try scenedetect.detect_scenes(vm) if available
        scene_list = None
        try:
            sm.detect_scenes(frame_source=vm)
        except TypeError:
            try:
                sm.detect_scenes(vm)
            except TypeError:
                # As a last attempt, try the module-level helper if present
                try:
                    from scenedetect import detect_scenes as sd_detect
                    sd_detect(vm, sm)
                except Exception as e:
                    # If all fails, re-raise for higher-level handling
                    raise RuntimeError("detect_scenes call failed for all known signatures") from e

        # get base_timecode if available for precise scene_list
        try:
            base_time = vm.get_base_timecode()
            scene_list = sm.get_scene_list(base_time)
        except Exception:
            # fallback to simpler getter
            try:
                scene_list = sm.get_scene_list()
            except Exception:
                # if still failing, return empty list
                scene_list = []

        return scene_list

    finally:
        try:
            vm.release()
        except Exception:
            pass

def extract_frames_upload(video_path, video_id, desired_fps=30, max_frames=300):
    cap = cv2.VideoCapture(video_path)
    if not cap.isOpened():
        raise RuntimeError("Cannot open video file for reading")

    video_fps = cap.get(cv2.CAP_PROP_FPS) or 30.0
    total_frames = int(cap.get(cv2.CAP_PROP_FRAME_COUNT) or 0)
    duration = total_frames / video_fps if video_fps else 0.0
    logger.info("Video opened: fps=%s total_frames=%s duration=%s", video_fps, total_frames, duration)

    # step in source frames for approx desired_fps
    step = max(1, int(round(video_fps / float(desired_fps)))) if desired_fps > 0 else 1
    logger.info("Extract step: take 1 frame every %s source frames", step)

    uploaded = 0
    idx = 0
    frame_no = 0

    # ensure tmp frames dir
    frames_tmp_dir = os.path.join(TMP_DIR, f"frames_{video_id}")
    os.makedirs(frames_tmp_dir, exist_ok=True)

    try:
        while True:
            ret, frame = cap.read()
            if not ret:
                break
            if frame_no % step == 0:
                # build local filename
                fname = f"frame_{idx:06d}.webp"
                local_path = os.path.join(frames_tmp_dir, fname)
                # write webp with quality ~80
                ok = cv2.imwrite(local_path, frame, [int(cv2.IMWRITE_WEBP_QUALITY), 80])
                if not ok:
                    logger.warning("cv2.imwrite failed for %s", local_path)
                # upload to S3
                s3_key = f"frames/{video_id}/{fname}"
                try:
                    s3.upload_file(local_path, BUCKET, s3_key)
                except Exception as e:
                    logger.exception("Failed upload %s to s3://%s/%s", local_path, BUCKET, s3_key)
                    raise
                # remove local file to save space
                try:
                    os.remove(local_path)
                except Exception:
                    pass
                uploaded += 1
                idx += 1
                if uploaded >= max_frames:
                    logger.info("Reached MAX_FRAMES %s", max_frames)
                    break
            frame_no += 1
    finally:
        cap.release()
        # remove frames tmp dir if empty
        try:
            if os.path.isdir(frames_tmp_dir):
                shutil.rmtree(frames_tmp_dir)
        except Exception:
            pass

    return uploaded, total_frames, duration, video_fps

def lambda_handler(event, context):
    logger.info("Lambda invoked; event keys=%s", list(event.keys()))
    start_ts = int(time.time())

    # Parse SQS
    try:
        record = event["Records"][0]
        bucket, key = parse_sqs_record(record)
    except Exception as e:
        logger.exception("Cannot parse SQS record: %s", e)
        raise

    logger.info("Processing s3://%s/%s", bucket, key)

    # Generate deterministic video_id
    video_id = make_deterministic_video_id(bucket, key)
    logger.info("video_id=%s", video_id)

    # Skip if already processed
    try:
        resp = table.get_item(Key={"video_id": video_id})
        if resp.get("Item", {}).get("status") == "processed":
            logger.info("Video %s already processed; skipping", video_id)
            return {"statusCode": 200, "body": f"already processed {video_id}"}
    except Exception as e:
        logger.warning("DynamoDB get_item failed: %s", e)

    # Mark start
    write_status(video_id, "processing", {
        "original_key": key,
        "processing_started_at": start_ts
    })

    local_video_path = os.path.join(TMP_DIR, os.path.basename(key))

    try:
        # Download video
        logger.info("Downloading from s3://%s/%s to %s", bucket, key, local_video_path)
        s3.download_file(bucket, key, local_video_path)

        # Detect scenes
        try:
            scene_list = detect_scenes_local(local_video_path)
        except Exception as e:
            logger.warning("Scene detection failed: %s", e)
            scene_list = []

        scenes_count = len(scene_list)
        logger.info("Detected %d scenes", scenes_count)
        for i, (start, end) in enumerate(scene_list[:10]):
            logger.info("Scene %d: %s -> %s", i, start, end)

        # Extract frames
        uploaded, total_frames, duration, video_fps = extract_frames_upload(
            local_video_path, video_id, desired_fps=EXTRACT_FPS, max_frames=MAX_FRAMES
        )
        logger.info("Uploaded %d frames (total=%s, duration=%.2fs, fps=%.2f)",
                    uploaded, total_frames, duration, video_fps)

        # Save metadata
        table.put_item(Item={
            "video_id": video_id,
            "original_key": key,
            "status": "processed",
            "frames_count": uploaded,
            "scenes_count": scenes_count,
            "frames_prefix": f"frames/{video_id}/",
            "processing_started_at": decimal.Decimal(start_ts),  # Convert to Decimal
            "processing_finished_at": decimal.Decimal(int(time.time())),  # Convert to Decimal
            "duration_seconds": decimal.Decimal(duration).quantize(decimal.Decimal('0.00')),  # Convert to Decimal with precision
            "total_frames": total_frames,  # Assuming this is an integer
            "video_fps": decimal.Decimal(video_fps).quantize(decimal.Decimal('0.00'))  # Convert to Decimal with precision
        })

    except Exception as e:
        logger.exception("Processing failed for %s", video_id)
        write_status(video_id, "failed", {
            "error": str(e),
            "processing_finished_at": int(time.time())
        })
        raise
    finally:
        # Cleanup local files
        for path in [local_video_path, os.path.join(TMP_DIR, f"frames_{video_id}")]:
            try:
                if os.path.exists(path):
                    if os.path.isfile(path):
                        os.remove(path)
                    else:
                        shutil.rmtree(path)
            except Exception as ce:
                logger.warning("Cleanup failed for %s: %s", path, ce)

    return {"statusCode": 200, "body": json.dumps({
        "video_id": video_id,
        "frames_uploaded": uploaded
    })}
```

## Bước 8: Kiểm Tra Hàm Lambda
1. Đảm bảo rằng hàm Lambda đã được cấu hình đúng.
2. Kiểm tra với một video mẫu để xác minh rằng các khung hình được trích xuất và xử lý chính xác.
3. Theo dõi các log trong CloudWatch để xem thông tin chi tiết về quá trình xử lý.

