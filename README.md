# 🚀 MySQL on Kubernetes

This project demonstrates how to deploy a **MySQL database** on a **Kubernetes cluster** using YAML manifests, expose it via a Kubernetes Service, and access it internally from within the cluster.

---

## 📋 Table of Contents

- [Prerequisites](#prerequisites)
- [Architecture](#architecture)
- [Step 1: Deploy MySQL Pod](#step-1-deploy-mysql-pod)
- [Step 2: Expose MySQL with a Service](#step-2-expose-mysql-with-a-service)
- [Step 3: Access MySQL from Within the Cluster](#step-3-access-mysql-from-within-the-cluster)
- [Troubleshooting](#troubleshooting)
- [Next Steps (Optional Enhancements)](#next-steps-optional-enhancements)
- [File Structure](#file-structure)
- [License](#license)

---

## ✅ Prerequisites

- A running Kubernetes cluster (Minikube, KIND, or cloud provider)
- `kubectl` CLI installed and configured
- Basic understanding of YAML and Kubernetes resources

---

## 📐 Architecture

```
+-------------------+       ClusterIP       +-----------------------+
| MySQL Client Pod  |  <----------------->  |     MySQL Pod         |
| (temporary shell) |                       |  +-----------------+  |
|                   |                       |  | MySQL Container |  |
+-------------------+                       |  +-----------------+  |
                                            +-----------------------+
                                                 mysql-service
```

---

## 🧱 Step 1: Deploy MySQL Pod

Create the file: `mysql-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  containers:
    - name: mysql
      image: mysql:8.0
      env:
        - name: MYSQL_ROOT_PASSWORD
          value: my-secret-password
      ports:
        - containerPort: 3306
```

Apply the manifest:

```bash
kubectl apply -f mysql-pod.yaml
```

---

## 🌐 Step 2: Expose MySQL with a Service

Create the file: `mysql-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  selector:
    app: mysql
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
  type: ClusterIP
```

Apply the manifest:

```bash
kubectl apply -f mysql-service.yaml
```

---

## 🔍 Step 3: Access MySQL from Within the Cluster

Start a temporary MySQL client pod:

```bash
kubectl run -it mysql-client --image=mysql:8.0 --rm --restart=Never -- bash
```

Connect to the MySQL server:

```bash
mysql -h mysql-service -uroot -p
```

> When prompted, enter the password: `my-secret-password`

---

---

## 🔍 Step 4: Access MySQL from outside the Cluster
Modify the service in the yaml file:
```bash
type: NodePort
```
Then get the external port:
```bash
kubectl get svc
```
Also get the node ip:
```bash
kubectl get nodes -o wide
```
Now try to connect to mysql from outside:
```bash
mysql -h <node-ip> -P <external-port> -uroot -p
```
---

## 🛠️ Troubleshooting

| Problem                                  | Solution                                                                 |
|------------------------------------------|--------------------------------------------------------------------------|
| `Can't connect to mysql-service`         | Ensure pod has `app: mysql` label, and service `selector` matches it    |
| `Connection refused` (Error 111)         | MySQL may be bound to `127.0.0.1`; change to `0.0.0.0` if needed         |
| Pod in `CrashLoopBackOff`                | Use `kubectl logs mysql` to check startup issues                        |
| No endpoints in service                  | Pod label missing or mismatch with service selector                     |
| DNS name not resolving                   | Check `kubectl exec` + `nslookup mysql-service` from client pod         |

---

## 📈 Next Steps (Optional Enhancements)

- 🔐 Use `Kubernetes Secrets` for secure password management
- 💾 Add `PersistentVolumeClaim` for MySQL data persistence
- 🔁 Migrate to `Deployment` or `StatefulSet` for better lifecycle control
- 🌍 Use `NodePort` or `LoadBalancer` service type for external access
- 🛡️ Add network policies, resource limits, and liveness/readiness probes
- 📦 Schedule automatic backups using `CronJob` or external tools

---

## 📂 File Structure

```bash
.
├── mysql-pod.yaml         # MySQL Pod manifest
└── mysql-service.yaml     # Service to expose MySQL internally
```

---

## 🪪 License

This project is open-sourced under the [MIT License](LICENSE).

---

## 🙋‍♂️ Author

**Himanshu Gogoi**  
[Your GitHub](https://github.com/your-username)  
Feel free to fork and modify!
