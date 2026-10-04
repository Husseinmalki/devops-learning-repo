Objective

Deploy two EC2 instances behind an ALB. The ALB must handle all incoming traffic. EC2 instances should not be accessible directly from the internet.
---
Tasks

1. Two EC2 Instances

- Launch two EC2 instances in the same VPC
- Use different availability zones where possible
- Install a simple web server using user-data
- Each instance should return different content for testing

---

2. Set Up the ALB

- Create an ALB in two public subnets
- Add HTTP (port 80) listener
- Create a Target Group
- Register both EC2 instances
- Configure a health check on the root path /

---
3. Security Groups

- ALB SG: allow HTTP from anywhere
- EC2 SG: allow HTTP only from the ALB SG
- Do not allow direct public access to EC2

---
4. Testing

- Visit the ALB DNS name
- Refresh to verify traffic alternates between both instances
- Confirm health checks are healthy

---
Bonus (Optional)

- Add a Route53 DNS name and point it to the ALB DNS name via ALIAS record type. 
- Add an HTTPS listener with ACM
- Add an Auto Scaling Group behind the ALB

---
Step 1 
Create the VPC
<img width="1631" height="691" alt="image" src="https://github.com/user-attachments/assets/be798d6b-4475-471d-9c2b-b07bcca9ef0b" />
Create the subnets
<img width="1265" height="50" alt="image" src="https://github.com/user-attachments/assets/b92a5d87-3892-4fdd-bdc1-bb3193523383" />
<img width="702" height="64" alt="image" src="https://github.com/user-attachments/assets/3a828d9e-dcee-493a-84ec-aa58ddd17a6e" />
Ensure they are in different AZs
Create the IGW and Attach to the VPC, ensure the route table points to the IGW.

---
Step 2
Setting up 2 different EC2 instances in the VPC, they should be in different AZs and have a simple web server interface running. We will reuse the data from the previous project

The user data

    #!/bin/bash
    dnf install -y httpd
    echo "Hello from $(hostname -f)" > /var/www/html/index.html 
    systemctl enable --now httpd 
    systemctl is-active httpd && echo "httpd is running"

---    
Debug moment:

The user data above was not starting the EC2 web server correctly, I had to ssh in and run the script manually. I tested with adding the line 
    systemctl start httpd
To another instance and it worked. From my knowledge the systemctl enable --now httpd command should do what systemctl start does.
After checking the logs

      [  727.833571] cloud-init[1947]: No match for argument: httpd
      [  727.850566] cloud-init[1947]: Error: Unable to find a match: httpd
      [  727.902387] cloud-init[1947]: /var/lib/cloud/instance/scripts/part-001: line 3: /var/www/html/index.html: No such file or directory
      [  727.909309] cloud-init[1947]: Failed to enable unit: Unit file httpd.service does not exist.

This means the dnf install -y httpd command was not running causing the other commands to fail. 
This does not make sense however as when the second instance was launched with systemctl start httpd it should have also failed.
After reading more of the logs, I found.

    [  367.776302] cloud-init[1947]: Error: Failed to download metadata for repo 'amazonlinux': Cannot prepare internal mirrorlist: Curl error (28): Timeout was reached for https://al2023-repos-eu-north-1-de612dc2.s3.dualstack.eu-north-1.amazonaws.com/core/mirrors/2023.12.20260930/x86_64/mirror.list?instance_id=i-0b9bb897009c7bdb8 [Connection timed out after 30000 milliseconds]
    [  727.797803] cloud-init[1947]: Amazon Linux 2023 Kernel Livepatch repository   0.0  B/s |   0  B     00:00
    [  727.798008] cloud-init[1947]: Errors during downloading metadata for repository 'kernel-livepatch':
    [  727.798173] cloud-init[1947]:   - Curl error (28): Timeout was reached for https://al2023-repos-eu-north-1-de612dc2.s3.dualstack.eu-north-1.amazonaws.com/kernel-livepatch/mirrors/al2023/x86_64/mirror.list?instance_id=i-0b9bb897009c7bdb8 [Connection timed out after 30000 milliseconds]
    [  727.798370] cloud-init[1947]:   - Curl error (28): Timeout was reached for https://al2023-repos-eu-north-1-de612dc2.s3.dualstack.eu-north-1.amazonaws.com/kernel-livepatch/mirrors/al2023/x86_64/mirror.list?instance_id=i-0b9bb897009c7bdb8 [Connection timed out after 30001 milliseconds]
    [  727.798573] cloud-init[1947]: Error: Failed to download metadata for repo 'kernel-livepatch': Cannot prepare internal mirrorlist: Curl error (28): Timeout was reached for https://al2023-repos-eu-north-1-de612dc2.s3.dualstack.eu-north-1.amazonaws.com/kernel-livepatch/mirrors/al2023/x86_64/mirror.list?instance_id=i-0b9bb897009c7bdb8 [Connection timed out after 30000 milliseconds]
    [  727.803695] cloud-init[1947]: Ignoring repositories: amazonlinux, kernel-livepatch

This showed that it was not connecting to the AWX linux repo and therefore failing to install the httpd.
Checking the system logs for my second instance, this never happened. 
I attributed this to the fact that I had not attached the IGW to the VPC before I created the EC2 instance.

---
Also ensure to select the correct subnet.
<img width="912" height="303" alt="image" src="https://github.com/user-attachments/assets/0d425084-de02-4aaa-9305-23630920ee35" />

---
Step 3
Create the ALBs

<img width="1176" height="520" alt="image" src="https://github.com/user-attachments/assets/fa5259e2-7e41-4de1-8db9-c0ddff914fbf" />
The ALBs will be internet facing.

Create the custom security group for the ALB.
<img width="1674" height="638" alt="image" src="https://github.com/user-attachments/assets/ee8eab83-ce82-4cdc-8586-5571aa246cd4" />
<img width="1397" height="216" alt="image" src="https://github.com/user-attachments/assets/e821cd30-79e5-48b0-9679-3a1c1d696030" />


You will need to create a target group before finishing the ALB
<img width="1032" height="772" alt="image" src="https://github.com/user-attachments/assets/f64814c9-ccb4-4714-8920-1883a2542249" />
<img width="1199" height="637" alt="image" src="https://github.com/user-attachments/assets/6625cc68-6117-4180-9117-c280fca72d23" />

Select the instances that we created. Add them to pending group.
<img width="1581" height="587" alt="image" src="https://github.com/user-attachments/assets/c9fdf9db-5757-46e9-bbc0-c9b78f9151b3" />

Return to the ALB
<img width="1606" height="714" alt="image" src="https://github.com/user-attachments/assets/4ccbba1e-1758-4739-8ec0-33e6e2bb4744" />
Select the target group we just created.
Create the ALB
<img width="1603" height="712" alt="image" src="https://github.com/user-attachments/assets/ed98b650-31b1-4c65-868a-8289c8e28290" />
This will take a few minutes to start.

While that is being created, lets go back and change our SGs for our EC2 instances to only allow http traffic from the ALB
<img width="1599" height="164" alt="image" src="https://github.com/user-attachments/assets/b81e146c-3728-499b-8243-859c6fce982c" />

Sidenote: when adjusting the HTTP rules you will have to delete the previous Inboud rules as they clash.

<img width="1637" height="361" alt="image" src="https://github.com/user-attachments/assets/63709f26-1bdb-4c5c-b6db-14a7bab6e276" />

---
Step 4
After that is all done, visit the ALB DNS name with http:// at the start
<img width="840" height="142" alt="image" src="https://github.com/user-attachments/assets/ef239a78-70e9-4438-9864-3c70697daf60" />
<img width="671" height="124" alt="image" src="https://github.com/user-attachments/assets/9ff15276-8579-4905-8f89-bc1c833c8259" />
After refreshing the page you can see the instances we are connecting to changing.
<img width="1450" height="465" alt="image" src="https://github.com/user-attachments/assets/d46eb3e8-48f0-480d-ae19-3f470e6d74f5" />
We can also see the instances are healthy.

---
Step 5
For the Bonus tasks I am unable to import my domain name into Route53 as I am on the free account and upgrading it does not work at the time of working on the project.
So I will stick with creating the auto scaling group.

First we must create a launch template for how these instances will run.
<img width="1218" height="489" alt="image" src="https://github.com/user-attachments/assets/e1181477-6e0b-4c23-950a-68811124b756" />

For the subnets choose not to include them,
<img width="1187" height="533" alt="image" src="https://github.com/user-attachments/assets/b68ee517-289b-4285-a9b7-a76c67b57916" />
Add the correct user data.

Head back to the auto scaling group and select the subnets that you want the Auto scaler to add/remove instances, in this case our subnets.
<img width="1319" height="622" alt="image" src="https://github.com/user-attachments/assets/236cb8bd-2a84-40bd-af9c-e32ced7bfbc9" />

Attach the ALB to the ASG so the instances sit behind the ALB.
<img width="1326" height="500" alt="image" src="https://github.com/user-attachments/assets/58718b05-6961-48d9-afb7-a99f62ed3b02" />

turn on ELB health checks, this will mean if an instance is unhealthy the auto scaling group can replace it.
<img width="1375" height="489" alt="image" src="https://github.com/user-attachments/assets/871760db-82df-489b-bc46-23624e3e5dd9" />
Keep the rest of the settings to its defaults and create the ASG.

Now let us test it.
<img width="451" height="20" alt="image" src="https://github.com/user-attachments/assets/f5ebdd4c-6d06-4cfc-8bdd-692a8a2c4d1c" />
After shutting down httpd on our server one we can see the auto scaling group created the new instance.

<img width="1632" height="199" alt="image" src="https://github.com/user-attachments/assets/78239ae0-6631-4e6a-8363-a09db4c7890a" />

Lets now start it back up and see what happens 
<img width="448" height="22" alt="image" src="https://github.com/user-attachments/assets/e0972865-6dd0-47d9-b703-f65cb394163a" />
<img width="347" height="135" alt="image" src="https://github.com/user-attachments/assets/2e0fd376-12c0-40b2-a399-712540eb508c" />

<img width="1370" height="39" alt="image" src="https://github.com/user-attachments/assets/4316b178-9509-417f-94ae-405ab43516b9" />
We can see the ASG is terminating the instance. 


---


Things that could be improved on. 
---
The EC2 instances could be in an private subnet and then use a NAT gateway in a public subnet to connect to the IGW to improve security.
2 SGs did not need to be created as both EC2 instances were going straight to the IGW and not through a NATGW on a private subnet.


Final Architecture.
---
<img width="3752" height="2968" alt="Cloud Architecture" src="https://github.com/user-attachments/assets/6af95295-93fe-4dd4-b496-813d416c77b5" />



