# Lab 1. Containerization and application layer load balancing

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

### a. Image tag should be Nginx v1.23.3.

I created Dockerfile and used nginx 1.23.3.

![image](https://github.com/user-attachments/assets/ee04ab18-66d0-4a77-a3aa-aece64095631)

### b. Create a custom index.html file and copy it to your docker image to replace the Nginx default webpage

My custom index.html.

![image](https://github.com/user-attachments/assets/60c07579-c6c1-4368-95da-9b32213fd7d4)

You can see how I replace index.html in punkt `a`.

### c. Build the image from the Dockerfile, tag it during build as nginx:<stX>, check/validate local images, and run your custom made docker image.

To build docker image, I use `docker build`.

![image](https://github.com/user-attachments/assets/3c909140-7ea3-4101-b833-e3db78d4270d)

Build is successfull.

### d. Access via browser and validate that your custom page is hosted.

Started container.

![image](https://github.com/user-attachments/assets/c56e3828-6693-45f4-bf15-206c23c2e71a)

Let's go to the browser and check the result.

![image](https://github.com/user-attachments/assets/2695fc52-cad7-4ff9-82be-eb31fce31898)

Everything works correct.

# Task 2: Work with multi-container environment

## 1. Create another Dockerfile similar to step 1.4 (Let’s call it container B), and an index.html with different content.

Let's change index.html content

![image](https://github.com/user-attachments/assets/54677769-8760-4ba4-a521-f0e8bb848ee4)

I changed text color from pink to purple.

Build:

![image](https://github.com/user-attachments/assets/1fd792f1-f7fb-4633-86e5-3fc757382e42)

## 2. Write a docker-compose file with the below properties

Let's write docker-compose.yml file

### a. Multi-build: Builds both Dockerfiles and runs both images.

I need to split Dockerfiles and index.html's 

![image](https://github.com/user-attachments/assets/61986d20-f11d-4203-b1b6-bf03e2120648)

Start writing docker-compose.yml

![image](https://github.com/user-attachments/assets/98891680-4aa7-46ce-a298-e2fa2821a950)

Section `build` is responsible for image building. `context` is directory which will be build context.

### b. Port mapping: Container A should listen to port 8080 and container B should listen to port 9090. (They host two different web pages)

Lets add ports mapping to docker-compose.

![image](https://github.com/user-attachments/assets/04e098a1-1212-486a-a8f8-3eb43894919f)

Section `ports` is responsible for port mapping. It contains list of port mapping. Each mapping has host port (left side) and container port (right side).

### c. Confirm both websites are accessible

Run docker-compose.yml !

`docker compose up -d --build`

![image](https://github.com/user-attachments/assets/98d50ebc-d4c3-406d-ac23-4c197da7c5dc)

Let;s check - how it works. 8080 port.

![image](https://github.com/user-attachments/assets/7d0cb62e-c523-4733-a6af-207f46884b46)

9090 port.

![image](https://github.com/user-attachments/assets/d12b1aac-b04e-40d2-bf71-64c1452fd072)

Everything works corrent.

### d. Volumes: Mount (bind) a directory from the host file system to Nginx containers and update the contents of index.html in the host file system, re-deploy and confirm in the browser that the web page's content is updated.

I created `index.html` is separate directory.

![image](https://github.com/user-attachments/assets/3aab6f7d-bd64-4e7f-b421-7d49b5735dc5)

Then added volumes section to docker compose

![image](https://github.com/user-attachments/assets/d940480c-948c-43d5-9c26-873a268299ab)

There is `ro` flag at the end of volume path. It mean that mount has read only permissions.

Let's see how it works.

`docker compose up -d --build --force-recreate`

![image](https://github.com/user-attachments/assets/298b7374-469f-43bd-9951-f4e74d2708a7)

I changed color of text to red.

![image](https://github.com/user-attachments/assets/511eb6dc-7aac-447e-a715-e988039450ac)

The result is:

![image](https://github.com/user-attachments/assets/14566f82-ccf0-48a8-9398-cb27035e1bf0)

I change color to blue.

![image](https://github.com/user-attachments/assets/9c65311c-d7ba-4a61-a81d-bbea8fef79be)

The result is:

![image](https://github.com/user-attachments/assets/a97e069d-88c4-4798-b387-700c0d3f2bce)

Everyting works correct!

# Task 3. Configure L7 Loadbalaner

## 1. Install Nginx in the host machine or add a third container in the docker-compose that will act as loadbalancer, and configure it in front of two containers in a manner that it should distribute the load in a Weighted Round Robin approach.

I decided to make third container. It will be standart nginx image. Let's see modified docker-compose.yml

![image](https://github.com/user-attachments/assets/71f55ace-8939-4b0c-8c36-fa8e2ddb981a)

Crucial changes:
1) Third contrainer nginx with volume to nginx.conf
2) No port forwarding on pink & purple containers.
3) Network `nginxnet`

Pink & purple don't need port forwarding, because we will connect to them through the balancer. Balancer can connect to them because they are in same network, moreover there is service discovery in docker compose (in file name space).

I also prepared nginx configuration:

![image](https://github.com/user-attachments/assets/f19d25e1-f370-4331-b0a7-874c11cdf435)

Server will accept requests on port 80 and forward to upstream. Upstream contains 2 adresses - ping and purple.

Let's execute:

![image](https://github.com/user-attachments/assets/e5414b7c-289f-4a1f-bdf8-a0afbca6e96e)

## 2. Access the page of Nginx ALB and validate, it is load-balancing the traffic (you see two different content per page reload).

To show how it works, I decided to use `curl`.

![image](https://github.com/user-attachments/assets/d8f76ddf-a723-47a7-b046-6d5227c28c2f)

Everything works correct!

# References

1. `docker --help`
2. https://docs.docker.com/compose/how-tos/networking/
3. https://docs.docker.com/reference/compose-file/build/
4. https://www.dmosk.ru/miniinstruktions.php?mini=nginx-balancing
