# D&S Lab 4: Kubernetes Basics

## Completed by Fedorov Alexey (tg: @ullibniss)

---

```
The lab goal is to get familiar with the main Kubernetes manifests, then try them on practice and deploy
application on Kubernetes cluster as well.

  Individual lab. Choose this lab flavor if you don't work with Kubernetes before and
  have to learn it from the beginning.
  While working on some tasks, keep in mind that you might wipe your created objects after
  the task completion before to go the next task in order do not mess some logically
  conflicted objects.
  We are going to work within the local cluster (minikube, k3s, MicroK8s...) but nothing
  prevents you from deploying a cluster on a real cloud like AWS EKS or On-premise baremetal solution.
```


# Task 1 - Preparation

## 1.1 Choose an application that has such characteristics as (points with "star" are not required but recommended)

I have chosen `JetBrains YouTrack` as the application. It meets all the requirements.

## 1.2 Get familiar with Kubernetes (k8s) and concepts.

Kubernetes automates deployment, scaling, and management of containerized applications, ensuring high availability and stability. It uses these core concepts to manage workloads efficiently across distributed systems:

1. **Cluster**: A Kubernetes cluster is a set of nodes that run containerized applications. It consists of a control plane (master) and worker nodes.  
2. **Node**: A node is a machine (physical or virtual) in the cluster. Worker nodes run Pods, while the master node manages the cluster.  
3. **Pod**: The smallest deployable unit in Kubernetes, a Pod contains one or more containers that share storage, network, and specifications.  
4. **Deployment**: A Deployment defines the desired state for Pods and ReplicaSets, enabling declarative updates, scaling, and rollbacks.  
5. **Service**: A Service provides a stable network endpoint to access Pods, abstracting their dynamic IP addresses with a consistent DNS or IP.  
6. **ConfigMap**: ConfigMaps store configuration data as key-value pairs, allowing you to decouple configuration from container images.  
7. **Namespace**: Namespaces divide a cluster into virtual sections, enabling resource isolation and organization for teams or projects.  

## 1.3 Install and set up the necessary tools

I installed the necessary tools using the [official documentation](https://kubernetes.io/ru/docs/tasks/tools/).

`kubectl`

![image](https://github.com/user-attachments/assets/47408a2d-4275-4325-b6c6-f7f829331ae8)

`minikube`

![image](https://github.com/user-attachments/assets/335c6f93-619d-4485-b266-f4c138b94b00)

## 1.4 Get access to Kubernetes Dashboard.

To get access to the dashboard, I started Minikube and enabled the `dashboard` plugin. Then I opened the Minikube dashboard.

![image](https://github.com/user-attachments/assets/dde99510-ae1a-4a08-8bb2-63415002c02c)

# Task 2 - k8s Nodes

## 2.1 After installing all required lab tools and starting minikube cluster, use kubectl commands   to get and describe your cluster nodes.

To get all cluster nodes, I used the following command:

```
kubectl get nodes
```

![image](https://github.com/user-attachments/assets/4ac9e4ff-6e5a-4b53-8cd5-793a91491279)

## 2.2 Get the more detailed information about the particular node (it's fine if you have one node in the cluster).

To get detailed info about node, I used `describe` command:

```
kubectl describe node minikube
```

![image](https://github.com/user-attachments/assets/758466a8-925b-4691-8725-5ee9044e9c9b)


## 2.{3,4} Get the OS and CPU information. Put the results into report.

To get `OS` information, I will simply grep the `describe` output:

```
kubectl describe node minikube | grep 'OS'
```

![image](https://github.com/user-attachments/assets/f0106e36-0c53-43d1-b0a3-24ebdbc85f97)

The same is for CPU information.

![image](https://github.com/user-attachments/assets/3f87fa97-2eab-4f69-9bb6-bbaed5fd44a7)


# Task 3 - k8s Pod

## 3.1 Figure out the necessary Pod spec fields.

Pod specification must include essential fields such as:

- `apiVersion` - defines the Kubernetes API version used.
- `kind` - specifies the type of resource (Pod).
- `metadata` - includes name and labels for the Pod.
- `spec`:
  - `containers` - defines the container image, ports, environment variables, and volume mounts.
  - `volumes` - if persistent storage is needed.
 
## 3.2 Write a Pod spec for your chosen application, deploy the application and run a pod.

YouTrack requires:

- An official `JetBrains YouTrack` Docker image.
- Persistent storage for data retention.
- A port to expose the service.

I created pod spec for `youtrack`, here is it:

![image](https://github.com/user-attachments/assets/a8293ca3-0cc8-4778-85f2-cc93ff534d78)

And then I applied this configuration:

![image](https://github.com/user-attachments/assets/39e9276f-5257-4516-89ed-11836fd35ee1)

## 3.3 With kubectl , get the pods, pod logs, describe pod, go into pod shell.

To check whether my pod running, I used:

```
kubectl get pods
```

![image](https://github.com/user-attachments/assets/d27ee6a5-95b0-46ae-914e-41816b857dc6)

I can also describe pod:

```
kubectl describe pod youtrack
```

![image](https://github.com/user-attachments/assets/496059f0-3014-4a50-9ad2-1c84ab6d3995)

To see the pod's logs, I used the following command:

```
kubectl logs youtrack
```

![image](https://github.com/user-attachments/assets/a4d0e445-9261-466b-9478-c90e8d6477d1)

To execute a command inside a pod, I can use the following command:

```
kubectl exec -it youtrack -- /bin/bash
```

![image](https://github.com/user-attachments/assets/f5b800cf-cdba-4d4b-8e54-2a6fd4f16781)

## 3.{4.5} Make sure that your app is working correctly inside Pod. Put the results into report.

To check whether my application works correctly, I need to access it. I can't access it via `localhost` because Minikube runs pods inside a container, and this container has its own IP. I will check the `nodes`:

```
kubectl get nodes -o wide
```

![image](https://github.com/user-attachments/assets/f6a38415-2af2-46b1-97cc-580e9bc98b6e)

Now let's access `Youtrack`. 

![image](https://github.com/user-attachments/assets/ccf793cf-04c6-49b8-82eb-112d38218422)

It is accessible. I activated it to check how it works.

![image](https://github.com/user-attachments/assets/7a3858a7-5874-4af1-bb99-c5543806a74f)

Finally, it works correct.

# Task 4 - k8s Service

```
We use k8s Services to make an application accessible from outside the Kubernetes
virtual network, provide an compatible IP address and link to DNS name to pod, to route
internal/external traffic within pods.
```

## 4.1 Figure out the necessary Service spec fields.

A Kubernetes Service allows Pods to communicate with each other and be accessed externally. The key fields in a Service spec are:

Fields:
- `apiVersion`: v1
- `kind`: Service
- `metadata`: Defines the Service name and labels.
- `spec`:
  - `selector`: Matches the target Pods.
  - `ports`: Defines the exposed ports.
  - `type`:
    - (one of three)
    - `ClusterIP` (default) - Internal communication only.
    - `NodePorr` - Exposes the Service on all nodes.
    - `LoadBalancer` - External access via a cloud provider.

## 4.2 Write a Service spec for your pod(s) and deploy the Service.

I've implemented spec for my pods:

![image](https://github.com/user-attachments/assets/eb16b4eb-793f-45ba-8cc1-cac55822a5f1)

Let's deploy it.

![image](https://github.com/user-attachments/assets/8d4e97c5-18e5-460b-b385-fa94f8adcc3f)

## 4.3 With kubectl , get the Services and describe them.

Let's check whether `youtrack` service presents:

![image](https://github.com/user-attachments/assets/c397d2a2-ae31-4b3a-9dfb-9caa240eff54)

It was successfully deployed.

Let's describe service:

```
kubectl describe service youtrack-service
```

![image](https://github.com/user-attachments/assets/03e10332-b02d-48bb-b363-75187fdf9404)

## 4.4 Make sure that pods can communicate between each other using DNS names, check pods addresses.

Unfortunately, our service has only 1 pod. Because of this, we can deploy test-pod, to check connectivity. Let's do it.

![image](https://github.com/user-attachments/assets/e4643fbb-7137-4839-9d87-9e3071c59ccb)

Test pod can connect to `youtrack-service` via `DNS` name.

If we check pods' IP addresses, we can see, that they are in the same network.

![image](https://github.com/user-attachments/assets/54a9b2f7-8642-4fe4-8331-7ce95a9fb8fe)

## 4.5. Delete any Pod, recreate it and check addresses again. Make sure that traffic is routed to the new Pod correctly.

Let's recreate `youtrack` pod.

![image](https://github.com/user-attachments/assets/857154c8-f51a-4f99-ac50-a9d10ba29a1e)

As we can see, when pod was recreated, it got new IP adress, but in the same network. Let's check connectivity.

![image](https://github.com/user-attachments/assets/e4124ac1-7729-4e3d-853f-9f8e5e69de1c)

It connects.

## 4.6 Learn about Loadbalancer and NodePort.

I've learnt.

## 4.{7,8} Deploy an external Service to access your application from outside, e.g., from your local host. Put the results into report.

To expose `YouTrack` outside `Kubernetes`, I will use a NodePort. Let's modify service configuration

![image](https://github.com/user-attachments/assets/e962a57e-9402-4c66-976a-bb9fec818806)

Redeploy it.

![image](https://github.com/user-attachments/assets/be55e7c8-5585-42d1-9a4e-6d7cc70e42a2)

Now I can access my application with `http://192.168.49.2:30080`. Ip is not changed, because lab is on localhost. But if It were cloud, I would be able to access service with public ip. 

![image](https://github.com/user-attachments/assets/d3a372dc-6d80-4c11-afe4-4f2d69420810)

# Task 5 - k8s Deployment

```
Deployment is Kubernetes manifest to manage Pods automatically by Controller. It helps to
release, scale and upgrade Pods.
```

## 5.1 Figure out the necessary Deployment spec fields.

A Deployment provides a declarative way to manage Pods. Key fields include:

Necessary Fields:
- `apiVersion`: apps/v1
- `kind`: Deployment
- `metadata`: Deployment name and labels.
- `spec`:
  - `replicas`: Number of desired Pods.
  - `selector`: Matches Pods with labels.
  - `template`:
    - `metadata`: Labels for Pods.
      - `spec`:
        - `containers`:
          - Image, ports, environment variables, resource limits.
         
## 5.2 Make sure that you wiped previous Pod manifests. Write a Deployment spec for your pod(s) and deploy the application.

I wiped previous `Pod manifests`.

![image](https://github.com/user-attachments/assets/d1547285-c174-424d-a03d-1765d2106395)

Next, I implemented Deployment.

![image](https://github.com/user-attachments/assets/a0ac4b28-3143-4ff3-bd72-afa5e9c84c84)

I applied it.

![image](https://github.com/user-attachments/assets/ee00aef8-90ad-4999-8943-76b9969c359e)

## 5.3 With kubectl , get the Deployments and describe them.

Let's verify `deployment` status:

```
kubectl get deployments
```

![image](https://github.com/user-attachments/assets/134e9095-8206-41fc-996b-7659ae636eb6)


Deployment been succesfully deployed. Now I will `describe` deployment.

![image](https://github.com/user-attachments/assets/0490c407-6636-4359-ae20-6e7948eec9a6)

## 5.4 Update your Deployment manifest to scale your application to three replicas.

To scale my application to three replicas, I simply need to modify `Replicas` field in configuration.

![image](https://github.com/user-attachments/assets/711b9024-187d-483c-9e9c-2e56c323d382)

Deploy:

![image](https://github.com/user-attachments/assets/db1618d5-dd2c-498c-a0fb-848752f418a5)

It successfully upscaled!

## 5.5 Access pod shell and logs using Deployment labels.

I will use Deployment labels to find pods related to `youtrack` deployment.

![image](https://github.com/user-attachments/assets/2a3cd115-1714-4f59-9504-2b59aede2765)

As we can see, there 3 pods.

Let's access shell of one of the pods.

![image](https://github.com/user-attachments/assets/90b265ba-c4e8-415f-96de-c2999939af49)

How I will check pod's logs.

![image](https://github.com/user-attachments/assets/2c7a124a-90aa-4f90-a253-7e0ac368d8f3)

## 5.6 . Make any application configuration change in your Deployment yaml and try to update the application. Monitor what are happened with pods (--watch).

Let's edit application configuration. I added environment variable to change base url.

![image](https://github.com/user-attachments/assets/352d54d1-a571-4f85-96d7-e77a6a67ef7e)

Deploy:

![image](https://github.com/user-attachments/assets/05e20a2f-d153-40bf-a9a1-3fe6611f61f4)

Using `kubectl get pods --watch`, we can see, that old pods are terminated, and new pods are running now.

## 5.7 Rollback to previous application version using Deployment.

To rollback to previous application version I need to check deployment history.

```
kubectl rollout history deployment youtrack-deployment
```

![image](https://github.com/user-attachments/assets/1b55c327-b545-4f65-aa6b-2fa902c14bac)

Unfortunately, there no cause specified for previous deployment.

I can rollback using version number. Let's do it.

![image](https://github.com/user-attachments/assets/42ab616a-5ca0-4f38-bcc3-b37668715c94)

As we can see, deployment rolled back. To verify that this is prevoius version, we can check, that there no environment variables.

![image](https://github.com/user-attachments/assets/f30de954-09cb-4696-8749-81051ce7ca24)

## 5.8 Set up requests and limits for CPU and Memory for your application pods. Provide a PoC that it works properly. Explain and show what is happened when app reaches the CPU usage limit? And memory limit?

To set limits for pods, I will use `requsts` and `limits` options. 

- `requests` - defines the minimum amount of CPU and memory guaranteed for a container.
- `limits` - defines the maximum amount of CPU and memory a container can use.

I modified configuration.

![image](https://github.com/user-attachments/assets/86982ea8-6b9b-4351-94b7-7cec8235ac95)

Deploy:

![image](https://github.com/user-attachments/assets/594a45d9-e4cb-437b-bc23-21df6efc3bdc)

Let's test limits.

### Test: CPU Limit Test

To simulate high CPU usage, I will use `yes` command.

![image](https://github.com/user-attachments/assets/9e1fbea0-ca1e-4bab-97f2-767110929a1f)

![image](https://github.com/user-attachments/assets/9e42aca7-7884-4bbf-b6d2-d1b52d98df7d)

As we can see, server consumens <500m of CPU. Limit works. We process reaches limit on CPU is starts throttling, because of queue of operations per tick.

### Memory Limit test

To simulate high Memory usage, I will use fork bomb ` :(){ :|:& };:`.

![image](https://github.com/user-attachments/assets/839022ab-9436-477c-9a03-754045c7c4b7)

![image](https://github.com/user-attachments/assets/c71198db-38e5-40d7-8196-c5d48934a98f)

We can see, that limit exceeded and then proccess terminated via OOM Killer. This happend because OOM killer gives mark (`oom_score`) to process and kills process with highest `oom_score` when any process reaches limit.

# Task 6 - k8s Secrets

```
Secret manifest is a quite similar to configMap . However, we use Secret to work with
confidential application data. Kubernetes encode secrets in Base64 format.
```

## 6.1 Figure out the necessary Secret spec fields.

A Secret in Kubernetes is used to store sensitive information, such as passwords, API keys, or TLS certificates.

Necessary fields:

```
- apiVersion: v1
- kind: Secret
- metadata: 
  - name: my-secret - the name of the Secret
- type: Opaque -  default type for generic Secrets
- data: - contains key-value pairs where values are base64-encoded
```

## 6.2 Create and apply a new Secret manifest. For example, it could be login and password to login to your app or something else.

First, I will create encoded creds.

![image](https://github.com/user-attachments/assets/6eac1673-e7f7-4e9e-a099-28c119e2ade9)

Next, create a manifest.

![image](https://github.com/user-attachments/assets/f06a803d-7827-4f47-846d-6afc1980baa9)

Apply:

![image](https://github.com/user-attachments/assets/225b81a5-63e5-4ffc-9e71-408a14eec037)

## 6.3 With kubectl , get and describe your secret(s).

Let's describe secret.

![image](https://github.com/user-attachments/assets/22a4230e-9c38-4f6f-b6af-5cbb09f4432d)

We can see name and size of secret data.

## 6.4 Decode your secret(s)

We can use `-o jsonpath` to get data from secret and `base64 --decode` to decode base64.

![image](https://github.com/user-attachments/assets/d440d2b4-52cb-4bd7-834c-201a968491c3)

## 6.5 Update your Deployment to reference to your secret as environment variable. 

To reference to secret I will specify `secretKeyRef` option. Modified Deployment manifest:

![image](https://github.com/user-attachments/assets/f869b3b6-c7e0-46d6-b2ce-70ddb030b007)

Deploy:

![image](https://github.com/user-attachments/assets/97cdffc2-424e-4d74-b1fd-79928650ce40)

## 6.6 Make sure that you are able to see your secret inside pod.

It is enough to check environment variables inside one of the pods.

![image](https://github.com/user-attachments/assets/5e6973a9-f6fc-4943-8a18-565606148ec8)

# Task 7 - k8s configMap

```
ConfigMap is Kubernetes manifest to store application configuration setting in two ways:
key-value pairs as environment variables and text (usually JSON) data as dedicated file into
container filesystem. We usually use configMap to store non sensitive data as plain text.
```

## 7.1 Figure out the necessary configMap spec fields.

A ConfigMap is used to store non-sensitive configuration data, such as:

- Environment variables
- Configuration files

Necessary fields:

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config - name of the ConfigMap
data: - contains key-value pairs as plain text.
  config_key1: "value1"
  config_key2: "value2"
```

## 7.{2,3} Modify your Deployment manifest to set up some app configuration via environment variables. Create a new configMap manifest. In data spec, put some app configuration as key-value pair (it could be the same as in previous exercise). In the Deployment.Pod spec add the connection to key-value pair from configMap yaml file.

Let's create configMap manifest.

![image](https://github.com/user-attachments/assets/f97e3df1-d0f7-4c5b-b62c-dcbebc8f5f19)

Apply:

![image](https://github.com/user-attachments/assets/1ab823f0-a5dd-4e1d-89f4-bfb038283214)

Deployment modification:

![image](https://github.com/user-attachments/assets/07bc3e64-6a74-4c06-8a3a-a3e7636be93b)

To specify configuration from configMap, I used `configMapKeyRef`.

Deploy:

![image](https://github.com/user-attachments/assets/0c4a8506-3daf-417b-9d4e-15acf8f66111)

Verification:

![image](https://github.com/user-attachments/assets/e34d989c-97d9-4e68-a969-781bb80392f4)

## 7.4. Create a new file like config.json file and put some json data into.

I created `config.json`.

![image](https://github.com/user-attachments/assets/1ffc6412-6ed1-4e9e-aba7-982e80d8421e)

# 7.{5,7} Create a new one configMap manifest. Connect configMap yaml file with config.json file to read the data from it. With kubectl , check the configMap details. Make sure that you see the data as plain text.

Let's create configMap from config.json:

```
kubectl create configmap youtrack-config-file --from-file=config.json
```

![image](https://github.com/user-attachments/assets/e830ae89-c3cd-488c-9af6-8ce872782a0e)

Verification:

![image](https://github.com/user-attachments/assets/035315a5-5159-40d8-b9d0-a0610efe72a3)

We can see file data as plaintext.

## 7.6 Update your Deployment to add Volumes and VolumeMounts.

I modified `youtrack-deployment`. Added volume for pod and volumeMount.

![image](https://github.com/user-attachments/assets/12cd2122-6422-45b9-885d-e83a7e4caf43)

Let's deploy:

![image](https://github.com/user-attachments/assets/1685a6da-37f0-44e6-9b4b-c282881b9811)

## Check the filesystem inside app container to show the loaded file data on the specified path.

Let's verify that volume mounted and file presents.

![image](https://github.com/user-attachments/assets/c0b52c9f-e845-4302-95a0-e04348ace1d6)

File presents, everything works correct!

# Task 8 - k8s Namespace

```
Namespace Kubernetes Manifest is designed for different projects and deployment
environments isolation. With Namespaces we can separate App 1 deployment from App 2
deployment, manage (and isolate) cluster resources for them, define users list to have
access either to App 1 or to App 2 deployment. Using Namespaces , we also can define a
different environments like DEV, TEST, STAGE. In that way, Namespaces is a required feature
for a real Kubernetes production clusters.
```

## 8.1 Figure out the necessary Namespace spec fields.

A Namespace in Kubernetes provides a logical separation for resources within a cluster.

Necessary fields:

```
apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace -  the name of the Namespace
```

## 8.2 Create a two different Namespaces in your k8s cluster.

I created namespaces manifests.

![image](https://github.com/user-attachments/assets/22ccf381-6b18-4026-9e93-8afaab1667fa)

Apply:

![image](https://github.com/user-attachments/assets/b5b39f0f-208b-4bfe-8d45-852683039bf7)

## 8.3 Using kubectl , get and describe your Namespaces.

Let's describe both namespaces `youtrack-one` and `youtrack-two`.

![image](https://github.com/user-attachments/assets/8e5c0ac1-401e-4bb9-ae3a-9d1979e49a99)

## 8.4 Deploy two different applications in two different Namespaces with kubectl. By the way, it's acceptable even just to deploy the same objects (same previous app) in the different Namespaces but with different resources names.

I will deploy `youtrack` to one namespace and `nginx` to another namespace. I have to modify `youtrack-deployment` and create `nginx-deployment`.

Youtrack deployment:

![image](https://github.com/user-attachments/assets/59ed0a9b-50bb-464c-9a1d-a956bd631655)

Nginx deployment:

![image](https://github.com/user-attachments/assets/fd62321f-ff10-4bab-bbed-91e7e9707b0e)

Let's deploy:

![image](https://github.com/user-attachments/assets/c9c543fd-6fab-48d8-9aa4-2b833ea17c2d)

As we can see, youtrack deployment recreated in new namespace, and nginx was also created.

## 8.5 With kubectl, get and describe pods from different Namespaces witn -n flag.

To get pods by namespace I need to use `-n` flag:

```
kubectl get pods -n youtrack-one
```

![image](https://github.com/user-attachments/assets/2924d798-d4ae-40a8-93eb-74abbaa609b5)

I can describe pod in concrete namespace with the same flag.

```
kubectl describe pod -n youtrack-two nginx-deployment-659b9cb59d-zh77k
```

![image](https://github.com/user-attachments/assets/320798fb-2ac7-4f29-acf4-fa1e1e78ad67)

![image](https://github.com/user-attachments/assets/7c6a1b21-f345-444c-bc7f-73ab76c9d670)

## 8.6 Can you see and can you connect to the resources from different Namespaces?

By default, Pods in different Namespaces can communicate by IP.

![image](https://github.com/user-attachments/assets/882ba2b3-e810-4d4e-935c-0fdf4b3e9277)

They can also communicate with FQDN if `service` is configured.

# References

- https://kubernetes.io/docs/tasks/tools/
- https://kubernetes.io/docs/concepts/workloads/pods/
- https://kubernetes.io/docs/concepts/services-networking/
- https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
- https://kubernetes.io/docs/concepts/configuration/configmap/
- https://kubernetes.io/docs/concepts/configuration/secret/
- https://kubernetes.io/docs/concepts/architecture/nodes/
- https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/
