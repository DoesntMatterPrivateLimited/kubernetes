# Version Upgrade Without Application Down

**step1: Check Current Kubernetes Version:**

```shell
kubectl get nodes -o wide
kubectl version --short
```
---

**step2: Backup etcd (Master Node):**

- Before upgrading the master node, back up etcd (critical for cluster recovery).

```shell
sudo ETCDCTL_API=3 etcdctl snapshot save /var/lib/etcd/snapshot.db
```
---

**step3: Upgrade Worker Node:**

- Drain the Worker Node
- Before upgrading, remove all workloads from the worker node

```shell
kubectl drain <worker-node-name> --ignore-daemonsets --delete-emptydir-data
```
- This will evict all non-DaemonSet pods from the worker.
---

**step4: Upgrade Kubernetes Components:**

- Update Package Repository

```shell
sudo apt update && sudo apt install -y kubeadm=<NEW_VERSION>
```
- Upgrade Kubelet & Kubectl

```shell
sudo apt install -y kubelet=<NEW_VERSION> kubectl=<NEW_VERSION>
```
- Restart Kubelet

```shell
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```
---

**step5: Upgrade Worker Node:**

- Upgrade Worker Node

```shell
sudo kubeadm upgrade node
```

---

**step6: Uncordon & Verify:**

- Once upgraded, allow scheduling again

```shell
kubectl uncordon <worker-node-name>
kubectl get nodes

```

---

# Upgrade Master Node

**step1: Upgrade Kubeadm on Master:**

```shell
sudo apt update && sudo apt install -y kubeadm=<NEW_VERSION>
```
- Check upgrade plan

```shell
kubeadm upgrade plan
```
- Perform upgrade

```shell
sudo kubeadm upgrade apply v<NEW_VERSION>
```

---

**step2: Upgrade Kubelet & Kubectl:**

```shell
sudo apt install -y kubelet=<NEW_VERSION> kubectl=<NEW_VERSION>
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

---

**step3: Verify & Restore Workloads:**

- Check Cluster Health

```shell
kubectl get nodes
kubectl get pods -A
kubectl cluster-info
```

---

**step4: Restore Workloads (If Needed):**

- If workloads were evicted, restart them manually

```shell
kubectl rollout restart deployment <your-deployment>
```



































