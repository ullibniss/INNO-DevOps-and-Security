#  D&S Lab 2: Infrastructure as Code with clouds (alternate lab flavor)

## Completed by Fedorov Alexey (tg: @ullibniss)

---

# Task 1 - IaC Theory

### **Ansible Directory:**
- **`ansible.cfg`**: The main configuration file for Ansible, defining settings like inventory location, logging, SSH settings, etc.  
- **`inventory` folder**: Stores inventory files defining target hosts and their groups.  
- **`roles` folder**: Contains reusable Ansible roles, organizing tasks, handlers, variables, templates, and defaults.  
  - **`tasks`**: Defines the sequence of actions executed in a playbook or role.  
  - **`defaults/group_vars`**: Stores default and group-specific variables used in playbooks.  
  - **`handlers`**: Contains actions that trigger on specific conditions .  
  - **`templates`**: Stores Jinja2 templates for configuration files dynamically rendered during execution.  
  - **`vars`**: Holds additional variable definitions for use in playbooks and roles.  
  - **`playbooks` folder**: Contains Ansible playbooks, which define automation workflows.  
  - **`meta` folder**: Holds metadata about a role, such as dependencies and author information.  

---

### **Terraform Directory:**
- **`main.tf`**: The primary configuration file defining Terraform resources and infrastructure.  
- **`variables.tf`**: Declares input variables to make Terraform configurations more flexible.
- **`output.tf`**: A Terraform configuration file that defines output values to display useful information after running terraform apply.

# Task 2 - Prepare your application

I decided to use my self-made balancing proxy with an example. The example demonstrates the functionality of a sidecar balancer and consists of five different applications:  

- **Master balancer**  
- **Sidecar balancer**  
- **Coffee service**  
- **Drink service**  
- **Milk service**  
- **Syrup service**  

The main idea is that the **Coffee service** allows users to order a random coffee, which consists of a type of coffee (**Drink service**), a type of milk (**Milk service**), and a flavor of syrup (**Syrup service**).  

Each of these three services has multiple replicas with different types, and the balancer helps the **Coffee service** select a random type for each by balancing requests.  

The architecture is:

![image](https://github.com/user-attachments/assets/0a76a93d-c62e-473d-a2b2-15b2b85b66cc)


**Language** - Java. Joke, i hate Java. **Golang**.

**Github** - https://github.com/ullibniss/INNO-Sevice-Mash

# Task 3 - Dockerize your application

I dockerized all the applications. I added Dockerfile and docker-compose.yml file. 

### Dockerfile

I dockerfile is multistaged. It is common practice for compiled languages like Golang. 

- 1st stage - builder
- 2nd stage - runner

Let's take a look of Dockerfile.

![image](https://github.com/user-attachments/assets/71ebf0a9-4dbc-403b-a530-6f890ae5f6a2)

It is the same for every service, there no reason to show all of them.

### Docker Compose

I also prepared docker-compose.yml.

For Drink service (Milk, Syrup are the same):

![image](https://github.com/user-attachments/assets/4fbef0dd-1480-4e8b-9f36-94a1bf9ef249)

For Coffe and Sidecar service

![image](https://github.com/user-attachments/assets/79baf4be-7e92-496e-af0b-14ef4f4434b6)

For Master LoadBalancer:

![image](https://github.com/user-attachments/assets/c620c229-1d3b-47a9-9ab9-0b1ab4ddec47)

This configuration is to run system on localhost. I will reconfigure it when I begin to implement cloud deploy.

# Task 4 - Deliver your application using Software Configuration Management

## 4.1 Get your personal cloud account.

```
Free tiers for a AWS, GCP and some other popular cloud providers doesn't worked in Russia more.
If you already have a such account, it may work and be enough for this lab.
If not, try other cloud providers with a free subscription, i.g.
Yandex.Cloud. If you will not be able to work with cloud, you have to proceed within
the local deployment for the whole Task 4. Thus, think about a local virtual machine for
the further tasks. Include a small explanation into the report why you were forced to work
locally.
```

I will use Timeweb. This my favourite Russian cloud. It have a fex cloud featurer, but it is cheap and has terraform provider.

## 4.2 Use Terraform to deploy your required cloud instance.

```
Please notice that to run terraform
init command you have to use VPN. Look for the best Terraform practices and try to follow
them. If for a some reason you will not able to use any cloud service, prepare a local VM
using Vargant tool. Include the explanation into the report about the inability to work with
cloud or if it was impossible for you to setup a VPN.
```

**Github**: https://github.com/ullibniss/INNO-Sevice-Mash/tree/master/terraform

### Preparation

Let's implement infrastructure with `terraform`. First we need to initilize provider and configure backend. I will hide all sensitive data.

`provider.tf`

![image](https://github.com/user-attachments/assets/c31a8a60-7927-4843-b316-987d43d8ee13)

I will 2 use providers:

- **Timeweb provider** - cloud provider.
- **Random provider** - provider to randomly generate passwords for instances.

`backend.tf`

![image](https://github.com/user-attachments/assets/38e82783-7fa8-47fe-a401-582c0b88f2c8)

There is no point in storing `terraform` manifests for cloud infrastructure on github without tf.state, if tf.state storing nowhere, except the localhost. It is very bad practice and can cause dizasters for infrastruction. I will store it in timeweb s3  service. In real case better backup this s3 bucket.

### Inplementation

I will build hardware infrastructure in private network. There will be only two public server - our Coffee service.

Let's implement it in separated files:

- `compute.tf`
- `network.tf`
- `firewall.tf`
- `variables.tf`
- `output.tf`

Let's start with `network.tf`:

![image](https://github.com/user-attachments/assets/87dbb6f4-1177-4b93-8f99-a853fffbaec3)

I prepared cloud VPC to use it in `compute.tf` to connect server to this network.

I also generated passwords and stored them in locals in `variables.tf`.

![image](https://github.com/user-attachments/assets/38160f95-88a5-4e78-975e-fe92f83f4343)

Next step is to create `output.tf` for generated password. We will have ability to see them.

![image](https://github.com/user-attachments/assets/c9acb941-e444-47a5-8d41-135727a57819)

After all preparations are ready we can create `compute.tf`.

![image](https://github.com/user-attachments/assets/3b80f282-f221-4ef0-a436-3d56055b9770)

Here is some different instances:

- `twc_presets` - server resources confgiuration. On **Timeweb** cloud we can create server with preset (like `flavor` in **Openstack**) or with our resource configuration. With preset is cheaper.
- `twc_os` - server's os preset.
- `twc_server` - is cloud vpc.
- `twc_floating_ip` - public ip address. it is floating, because if we will recreate it, the ip address will change randomly.

I created 5 servers: master-lb-server, coffee-server, drink-server, milk-server and syrup-server. All servers are the same, except the coffee-server. It has it's own public ip.  

Moreover every service has provision via cloud-init. There 2 cloud init files for coffee and others servers.

`coffee.yaml`

![image](https://github.com/user-attachments/assets/9bbffcbe-24b8-41a2-8a4d-bfb6d507453c)

`others.yaml`

![image](https://github.com/user-attachments/assets/2e05388c-d82b-4daa-87e8-9fea3e0e858b)

I need to say, that ssh conenction to any host provided through coffee server. Use can see how I configured firewalld for forwarding.

The last thing is WAF. I configured it in `firewalld.tf`. I configured it only for `coffee-server`, but to be honest it is nessesary for every host in private network. I decided that for lab, `coffee-server` is enough.

![image](https://github.com/user-attachments/assets/8f302ba7-4420-4baf-9409-b9a7b8bebff1)

You can see filewall and rules instances. Rules provide access for:

- 443 - HTTTPS
- 1900 - SSH to master-lb-server
- 1901 - SSH to coffee-server
- 1902 - SSH to drink service
- 1903 - SSH to milk service
- 1904 - SSH to syrup service

### Deploy

Let's deploy our hardware infrastructure.

Firstly I need to init terraform. 

```
terraform init
```

![image](https://github.com/user-attachments/assets/85113055-756c-4ccf-a0b3-fb15e99aa41e)

Next let's plan infrastructure and verify that everything coffect.  

```
terraform plan
```

![image](https://github.com/user-attachments/assets/c49740ee-25f2-4d46-a729-bd42d8637d37)

Now we can verify that configuration is worikng and terraform will create exactly what we need.

Next step is to apply our configuration. 

```
terraform apply
```

![image](https://github.com/user-attachments/assets/488d37f9-7f20-4e9c-a47c-275cf14096f2)

Cloud rejected my `leet` network `192.133.7.0/24`, so I have to change it to `192.168.7.0/24`.

Try again and everyting deployed!

![image](https://github.com/user-attachments/assets/d6b1d335-ec30-48a4-9015-f50bba59f03e)

We can proof it in web interface.

![image](https://github.com/user-attachments/assets/0e899dcb-9f05-4bf8-8307-afa56e8c4bfc)

![image](https://github.com/user-attachments/assets/803d3e1b-4791-405c-9137-512dd2441aa5)

Let's try to connect to master load balance to proof, that forwarding works correct.

![image](https://github.com/user-attachments/assets/74062a16-8ba0-48ce-8848-8fd0ca3a56b4)

Everything works. Deploy if hardware infrastructure finished!

## 4.3. Choose Software Configuration Management (SCM) tool.

```
Ansible is the industry standard. Of course, we have other
SCM solutions such as SaltStack, Puppet, Chef. You can try them
but remember that it is probably more difficult to work with these
tools and you are responsible for your choice.
```

I decided to use Ansible. I also have good experience with SaltStack, but I think it is too hard (complex) for lab.

## 4.4 Using SCM tool, write a playbook tasks to deliver and run your application to cloud instance/local VM.

```
Try to separate your configuration files into inventory/roles/playbooks files.
In real practice, we almost newer use poor playbooks where everything all in one.
```

**Github**: https://github.com/ullibniss/INNO-Sevice-Mash/tree/master/ansible  

### Preparation

I will use the following folder structure for `ansible`:

![image](https://github.com/user-attachments/assets/360156b1-12f9-4435-9bc6-9a53e04c8698)

I will show implementation of `ansible` roles and playbooks only for coffee service, because others are almost the same. You can find them on github.

### Inplementation

First of all we need to implement `role` for servicce. Let's start with tasks, then I will add variables for parametrization in they needed.

Role will have 5 tasks. 1 of them is auxiliary and 4 are for usage:

- `main.yml` - this is task that is responsible to proxy tasks call to concrete task inside role. I will show it further.
- `build.yml` - this is task for building our server. Actually building docker image.
- `deploy.yml` - this is task to deploy service. It will render all the files to build and run service.
- `start.yml` - this is task to start service.
- `stop.yml` - this is task to stop service.

Let's start implementation:

`main.yml`

![image](https://github.com/user-attachments/assets/57f3ca22-1bba-4161-8a1c-0418232d25c1)

As we can see, when I will implement playbook, I will use variable `coffee_task` to call concrete task.

`build.yml`

![image](https://github.com/user-attachments/assets/06abdaa3-bffc-4f4e-91dc-8acd0505ec4b)

Here we can see default `docker build` command. There are 2 build - `coffee` and `sidecar`, because `sidecar` used in pair with coffee. To be honest, it is too small, to implement separate role. We also can see first two real variables here. I using `coffee_tag`, because use `latest` is bad practice. But default value will be `latest` :).

`deploy.yml`

![image](https://github.com/user-attachments/assets/9ce95a6d-18e0-4bd8-80d7-8dc3b8cb8437)

The most interesting task in my opinion is `deploy.yml`. I clone repo of service to for build. If there were `docker registry`, i would build it on local or separate machine. But in this case, I will build on target server. 

`start.yml`

![image](https://github.com/user-attachments/assets/026fe295-11cc-4d61-bf32-707e4d1ac200)

`stop.yml`

![image](https://github.com/user-attachments/assets/6850f2aa-22ae-40a1-b8ea-d296611b5324)

There are simple start and stop `docker-compose` tasks. Nothing interesting.

When tasks are ready, I need to implement `defaults/main.yml` with default variables.

![image](https://github.com/user-attachments/assets/07a9d3a6-0477-4c57-963c-1fb607039a43)

You might have noticed that there no variables `default_user` and `deploy_dir`. This variables have no default parameter and must be defined separately. This is because they we dont know in what context our role will be used. And this variable may be general, if there more that one service on server.

Last thing is to modify `docker-compose.yml` as I promised earlier.

![image](https://github.com/user-attachments/assets/118c013d-27af-45e7-8b8d-4c0ca0e58185)

Key changes:

- Image tag is also templated.
- Still we have separate servers, we dont need external network. Because of this, I will create bridge between `coffee` and `sidecar-lb` inside docker-compose.
- `MASTER_URL` variable changed with `master load balances`'s IP.

Okay, when we finished role implementation, we can implement playbooks and inventories.

There will be 4 playbooks. They are the same as the role tasks, except the `main` task. All playbooks are the same. Only thing that changes is `coffee_task` variable.

`deploy.yml`

![image](https://github.com/user-attachments/assets/68e85dfd-581f-4bc3-b6ff-3eb14502685b)

`build.yml`

![image](https://github.com/user-attachments/assets/7e9d9869-67ff-47a7-a389-540c7f2c37ac)

To make our role available for playbooks, I will configure `ansible.cfg` file.

![image](https://github.com/user-attachments/assets/ee5ffbff-35d0-4db2-a865-9d4f9b76cafe)

Settings:

- `host_key_checking = False` - apply `-o HostKeyChecking=no` flag for `ssh` connection
- `roles_path=../roles` - is like PATH variable, but for roles
- `inventory=localhost.cfg` - configures default inventory file.

Now we can implement inventory file. There are inventories: `localhost.cfg` and `remote.cfg`

`localhost.cfg`

![image](https://github.com/user-attachments/assets/858c3bd2-9e66-4cec-a0d4-226efda8ba0f)

`remote.cfg`

![image](https://github.com/user-attachments/assets/a7b6b0a8-b825-4ec1-8062-2635976f589c)

I also implemented roles and playbooks for other services, you can check it on **[Github](https://github.com/ullibniss/INNO-Sevice-Mash/tree/master/ansible)**

### Deployment


