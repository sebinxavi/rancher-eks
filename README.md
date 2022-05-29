In this blog, we’re demonstrating how to configure Rancher and add an existing AWS EKS cluster to it. Rancher is an open source software platform that is used for managing the Kubernetes clusters. 


## Prerequisites

1. EC2 Instance Ubuntu
2. EKS cluster 

### Step 1: Install Docker on EC2 instance (Rancher Server)

I used to follow below steps to install latest Docker version in Ubuntu
~~~
sudo apt-get update
~~~
~~~
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
~~~
~~~
sudo mkdir -p /etc/apt/keyrings
~~~
~~~
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
~~~
~~~
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
~~~
~~~
sudo apt-get update
~~~
~~~
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
~~~
![alt text](https://i.ibb.co/xFY5J74/Docker-Installation.png)

### Step 2: Run the Latest Rancher image in Docker
~~~
 sudo docker run -d --restart=always -p 80:80 -p 443:443 -v /opt/rancher:/var/lib/rancher --privileged rancher/rancher:latest
~~~
![alt text](https://i.ibb.co/n0GjYtq/Docker-Rancher.png)
### Step 3:  Setup the Application Load Balancer
- Configure the ALB and Target group
![alt text](https://i.ibb.co/GRzXT66/Target-Group.png)
- Point domain to ALB endpoint in Route 53
![alt text](https://i.ibb.co/BgppwHX/Route-53.png)

### Step 4: Access Rancher URL in browser
![alt text](https://i.ibb.co/1KFbkPN/Welcome-page-rancher.png)

### Step 5: Login Rancher 
- Follow instructions which appears in welcome page
![alt text](https://i.ibb.co/CnPhtYs/Rancher-Password.png)
- Login Rancher
![alt text](https://i.ibb.co/px3bnp2/Welcome-page-rancher-2.png)

### Step 6: Add EKS cluster to Rancher
- Click on clusters and choose Generic

![alt text](https://i.ibb.co/HpchZqw/Rancher-3.png)

- Add the Cluster name and click on Create
![alt text](https://i.ibb.co/YWZs5CY/Adding-Cluster-to-rancher.png)

### Step 7: Apply the kubectl command
![alt text](https://i.ibb.co/MMQ6zt9/kubectl-apply-rancher.png)
~~~
 $ kubectl apply -f https://domain/v3/import/xxxxxxxxxx.yaml
clusterrole.rbac.authorization.k8s.io/proxy-clusterrole-kubeapiserver created
clusterrolebinding.rbac.authorization.k8s.io/proxy-role-binding-kubernetes-master created
namespace/cattle-system created
serviceaccount/cattle created
clusterrolebinding.rbac.authorization.k8s.io/cattle-admin-binding created
secret/cattle-credentials-3486dd8 created
clusterrole.rbac.authorization.k8s.io/cattle-admin created
deployment.apps/cattle-cluster-agent configured
service/cattle-cluster-agent created
~~~
~~~
$ kubectl get all -n cattle-system
NAME                                        READY   STATUS    RESTARTS   AGE
pod/cattle-cluster-agent-68cb8c4784-mzfgc   1/1     Running   0          2m14s
pod/cattle-cluster-agent-68cb8c4784-xkg7r   1/1     Running   0          2m15s
NAME                           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/cattle-cluster-agent   ClusterIP   172.20.125.115   <none>        80/TCP,443/TCP   52d
NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cattle-cluster-agent   2/2     2            2           52d
NAME                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/cattle-cluster-agent-65bcff69d    0         0         0       52d
replicaset.apps/cattle-cluster-agent-68cb8c4784   2         2         2       2m15s
replicaset.apps/cattle-cluster-agent-68fb87d757   0         0         0       52d
replicaset.apps/cattle-cluster-agent-745894d768   0         0         0       2m55s
~~~

### Step 8: Access the Rancher UI
- Once that is done, up and running cluster metrics will show as below.
![alt text](https://i.ibb.co/h8q9xnP/Cluster-Info.png)

## Summary
We can add an existing Kubernetes cluster or create a new kubernetes cluster using Rancher and scale up and scale down multiple nodes, pods etc. 
