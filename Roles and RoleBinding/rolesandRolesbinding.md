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
