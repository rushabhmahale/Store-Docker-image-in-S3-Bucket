# Store-Docker-image-in-S3-Bucket

## What is Docker? 
Docker is an open platform for developing, shipping, and running applications. Docker enables you to separate your applications from your infrastructure so you can deliver software quickly. With Docker, you can manage your infrastructure in the same ways you manage your applications. By taking advantage of Docker’s methodologies for shipping, testing, and deploying code quickly, you can significantly reduce the delay between writing code and running it in production.

Refer this document:- https://docs.docker.com/get-started/overview/

## Architecture Diagram 
![docker-s3 drawio (1)](https://user-images.githubusercontent.com/63963025/202839651-5b5b207e-e772-4a95-b36c-151628cf27dc.png)

## What is Containers in Docker? 
A container is a standard unit of software that packages up code and all its dependencies so the application runs quickly and reliably from one computing environment to another. A Docker container image is a lightweight, standalone, executable package of software that includes everything needed to run an application: code, runtime, system tools, system libraries and settings. Container images become containers at runtime and in the case of Docker containers images become containers when they run on Docker Engine.

Refer this document:- https://www.docker.com/resources/what-container/

## What is S3 bucket in AWS? 
Amazon Simple Storage Service (Amazon S3) is an object storage service that offers industry-leading scalability, data availability, security, and performance. Customers of all sizes and industries can use Amazon S3 to store and protect any amount of data for a range of use cases, such as data lakes, websites, mobile applications, backup and restore, archive, enterprise applications, IoT devices, and big data analytics. Amazon S3 provides management features so that you can optimize, organize, and configure access to your data to meet your specific business, organizational, and compliance requirements.

Refer this document:-  https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html

## Steps to be followed:
### Step 1 Create Build Server where your developer will be create a application. 

- Go to Ec2 Instance create instance name with <b>build server</b>. 
<img width="1182" alt="image" src="https://user-images.githubusercontent.com/63963025/202470596-1db270c1-e1bb-4f78-a39e-8eb5bb6f8819.png">

Refer this document:- https://docs.aws.amazon.com/efs/latest/ug/gs-step-one-create-ec2-resources.html
- Now SSH to that machine and install <b>Docker</b>. 
- Use below command to install <b>Docker</b>.
- This command will update machine. 
```
sudo yum update -y 
```
<img width="1440" alt="image" src="https://user-images.githubusercontent.com/63963025/202471162-d7fd0809-e249-488a-8543-ad5cfbf4c449.png">

- This command will install Docker.
```
sudo yum install docker -y 
```
<img width="1428" alt="image" src="https://user-images.githubusercontent.com/63963025/202471793-d744d33d-5f1b-4bea-b9fe-278ee289ffd0.png">

- Now lets give docker as root privileges sudo power. In y case i am using Amazon linux 2 AMI if you are using different kind of OS you can use that username more information go through this document - https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connection-prereqs.html
```
sudo usermod -aG docker ec2-user
```

- As we have given Docker as a sudo power restart the service using this command given below. 
```
 sudo systemctl restart docker 
```
- Check the status of Docker 
```
 sudo systemctl status docker 
```
<img width="1423" alt="image" src="https://user-images.githubusercontent.com/63963025/202474701-f53a757d-b9c4-4e32-ae7d-10c8d3687b12.png">

- Our service is up and running. 

### Step 2 Create Application and Dockerfile 

- To create Application we have to create directory. 
```
mkdir docker-app
```
<img width="434" alt="image" src="https://user-images.githubusercontent.com/63963025/202475640-df26ee61-eba1-403d-b49a-8dacec2d36cf.png">

- Lets create our simple HTML Application. 
```
sudo vi index.html
```
- Add this content given below.
```
<body bgcolor = 'Aqua'>

   <h1>Hello running from Docker</h1>

</body>
```
<img width="367" alt="image" src="https://user-images.githubusercontent.com/63963025/202477825-7cef9778-1e17-49ca-b8a4-99cb24a36fe9.png">


- Final step create Dockerfile.
```
FROM nginx
COPY index.html /usr/share/nginx/html
```
<img width="345" alt="image" src="https://user-images.githubusercontent.com/63963025/202477709-e25e427a-0e23-4bf4-be87-f48f1b1fa637.png">

### Step 3 Create Docker image 

## What is Docker image? 
A Docker image is a read-only template that contains a set of instructions for creating a container that can run on the Docker platform. It provides a convenient way to package up applications and preconfigured server environments, which you can use for your own private use or share publicly with other Docker users. Docker images are also the starting point for anyone using Docker for the first time.

refer this document:- https://docs.docker.com/engine/reference/commandline/image/

- Let's Build docker image. -t means tag image, (.) this mean current folder location where Dockerfile present. 
```
docker build -t dockerapp:v1 . 
```
<img width="718" alt="image" src="https://user-images.githubusercontent.com/63963025/202479691-b019d5b9-6168-4c8f-8b56-5cf61bb84740.png">

- Check image is been created or not. 
```
docker images
```
<img width="558" alt="image" src="https://user-images.githubusercontent.com/63963025/202479977-3e86ad38-1b94-4890-886d-83095dc72201.png">

### Step 4 Test the image 

- Before upload image in S3 Bucket. check the container is working or not.

- Create a container <b>-d</b> is detach mode where your container will be run in background. <b>-it</b> interactive terminal and we have done here port mapping we have expose our application at 5000 and our docker is running at 80 port number we have map our ec2 port number at 5000. 
```
docker run -d -it -p <host-port>:<container-port> <image-name>:<version-name or tag-name>
```
- example
```
docker run -d -it -p 5000:80 dockerapp:v1 
```
- Docker check running container docker ps.
<img width="1286" alt="image" src="https://user-images.githubusercontent.com/63963025/202483928-8be6bb7d-d3e8-4143-bb19-20d9ffa2bc25.png">

- We have to change in our Ec2 security group. Go to Ec2 machine --> Security --> Security Group change inbound rule 
<img width="1331" alt="image" src="https://user-images.githubusercontent.com/63963025/202485319-ce5b5238-33a9-452f-ae51-f9362162ca39.png">

- Here we go 
<img width="473" alt="image" src="https://user-images.githubusercontent.com/63963025/202485446-0611a98a-c1ed-488d-ae7d-18efa4f30216.png">

- Lets share this image to testing team using S3 bucket 

### Step 5 Create Test server and install Docker

- Create test server. 
<img width="1167" alt="image" src="https://user-images.githubusercontent.com/63963025/202692986-186dd677-e121-42f5-971f-54fd4cfbf521.png">

- Use below command to install <b>Docker</b>.
- This command will update machine. 
```
sudo yum update -y 
```
<img width="1440" alt="image" src="https://user-images.githubusercontent.com/63963025/202471162-d7fd0809-e249-488a-8543-ad5cfbf4c449.png">

- This command will install Docker.
```
sudo yum install docker -y 
```
<img width="1428" alt="image" src="https://user-images.githubusercontent.com/63963025/202471793-d744d33d-5f1b-4bea-b9fe-278ee289ffd0.png">

- Now lets give docker as root privileges sudo power. In y case i am using Amazon linux 2 AMI if you are using different kind of OS you can use that username more information go through this document - https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connection-prereqs.html
```
sudo usermod -aG docker ec2-user
```

- As we have given Docker as a sudo power restart the service using this command given below. 
```
 sudo systemctl restart docker 
```
- Check the status of Docker 
```
 sudo systemctl status docker 
```
<img width="1423" alt="image" src="https://user-images.githubusercontent.com/63963025/202474701-f53a757d-b9c4-4e32-ae7d-10c8d3687b12.png">

- Our service is up and running. 

### Step 6 Create S3 Bucket 

- Create S3 Bucket in ap-south-1 (Mumbai) region.
<img width="1105" alt="image" src="https://user-images.githubusercontent.com/63963025/202693757-c44ebb80-b0e2-49b5-a5f0-71afa7a6ca45.png">

### Step 7 Assign Role for Ec2 machine 

- Go to IAM --> Create Role. 
<img width="1075" alt="image" src="https://user-images.githubusercontent.com/63963025/202694301-d22492d6-c4f3-468b-9b62-6c64018625a8.png">

- Trusted entity type select <b>AWS service</b>.
<img width="825" alt="image" src="https://user-images.githubusercontent.com/63963025/202694507-1328ef88-a202-4e9a-9c5f-b1331fe4374e.png">

- Use case Ec2 select Next.
<img width="911" alt="image" src="https://user-images.githubusercontent.com/63963025/202694558-b3fe845b-0a01-4e49-95f1-4a59ec557f08.png">

- Name Role Ec2-S3-Role create role.
<img width="1021" alt="image" src="https://user-images.githubusercontent.com/63963025/202696606-c67a09c8-4c56-498a-99cc-8361615bc023.png">

- Go to Ec2 vm Action --> Security --> Modify IAM Role --> Attach Role that you have created <b>Ec2-S3-role</b>
<img width="1200" alt="image" src="https://user-images.githubusercontent.com/63963025/202697079-7d1b4d28-9afa-4f53-913b-7aa8d6bd4426.png">

<img width="827" alt="image" src="https://user-images.githubusercontent.com/63963025/202697267-6b87dc73-3a3c-4734-bafd-6668c2101c2e.png">


## Why we have assign IAM Role to Ec2 Machine? 
We want authentication betwenn S3 and Ec2 Instance so we have 2 method also creating user and configure aws profile but <b>this is not best practice</b>. That's why we are using Role to authenticate between resources. In GCP if we want to authentication between resources we use service account there. 

More to know about role:- https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html </br>
More to know about Service account:- https://cloud.google.com/iam/docs/service-accounts

- Same assign the IAM Role to Test server 

### Step 8 Create rar file

- In this process we are going to make rar file of image. Before pushing image to S3 where your image is been store let's have look. 

- Go to this file location in root location. 
```
cd /var/lib/docker 
```

- Your image is been stored in this location <b>overlay2</b> directory.
<img width="1244" alt="image" src="https://user-images.githubusercontent.com/63963025/202836940-1ca50d30-5a9b-4e52-b4f3-bfe9fa097a9c.png">

- Image are store in layer form.

- Lets create package of image. This command will show all the image you are having.
```
docker images
```

- We will create rar file of image. (-o) means output save command will help you to save the image in rar format. 
- Syntax
```
docker save -o <save-your-rar-filename> <docker-image that you have created>:<version-tags> 
```
- Example
```
docker save -o dockerapp.rar dockerapp:v1 
```
<img width="637" alt="image" src="https://user-images.githubusercontent.com/63963025/202837169-e1521376-da83-4501-8b2e-26fba9cd1d36.png">

Refer this document:- https://docs.docker.com/engine/reference/commandline/save/

### Step 9  Push your image to S3 Bucket 

- We upload this rar file in the S3 Bucket. To copy the files from EC2 to S3.
- Syntax
```
aws s3 cp <rar-file or any file/folder> s3://<S3BucketName>
```
- Example
```
aws s3 cp dockerapp.rar s3://docker-s3953
```
- This command will help you to list your bucket object.
```
aws s3 ls s3://docker-s3953
```
<img width="626" alt="image" src="https://user-images.githubusercontent.com/63963025/202837706-df8c66ec-0406-45e4-ab87-6360ce5abcf6.png">

- You can seee here we have successfully uploaded object in bucket. 
<img width="1077" alt="image" src="https://user-images.githubusercontent.com/63963025/202838387-74b362f8-c081-4b9a-984d-34944a5eae09.png">

### Step 10 we will call the object to test-server 
- Go to Test-server check if there is image available or not. 
```
docker images
```
<img width="439" alt="image" src="https://user-images.githubusercontent.com/63963025/202838511-3fd70658-0347-456a-948d-5372a2dbab03.png">

- Lets call object in the test-server. 
- Syntax 
```
aws s3 cp  <s3://BUCKET_NAME/FOLDER/OBJECT> <YOUR PATH WHERE YOU WANT TO SAVE YOUR OBJECT>
```
- Example
```
aws s3 cp  s3://docker-s3953/dockerapp.rar /home/ec2-user
```
<img width="800" alt="image" src="https://user-images.githubusercontent.com/63963025/202838738-927496ca-24ac-4a9f-9008-bdf2bd309c1c.png">

- Image is in layer form. We will load the image. Using this command 
```
docker load -i myapp.rar
```

 <img width="915" alt="image" src="https://user-images.githubusercontent.com/63963025/202838869-85a60b75-a3e0-4952-a9dc-d91d938e9b45.png">
 
- Here we go 
<img width="540" alt="image" src="https://user-images.githubusercontent.com/63963025/202838931-df544422-1173-41f7-bb04-e1fdf3c7257a.png">

### Step 11 run the container for testing 

- Let's test our application is working or not. 
```
docker run -dit -p 8080:80 dockerapp:v1 
```
<img width="1272" alt="image" src="https://user-images.githubusercontent.com/63963025/202839008-04cb9356-ba8c-48be-b3aa-442c5c8b6c5b.png">

- Make sure you have change security group if you are not able to access.
<img width="1329" alt="image" src="https://user-images.githubusercontent.com/63963025/202839063-7830f647-8421-4f68-a62d-335dd6f82f72.png">

- Here we are
<img width="425" alt="image" src="https://user-images.githubusercontent.com/63963025/202839075-f9d84a0b-44f1-4dae-bd62-42b81f59b70f.png">

## What is Dangling images? 
Dangling images are untagged Docker images that aren't used by a container or depended on by a descendant. They usually serve no purpose but still consume disk space.

Refer this document:- https://docs.docker.com/engine/reference/commandline/image_prune/
