**Here's a detailed step-by-step guide to simulate Roles and RoleBindings in your kubeadm Kubernetes setup by creating `dev` and `admin` users.**

**Step 1: Create User Certificates**
- Kubernetes uses client certificates for authentication. You need to create certificates for dev and admin.

- 1.1 Create Private Keys for Users
- Run the following commands on the Kubernetes master node:

```
mkdir -p $HOME/k8s-users
cd $HOME/k8s-users

openssl genrsa -out dev.key 2048
openssl genrsa -out admin.key 2048
```

- 1.2 Create Certificate Signing Requests (CSRs)

```
openssl req -new -key dev.key -out dev.csr -subj "/CN=dev/O=developers"
openssl req -new -key admin.key -out admin.csr -subj "/CN=admin/O=admins"
```
- CN=dev: Username (dev)
- O=developers: Group (developers)
- CN=admin: Username (admin)
- O=admins: Group (admins)


- 1.3 Sign Certificates with Kubernetes CA
- Find your Kubernetes CA files:  

```
ls /etc/kubernetes/pki/
```
- Sign the certificates:

```
sudo openssl x509 -req -in dev.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out dev.crt -days 365 -sha256
sudo openssl x509 -req -in admin.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out admin.crt -days 365 -sha256
```
---

**Step 2: Configure Kubeconfig for Users**

- Now, configure kubectl access for these users.

- 2.1 Add User to Kubeconfig

```
kubectl config set-credentials dev --client-certificate=$HOME/k8s-users/dev.crt --client-key=$HOME/k8s-users/dev.key
kubectl config set-credentials admin --client-certificate=$HOME/k8s-users/admin.crt --client-key=$HOME/k8s-users/admin.key
```
- 2.2 Create User Contexts

```
kubectl config set-context dev-context --cluster=kubernetes --user=dev
kubectl config set-context admin-context --cluster=kubernetes --user=admin
```
- 2.3 Verify Contexts

```
kubectl config get-contexts
```
---

**Step 3: Create RBAC Roles and Bindings**

- 3.1 Create a Role for dev User
- The dev user should have read-only access to pods in the default namespace.

```
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: dev-role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
EOF
```

- 3.2 Bind dev User to Role

```
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-rolebinding
  namespace: default
subjects:
- kind: User
  name: dev
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: dev-role
  apiGroup: rbac.authorization.k8s.io
EOF
```

- 3.3 Create a ClusterRole for admin
- The admin user should have full access to all namespaces.

```
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: admin-clusterrole
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
EOF
```

- 3.4 Bind admin User to ClusterRole

```
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-clusterrolebinding
subjects:
- kind: User
  name: admin
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: admin-clusterrole
  apiGroup: rbac.authorization.k8s.io
EOF
```

---

**Step 4: Test User Access**

- 4.1 Test as dev User
- Switch to the dev context:

```
kubectl config use-context dev-context
```
- Try listing pods in default namespace:

```
kubectl get pods -n default
```
- Expected: Works ✅
- Now, try deleting a pod:

```
kubectl delete pod <pod-name> -n default
```
- Expected: Permission Denied ❌

- 4.2 Test as admin User
- Switch to admin context:

```
kubectl config use-context admin-context
```
- Try listing pods in all namespaces:

```
kubectl get pods --all-namespaces
```
- Expected: Works ✅
- Try deleting a pod:

```
kubectl delete pod <pod-name> -n default
```
- Expected: Works ✅

---

**Step 5: Cleanup (Optional)**

- If you want to remove the configurations:

```
kubectl delete role dev-role -n default
kubectl delete rolebinding dev-rolebinding -n default
kubectl delete clusterrole admin-clusterrole
kubectl delete clusterrolebinding admin-clusterrolebinding
rm -rf $HOME/k8s-users
```












