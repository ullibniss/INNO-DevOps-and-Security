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

I dockerized all the applications. I added a Dockerfile and a `docker-compose.yml` file.

### Dockerfile

My Dockerfile is multi-staged. It is a common practice for compiled languages like Golang.

- 1st stage - builder
- 2nd stage - runner

Let's take a look at the Dockerfile.

![image](https://github.com/user-attachments/assets/71ebf0a9-4dbc-403b-a530-6f890ae5f6a2)

It is the same for every service; there is no reason to show all of them.

### Docker Compose

I also prepared a `docker-compose.yml`.

For Drink service (Milk, Syrup are the same):

![image](https://github.com/user-attachments/assets/4fbef0dd-1480-4e8b-9f36-94a1bf9ef249)

For Coffe and Sidecar service

![image](https://github.com/user-attachments/assets/79baf4be-7e92-496e-af0b-14ef4f4434b6)

For Master LoadBalancer:

![image](https://github.com/user-attachments/assets/c620c229-1d3b-47a9-9ab9-0b1ab4ddec47)

This configuration is to run the system on localhost. I will reconfigure it when I begin to implement cloud deployment.

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

I will use Timeweb. This is my favorite Russian cloud. It has a few cloud features, but it is cheap and has a Terraform provider.

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

Let's implement the infrastructure with `terraform`. First, we need to initialize the provider and configure the backend. I will hide all sensitive data.

`provider.tf`

![image](https://github.com/user-attachments/assets/c31a8a60-7927-4843-b316-987d43d8ee13)

I will 2 use providers:

- **Timeweb provider** - cloud provider.
- **Random provider** - provider to randomly generate passwords for instances.

`backend.tf`

![image](https://github.com/user-attachments/assets/38e82783-7fa8-47fe-a401-582c0b88f2c8)

There is no point in storing `terraform` manifests for cloud infrastructure on GitHub without `tf.state` if `tf.state` is stored nowhere except on localhost. It is very bad practice and can cause disasters for the infrastructure. I will store it in Timeweb's S3 service. In a real case, it is better to back up this S3 bucket.

### Inplementation

I will build the hardware infrastructure in a private network. There will be only two public servers—our Coffee service.

Let's implement it in separated files:

- `compute.tf`
- `network.tf`
- `firewall.tf`
- `variables.tf`
- `output.tf`

I started with `network.tf`:

![image](https://github.com/user-attachments/assets/87dbb6f4-1177-4b93-8f99-a853fffbaec3)

I prepared a cloud VPC to use it in `compute.tf` to connect the server to this network.

I also generated passwords and stored them in locals in `variables.tf`.

![image](https://github.com/user-attachments/assets/38160f95-88a5-4e78-975e-fe92f83f4343)

The next step is to create `output.tf` for the generated passwords. We will have the ability to view them.

![image](https://github.com/user-attachments/assets/c9acb941-e444-47a5-8d41-135727a57819)

After all preparations are ready, we can create `compute.tf`.

![image](https://github.com/user-attachments/assets/3b80f282-f221-4ef0-a436-3d56055b9770)

Here are some different instances:

- `twc_presets` - server resource configuration. On **Timeweb** cloud, we can create a server with a preset (like `flavor` in **Openstack**) or with our custom resource configuration. Using a preset is cheaper.
- `twc_os` - the server's OS preset.
- `twc_server` - the cloud VPC.
- `twc_floating_ip` - the public IP address. It is floating because if we recreate it, the IP address will change randomly.

I created 5 servers: `master-lb-server`, `coffee-server`, `drink-server`, `milk-server`, and `syrup-server`. All servers are the same, except for the `coffee-server`, which has its own public IP.

Moreover, every service has provisioning via cloud-init. There are 2 cloud-init files: one for the coffee server and one for the other servers.

`coffee.yaml`

![image](https://github.com/user-attachments/assets/9bbffcbe-24b8-41a2-8a4d-bfb6d507453c)

`others.yaml`

![image](https://github.com/user-attachments/assets/2e05388c-d82b-4daa-87e8-9fea3e0e858b)

I need to mention that SSH connections to any host are provided through the coffee server. You can see how I configured `firewalld` for forwarding.

The last thing is the WAF. I configured it in `firewalld.tf`. I set it up only for the `coffee-server`, but to be honest, it is necessary for every host in the private network. For this lab, I decided that configuring it for the `coffee-server` is sufficient.

![image](https://github.com/user-attachments/assets/8f302ba7-4420-4baf-9409-b9a7b8bebff1)

You can see the firewall and rules instances. The rules provide access for:

- **443** - HTTPS  
- **1900** - SSH to `master-lb-server`  
- **1901** - SSH to `coffee-server`  
- **1902** - SSH to `drink-service`  
- **1903** - SSH to `milk-service`  
- **1904** - SSH to `syrup-service`

### Deploy

Let's deploy our hardware infrastructure.

First, I need to initialize Terraform.

```
terraform init
```

![image](https://github.com/user-attachments/assets/85113055-756c-4ccf-a0b3-fb15e99aa41e)

Next, let's plan the infrastructure and verify that everything is correct.

```
terraform plan
```

![image](https://github.com/user-attachments/assets/c49740ee-25f2-4d46-a729-bd42d8637d37)

Now we can verify that the configuration is working and Terraform will create exactly what we need.

The next step is to apply our configuration.

```
terraform apply
```

![image](https://github.com/user-attachments/assets/488d37f9-7f20-4e9c-a47c-275cf14096f2)

Cloud rejected my `leet` network `192.133.7.0/24`, so I have to change it to `192.168.7.0/24`.

Try again, and everything is deployed!

![image](https://github.com/user-attachments/assets/d6b1d335-ec30-48a4-9015-f50bba59f03e)

We can prove it in the web interface.

![image](https://github.com/user-attachments/assets/0e899dcb-9f05-4bf8-8307-afa56e8c4bfc)

![image](https://github.com/user-attachments/assets/803d3e1b-4791-405c-9137-512dd2441aa5)

Let's try to connect to the master load balancer to prove that forwarding works correctly.

![image](https://github.com/user-attachments/assets/74062a16-8ba0-48ce-8848-8fd0ca3a56b4)

Everything works. Deploy of the hardware infrastructure is finished!

## 4.3. Choose Software Configuration Management (SCM) tool.

```
Ansible is the industry standard. Of course, we have other
SCM solutions such as SaltStack, Puppet, Chef. You can try them
but remember that it is probably more difficult to work with these
tools and you are responsible for your choice.
```

I decided to use Ansible. I also have good experience with SaltStack, but I think it is too complex for the lab.

## 4.4 Using SCM tool, write a playbook tasks to deliver and run your application to cloud instance/local VM.

```
Try to separate your configuration files into inventory/roles/playbooks files.
In real practice, we almost newer use poor playbooks where everything all in one.
```

**Github**: https://github.com/ullibniss/INNO-Sevice-Mash/tree/master/ansible  

### Preparation

I will use the following folder structure for `Ansible`:

![image](https://github.com/user-attachments/assets/360156b1-12f9-4435-9bc6-9a53e04c8698)

I will show the implementation of `Ansible` roles and playbooks only for the coffee service, because the others are almost the same. You can find them on GitHub.

### Inplementation

First of all, we need to implement the `role` for the service. Let's start with the tasks, and then I will add variables for parameterization if needed.

The role will have 5 tasks, with one auxiliary task and 4 for actual usage:

- `main.yml` - This task is responsible for proxying task calls to the concrete tasks inside the role. I will show it further.
- `build.yml` - This task is for building our server, specifically for building the Docker image.
- `deploy.yml` - This task is for deploying the service. It will render all the files needed to build and run the service.
- `start.yml` - This task is for starting the service.
- `stop.yml` - This task is for stopping the service.

Let's start with the implementation:

`main.yml`

![image](https://github.com/user-attachments/assets/57f3ca22-1bba-4161-8a1c-0418232d25c1)

As we can see, when I implement the playbook, I will use the variable `coffee_task` to call the concrete task.

`build.yml`

![image](https://github.com/user-attachments/assets/06abdaa3-bffc-4f4e-91dc-8acd0505ec4b)

Here we can see the default `docker build` command. There are 2 builds — `coffee` and `sidecar`, because `sidecar` is used in conjunction with coffee. To be honest, it's too small to implement a separate role. We also see the first two real variables here. I'm using `coffee_tag` because using `latest` is bad practice. However, the default value will be `latest` :).

`deploy.yml`

![image](https://github.com/user-attachments/assets/9ce95a6d-18e0-4bd8-80d7-8dc3b8cb8437)

The most interesting task, in my opinion, is `deploy.yml`. I clone the repository of the service for the build. If there were a `docker registry`, I would build it on a local or separate machine. But in this case, I will build it on the target server.

`start.yml`

![image](https://github.com/user-attachments/assets/026fe295-11cc-4d61-bf32-707e4d1ac200)

`stop.yml`

![image](https://github.com/user-attachments/assets/6850f2aa-22ae-40a1-b8ea-d296611b5324)

There are simple start and stop `docker-compose` tasks. Nothing too interesting.

Once the tasks are ready, I need to implement `defaults/main.yml` with default variables.

![image](https://github.com/user-attachments/assets/07a9d3a6-0477-4c57-963c-1fb607039a43)

You might have noticed that there are no variables `default_user` and `deploy_dir`. These variables have no default values and must be defined separately. This is because we don't know in what context our role will be used. Additionally, these variables may be general if there is more than one service on the server.

The last thing is to modify `docker-compose.yml`, as I promised earlier.

![image](https://github.com/user-attachments/assets/118c013d-27af-45e7-8b8d-4c0ca0e58185)

Key changes:

- The image tag is also templated.
- Since we still have separate servers, we don't need an external network. Because of this, I will create a bridge between `coffee` and `sidecar-lb` inside `docker-compose`.
- The `MASTER_URL` variable has been changed to the `master load balancer`'s IP.

Okay, once we finish the role implementation, we can implement the playbooks and inventories.

There will be 4 playbooks. They are the same as the role tasks, except for the `main` task. All playbooks are similar. The only thing that changes is the `coffee_task` variable.

`deploy.yml`

![image](https://github.com/user-attachments/assets/68e85dfd-581f-4bc3-b6ff-3eb14502685b)

`build.yml`

![image](https://github.com/user-attachments/assets/7e9d9869-67ff-47a7-a389-540c7f2c37ac)

To make our role available for the playbooks, I will configure the `ansible.cfg` file.

![image](https://github.com/user-attachments/assets/ee5ffbff-35d0-4db2-a865-9d4f9b76cafe)

Settings:

- `host_key_checking = False` - applies the `-o HostKeyChecking=no` flag for `ssh` connections.
- `roles_path=../roles` - This is like the `PATH` variable but for roles.
- `inventory=localhost.cfg` - Configures the default inventory file.

Now we can implement the inventory file. There will be two inventories: `localhost.cfg` and `remote.cfg`.

`localhost.cfg`

![image](https://github.com/user-attachments/assets/858c3bd2-9e66-4cec-a0d4-226efda8ba0f)

`remote.cfg`

![image](https://github.com/user-attachments/assets/a7b6b0a8-b825-4ec1-8062-2635976f589c)

I also implemented roles and playbooks for other services. You can check them out on **[Github](https://github.com/ullibniss/INNO-Sevice-Mash/tree/master/ansible)**.

### Deployment

Let's start the deployment. To deploy our services to remote servers, I will use the following command:

```
ansible-playbook -i remote.cfg {deploy.yml,build.yml,start.yml} -K
```
![image](https://github.com/user-attachments/assets/4da960c8-0d0f-403a-84c7-255db25c8acd)

![image](https://github.com/user-attachments/assets/989aaff0-3d59-4f9c-a895-2b02fdc50788)

As we can see all tasks executed correctly.

I repeated this process for all other services. And as a result I've got:

![image](https://github.com/user-attachments/assets/abf91be4-a609-4661-822c-200a2ab65182)

This indicates that the system works correctly. Let me explain:

- If any of the milk, syrup, or drink didn't work, we would not have all the parts of the message.
- If any of the master LB, sidecar LB, or coffee didn't work, we would not have any response.

We can also see the result in the web browser:

![image](https://github.com/user-attachments/assets/6f7ec376-dabd-4fa6-aaa8-5478cae9c267)

### What could be done better:

1) I could use a `docker registry` instead of building images on the servers.
2) I could use `Vault` to store sensitive data and load server credentials from there via the `ansible hvac` module.
3) I could use a real NAT server with a public IP instead of using the coffee-server.

# Task 5 - Play with SCM cases

While completing the previous tasks, I implemented one more role. I decided to show it in task 5.

This role is for advanced server provisioning. It will perform 3 actions:

1) Configure the `/hosts` file and set aliases for the local network.
2) Configure `ssh` by deploying keys and applying some security options.
3) Install the necessary packages. In this example, it installs the Docker packages.

Each responsibility is handled by a separate task.

- `manage_network.yml`

![image](https://github.com/user-attachments/assets/b484c71f-18cb-43da-8cce-cd7490b0a648)

It renders the following file to `/etc/hosts`:

![image](https://github.com/user-attachments/assets/30043b7c-b835-4cd8-8f08-868e33ae79bb)

Here is variable:

![image](https://github.com/user-attachments/assets/72734598-2bfd-4a4a-a806-f6bdbfd65a41)

- `manage_keys.yml`

![image](https://github.com/user-attachments/assets/3663de2c-6e0a-457a-9c2a-89e5fbb3eb72)

This task renders keys from templates and configures ssh.

- `manage_packages.yml`

![image](https://github.com/user-attachments/assets/6e573e0c-5228-470c-a273-22828b783cc3)

As we can see, `apt` updated the package list and installed the Docker packages. It is important to install Docker this way (not `docker.io`) because it includes security updates.

# References

- https://github.com/timeweb-cloud/terraform-provider-timeweb-cloud
- https://registry.terraform.io/providers/hashicorp/random/latest/docs
- https://cloudinit.readthedocs.io/en/latest/
- https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html
- https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html
- `ansible-doc` command, to see documentation of modules.
