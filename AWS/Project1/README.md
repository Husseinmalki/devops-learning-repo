Objective

Create a custom VPC with one public and one private subnet, set up the correct routing for internet access, and deploy EC2 instances across them.

Tasks
---

1. Create the VPC

- Custom VPC (e.g. 10.0.0.0/16)
- One public subnet
- One private subnet

---
2. Internet Access

- Create and attach an Internet Gateway
- Create an Elastic IP
- Create a NAT Gateway in the public subnet

---
3. Route Tables


- Public route table → default route via IGW
- Private route table → default route via NAT Gateway

---
4. EC2 Instances


- Public EC2: launch in public subnet with public IP
- Private EC2: launch in private subnet without public IP

---
5. Security


- Public EC2 SG: allow SSH/HTTP only from your IP
- Private EC2 SG: allow only internal access (e.g. from public EC2 or Bastion host)

---
Bonus (Optional)
 

- Deploy a Bastion Host to access the private EC2
- Enable CloudWatch monitoring on instances

We will be assuming this Project will be for an enterprise business with a large amount of users.

---
Step 1 
<img width="1870" height="305" alt="image" src="https://github.com/user-attachments/assets/ba70c13e-8418-40fe-8ac4-0dbd7b3284be" />

---
Step 2 
Add a tag and click create VPC at the bottom of the screen.
<img width="1593" height="829" alt="image" src="https://github.com/user-attachments/assets/5f203b76-36b1-4d47-b5ab-4cd9195b48ab" />

<img width="1857" height="790" alt="image" src="https://github.com/user-attachments/assets/68fe9624-a8c1-4675-8e0e-649a60e301c2" />

---
Step 3
Click subnet on the side bar and create subnet 
<img width="1866" height="330" alt="image" src="https://github.com/user-attachments/assets/3f525dda-97a1-4e2f-be2c-7cae044536ee" />

---
Step 4

Click the VPC we just created <img width="1225" height="308" alt="image" src="https://github.com/user-attachments/assets/38fbe496-54e1-4cc9-add9-ad3db34b91b7" />

Add the availability zone if required, then create the first subnet. We will be using the /20 subnet mask as it gives a reasonable amount of IPs (4096 IPs) for an enterprise business to use.
<img width="1630" height="647" alt="image" src="https://github.com/user-attachments/assets/2cceab7d-723c-4b6d-89b2-1b055e3de4fa" />

Do the same for the public Subnet <img width="1618" height="704" alt="image" src="https://github.com/user-attachments/assets/6e7bd2ca-ddc9-4810-8072-e20a7c20fab8" />

Make sure to add the tags 

They should now show up on the Subnets tab <img width="1617" height="61" alt="image" src="https://github.com/user-attachments/assets/0adf5f88-f544-4c31-876a-68b6f122e25a" />

---
Step 5

Click on internet gateway on the sidebar
<img width="1871" height="332" alt="image" src="https://github.com/user-attachments/assets/7c0988a1-b1dd-42cd-b274-e8d3c3705f12" />
Click create Internet gateway
<img width="1556" height="479" alt="image" src="https://github.com/user-attachments/assets/7873a2f7-efac-44d6-968e-a3df60f3c666" />
Add a Tag
<img width="1625" height="431" alt="image" src="https://github.com/user-attachments/assets/4676bc75-7bb6-4705-827e-7f43d5224a65" />
The gateway will not work as it is still detached from the VPC, we will need to attach it.
<img width="1626" height="365" alt="image" src="https://github.com/user-attachments/assets/e8251b5f-0a1e-41c8-b15e-06188e51bd7f" />
Click on the Internet gateway - Actions - Attach to VPC
<img width="1467" height="320" alt="image" src="https://github.com/user-attachments/assets/b7ffcfd3-f8b8-4e5a-848a-156950ec271c" />
Select the created VPC and attach. <img width="611" height="164" alt="image" src="https://github.com/user-attachments/assets/3f689b28-8845-474a-8e70-e8a29630e5f7" />

---
Step 6
We will create an elastic IP now
Click elastic IP and allocate elastic IP address.
<img width="1864" height="460" alt="image" src="https://github.com/user-attachments/assets/4a397a83-3bd5-4c55-ad88-91d15945194b" />

Select the network border group that your VPC uses, we use eu-west-2a so the IP address will belong within this AZ.
<img width="1697" height="690" alt="image" src="https://github.com/user-attachments/assets/5c2e9e4a-368e-4e27-99fc-419dcfc15ad3" />
<img width="1605" height="715" alt="image" src="https://github.com/user-attachments/assets/8e1b5e36-4a8d-491d-ab2b-af7110617466" />

---
Step 7 
Create a NAT gateway
<img width="1866" height="517" alt="image" src="https://github.com/user-attachments/assets/397ca5b0-6f8f-42ea-a449-6cc470af35e7" />
When selecting the options, select zonal for the availability mode, Select the public subnet to attach it to and then select the elastic IP created.
<img width="1733" height="775" alt="image" src="https://github.com/user-attachments/assets/df3e610d-0b5d-4c26-adf2-144f2da27b8b" />
When created it will give a pending state, this takes some time.
<img width="1863" height="606" alt="image" src="https://github.com/user-attachments/assets/13fe57d0-3778-4cc1-acca-67aeca2b74de" />
After a minute it will become available, with the public and private IPs attached to it.
<img width="1617" height="160" alt="image" src="https://github.com/user-attachments/assets/d8303e11-3179-417b-872b-e3f72fdc99e5" />

---
Step 8
Click on route table and then click Add route table, we will need a route table for each subnet.
<img width="1860" height="315" alt="image" src="https://github.com/user-attachments/assets/23876cdc-39c8-4bd3-9667-e8295ba9f05b" />
Create one for the private subnet and then one for the public.
<img width="1672" height="557" alt="image" src="https://github.com/user-attachments/assets/0a9eb602-b25d-43a9-950b-92a270be122a" />
<img width="1690" height="561" alt="image" src="https://github.com/user-attachments/assets/3cb78690-a98e-4f22-ba64-24f4bd17ffbb" />
Ensure they are attached to the correct VPC.

Click on to the private route table we created. Click on edit routes.
<img width="1624" height="492" alt="image" src="https://github.com/user-attachments/assets/15085b72-f81b-4465-b659-55d553c16c19" />

Add the 0.0.0.0/0 route pointing to the NAT gateway, this means any traffic that does not have an IP address specified will be directed towards the NAT gateway.
<img width="1700" height="351" alt="image" src="https://github.com/user-attachments/assets/07e87ab8-a073-4328-ab83-c2186c404e1d" />

Do the same for the public route table, but instead of a NAT gateway, we will do the Internet gateway.
<img width="1662" height="345" alt="image" src="https://github.com/user-attachments/assets/54f6ab5c-023d-4104-b085-6593c148506d" />

The route tables have been created, we must now associate each route table to the subnets we created.
<img width="1862" height="407" alt="image" src="https://github.com/user-attachments/assets/5c1a4b08-cb30-4207-b09e-c6a3dcc2f600" />
Head back to each subnet and Click route table - edit route table association 
<img width="1623" height="758" alt="image" src="https://github.com/user-attachments/assets/52a885a5-5531-4066-a499-25af3fd394f9" />

Then change the route table ID to the correct one. e.g Private route table for private subnet etc...

Same done here for public 
<img width="1695" height="483" alt="image" src="https://github.com/user-attachments/assets/b3b91737-994d-4d0a-9c13-cfb75b61f99f" />

This is how your Network should look right now.
<img width="1619" height="751" alt="image" src="https://github.com/user-attachments/assets/41f765ee-2349-41b4-95c7-a852bb1ad92b" />


---
Step 9
Search for EC2 in the search bar and click launch instance.
<img width="1305" height="855" alt="image" src="https://github.com/user-attachments/assets/c5505163-0a55-4048-9a29-344b42fac7b8" />

Enter a name and an image, any type of image will work for this project, we will go with a basic AWS linux image.
<img width="1195" height="821" alt="image" src="https://github.com/user-attachments/assets/ecef67f4-805f-4f2d-b688-93f6678aa8c6" />
Key pair is not needed for this project as we will not be accessing these VMs.

For the network settings select the VPC and the correct subnet we want to add this EC2 instance to - Click enable Auto-assign public IP, then click create a new security group and allow SSH traffic from my ip. 
<img width="1091" height="796" alt="image" src="https://github.com/user-attachments/assets/c052b3c8-4e87-43c1-9b57-8934326e4e68" />

Then click launch instance. 
<img width="1373" height="78" alt="image" src="https://github.com/user-attachments/assets/e6f07fe6-64d0-44a8-86cb-824d5ea35701" />


Repeat the steps but now in the private subnet.
<img width="1111" height="799" alt="image" src="https://github.com/user-attachments/assets/d7e57596-48be-4dea-83ae-6cbf28a64171" />
For the private EC2 instance, set the access via ssh by custom source type of the Public SG we created earlier.
and launch 
<img width="596" height="70" alt="image" src="https://github.com/user-attachments/assets/6ca685bd-dce4-4294-aebc-94b44326351e" />

There should now be 2 instances running.
<img width="1582" height="65" alt="image" src="https://github.com/user-attachments/assets/978cf5f4-4ede-4fdb-8280-925993bf3ded" />

As we did not add the inbound Security group rule we will add it now on the public security group 
<img width="1862" height="556" alt="image" src="https://github.com/user-attachments/assets/c2c57926-3383-46f5-af71-c35b8d92a059" />

We add an HTTP type rule and then select my IP.
<img width="1669" height="387" alt="image" src="https://github.com/user-attachments/assets/2df43b69-65f8-4acc-b5bd-cab0fa03055b" />




To test our EC2 instance we added this User data to create a simple web server we could access on HTTP
<img width="1677" height="692" alt="image" src="https://github.com/user-attachments/assets/f8e6b286-056c-4541-80ac-ee31f3076105" />

---
Debug:
We are trying to access our public EC2 instance on an http webserver but are met with 
      
      C:\Users\Husse>curl -v http://18.133.226.224
      *   Trying 18.133.226.224:80...
      * connect to 18.133.226.224 port 80 from 0.0.0.0 port 58140 failed: Connection refused
      * Failed to connect to 18.133.226.224:80 after 2082 ms: Could not connect to server
      * closing connection #0
      curl: (7) Failed to connect to 18.133.226.224:80 after 2082 ms: Could not connect to server

We are able to reach the instance but nothing is listening on port 80

After checking the instance logs we can see that the user data script that we added never ran. 
Meaning there was nothing listening on port 80.
We can try and fix this by terminating the instance and relaunching it with the user data added before the first initialisation.

<img width="1081" height="586" alt="image" src="https://github.com/user-attachments/assets/9966e100-16e7-4336-90cd-d2982da3dc38" />
We already have the SG created so we just select that along with the other options.

Clicking advanced options and then at the bottom, add the user data.
<img width="1083" height="479" alt="image" src="https://github.com/user-attachments/assets/c15b7f41-f11f-4f9c-8032-71e19a75ad8d" />

      #!/bin/bash
      dnf install -y httpd
      echo "<h1>Hello from $(hostname -f)</h1>" > /var/www/html/index.html
      systemctl enable --now httpd
      systemctl is-active httpd && echo "httpd is running"

Now it works
<img width="776" height="84" alt="image" src="https://github.com/user-attachments/assets/2f887238-62ac-40dd-9843-cdd4c6213714" />

And we can SSH into it.
<img width="747" height="427" alt="image" src="https://github.com/user-attachments/assets/f504b2f4-1cf3-414a-9a7d-60b3c5fbeeb6" />


---
Step 9 

We can now connect to the Private EC2 instance via the Public EC2 instance using the public EC2 instance as a bastion host/jumpbox.
We can use the -A option on SSH to pass on the key to the jump box.

      hussein@bench:~/.ssh$ ssh -A -i "PubKey.pem" ec2-user@18.130.239.163
      ** WARNING: connection is not using a post-quantum key exchange algorithm.
      ** This session may be vulnerable to "store now, decrypt later" attacks.
      ** The server may need to be upgraded. See https://openssh.com/pq.html
         ,     #_
         ~\_  ####_        Amazon Linux 2023
        ~~  \_#####\
        ~~     \###|
        ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
         ~~       V~' '->
          ~~~         /
            ~~._.   _/
               _/ _/
             _/m/'
      Last login: Sat Oct  3 14:40:24 2026 from 80.2.186.171

From this we can simply just ssh into the private EC2 instance.

      [ec2-user@ip-10-0-18-52 ~]$ ssh  ec2-user@10.0.14.180
      
         ,     #_
         ~\_  ####_        Amazon Linux 2023
        ~~  \_#####\
        ~~     \###|
        ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
         ~~       V~' '->
          ~~~         /
            ~~._.   _/
               _/ _/
             _/m/'
      Last login: Sat Oct  3 14:51:24 2026 from 10.0.18.52

---
Step 10

To enable cloudwatch monitoring, we need to configure a role to allow the SSM agent to connect to the Systems manager. Without the role we get this error.
<img width="1524" height="31" alt="image" src="https://github.com/user-attachments/assets/382d8ad9-2b7f-4702-87d0-5a1e0cfb64af" />

Head to IAM and create a new role.
<img width="1673" height="584" alt="image" src="https://github.com/user-attachments/assets/ea737d9b-6e63-4439-a374-7d171406bc31" />

Add these roles 
<img width="1583" height="441" alt="image" src="https://github.com/user-attachments/assets/15c7d7de-c1a2-4776-8b70-32ae862409ce" />
<img width="1565" height="375" alt="image" src="https://github.com/user-attachments/assets/66b2f55a-9482-4639-ae3e-db3ae6408314" />

Give it a name and then create it.

Go back to the EC2 instance, Actions - Security - Modify IAM role
Add the role

<img width="1666" height="301" alt="image" src="https://github.com/user-attachments/assets/b49861e2-9555-430b-a984-0b2a5cae4d2c" />

After restarting the SSM agent on the EC2 instance, it should be connected.

<img width="1537" height="336" alt="image" src="https://github.com/user-attachments/assets/b530322f-e977-44a4-8153-d3c522999e42" />
It will now allow you to configure a Cloudwatch agent
<img width="1298" height="292" alt="image" src="https://github.com/user-attachments/assets/6cb71dc4-8758-4b42-a354-edb77d7b2f83" />

This role must be created for the Cloudwatch agent
<img width="795" height="83" alt="image" src="https://github.com/user-attachments/assets/1c8772b8-c797-4628-8dd7-c57369ef7829" />

It should be installed and ready to configure 
<img width="1305" height="284" alt="image" src="https://github.com/user-attachments/assets/b7b20cc5-6c81-499f-9795-4cbf8f0c7fbd" />

The cloudwatch agent is now able to send metrics 
<img width="1543" height="422" alt="image" src="https://github.com/user-attachments/assets/b1aa8e96-a274-46da-aec6-d3056050956f" />




