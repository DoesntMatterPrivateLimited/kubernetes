# OS Upgrade Without Application Down

**step1: Verify Cluster & Workloads:**
  
- Before starting the upgrade, check the current state of your cluster:

```shell
kubectl get nodes
kubectl get pods -A -o wide
```
- Ensure your worker node is Ready.
- Identify which workloads are running on it.
---
**step2: Temporarily Move Critical Workloads to Master (if possible):**

- If your master node allows workloads (i.e., does not have the NoSchedule taint), you can schedule some workloads on it.

- Check if the master allows scheduling:

```shell
kubectl describe node <master-node-name> | grep Taints
```
- If the master is tainted (NoSchedule), remove the taint temporarily:

```shell
kubectl taint nodes <master-node-name> node-role.kubernetes.io/control-plane:NoSchedule-
```

- Now, if you have any Deployment-based workloads, you can scale them up to replicate pods on the master:

```shell
kubectl scale deployment <your-deployment> --replicas=2
```
**Note:** If your workloads require multiple nodes (e.g., stateful apps like databases), consider setting up a temporary backup.

---

















































