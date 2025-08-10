---
title : "Tạo Private Windows EC2"
date :  "`r Sys.Date()`" 
weight : 6
chapter : false
pre : " <b> 2.1.6 </b> "
---

1. Truy cập [giao diện quản trị dịch vụ EC2](https://console.aws.amazon.com/ec2/v2/home)
  + Click **Instances**.
  + Click **Launch instances**.
  
2. Tại trang **Step 1: Choose an Amazon Machine Image (AMI)**.
  + Kéo chuột xuống phía dưới.
  + Click **Select** để lựa chọn AMI **Microsoft Windows Server 2019 Base**.
  
![EC2](/images/2.prerequisite/034-createec2.png)

3. Tại trang **Step 2: Choose an Instance Type**.
 + Click chọn Instance type **t2.micro**.
 + Click **Next: Configure Instance Details**.
 
![EC2](/images/2.prerequisite/029-createec2.png)

4. Tại trang **Step 3: Configure Instance Details**
  + Tại mục **Network** chọn **Lab VPC**.
  + Tại mục **Subnet** chọn **Lab Private Subnet**.
  + Tại mục **Auto-assign Public IP** chọn **Use subnet setting (Disable)**
  + Click **Next: Add Storage**.

![EC2](/images/2.prerequisite/035-createec2.png)

5. Click **Next: Add Tags** để chuyển sang bước kế tiếp.
  + Click **Next: Configure Security Group** để chuyển sang bước kế tiếp.


6. Tại trang **Step 6: Configure Security Group**.
  + Chọn **Select an existing security group**.
  + Chọn security group **SG Private Windows Instance**.
  + Click **Review and Launch**.

![EC2](/images/2.prerequisite/036-createec2.png)

7. Hộp thoại cảnh báo hiện lên vì chúng ta không cấu hình tường lửa cho phép kết nối vào port 22, Click **Continue** để tiếp tục.

8. Tại trang **Step 7: Review Instance Launch**.
  + Click **Launch**.

9. Tại hộp thoại **Select an existing key pair or create a new key pair**.
  + Click chọn **Choose an existing key pair**.
  + Tại mục **Key pair name** chọn **LabKeypair**.
  + Click chọn **I acknowledge that I have access to the corresponding private key file, and that without this file, I won't be able to log into my instance.**.
  + Click **Launch Instances** để tạo máy chủ EC2.

10. Click **View Instances** để quay lại danh mục EC2 instances.

11. Click vào biểu tượng edit dưới cột **Name**.
  + Tại hộp thoại **Edit Name** điền **Private Windows Instance**.
  + Click **Save**.

![EC2](/images/2.prerequisite/033-createec2.png)

Tiếp theo chúng ta sẽ tiến hành tạo các IAM Role để phục vụ cho Session Manager.