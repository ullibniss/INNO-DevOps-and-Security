![image](https://github.com/user-attachments/assets/7732d3ae-9b24-4d4a-8215-6cb35d63a557)# Lab 1. Containerization and application layer load balancing

## Done by Fedorov Alexey

# Task 1. Get familiar with Docker Engine

## 1. Pull Nginx v1.23.3 image from dockerhub registry and confirm it is listed in local images

To pull image, i used command `docker pull`.

![image](https://github.com/user-attachments/assets/c140ec1d-bb58-45ec-9820-447a906cc162)

To list images, i used command `docker images (docker image ls)`.

![image](https://github.com/user-attachments/assets/c696dd48-a7c2-4f25-af17-44ec4a1e329d)

Image pulled correctly.

## 2. Run the pulled Nginx as a container with the below properties

a. Map the port to 8080.
b. Name the container as nginx-<stX>.
c. Run it as daemon .

To follow the lists of requirements, I will use the following flags:

- `-p` - sets port mapping
- `-n` - sets container name
- `-d` - runs container in background (as a daemon)

Command is:

`docker run -p 80:8080 --name nginx-fedorov-alexey -d nginx:1.23.3`

Container started successfully.

****![image](https://github.com/user-attachments/assets/6c0c8711-7949-405b-940d-1fb001aaa929)

## 3. Confirm port mapping

### a. List open ports in host machine.

I used `ss` command to list ports 

![image](https://github.com/user-attachments/assets/60222256-da51-4d22-b315-c4abb3079b98)

Port is present on host machine.

### b. List open ports inside the running container.

To list ports inside container, I need to install `netstat` or `ss` inside container. Lets do it.

I attached to container shell using exec with interactive mode.

![image](https://github.com/user-attachments/assets/f1b6886a-07a7-4722-b888-583b4470faf9)

Installed iproute2 package, that contains `ss` utility.

![image](https://github.com/user-attachments/assets/fd29dd6b-5618-4dd9-b522-45b1915af573)

Now we can see that port allocated.

![image](https://github.com/user-attachments/assets/29b60d96-d993-4e82-83a3-f95b25483f81)

### c. Access the page from your browser.

Going to `127.0.0.1:8080` in my browser.

![image](https://github.com/user-attachments/assets/991f026f-ea78-4ef6-a8c4-971ec8bc838a)

Everything works correct.

## 4. Create a Dockerfile similar to the below properties (let’s call it container A).


