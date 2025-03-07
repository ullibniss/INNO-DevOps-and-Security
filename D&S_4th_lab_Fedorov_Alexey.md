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

I have choosen `JetBrains Youtrack` tracker as application. It meets all the requirements.

### 1.2 Get familiar with Kubernetes (k8s) and concepts.

I got.

### 1.3 Install and set up the necessary tools

I installed necessary tools using [official documentation](https://kubernetes.io/ru/docs/tasks/tools/). 

`kuberctl`

![image](https://github.com/user-attachments/assets/47408a2d-4275-4325-b6c6-f7f829331ae8)

`minikube`

![image](https://github.com/user-attachments/assets/335c6f93-619d-4485-b266-f4c138b94b00)

# 1.4 Get access to Kubernetes Dashboard.

To get access to dashboard, I started minikube and enabled `dashboard` plugin. Then I opened minikube dashboard.

![image](https://github.com/user-attachments/assets/dde99510-ae1a-4a08-8bb2-63415002c02c)

# Task 2 - k8s Nodes

# 2.1 After installing all required lab tools and starting minikube cluster, use kubectl commands   to get and describe your cluster nodes.

