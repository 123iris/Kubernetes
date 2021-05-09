# Steps to Install Kubernetes on Centos VM

## Introduction

* Kubernetes is a tool for orchestrating and managing Docker containers at scale on on-prem server or across hybrid cloud environments. 
* Kubeadm is a tool provided with Kubernetes to help users install a production ready Kubernetes cluster with best practices enforcement. 
* This tutorial will demonstrate how one can install a Kubernetes Cluster on Ubuntu 20.04 with kubeadm.
* To use Kubernetes, you need to install a containerization engine. <br>
* Currently, the most popular container solution is Docker. 

There are two server types used in deployment of Kubernetes clusters:

* **Master:** A Kubernetes Master is where control API calls for the pods, replications controllers, services, nodes and other components of a Kubernetes cluster are executed.
* **Node:** A Node is a system that provides the run-time environments for the containers. A set of container pods can span multiple nodes.


Minimum Requirement:

* The minimum requirements for the viable setup are:
    * **Memory:** 2 GiB or more of RAM per machine
    * **CPUs:** At least 2 CPUs on the control plane machine.
    * **Internet connectivity** for pulling containers required (Private registry can also be used)
    * **Full network connectivity** between machines in the cluster – This is private or public


## Step 0: Install Docker

* Docker needs to be installed on Ubuntu, both on the Master Node and the Worker Nodes. [Docker install](https://github.com/123iris/Docker/blob/master/Docker%20SetUp/Docker%20install%20on%20Ubuntu%2020.md)

## Step 1: Update & Install packages

* Provision the servers to be used in the deployment of Kubernetes on Ubuntu 20.04. 
* The setup process will vary depending on the virtualization or cloud environment you’re using.

Once the servers are ready, update them.

```
syed@k8sMaster:~$ sudo apt -y install vim git curl wget tmux
syed@k8sMaster:~$ sudo apt -y update
syed@k8sMaster:~$ sudo apt -y upgrade && sudo systemctl reboot
```
## Step 2: Add Kubernetes Signing Key

Since you are downloading Kubernetes from a non-standard repository, it is essential to ensure that the software is authentic. This is done by adding a signing key.

* Install apt packages.

```
syed@k8sMaster:~$ sudo apt -y install curl apt-transport-https
```
* Enter the following to add a signing key:

```
syed@k8sMaster:~$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

ok
```

* Update & reboot.

```
syed@k8sMaster:~$ sudo apt -y update 
syed@k8sMaster:~$ sudo reboot
```

## Step 3: Configure Kubernetes Repository

* Once the servers are rebooted, add Kubernetes repository for Ubuntu 20.04 to all the servers.

```
syed@k8sMaster:~$ echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

## Step 4: Kubernetes Installation Tools

* Kubeadm (Kubernetes Admin) is a tool that helps initialize a cluster. It fast-tracks setup by using community-sourced best practices. Kubelet is the work package, which runs on every node and starts containers. The tool gives you command-line access to clusters.

```
syed@k8sMaster:~$ sudo apt -y install wget kubelet kubeadm kubectl
```
```
syed@k8sMaster:~$ sudo apt-mark hold kubelet kubeadm kubectl

kubelet set on hold.
kubeadm set on hold.
kubectl set on hold.
```
Allow the process to complete.

* Verify the installation with:

```
syed@k8sMaster:~$ kubeadm version

kubeadm version: &version.Info{ Major:"1", 
                                Minor:"21", 
                                GitVersion:"v1.21.0",
                                GitCommit:"cb303e613a121a29364f75cc67d3d580833a7479",
                                GitTreeState:"clean", 
                                BuildDate:"2021-04-08T16:30:03Z", 
                                GoVersion:"go1.16.1", 
                                Compiler:"gc", Platform:"linux/amd64" }
```
				or
```				
syed@k8sMaster:~$ kubectl version --client && kubeadm version

Client Version: version.Info{Major:"1", Minor:"21", GitVersion:"v1.21.0", GitCommit:"cb303e613a121a29364f75cc67d3d580833a7479", GitTreeState:"clean", BuildDate:"2021-04-08T16:31:21Z", GoVersion:"go1.16.1", Compiler:"gc", Platform:"linux/amd64"}
kubeadm version: &version.Info{Major:"1", Minor:"21", GitVersion:"v1.21.0", GitCommit:"cb303e613a121a29364f75cc67d3d580833a7479", GitTreeState:"clean", BuildDate:"2021-04-08T16:30:03Z", GoVersion:"go1.16.1", Compiler:"gc", Platform:"linux/amd64"}

```
 
* Repeat for each node.


## Step 5: Disable Swap

* Start by disabling the swap memory on each server:

```
syed@k8sMaster:~$ sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
syed@k8sMaster:~$ sudo swapoff -a
```

## Step 6: Update Iptables Settings

* Set the net.bridge.bridge-nf-call-iptables to ‘1’ in your sysctl config file. 
  This ensures that packets are properly processed by IP tables during filtering and port forwarding.
  
* Configure sysctl.

```
syed@k8sMaster:~$ sudo modprobe overlay
```
```
syed@k8sMaster:~$ sudo modprobe br_netfilter
```
```
syed@k8sMaster:~$ sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
```
```
syed@k8sMaster:~$ sudo sysctl --system
```

## Step 7: Disable SELINUX 

* The containers need to access the host filesystem. SELinux needs to be set to permissive mode, which effectively disables its security functions.
Use following commands to disable SELinux:

* First, I need to install selinux packages:

```
syed@k8sMaster:~$ sudo apt install policycoreutils 
syed@k8sMaster:~$ sudo apt install policycoreutils-python-utils -y
```

* Disable SElinux:

```
syed@k8sMaster:~$ sudo setenforce 0
setenforce: SELinux is disabled

syed@k8sMaster:~$ sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

## Step 8: Configure Firewall

The nodes, containers, and pods need to be able to communicate across the cluster to perform their functions.
Firewalld is enabled in Ubuntu by default on the front-end. Add the following ports by entering the listed commands.

By default ubuntu 20 uses ufw firewall. so start & enable ufw

```
syed@k8sMaster:~$ sudo systemctl start ufw

syed@k8sMaster:~$ sudo systemctl enable ufw

Synchronizing state of ufw.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable ufw


syed@k8sMaster:~$ sudo ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup

```

On the Master Node enter:

```
syed@k8sMaster:~$  sudo ufw allow from any to any port 22 proto tcp
Rules updated
Rules updated (v6)
syed@k8sMaster:~$  sudo ufw allow from any to any port 443 proto tcp
Rules updated
Rules updated (v6)

syed@k8sMaster:~$ sudo ufw allow from any to any port 80 proto tcp
Rules updated
Rules updated (v6)

syed@k8sMaster:~$ sudo ufw allow from any to any port 6443 proto tcp
Rules updated
Rules updated (v6)

syed@k8sMaster:~$ sudo ufw allow from any to any port 2379 proto tcp
Rules updated
Rules updated (v6)

syed@k8sMaster:~$ sudo ufw allow from any to any port 2380 proto tcp
Rules updated
Rules updated (v6)

syed@k8sMaster:~$ sudo ufw allow from any to any port 10250 proto tcp
Rules updated
Rules updated (v6)

syed@k8sMaster:~$ sudo ufw allow from any to any port 10251 proto tcp
Rules updated
Rules updated (v6)

syed@k8sMaster:~$ sudo ufw allow from any to any port 10252 proto tcp
Rules updated
Rules updated (v6)

syed@k8sMaster:~$ sudo ufw allow from any to any port 10255 proto tcp
Rules updated
Rules updated (v6)

syed@k8sMaster:~$ sudo ufw reload
```
List open ports

```
syed@k8sMaster:~$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
443/tcp                    ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
6443/tcp                   ALLOW       Anywhere                  
2379/tcp                   ALLOW       Anywhere                  
2380/tcp                   ALLOW       Anywhere                  
10250/tcp                  ALLOW       Anywhere                  
10251/tcp                  ALLOW       Anywhere                  
10252/tcp                  ALLOW       Anywhere                  
10255/tcp                  ALLOW       Anywhere                  
443/tcp (v6)               ALLOW       Anywhere (v6)             
80/tcp (v6)                ALLOW       Anywhere (v6)             
6443/tcp (v6)              ALLOW       Anywhere (v6)             
2379/tcp (v6)              ALLOW       Anywhere (v6)             
2380/tcp (v6)              ALLOW       Anywhere (v6)             
10250/tcp (v6)             ALLOW       Anywhere (v6)             
10251/tcp (v6)             ALLOW       Anywhere (v6)             
10252/tcp (v6)             ALLOW       Anywhere (v6)             
10255/tcp (v6)             ALLOW       Anywhere (v6)             
```

On Each Worker Node enter:

```
syed@k8sWorker0:~$ sudo firewall-cmd --permanent --add-port=10251/tcp
success
syed@k8sWorker0:~$ sudo firewall-cmd --permanent --add-port=10255/tcp
success
syed@k8sWorker0:~$ firewall-cmd --reload
success
```

# How to Deploy a Kubernetes Cluster

## Step 1: Create Cluster with kubeadm

* Initialize a cluster by executing any one of the following command:
Calico Network, If you want Calico-Pod Network, execute the below command:

```
syed@k8sMaster:~$ kubeadm init --apiserver-advertise-address=<ip-address-of-kmaster-vm> --pod-network-cidr=192.168.0.0/16
```

# References

1. [computingforgeeks.com-ubuntu.20](https://computingforgeeks.com/deploy-kubernetes-cluster-on-ubuntu-with-kubeadm/)
1. [phoenixnap.com-ubuntu-18](https://phoenixnap.com/kb/install-kubernetes-on-ubuntu)