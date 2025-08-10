---
title : "Create Public instance"
date : "`r Sys.Date()`"
weight : 5
chapter : false
pre : " <b> 2.1.5 </b> "
---

1. Go to [EC2 service management console](https://console.aws.amazon.com/ec2/v2/home)
  + Click **Instances**.
  + Click **Launch instances**.
  
![EC2](/images/2.prerequisite/027-createec2.png)

2. On the **Step 1: Choose an Amazon Machine Image (AMI)** page.
  + Click **Select** to select AMI **Amazon Linux 2 AMI**.
  
![EC2](/images/2.prerequisite/028-createec2.png)

3. On the **Step 2: Choose an Instance Type** page.
 + Click on Instance type **t2.micro**.
 + Click **Next: Configure Instance Details**.
 
![EC2](/images/2.prerequisite/029-createec2.png)

4. At **Step 3: Configure Instance Details** page
  + In the **Network** section, select **Lab VPC**.
  + In the **Subnet** section, select **Lab Public Subnet**.
  + In the **Auto-assign Public IP** section, select **Use subnet setting (Enable)**
  + Click **Next: Add Storage**.

![EC2](/images/2.prerequisite/030-createec2.png)

5. Click **Next: Add Tags** to move to the next step.
  + Click **Next: Configure Security Group** to move to the next step.


6. On page **Step 6: Configure Security Group**.
  + Select **Select an existing security group**.
  + Select security group **SG Public Linux Instance**.
  + Click **Review and Launch**.

![EC2](/images/2.prerequisite/031-createec2.png)

7. The warning dialog box appears because we do not configure the firewall to allow connections to port 22, Click **Continue** to continue.

8. At page **Step 7: Review Instance Launch**.
  + Click **Launch**.

9. In the **Select an existing key pair or create a new key pair** dialog box.
  + Click to select **Create a new key pair**.
  + In the **Key pair name** field, enter **LabKeypair**.
  + Click **Download Key Pair** and save it to your computer.
  + Click **Launch Instances** to create EC2 server.

![EC2](/images/2.prerequisite/032-createec2.png)

10. Click **View Instances** to return to the list of EC2 instances.

11. Click the edit icon under the **Name** column.
  + In the **Edit Name** dialog box, enter **Public Linux Instance**.
  + Click **Save**.

![EC2](/images/2.prerequisite/033-createec2.png)

Next, we will do the same to create an EC2 Instance Windows running in the Private subnet.