# Minio Setup

**step1: Verify StorageClass Availability:**
- Before setting up MinIO, check if a StorageClass is available

```shell
kubectl get storageclass
```
- If no storage class exists, we must create one.

---

**step2: Create a StorageClass (Prerequisite):**

- We will define a StorageClass suitable for MinIO. Choose the correct provisioner based on your environment
- For Local Kubernetes (Bare Metal, Minikube)

```shell
cat <<EOF | kubectl apply -f -
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
EOF
```
- This StorageClass is static, meaning PersistentVolumes (PVs) must be created manually.

---
- For Cloud Providers (AWS, GCP, Azure)
- Use the appropriate provisioner

```shell
cat <<EOF | kubectl apply -f -
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/aws-ebs  # Change to match your cloud provider
volumeBindingMode: WaitForFirstConsumer
EOF
```
- For GCP: ``provisioner: `kubernetes.io/gce-pd``
- Fo`r Azure: ``provisioner: kubernetes.io/azure-disk``

---

**step3: Create a PersistentVolume (PV):**
- Since we're using ``kubernetes.io/no-provisioner``, we need to manually create a PersistentVolume.

```shell
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: minio-pv
  namespace: minio
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: standard
  hostPath:
    path: "/mnt/data/minio"
EOF
```
---

**step4: Create a PersistentVolumeClaim (PVC):**
- Now, define a PVC that binds to our PV.

```shell
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: minio-pvc
  namespace: minio
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: standard
EOF
```
---

**step5: Deploy MinIO Service:**
- Since you wanted NodePort, update the Service configuration

```shell
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: minio-service
  namespace: minio
spec:
  selector:
    app: minio
  ports:
  - name: api
    protocol: TCP
    port: 9000
    targetPort: 9000
    nodePort: 30090
  - name: console
    protocol: TCP
    port: 9001
    targetPort: 9001
    nodePort: 30091
  type: NodePort
EOF
```
---

**step6: Deploy MinIO StatefulSet:**
- Now deploy MinIO using a StatefulSet.

```shell
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: minio
  namespace: minio
spec:
  selector:
    matchLabels:
      app: minio
  serviceName: "minio-service"
  replicas: 1
  template:
    metadata:
      labels:
        app: minio
    spec:
      containers:
      - name: minio
        image: minio/minio
        args:
        - server
        - /data
        - "--console-address"
        - ":9001"
        env:
        - name: MINIO_ACCESS_KEY
          value: "minioadmin"
        - name: MINIO_SECRET_KEY
          value: "minioadmin"
        ports:
        - containerPort: 9000
          name: api
        - containerPort: 9001
          name: console
        volumeMounts:
        - name: minio-storage
          mountPath: "/data"
  volumeClaimTemplates:
  - metadata:
      name: minio-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
      storageClassName: standard
EOF
```
---

**step7: Verify MinIO Deployment:**
- Check if all resources are created correctly

```shell
kubectl get pods -n minio
kubectl get svc -n minio
kubectl get pvc -n minio
kubectl get pv
```
---

**step8: Access MinIO:**
- MinIO API: `http://<NodeIP>:30090`
- MinIO Console: `http://<NodeIP>:30091`
- Login Credentials:
- Username: `minioadmin`
- Password: `minioadmin`













