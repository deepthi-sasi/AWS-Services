### Demo Project - Deploy App on EC2 from Jenkins Pipeline (using Docker)

### Topics of the Demo Project

### CD - Deploy Application from Jenkins Pipeline to EC2 Instance (automatically with docker)

### Technologies Used

* AWS 
* Jenkins 
* Docker 
* Linux 
* Git 
* Java 
* Maven 
* Docker Hub

### Project Description

* Prepare AWS EC2 Instance for deployment (Install Docker)
* Create ssh key credentials for EC2 server on Jenkins 
* Extend the CI pipeline with a deploy step that SSHs into the EC2 instance and deploys the newly built Docker image from Jenkins 
* Configure security group on EC2 Instance to allow access to our web application

Steps to prepare AWS EC2 instance for deployment (install Docker)

We already installed Docker on EC2. See [Demo Project 1, Steps to Install Docker on the EC2 Instance.](https://github.com/deepthi-sasi/AWS-Services/tree/main/1.deploy-app-on-ec2-manually)

Steps to create ssh key credentials for EC2 server on Jenkins

1. Add SSH Agent Plugin to Jenkins
   Go to Dashboard → Manage Jenkins → Manage Plugins and install the SSH Agent plugin.

2. Add EC2 SSH Credentials to Jenkins

Open the devops-bootcamp-multibranch-pipeline → Credentials → Global credentials
Click Add Credentials and configure:

Kind: SSH Username with private key
ID: ec2-server-key
Username: ec2-user
Private Key: Paste the contents of ~/.ssh/docker-server.pem

Click Create

3. Allow Jenkins to SSH into EC2

In AWS Console go to EC2 → Instances → web-server → Security → security-group-docker-server
Under Inbound Rules, click Edit inbound rules → Add rule
Add:

Type: SSH | Port: 22 | Source: Custom — <jenkins-server-ip>/32

Click Save rules

#### Steps to extend the CI pipeline with deploy step to deploy newly built image from Jenkins server

Step 1: Open the Jenkinsfile in the application project, which is built in the aws-multibranch pipeline ([aws-java-maven-app](https://github.com/deepthi-sasi/aws-java-maven-app)) and add the following stage:

        stage('Deploy Application') {
            steps {
                script {
                    echo 'deploying Docker image to EC2 server...'
                    def dockerCmd = 'docker run -d -p 8000:8080 deepthisasi/demo-app:java-maven-app-1.1'
                    sshagent(['ec2-server-key']) {
                    sh "ssh -o StrictHostKeyChecking=no ec2-user@34.253.XX.XX ${dockerCmd}"
                    }
                }
            }
        }
The option `-o StrictHostKeyChecking=no` is necessary to avoid ssh asking whether the server should be added to the known hosts.

#### Step 2: To allow EC2 to pull a Docker image from our private repository on DockerHub, we have to login from EC2 to DockerHub once.

ssh -i ~/.ssh/docker-server.pem ec2-user@35.156.XX.XX
docker login

This will create an entry in /home/ec2-user/.docker/config.json and keep the ec2-user logged in.

#### Steps to configure security group on EC2 instance to allow access to our web application

To allow internet access to the application, add an inbound firewall rule — Custom TCP, port 8000, source 0.0.0.0/0.

Now open the browser and navigate to http://34.253.XX.XX:8000 to see the application in action.