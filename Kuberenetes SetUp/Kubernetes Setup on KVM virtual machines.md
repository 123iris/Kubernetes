# Steps to Install Kubernetes on Centos VM

* To use Kubernetes, you need to install a containerization engine. <br>
* Currently, the most popular container solution is Docker. 
* Docker needs to be installed on CentOS, both on the Master Node and the Worker Nodes. [Docker install](https://github.com/123iris/Docker/blob/master/Docker%20install%20on%20centos.md)

#### Step 1: Configure Kubernetes Repository

* The steps are followed from [here](https://phoenixnap.com/kb/how-to-install-kubernetes-on-centos)
* Kubernetes packages are not available from official CentOS 7 repositories.
* This step needs to be performed on the Master Node, and each Worker Node you plan on utilizing for your container setup.
* You must be the root user ( sudo won't work here ).
* Copy & paste entire command from cat << EOF > to EOF. Note: don't just copy 1st line  
* Enter the following command to retrieve the Kubernetes repositories.

```
[root@k8Master1 mosipuser]# cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
[root@k8Master1 mosipuser]# yum clean all
[root@k8Master1 mosipuser]# yum list all
```

#### Step 2: Install kubelet, kubeadm, and kubectl

* These 3 basic packages are required to be able to use Kubernetes. Install the following package(s) on each node:

```
[mosipuser@k8Master1 ~]$ sudo yum install -y kubelet kubeadm kubectl

Loaded plugins: fastestmirror, product-id, search-disabled-repos, subscription-manager
This system is not registered with an entitlement server. You can use subscription-manager to register.
Determining fastest mirrors
.....
.....
Complete!
```

```
[mosipuser@k8Master1 ~]$ sudo systemctl enable kubelet
```

```
[mosipuser@k8Master1 ~]$ sudo systemctl start kubelet
```
#### Step 3: Set Hostname on Nodes

* This is not Required. It will your wish to follow this Step.
* To give a unique hostname to each of your nodes, use this command:

```
[mosipuser@k8Master1 ~]$ sudo hostnamectl set-hostname master-node
```
<p style="text-indent:100px">or</p>

```
[mosipuser@k8Master1 ~]$ sudo hostnamectl set-hostname worker-node1
```

* In this example, the master node is now named master-node, while a worker node is named worker-node1.

Make a host entry or DNS record to resolve the hostname for all nodes:

```
[mosipuser@k8Master1 ~]$ sudo vi /etc/hosts
```
With the entry:

```
192.168.1.10 master.phoenixnap.com master-node
192.168.1.20 node1. phoenixnap.com node1 worker-node
```
#### Step 4: Configure Firewall

* The nodes, containers, and pods need to be able to communicate across the cluster to perform their functions.<br> Firewalld is enabled in CentOS by default on the front-end. Add the following ports by entering the listed commands.

On the Master Node enter:

```
[mosipuser@k8Master1 ~]$ sudo firewall-cmd --permanent --add-port=6443/tcp
success
[mosipuser@k8Master1 ~]$ sudo firewall-cmd --permanent --add-port=2379-2380/tcp
success
[mosipuser@k8Master1 ~]$ sudo firewall-cmd --permanent --add-port=10250/tcp
success
[mosipuser@k8Master1 ~]$ sudo firewall-cmd --permanent --add-port=10251/tcp
success
[mosipuser@k8Master1 ~]$ sudo firewall-cmd --permanent --add-port=10252/tcp
success
[mosipuser@k8Master1 ~]$ sudo firewall-cmd --permanent --add-port=10255/tcp
success
[mosipuser@k8Master1 ~]$ sudo firewall-cmd --reload
success
```

On Each Worker Node enter:

```
[mosipuser@k8Worker0 ~]$ sudo firewall-cmd --permanent --add-port=10251/tcp
success
[mosipuser@k8Worker0 ~]$ sudo firewall-cmd --permanent --add-port=10255/tcp
success
[mosipuser@k8Worker0 ~]$ firewall-cmd --reload
success
```
#### Step 5: Update Iptables Settings

* Set the net.bridge.bridge-nf-call-iptables to ‘1’ in your sysctl config file. This ensures that packets are properly processed by IP tables during filtering and port forwarding.

```
[root@k8Master1 mosipuser]# cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

```
[root@k8Master1 mosipuser]# sysctl --system
```

#### Step 6: Disable SELinux

* The containers need to access the host filesystem. SELinux needs to be set to permissive mode, which effectively disables its security functions.

Use following commands to disable SELinux:

```
[root@k8Master1 mosipuser]# sudo setenforce 0
[root@k8Master1 mosipuser]# sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

```

#### Step 7: Disable SWAP

* Lastly, we need to disable SWAP to enable the kubelet to work properly:

```
[root@k8Master1 mosipuser]# sudo sed -i '/swap/d' /etc/fstab
[root@k8Master1 mosipuser]# sudo swapoff -a
```


# How to Deploy a Kubernetes Cluster

#### Step 1: Create Cluster with kubeadm

* Initialize a cluster by executing any one of the following command:

Calico Network, If you want Calico-Pod Network, execute the below command: 

```
 kubeadm init --apiserver-advertise-address=<ip-address-of-kmaster-vm> --pod-network-cidr=192.168.0.0/16
```

Flannel Network, If you want flannel-Pod Network, execute the below command:

```
kubeadm init --apiserver-advertise-address=<ip-address-of-kmaster-vm> --pod-network-cidr=100.96.0.0/16
```

* When you run this command this may generate error 

```
[mosipuser@k8Master1 ~]$ kubeadm init --apiserver-advertise-address=192.168.122.175 --pod-network-cidr=192.168.0.0/16
[init] Using Kubernetes version: v1.20.0
[preflight] Running pre-flight checks
    [WARNING Firewalld]: firewalld is active, please ensure ports [6443 10250] are open or your cluster may not function correctly
error execution phase preflight: [preflight] Some fatal errors occurred:
    [ERROR NumCPU]: the number of available CPUs 1 is less than the required 2
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
```

* Because, you must be 'root user' or must have sudo access and machine must have 2GB RAM and minimum 2 vcpu's


* So, to do that, On host machine, Follow these steps:
 
 
Edit each VM and make it to 2GB RAM and minimum 2 vcpu's :

```
syed@syed-Latitude-3490:~$ virsh edit k8Master1 
```

```
 <memory unit='KiB'>2097152</memory>
 <currentMemory unit='KiB'>2097152</currentMemory>
 <vcpu placement='static'>2</vcpu
```

Shutdown and Start that VM machine

```
syed@syed-Latitude-3490:~$ virsh shutdown k8Master1 
Domain k8Master1 is being shutdown

syed@syed-Latitude-3490:~$ virsh start k8Master1 
Domain k8Master1 started
```

Now, Run the following command:

```
[mosipuser@k8Master1 ~]$ sudo kubeadm init --apiserver-advertise-address=192.168.122.175 --pod-network-cidr=192.168.0.0/16
[init] Using Kubernetes version: v1.20.0
[preflight] Running pre-flight checks
    [WARNING Firewalld]: firewalld is active, please ensure ports [6443 10250] are open or your cluster may not function correctly
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
....
....
To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
....
....  
kubeadm join 192.168.122.67:6443 --token dpms5h.8b4dxwl9mdjc4veb \
    --discovery-token-ca-cert-hash sha256:59a9c5cae741b57f4d16b07a6d381a7512118af8ba7401e377d06c6f8db50c3f
```

#### Step 2: Manage Cluster as Regular User

* To start using the cluster you need to run it as a regular user by typing:

```
[mosipuser@k8Master1 ~]$ mkdir -p $HOME/.kube
[mosipuser@k8Master1 ~]$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[mosipuser@k8Master1 ~]$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
* To verify, if kubectl is working or not, run the following command:

```
[mosipuser@k8Master1 ~]$ kubectl get pods -o wide --all-namespaces
NAMESPACE     NAME                                READY   STATUS    RESTARTS   AGE     IP                NODE        NOMINATED NODE   READINESS GATES
kube-system   coredns-74ff55c5b-llflc             0/1     Pending   0          6m8s    <none>            <none>      <none>           <none>
kube-system   coredns-74ff55c5b-vlxpz             0/1     Pending   0          6m8s    <none>            <none>      <none>           <none>
kube-system   etcd-k8master1                      1/1     Running   0          6m16s   192.168.122.175   k8master1   <none>           <none>
kube-system   kube-apiserver-k8master1            1/1     Running   0          6m16s   192.168.122.175   k8master1   <none>           <none>
kube-system   kube-controller-manager-k8master1   1/1     Running   0          6m16s   192.168.122.175   k8master1   <none>           <none>
kube-system   kube-proxy-kcwpp                    1/1     Running   0          6m9s    192.168.122.175   k8master1   <none>           <none>
kube-system   kube-scheduler-k8master1            1/1     Running   0          6m16s   192.168.122.175   k8master1   <none>           <none>
```
It may take time to start Running, wait for it.


#### Step 3: Set Up Pod Network

* There are two types of Pod Network 

```
1. CALICO POD NETWORK
2. FLANNEL POD NETWORK
```
* You will notice from the previous command, that all the pods are running except one: ‘kube-dns’. 
* For resolving this we will install a pod network.
 
To install the CALICO pod network, run the following command:

```
[mosipuser@k8Master1 ~]$ curl https://docs.projectcalico.org/manifests/calico.yaml -O
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  183k  100  183k    0     0  30073      0  0:00:06  0:00:06 --:--:-- 39695
```
```
[mosipuser@k8Master1 ~]$ kubectl apply -f calico.yaml
configmap/calico-config unchanged
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org configured
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org configured
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org configured
....
....
serviceaccount/calico-kube-controllers unchanged
poddisruptionbudget.policy/calico-kube-controllers created

```

If there is an error to download that calico.yaml file.
You can get this file from this github repository.
* Download Calico.yaml file from this github repository and run the following command

```
[mosipuser@k8Master1 ~]$ kubectl apply -f https://github.com/123iris/Kubernetes/calico.yaml 

```

To verify, if kubectl is working or not, run the following command:

```
[mosipuser@k8Master1 ~]$ kubectl get pods -o wide --all-namespaces
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE     IP               NODE        NOMINATED NODE   READINESS GATES
kube-system   calico-kube-controllers-744cfdf676-9vm57   1/1     Running   0          2m56s   192.168.145.1    k8master1   <none>           <none>
kube-system   calico-node-p2dks                          1/1     Running   0          2m56s   192.168.122.67   k8master1   <none>           <none>
kube-system   coredns-74ff55c5b-7s86p                    1/1     Running   0          6m      192.168.145.2    k8master1   <none>           <none>
kube-system   coredns-74ff55c5b-fz9h8                    1/1     Running   0          6m      192.168.145.3    k8master1   <none>           <none>
kube-system   etcd-k8master1                             1/1     Running   0          6m8s    192.168.122.67   k8master1   <none>           <none>
kube-system   kube-apiserver-k8master1                   1/1     Running   0          6m8s    192.168.122.67   k8master1   <none>           <none>
kube-system   kube-controller-manager-k8master1          1/1     Running   0          6m8s    192.168.122.67   k8master1   <none>           <none>
kube-system   kube-proxy-8dm4r                           1/1     Running   0          6m      192.168.122.67   k8master1   <none>           <none>
kube-system   kube-scheduler-k8master1                   1/1     Running   0          6m8s    192.168.122.67   k8master1   <none>           <none>
```

NOTE: It Will take time to initialize the pods. Please Have Patience !!!


If you want to Install FLANNEL POD NETWORK

download the kube-flannel.yml file 

```
[mosipuser@k8Master1 ~]$ curl https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml -O
[mosipuser@k8Master1 ~]$ kubectl apply -f kube-flannel.yml
```

If there is an error to download that kube-flannel.yml file.
You can get this file from this github repository.
* Download kube-flannel.yaml file from this github repository and run the following command

```
[mosipuser@k8Master1 ~]$ kubectl apply -f https://github.com/123iris/Kubernetes/kube-flannel.yml 
```

#### Step 4: Install Kubernetes Dashboard

* To install Kubernetes Dashboard, Install nginx on VM machine. [link](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-centos-7)

```
sudo yum install epel-release -y
sudo yum install nginx -y
sudo systemctl start nginx
sudo firewall-cmd --permanent --zone=public --add-service=http 
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
sudo systemctl enable nginx
sudo systemctl start nginx
```

To test whether nginx is installed in your machine, use this logic

```
http://server_domain_name_or_IP/
```

*  Next, we will install the dashboard. To install the Dashboard, run the following command:

```
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
```

If there is an Error to Open the Above link, you can get the kubernetes-dashboard.yaml file from your Git-Repository 

```
kubectl create -f https://github.com/123iris/Kubernetes/kubernetes-dashboard.yaml
```

* To check whether Your dashboard is now ready with it’s the pod in the running state. 

```
[mosipuser@k8Master1 ~]$ kubectl get pods -o wide --all-namespaces
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE    IP               NODE        NOMINATED NODE   READINESS GATES
kube-system   calico-kube-controllers-744cfdf676-9vm57   1/1     Running   0          136m   192.168.145.1    k8master1   <none>           <none>
kube-system   calico-node-p2dks                          1/1     Running   0          136m   192.168.122.67   k8master1   <none>           <none>
kube-system   coredns-74ff55c5b-7s86p                    1/1     Running   0          139m   192.168.145.2    k8master1   <none>           <none>
kube-system   coredns-74ff55c5b-fz9h8                    1/1     Running   0          139m   192.168.145.3    k8master1   <none>           <none>
kube-system   etcd-k8master1                             1/1     Running   0          139m   192.168.122.67   k8master1   <none>           <none>
kube-system   kube-apiserver-k8master1                   1/1     Running   0          139m   192.168.122.67   k8master1   <none>           <none>
kube-system   kube-controller-manager-k8master1          1/1     Running   0          139m   192.168.122.67   k8master1   <none>           <none>
kube-system   kube-proxy-8dm4r                           1/1     Running   0          139m   192.168.122.67   k8master1   <none>           <none>
kube-system   kube-scheduler-k8master1                   1/1     Running   0          139m   192.168.122.67   k8master1   <none>           <none>
kube-system   kubernetes-dashboard-6ff6454fdc-xw4n5      1/1     Running   0          90m    192.168.145.4    k8master1   <none>           <none>
```

* InOrder to Access Kubernetes DashBoard of VM machine remotely. Follow these steps

```
[mosipuser@k8Master1 ~]$ kubectl -n kube-system edit service kubernetes-dashboard
```
change the spec.type ClusterIP to NodePort

```

spec:
  clusterIP: 10.102.90.180
  clusterIPs:
  - 10.102.90.180
  externalTrafficPolicy: Cluster
  ports:
  - nodePort: 30536
    port: 443
    protocol: TCP
    targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard
  sessionAffinity: None
  type: NodePort   ## here change ClusterIP to NodePort
  
```

* To check whether Kubernetes Dashboard service is available use the below command. 

```

[mosipuser@k8Master1 ~]$ kubectl get services -n kube-system
NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
kube-dns               ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP,9153/TCP   178m
kubernetes-dashboard   NodePort    10.102.90.180   <none>        443:30536/TCP            129m

```

* Now, Create a Dashboard service account using the following commands:

```

[mosipuser@k8Master1 ~]$ kubectl create serviceaccount dashboard -n default
serviceaccount/dashboard created

[mosipuser@k8Master1 ~]$ kubectl create clusterrolebinding dashboard-admin -n default --clusterrole=cluster-admin  --serviceaccount=default:dashboard
clusterrolebinding.rbac.authorization.k8s.io/dashboard-admin created

[mosipuser@k8Master1 ~]$ kubectl get secret $(kubectl get serviceaccount dashboard -o jsonpath="{.secrets[0].name}") -o jsonpath="{.data.token}" | base64 --decode

# copy this below token. It is very important InOrder to login in Kubernetes Dashboard
eyJhbGciOiJSUzI1NiIsImtpZCI6Ik1hQ3lVdGE4SmViVTIyVmQ0R19FRUpEV3dMZHlkQW5Kd2dIa09nX2doWWcifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRhc2hib2FyZC10b2tlbi02MmZoeCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJkYXNoYm9hcmQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJkMmM1YzQ3My03M2ZjLTQ0NTEtOTc5NS1iMDA3YmRjYjA3ODgiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVmYXVsdDpkYXNoYm9hcmQifQ.abEKZaUphDEvQhHT02JEVWYxjjfN5ahoEieM0yQHfu2CJ8WGntcTrAy67z_ocO-7o_4_k08hn_yTGP06w_zOTVw3sRdUWKyORu0Bhcab_o1KOFKkjpXBOkCZSCOflJPHTKUDZk944ckpG5JDzFVM6PXqzi3cY0mOCBkda24_4Hf4Z1kdaBYu0Fctmf19bMlHsPkyNsaU2uejhZ04MTVGZFXyulKkMDwPbaUsggkAQ_ysj1-B1qcQ0ma6qyypgv2ePczsAXKv9kiRbEI3lxP2SRzSgxUwTAiVLDhZyp4Zn6vHHjNTGL6RhPdwBPIynBG5Vs6OiEwLl_k94DYHmeGHyA

```

* InOrder to Access Kubernetes Dashboard on local machine ( i.e. On master machine ), run the following command:

```
[mosipuser@k8Master1 ~]$ kubectl proxy
Starting to serve on 127.0.0.1:8001

```
Open browser and type:

```
http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/
```

* InOrder to Access Kubernetes Dashboard copy port number from below command. [i.e port 30536]

```
[mosipuser@k8Master1 ~]$ kubectl get services -n kube-system
NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
kube-dns               ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP,9153/TCP   178m
kubernetes-dashboard   NodePort    10.102.90.180   <none>        443:30536/TCP            129m
```

*  Open Browser, And type the https://<VM IP ADDRESS>:30536/ 
   Select Token Option
   

```
https://<IP ADDRESS of VM>:<NodePort>
```

```
https://192.168.122.67:30536/
```

**Note : when open this "https://192.168.122.67:30536/" you may get warning. Don't be afraid.**

**Click on Advance option ---> Click on Proceed with https://192.168.122.67:30536/**

* Copy the output from the below command and paste it in browser and click on sign-In:

```

[mosipuser@k8Master1 ~]$ kubectl get secret $(kubectl get serviceaccount dashboard -o jsonpath="{.secrets[0].name}") -o jsonpath="{.data.token}" | base64 --decode

eyJhbGciOiJSUzI1NiIsImtpZCI6Ik1hQ3lVdGE4SmViVTIyVmQ0R19FRUpEV3dMZHlkQW5Kd2dIa09nX2doWWcifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRhc2hib2FyZC10b2tlbi02MmZoeCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJkYXNoYm9hcmQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJkMmM1YzQ3My03M2ZjLTQ0NTEtOTc5NS1iMDA3YmRjYjA3ODgiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVmYXVsdDpkYXNoYm9hcmQifQ.abEKZaUphDEvQhHT02JEVWYxjjfN5ahoEieM0yQHfu2CJ8WGntcTrAy67z_ocO-7o_4_k08hn_yTGP06w_zOTVw3sRdUWKyORu0Bhcab_o1KOFKkjpXBOkCZSCOflJPHTKUDZk944ckpG5JDzFVM6PXqzi3cY0mOCBkda24_4Hf4Z1kdaBYu0Fctmf19bMlHsPkyNsaU2uejhZ04MTVGZFXyulKkMDwPbaUsggkAQ_ysj1-B1qcQ0ma6qyypgv2ePczsAXKv9kiRbEI3lxP2SRzSgxUwTAiVLDhZyp4Zn6vHHjNTGL6RhPdwBPIynBG5Vs6OiEwLl_k94DYHmeGHyA

```

* When you try to sign-In. It will display Unknown server 404 ERROR 

```
Unknown Server Error (404)
the server could not find the requested resource
Redirecting to previous state in 3 seconds...

```
* **It will take 3 seconds to get back to login page. So, Within this 3 seconds click on create icon. Then you will be able to use Kubernetes Dashboard remotely.**


# Reset Kubernetes Cluster

If you want to reset Kubernetes Cluster on some particular nodes. Execute this command 

```
kubeadm reset
```

# Join the Cluster 

* If you want to join a particular Kubernetes cluster. Get the kubeadm join command from master node

* **Make sure you are root user or have sudo access**.

Example :

On Worker Node,

```
[mosipuser@k8Worker0 ~]$ sudo kubeadm join 192.168.122.67:6443 --token dpms5h.8b4dxwl9mdjc4veb \
     --discovery-token-ca-cert-hash sha256:59a9c5cae741b57f4d16b07a6d381a7512118af8ba7401e377d06c6f8db50c3f
```

# Check Kubernetes nodes

* Execute the below command to get to know how machine nodes are available on Kubernetes cluster.
* If Roles are none, it means it is a worker node.
* It will take time for machines to get 'Ready' STATUS. So please wait for sometime !!!

On Master Node,

```
[mosipuser@k8Master1 ~]$ kubectl get nodes
NAME        STATUS   ROLES                  AGE   VERSION
k8master1   Ready    control-plane,master   21h   v1.20.0
k8worker0   Ready    <none>                 16m   v1.20.0
```

If you Execute this command on Worker Node, It will through error

On Worker Node,

```
[mosipuser@k8Worker0 ~]$ kubectl get nodes
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

# References 

1. [phoenixnap--How-to-install-kubernetes-on-centos ](https://phoenixnap.com/kb/how-to-install-kubernetes-on-centos)
2. [Edureka--Install-kubernetes-on-ubuntu](https://www.edureka.co/blog/install-kubernetes-on-ubuntu)
3. [digitalocean--Nginx on centos ](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-centos-7)
4. [docs.projectcalico.org](https://docs.projectcalico.org/getting-started/kubernetes/self-managed-onprem/onpremises)
5. [raw.githubusercontent.com-kube-flannel.yml](https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml)
6. [raw.githubusercontent.com-kubernetes-dashboard.yaml](https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml)
7. [stackoverflow.com](https://stackoverflow.com/questions/39864385/how-to-access-expose-kubernetes-dashboard-service-outside-of-a-cluster)