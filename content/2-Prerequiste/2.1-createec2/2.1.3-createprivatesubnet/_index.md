---
title : "Create Private Subnet"
date : "`r Sys.Date()`"
weight : 3
chapter : false
pre : " <b> 2.1.3 </b> "
---

#### Create Private Subnet

1. Click **Subnets**.
   + Click **Create subnet**.

![VPC](/images/2.prerequisite/017-createsubnet.png)

2. At the **Create subnet** page.
   + In the **VPC ID** section, click **Lab VPC**.
   + In the **Subnet name** field, enter **Lab Private Subnet**.
   + In the **Availability Zone** section, select the first Availability zone.
   + In the field **IPv4 CIRD block** enter **10.10.2.0/24**.

![VPC](/images/2.prerequisite/018-createsubnet.png)

3. Scroll to the bottom of the page, click **Create subnet**.

The next step is to create the necessary security groups for the lab.