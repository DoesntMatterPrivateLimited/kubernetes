# OS Upgrade Without Application Down

 **step1: Verify Cluster & Workloads:**
  
- Before starting the upgrade, check the current state of your cluster:

```shell
kubectl get nodes
kubectl get pods -A -o wide
```
- Ensure your worker node is Ready.
- Identify which workloads are running on it.
