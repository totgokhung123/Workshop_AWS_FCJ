---
title : "Create security groups"
date : "`r Sys.Date()`"
weight : 4
chapter : false
pre : " <b> 2.1.4 </b> "
---

#### Create security groups

In this step, we will proceed to create the security groups used for our instances. As you can see, these security groups will not need to open traditional ports to **ssh** like port **22** or **remote desktop** through port **3389**.

#### Create security group for Linux instance located in public subnet

1. Go to [VPC service management console](https://console.aws.amazon.com/vpc)
  + Click **Security Group**.
  + Click **Create security group**.

![SG](/images/2.prerequisite/019-createsg.png)

3. In the **Security group name** field, enter **SG Public Linux Instance**.
  + In the **Description** section, enter **SG Public Linux Instance**.
  + In the **VPC** section, click the **X** to reselect the **Lab VPC** you created for this lab.

![SG](/images/2.prerequisite/020-createsg.png)

4. Keep **Outbound rule**, drag the mouse to the bottom.
  + Click **Create security group**.

{{%notice tip%}}
As you can see, the security group we created to use for Linux public instances will not need to open traditional ports to **ssh** like port **22**.
{{%/notice%}}


#### Create a security group for a Windows instance located in a private subnet

1. After successfully creating a security group for the Linux instance located in the public subnet, click the Security Groups link to return to the Security groups list.

![SG](/images/2.prerequisite/021-createsg.png)

2. Click **Create security group**.

3. In the **Security group name** field, enter **SG Private Windows Instance**.
  + In the **Description** section, enter **SG Private Windows Instance**.
  + In the **VPC** section, click the **X** to reselect the **Lab VPC** you created for this lab.

![SG](/images/2.prerequisite/022-createsg.png)

4. Scroll down.
  + Add **Outbound rule** to allow TCP 443 connection to 10.10.0.0/16 ( CIDR of **Lab VPC** we created)
  + Click **Create security group**.

![SG](/images/2.prerequisite/023-createsg.png)

{{%notice tip%}}
For the Instance in the private subnet, we will connect to the **Session Manager** endpoint over a TLS encrypted connection, so we need to allow outbound connection from our instance to VPC CIDR through port 443.
{{%/notice%}}


#### Create security group for VPC Endpoint

1. In this step, we will create security group for VPC Endpoint of **Session Manager**.
2. After successfully creating the security group for the Windows instance in the private subnet, click the Security Groups link to return to the Security groups list.
3. Click **Create security group**.
4. In the **Security group name** field, enter **SG VPC Endpoint**.
  + In the **Description** section, enter **SG VPC Endpoint**.
  + In the **VPC** section, click the **X** to reselect the **Lab VPC** you created for this lab.

![SG](/images/2.prerequisite/024-createsg.png)

5. Scroll down.
  + Delete **Outbound rule**.
  
![SG](/images/2.prerequisite/025-createsg.png)

6. Add **Inbound rule** allowing TCP 443 to come from 10.10.0.0/16 ( CIDR of **Lab VPC** we created ).
  + Click **Create security group**.

![SG](/images/2.prerequisite/026-createsg.png)

So we are done creating the necessary security groups for EC2 instances and VPC Endpoints.