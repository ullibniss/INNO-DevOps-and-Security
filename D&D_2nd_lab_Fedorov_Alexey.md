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

### Infrastucture

I will build hardware infrastructure in private network. There will be only two public server - our Coffee service.

Let's implement it in separated files:

- `compute.tf`
- `network.tf`
- `firewall.tf`
- `variables.tf`
- `output.tf`


