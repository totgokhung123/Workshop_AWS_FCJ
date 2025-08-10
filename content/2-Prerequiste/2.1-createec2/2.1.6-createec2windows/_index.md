---
title : "Create Private Instance"
date : "`r Sys.Date()`"
weight : 6
chapter : false
pre : " <b> 2.1.6 </b> "
---

1. Go to [EC2 service management console](https://console.aws.amazon.com/ec2/v2/home)
  + Click **Instances**.
  + Click **Launch instances**.
  
2. On the **Step 1: Choose an Amazon Machine Image (AMI)** page.
  + Drag the mouse down.
  + Click **Select** to select AMI **Microsoft Windows Server 2019 Base**.
  
![EC2](/images/2.prerequisite/034-createec2.png)

3. On the **Step 2: Choose an Instance Type** page.
 + Click on Instance type **t2.micro**.
 + Click **Next: Configure Instance Details**.
 
![EC2](/images/2.prerequisite/029-createec2.png)

4. At **Step 3: Configure Instance Details** page
  + In the **Network** section, select **Lab VPC**.
  + In the **Subnet** section, select **Lab Private Subnet**.
  + At **Auto-assign Public IP** select **Use subnet setting (Disable)**
  + Click **Next: Add Storage**.

![EC2](/images/2.prerequisite/035-createec2.png)

5. Click **Next: Add Tags** to move to the next step.
  + Click **Next: Configure Security Group** to move to the next step.


6. On page **Step 6: Configure Security Group**.
  + Select **Select an existing security group**.
  + Select security group **SG Private Windows Instance**.
  + Click **Review and Launch**.

![EC2](/images/2.prerequisite/036-createec2.png)

7. The warning dialog box appears because we do not configure the firewall to allow connections to port 22, Click **Continue** to continue.

8. At page **Step 7: Review Instance Launch**.
  + Click **Launch**.

9. In the **Select an existing key pair or create a new key pair** dialog box.
  + Click **Choose an existing key pair**.
  + In the **Key pair name** section, select **LabKeypair**.
  + Click **I acknowledge that I have access to the corresponding private key file, and that without this file, I won't be able to log into my instance.**.
  + Click **Launch Instances** to create EC2 server.

10. Click **View Instances** to return to the list of EC2 instances.

11. Click the edit icon under the **Name** column.
  + In the **Edit Name** dialog box, enter **Private Windows Instance**.
  + Click **Save**.

![EC2](/images/2.prerequisite/033-createec2.png)

Next, we will proceed to create IAM Roles to serve the Session Manager.