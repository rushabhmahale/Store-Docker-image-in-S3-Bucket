# Store-Docker-image-in-S3-Bucket

## What is Docker? 
Docker is an open platform for developing, shipping, and running applications. Docker enables you to separate your applications from your infrastructure so you can deliver software quickly. With Docker, you can manage your infrastructure in the same ways you manage your applications. By taking advantage of Docker’s methodologies for shipping, testing, and deploying code quickly, you can significantly reduce the delay between writing code and running it in production.

Refer this document:- https://docs.docker.com/get-started/overview/

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
- Docker check running container docker ps 
<img width="1286" alt="image" src="https://user-images.githubusercontent.com/63963025/202483928-8be6bb7d-d3e8-4143-bb19-20d9ffa2bc25.png">

- We have to change in our Ec2 security group. Go to Ec2 machine --> Security --> Security Group change inbound rule 
<img width="1331" alt="image" src="https://user-images.githubusercontent.com/63963025/202485319-ce5b5238-33a9-452f-ae51-f9362162ca39.png">

- Here we go 
<img width="473" alt="image" src="https://user-images.githubusercontent.com/63963025/202485446-0611a98a-c1ed-488d-ae7d-18efa4f30216.png">

- Lets share this image to testing team using S3 bucket 

### Step 5 
