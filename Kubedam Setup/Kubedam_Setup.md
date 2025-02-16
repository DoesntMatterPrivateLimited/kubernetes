# Kubedam Setup on VM

## Prerequisites:

   - 2 VM
   - Master VM
   - Worker VM

 ### Create VM (ubuntu 22.04)


1. **Nat Networking:**
  - Open VirtualBox > go to `tools` > Create > Properties > Apply.
    
   ![Screenshot ](https://i.imgur.com/Icfo9p2.png)
   ![Screenshot ](https://i.imgur.com/Dy71vSv.png)

2. **Create VM:** 
  - Create Master VM and Worker VM
  - Go to VirtualBox > Setting > Network > add Network Both VirtualMachine Adapter1 `NAT Network` and Adapter2 `Bridged Adapter` > ok.
  - Inside VM open Terminal type `nmtui` > Edit a Connection > edit `wired connection 2` > add IPV4 Configuration > ok.

Install UFW on master and worker

```shell
sudo ufw status
sudo ufw enable
sudo ufw allow 22
```
```shell
sudo apt install openssh-server
```
```shell
ssh-keygen
```
- take ssh on your local

```shell
ssh master@<your_ip_addr>
ssh worker@<your_ip_addr>
```
- add your contaient as per below image

```shell
sudo vim /etc/hosts
```
![image](https://github.com/user-attachments/assets/e5c4bb58-2e90-4c56-a037-7a9a15050281)


### Initial Node Setup

- **Step1: Update cluster:**

```
sudo apt-get update
```
---

- **Step 2: Disable Swap**

```
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```
---
- **Step 3: Add Kernel Parameters**

```
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
```
`sudo modprobe overlay`

`sudo modprobe br_netfilter`

```
sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
```

**Apply sysctl params without reboot**

```
sudo sysctl --system
```

---
### Containerd Runtime Setup and Configuration

`apt install curl -y`

```
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```
```
sudo apt update
```

```
sudo apt install -y containerd.io
```
```
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
```
```
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
```
```
sudo systemctl restart containerd
sudo systemctl enable containerd
systemctl status containerd
```
---
### Installing Kubeadm, Kubelet, and Kubectl

```
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
```
sudo mkdir -p /etc/apt/keyrings/
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
```
sudo apt-get update
apt-cache policy kubelet | head -n 20
```

> Install the required packages, if needed we can request specific version

```
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

**check status of the container** 

`sudo systemctl status kubelet.service`

`sudo systemctl status containerd.service`

> with no cluster configuration yet, kubelet will be crash looping. Bootstraping a cluster will help solve this 

---



































































