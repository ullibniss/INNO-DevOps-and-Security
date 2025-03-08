![image](https://github.com/user-attachments/assets/4ace1517-9086-45c7-9c38-c8d948f6bfd6)![image](https://github.com/user-attachments/assets/c240ec84-cf0d-4844-a7e6-c5aafad03f05)# D&S Lab 4: Kubernetes Basics

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

I have choosen `JetBrains Youtrack` tracker as application. It meets all the requirements.

## 1.2 Get familiar with Kubernetes (k8s) and concepts.

I got.

## 1.3 Install and set up the necessary tools

I installed necessary tools using [official documentation](https://kubernetes.io/ru/docs/tasks/tools/). 

`kuberctl`

![image](https://github.com/user-attachments/assets/47408a2d-4275-4325-b6c6-f7f829331ae8)

`minikube`

![image](https://github.com/user-attachments/assets/335c6f93-619d-4485-b266-f4c138b94b00)

## 1.4 Get access to Kubernetes Dashboard.

To get access to dashboard, I started minikube and enabled `dashboard` plugin. Then I opened minikube dashboard.

![image](https://github.com/user-attachments/assets/dde99510-ae1a-4a08-8bb2-63415002c02c)

# Task 2 - k8s Nodes

## 2.1 After installing all required lab tools and starting minikube cluster, use kubectl commands   to get and describe your cluster nodes.

To get all cluster nodes I used the following command:

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

To get `OS` information I will simply grep `describe` output:

```
kubectl describe node minikube | grep 'OS'
```

![image](https://github.com/user-attachments/assets/f0106e36-0c53-43d1-b0a3-24ebdbc85f97)

THe same is for CPU information.

![image](https://github.com/user-attachments/assets/3f87fa97-2eab-4f69-9bb6-bbaed5fd44a7)


# Task 3 - k8s Pod

## 3.1 Figure out the necessary Pod spec fields.

Pod specification must include essential fields such as:

- `apiVersion` - Defines the Kubernetes API version used.
- `kind` - Specifies the type of resource (Pod).
- `metadata` - Includes name and labels for the Pod.
- `spec`:
  - `containers` - Defines the container image, ports, environment variables, and volume mounts.
  - `volumes` - if persistent storage is needed.
 
## 3.2 Write a Pod spec for your chosen application, deploy the application and run a pod.

YouTrack requires:

- An official `JetBrains YouTrack` Docker image.
- Persistent storage for data retention.
- A port to expose the service.

I created pod spec for youtrack, here is it:

![image](https://github.com/user-attachments/assets/a8293ca3-0cc8-4778-85f2-cc93ff534d78)

And then I applied this configuration:

![image](https://github.com/user-attachments/assets/39e9276f-5257-4516-89ed-11836fd35ee1)

## 3.3 With kubectl , get the pods, pod logs, describe pod, go into pod shell.

To check whether my pod running, i used:

```
kubectl get pods
```

![image](https://github.com/user-attachments/assets/d27ee6a5-95b0-46ae-914e-41816b857dc6)

I can also describe pod:

```
kubectl describe pod youtrack
```

![image](https://github.com/user-attachments/assets/496059f0-3014-4a50-9ad2-1c84ab6d3995)

To see pod's logs I used the following command:

```
kubectl logs youtrack
```

![image](https://github.com/user-attachments/assets/a4d0e445-9261-466b-9478-c90e8d6477d1)

To execute command insibe pod, i can use the following command:

```
kubectl exec -it youtrack -- /bin/bash
```

![image](https://github.com/user-attachments/assets/f5b800cf-cdba-4d4b-8e54-2a6fd4f16781)

## 3.{4.5} Make sure that your app is working correctly inside Pod. Put the results into report.

To check whether my application works correct, I need to access it. I can't access it by `localhost`, because minicube run pods inside the container and this container is also has it's own IP. I will check `node`:

```
kubectl get nodes -o wide
```

![image](https://github.com/user-attachments/assets/f6a38415-2af2-46b1-97cc-580e9bc98b6e)

Now let's access `Youtrack`. 

![image](https://github.com/user-attachments/assets/ccf793cf-04c6-49b8-82eb-112d38218422)

It is accessible, I will activate it to check how it works. 

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

