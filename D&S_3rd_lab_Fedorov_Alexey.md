# D&S Lab 3: CI/CD Infrastructure

## Completed by Fedorov Alexey (tg: @ullibniss)

---

```
In this lab, we will focus on how to set up CI/CD infrastructure by deploying Gitlab on-premise:

- Self-managed Gitlab server
- Gitlab Runner
- Deployment server
- CI/CD pipeline
- Web application with test case
- Managing disaster scenarios
```

# Task 1 - Infra Deployment

## 1.1.​ Deploy three VMs that you will be using as Gitlab Server, Gitlab Runner, and the deployment server. Make sure VMs can reach each other.

I deployed 3 VMs and also configured the network.

![image](https://github.com/user-attachments/assets/0de08d05-b369-4827-b10c-561e1798fc28)

Let's check connectivity. 

![image](https://github.com/user-attachments/assets/74c829fc-ed9a-46e7-8f7e-4b26c7e2aaf0)

![image](https://github.com/user-attachments/assets/650d63cd-9476-4637-9ff7-2050d4fffc91)

![image](https://github.com/user-attachments/assets/02b74703-2e8b-47d5-9daf-b28bf3505ba6)

We can connect between hosts!

## 1.2 Set up Gitlab Server (VM1), and create a docker-compose file with the below configs:

```
a.​ Pull the Gitlab EE or CE edition
b.​ Name the running container as <stx>-gitlab
c.​ Map container ports 80 and 22 to host machine
d.​ Expose the Gitlab server as <stx>.sne.com. (Hint: find the right env variable to pass to the container to
update the configs of Gitlab. Update the hosts file to resolve the mentioned DNS record)
e.​ Bind the necessary directories of the Gitlab server container to the host machine (e.g. logs, app data,
configs…)
f.​ Run the docker-compose file and make sure the configs are working.
g.​ Access the Gitlab server and log in, create a project name it as <stx>-repo.
h.​ etc…
```

### 1.2.a Pull the Gitlab EE or CE edition

Let's pull the `GitLab CE` image. I will use the fixed version `17.7.6-ce.0`. This is important because if I use `latest`, it could update suddenly and introduce disruptive changes.

```
docker pull gitlab/gitlab-ce:17.7.6-ce.0
```

![image](https://github.com/user-attachments/assets/316b3dba-c46f-4cd9-a3c9-81438d8d2dc3)

### 1.2.{b,c,d,e} Name the running container as <stx>-gitlab. Map container ports 80 and 22 to host machine. Expose the Gitlab server as <stx>.sne.com. (Hint: find the right env variable to pass to the container to update the configs of Gitlab. Update the hosts file to resolve the mentioned DNS record). Bind the necessary directories of the Gitlab server container to the host machine (e.g. logs, app data, configs…)

Here I implemented the `docker-compose.yml`.

![image](https://github.com/user-attachments/assets/0b1d1505-6351-4f21-a10e-dca3710a16ae)

There is nothing to describe about the `b` and `c` tasks. The GitLab environment is configured through the `GITLAB_OMNIBUS_CONFIG` variable. Moreover, we can configure any option from `gitlab.rb` inside this variable.

I also created volumes for GitLab data persistence:

- `/opt/gitlab/config:/etc/gitlab` - GitLab configuration directory
- `/opt/gitlab/logs:/var/log/gitlab` - GitLab logs directory
- `/opt/gitlab/data:/var/opt/gitlab` - GitLab data directory.

### 1.2.f Run the docker-compose file and make sure the configs are working.

Let's run docker compose.

![image](https://github.com/user-attachments/assets/680d46f7-d51e-4ab2-a691-825d2925d7a5)

I need to mention that binding port `22` for the container works because I reconfigured `sshd` on the host to use `Port 2222`.

![image](https://github.com/user-attachments/assets/73554395-40d1-42c5-b568-e9c633a52ea6)

We can see that the container is running. GitLab needs some time to start.

### 1.2.g Access the Gitlab server and log in, create a project name it as <stx>-repo

Now I can access GitLab through the browser.

![image](https://github.com/user-attachments/assets/6de1974d-30b3-4a81-a6ef-6d9f8f29ae29)

I need to specify the port because I am using a NAT adapter on the VM, and port 80 is forwarded to port 8080 on the host. Unfortunately, VirtualBox doesn't allow forwarding port 80.

To access the root user, I need to set my password, which I can do with `gitlab-rake` inside the container.

![image](https://github.com/user-attachments/assets/ececab52-8f0c-4648-b316-c85c12d0c67d)

Let's create repository.

![image](https://github.com/user-attachments/assets/2f29ac94-2e87-498d-ab0d-4aa1d5f8cbd4)

As a result, we have created the repository.

![image](https://github.com/user-attachments/assets/3dd218f3-a36f-44e2-9107-a9ffdea722a3)

## 1.3 Set up the Gitlab Runner (VM2), don’t use the docker approach this time.

### 1.3.a​ Install and configure shared Gitlab Runner.

First, I need to add the GitLab Runner repository.

```
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | sudo bash
```

![image](https://github.com/user-attachments/assets/232a913d-077f-4a12-9429-383a166abc7e)

Now I can install `gitlab-runner`.

![image](https://github.com/user-attachments/assets/7b6ce916-499e-4e7a-be69-88618dd52d88)

Let's start the `gitlab-runner` and enable it in `systemd`.

![image](https://github.com/user-attachments/assets/a7c16d5f-9f08-489e-b66e-8e4d6006ee7c)


### 1.3.b Explain what is the Gitlab runner executor and set the executor type to shell

The executor is the mechanism used by the GitLab Runner to run jobs. It determines the environment in which the jobs are executed. Common executor types include:

- **shell**: Runs jobs directly on the host machine.
- **docker**: Runs jobs in Docker containers.
- **kubernetes**: Runs jobs in Kubernetes pods.

When registering the GitLab Runner, I will be prompted to choose an executor. I will select `shell`.

### 1.3.{c,d} Set Gitlab Runner tag to <stx>-runner. (You will be using this tag in the pipeline in the coming task). Authenticate your Gitlab runner with Gitlab server, and validate.

Let's register the GitLab Runner with the GitLab server.

![image](https://github.com/user-attachments/assets/b2c34e16-29c7-499b-8978-90275978aef8)

As you can see, I specified the tag `st17-runner` and the executor `shell`.

Let's verify that the runner is registered.

![image](https://github.com/user-attachments/assets/34c3726d-fd69-47db-98fc-f47eb8f65b33)

The runner is registered! I will verify that it works in the next tasks.

## 1.4 Set up the Deployment Server (VM3).

### 1.4.a Set up authentication of your Gitlab runner to be able to deploy to the deployment server.

To authenticate the GitLab Runner on VM3, we need to create an SSH key. Let's do it.

```
ssh-keygen
```

![image](https://github.com/user-attachments/assets/980fa84a-a52b-45c9-9487-ffd731f5cf60)

Then, I will add the `gitlab_runner.pub` public key to the authorized keys on VM3.

![image](https://github.com/user-attachments/assets/d0a63410-2ea2-4596-a08a-14eb2e644017)

You can see two keys. The first is my own key, which allows me to connect to the VM without a password.

To validate the authentication, we need to prove that SSH from VM2 to VM3 works without a password.

![image](https://github.com/user-attachments/assets/33b68cfe-3902-4780-a038-f2e251b821e5)

It works! We can use SSH in the same way in jobs.

# Task 2 - Create CI/CD Pipeline

## 2.1 Clone the project you have created in step 1.2.g.

Let's clone it in Lab 2 directory.

![image](https://github.com/user-attachments/assets/fc14708f-4919-4c43-97fa-32826dad37a2)

## 2.2 Write a simple web application in any programming language. (E.g. Random text or Addition of two numbers)

I implemented the `A+B` Golang application.

It works like this:

![image](https://github.com/user-attachments/assets/241bae71-69a7-4789-84b1-2cdb55c9d96b)

I also implemented tests for this application:

![image](https://github.com/user-attachments/assets/57f3ec40-cd3e-4ee5-879a-d116920d65d6)

![image](https://github.com/user-attachments/assets/fc4c5a21-ef8b-46af-9e97-fcb6da63b353)

## 2.3 Create CI/CD pipeline (.gitlab-ci.yml)

Let's implement the CI/CD pipeline.

We need to start with the stages.

![image](https://github.com/user-attachments/assets/e9271f33-60fe-4193-9023-2c6f16c59eb3)

Here is all five stages:

- [CI] Build
- [CI] Test
- [CI] Docker build
- [CI] Docker push
- [CD] Deploy

### 2.3.a CI stages of the pipeline should.

i.​ Build the application

![image](https://github.com/user-attachments/assets/ce68bb00-d7c9-4bd3-b8b9-aaea9c75f773)

ii.​ Run test (to check the application works ok)

![image](https://github.com/user-attachments/assets/ee9098c1-7637-49d2-875c-8d5bb7f30208)

iii.​ Build docker image (Note: you need Dockerfile)

![image](https://github.com/user-attachments/assets/53cb6017-adee-4aef-86d9-445bbb63e677)

iv.​ Push to your docker hub account.

![image](https://github.com/user-attachments/assets/d18a0592-2972-42b0-a73a-3cd17d420816)

### 2.3.b CD stages of the pipeline should:

i.​ Pull the docker image and deploy it on the deployment server

![image](https://github.com/user-attachments/assets/230061e6-1a5e-4a9e-94cd-bec938dac940)

## 2.4.​ Validate that the deployment is successful by accessing the web app via the browser on deployment server side.

We can see that the entire pipeline is successful.

![image](https://github.com/user-attachments/assets/36520e38-c1b1-42e6-aa85-7a78d29fa717)

Let's verify that our application works. I forwarded port 8080 from the VM to port 9001 on the host.

![image](https://github.com/user-attachments/assets/79dc00fe-3623-4c2c-9a2f-9c32e0fb5343)

![image](https://github.com/user-attachments/assets/61125d80-64cd-4b72-afee-cd6883534622)

Everything works correct!

# Task 3 - Polish the CICD

## 3.5 Update the CD stages to be able to deploy the web application using Ansible.

First of all, I implemented Ansible roles and playbooks for the application. Let's take a look at the key tasks:

`deploy.yml`

![image](https://github.com/user-attachments/assets/1d140ac7-f80e-470b-b05b-baaf70f0b98c)

`start.yml`

![image](https://github.com/user-attachments/assets/0b71c003-cbaf-4463-b762-018121f23fcc)

And modified the job.

![image](https://github.com/user-attachments/assets/8cb6e00f-f2e3-4d15-82ad-3440495ea989)

Let's check, how it works.

![image](https://github.com/user-attachments/assets/9060e9a0-35e6-46be-923e-64cf49b74f8c)

As a result, I got the deployment with Ansible.

## 3.6 Update the pipeline to support multi-branch (e.g. master and develop) and jobs should be triggered based on the specific target branch.

There is the `rules` option in GitLab CI/CD configuration that allows us to run the pipeline conditionally, depending on triggers. I've already configured this option in my jobs, but only for the `master` branch. Let's modify it.

I will execute CI jobs for both the `develop` and `master` branches, and CD jobs only on the `master` branch.
CI jobs:

![image](https://github.com/user-attachments/assets/a1167653-b2fa-47b3-a2d4-f41b8f63cae0)

CD job:

![image](https://github.com/user-attachments/assets/e5d10e37-8077-47fc-92ee-bcee2eb665ac)

The next step is to check how it works. Let's compare the two pipelines.

![image](https://github.com/user-attachments/assets/f26fa1e1-1280-44ad-bbea-3ca953dea567)

As we can see, the `develop` pipeline doesn't have the last job.

## 3.7 Update keywords such as cache, artifact, needs, and dependencies to have more control of pipeline execution.

To update the keywords, here's what they do:

- **cache** - Stores and reuses files between pipeline runs to speed up jobs (e.g., dependency directories).
- **artifact** - Saves files generated in a job for use in later jobs or for download (e.g., build outputs).
- **needs** - Allows jobs to run out of order by defining direct dependencies between jobs.
- **dependencies** - Specifies which artifacts from upstream jobs are needed for the current job.

Unfortunately, I don't need keywords like `artifact` and `dependencies` in my pipeline. The reason is that the only artifact in my pipeline is the `docker image`, which is stored in external storage and downloaded with Docker.

The `cache` keyword will only be used in the build task.

![image](https://github.com/user-attachments/assets/7637ea3e-49b4-43d3-88b3-f7f48b207415)

It should improve job speed.

As for `needs`, I will insert it into every job to specify the previous job.

![image](https://github.com/user-attachments/assets/1028d864-91b2-4d2d-b543-88322bd78412)

Let's check if the updated jobs work.

![image](https://github.com/user-attachments/assets/a9e01acc-7802-4374-8447-501d7844b4d7)

Everything works correct!

# References

- https://docs.gitlab.com/install/docker/installation/
- https://docs.gitlab.com/runner/configuration/
- https://docs.gitlab.com/ci/
- https://docs.gitlab.com/ci/yaml/
