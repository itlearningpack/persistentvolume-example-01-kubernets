# PersistentVolumeClaim(PVC) and PersistentVolume(PV)
## Today, we will see the process of attaching persistent storage to a Kubernetes pod and persisting the logs of the Nginx pod. This involves creating PersistentVolumes (PV) and PersistentVolumeClaims (PVC).
we establish a connection between the Pod, Persistent Volume Claim (PVC), and Persistent Volume (PV) to ensure a robust storage solution. 
To facilitate this, the PVC is bound to the PV, forming a link in the storage chain. The PV, in turn, serves as the bridge between the Kubernetes cluster and the actual storage infrastructure.
This approach is particularly valuable in scenarios where the pod may need to be replaced for any reason. By storing the content on the PV, we guarantee that essential data is retained and remains accessible even after pod replacements, contributing to a more resilient and reliable system.

### Prerequisites
**.    A Kubernetes cluster with the kubectl command-line tool configured to communicate with the cluster.**

**.    Basic knowledge of Kubernetes concepts such as Pods, PersistentVolumes, and PersistentVolumeClaims.**

## 1. Create a PersistentVolume and PersistentVolumeClaim
Create a YAML file, for example, pv1.yaml to define a PersistentVolume. Modify the storage size, access mode, and nfs accordingly.
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
 name: pv1
 labels:
  pv: v1
spec:
 accessModes:
  - ReadWriteMany
 capacity:
  storage: 2Gi
 storageClassName: nfs
 mountOptions:
  - hard
  - nfsvers=4.1
 nfs:
  server: 192.168.126.11
  path: /path
```

Apply the PersistentVolume configuration:

```
kubectl apply -f pv1.yaml
```

Create a YAML file, for example, pvc1.yaml to define a PersistentVolumeClaim that refers to the previously created PersistentVolume.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
 name: pvc1
 labels:
  pvc1: v1
spec:
 storageClassName: nfs
 accessModes:
  - ReadWriteMany
 resources:
  requests:
   storage: 2Gi
```
Apply the PersistentVolumeClaim configuration:

```
kubectl apply -f pvc1.yaml
```
## 2. Create Nginx Pod to use Persistent Storage
In the pod configuration I have included the PersistentVolumeClaim in the volumes section and Modified the volumeMounts section to mount the volume within the container
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      volumes:
       - name: content
         persistentVolumeClaim:
          claimName: pvc1
      containers:
      - image: nginx:latest
        name: nginx
        imagePullPolicy: IfNotPresent
        volumeMounts:
         - name: content
           mountPath: /usr/share/nginx/html
```
Apply the Pod configuration:

```
kubectl apply -f nginx.yaml
```

