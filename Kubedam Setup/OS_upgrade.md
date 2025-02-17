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
 **step3: Cordon & Drain the Worker Node:**

 - Cordoning prevents new pods from being scheduled while draining safely removes existing ones.

```shell
kubectl cordon <worker-node-name>
kubectl drain <worker-node-name> --ignore-daemonsets --delete-emptydir-data
```
---
**step4: Upgrade the Worker Node OS:**

- Now, upgrade the OS on the worker node.

```shell
sudo apt update && sudo apt upgrade -y
sudo reboot
```
---
**step5: Rejoin the Worker Node to the Cluster:**

- After the node reboots, ensure it re-joins the cluster.

-Check Node Status.

```shell
kubectl get nodes
```
- If the node is NotReady, restart kubelet.

```shell
sudo systemctl restart kubelet
```
- If the worker node is not joining automatically, re-run the kubeadm join command:

```shell
sudo kubeadm join <master-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```
---

**step6: Uncordon the Worker Node & Verify:**

- Once the worker node is Ready, allow scheduling again.

```shell
kubectl uncordon <worker-node-name>
```
- Verify that all pods are running correctly.

```shell
kubectl get pods -A -o wide
```
---

**step7: Restore Master Node (If Taint Was Removed):**

- If you removed the taint from the master in Step 2, reapply it to prevent scheduling workloads on it.

```shell
kubectl taint nodes <master-node-name> node-role.kubernetes.io/control-plane:NoSchedule
```












































