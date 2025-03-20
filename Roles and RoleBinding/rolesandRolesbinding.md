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
-- 1.3 Sign Certificates with Kubernetes CA
  


























