---
title : "Video & Processing"
date : "`r Sys.Date()`"
weight : 4
chapter : false
pre : " <b> 4. </b> "
---

After videos are uploaded to the system, the next stage is processing them to extract key frames for analysis and search. This process uses AWS Lambda to automatically perform scene detection, extract frames at intervals, compress images (WebP), and store them in S3. Video and frame metadata are saved to DynamoDB for later search steps.

![alt text](/images/4.s3/arc-4.png)

In this section, you will practice:
- Creating an S3 bucket to store frames.
- Configuring Lambda to automatically extract frames from videos.
- Saving metadata to DynamoDB.

### Content:

  - [Quy trình xử lý video thành frame](./4.1-Create-S3-Lambda-DynamoDB/)
  - [Testing trích xuất video](./4.2-Testing-processing/)