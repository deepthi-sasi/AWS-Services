### Demo Project - Manually Deploy App on EC2

### Topics of the Demo Project
Deploy Web Application on EC2 Instance (manually)

### Technologies Used
* AWS
* Docker 
* Linux

**Project Description**

* Create and configure an EC2 Instance on AWS 
* Install Docker on remote EC2 Instance 
* Deploy Docker image from private Docker repository on EC2 Instance

### Steps to Create and Configure an EC2 Instance on AWS

### Launch EC2 Instance
```
1. Navigate: AWS Console > Services > Compute > EC2 > Launch instance

2. Configure Instance:
    - Name: web-server
    - Add tag: `Type` → "web-server-with-docker"
    - AMI: Amazon Linux
    - Instance type: `t2.micro` (free tier)

3. Key Pair:
    - Create new key pair: "docker-server"
    - Type: RSA, Format: .pem
    - Download the private key file

4. Network Settings:
    - Keep default VPC/subnet
    - Enable "Auto-assign public IP"
    - Create security group: "security-group-docker-server"
    - SSH rule: Change source from "Anywhere" to "My IP"

5. Launch: Keep storage defaults, set 1 instance, click "Launch instance"

```
### Install Docker on remote EC2 Instance 

**Step 2: Install Docker on EC2** 

**Execute the following commands on the EC2 terminal:**

```sudo yum update
sudo yum install docker
sudo service docker start
# add the ec2-user to the docker group 
# to avoid having to use sudo for every docker command
sudo usermod -aG docker ec2-user
# the last command will be effective only after a re-login
exit
```
Steps to Deploy Docker Image from Private Docker Repository on EC2 Instance

Step 1: Clone the 'react-nodejs-example' application from [GitHub](https://github.com/techworld-with-nana/react-nodejs-example) and build the image with the Dockerfile using the below command.

`docker build -t deepthisasi/demo-app:react-nodejs-1.1 .`

**Step 2: Now switch back to the EC2 terminal, login to DockerHub, pull the image and start a container from it:**

```
ssh -i ~/.ssh/docker-server.pem ec2-user@34.253.XX.XX
docker login
docker pull deepthisasi/demo-app:react-nodejs-1.1
docker run -d -p 3000:3080 deepthisasi/demo-app:react-nodejs-1.1

```

**Step 3: Make the app accessible from the browser**
Open the AWS web console, go to "EC2 Dashboard" > "Instances" and check the 'web-server' instance. Select the "Security" tab below and click on the link for the 'security-group-docker-server'. Open the "Inbound rules" tab and press the "Edit inbound rules" button. Press "Add rule" and enter a rule of type "Custom TCP" for port 3000 with source "Anywhere-IPv4". Press "Save rules".

Now open the browser and navigate to http://34.253.XXX.XX:3000 to see the application in action.