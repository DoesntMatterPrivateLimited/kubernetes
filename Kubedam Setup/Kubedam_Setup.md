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
![image](https://github.com/user-attachments/assets/a1be78ff-b56d-4304-9279-b5d441a77f69)


### Initial Node Setup

- **Step1: Update cluster:**

```
sudo apt-get update
```
---




































































