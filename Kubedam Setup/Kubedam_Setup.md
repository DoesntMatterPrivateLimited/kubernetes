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

Install UFW

```shell
sudo ufw status
sudo ufw enable
sudo ufw allow 22
```


