# Week 4
## Cai dat moi truong minikube
### Chuan bi : 
1. Install kubectl 
   - Install curl 
    ```sh
    sudo apt install curl 
    ```
   - Dowload latest release with the command:
    ```sh
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    ```
   - Install kubectl
    ```sh
    sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
    ```
   - Test to ensure the version you installed is up-to-date:
    ```sh
    kubectl version --client
    ```
    > Ket qua thuc hien <br/>
  ![alt text]( "")
2. Install minikube
    - Recommended configuration
    ```sh
        2 CPUs or more
        2GB of free memory
        20GB of free disk space
        Internet connection
        Container or virtual machine manager, such as: Docker,      Hyperkit, Hyper-V, KVM, Parallels, Podman, VirtualBox, or VMWare
    ```
    - Dowload latest release
    ```sh
    curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
    ```
    - Install 
    ```sh 
    sudo install minikube-linux-amd64 /usr/local/bin/minikube
    ```
3. Start a cluster using docker driver
   ```sh
   minikube start --driver=docker
   ```
   To make docker the default driver:
   ```sh
   minikube config set driver docker
   ```
    > Tạo một local Kubernetes gồm 1 node chạy trên  container minikube <br/>
  ![alt text]( "") 

    > kubectl version <br/>
  ![alt text]( "")

## Cac buoc thuc hien deploy wordpress
- Tao thu muc lam viec 
  ```sh 
  mkdir minikube_test
  cd minikube_test
  ```
1. Deploy mariadb 
    - Tao PersistentVolumes (mariadb-pv) 
    - Tao PersistentVolumeClaims (mariadb-pvc)
    - Tao service ,deployment mariadb 
- Tao PersistentVolumes
    ```sh 
    touch mariadb_PV.yaml
    ```
    > file mariadb_PV.yaml
```sh
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mariadb-pv
spec:
  storageClassName: manual
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```
- Tao PersistentVolumeClaim
  ```sh
  touch mariadb_PVC.yaml
  ```
> file mariadb_PVC.yaml
```sh
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mariadb-pvc
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```
- Deployment mariadb
  ```sh
  touch deploy-mariadb.yaml
  ```
> file deploy-mariadb.yaml
```sh
---
apiVersion: v1
kind: Service
metadata:
  name: mariadb
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mariadbbitnami
  clusterIP: None
---  
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mariadb
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mariadbbitnami
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: mariadbbitnami
    spec:
      initContainers:
      - name: volume-permissions
        image: busybox
        command: ['sh', '-c', 'chmod -R g+rwX /bitnami']
        volumeMounts:
        - mountPath: /bitnami
          name: mariadb-data
      containers:
      - image: bitnami/mariadb:latest
        name: mariadbbitnami
        env:
        - name: MARIADB_USER
          value: bn_wordpress
        - name: MARIADB_PASSWORD
          value: bitnami
        - name: MARIADB_DATABASE
          value: bitnami_wordpress
        - name: ALLOW_EMPTY_PASSWORD
          value: "yes"
        ports:
        - containerPort: 3306
          name: mariadbbitnami
        volumeMounts:
        - name: mariadb-data
          mountPath: /bitnami
      volumes:
      - name: mariadb-data
        persistentVolumeClaim:
          claimName: mariadb-pvc
---
```
> Note: command phía dưới dùng để cấp quyền ghi cho container mariadb trên persistent directory  
```sh
initContainers:
      - name: volume-permissions
        image: busybox
        command: ['sh', '-c', 'chmod -R g+rwX /bitnami']
        volumeMounts:
        - mountPath: /bitnami
          name: mariadb-data
```
- Deploy mariadb: 
  ```sh
  kubectl apply -f mariadb_PV.yaml
  kubectl apply -f mariadb_PVC.yaml
  kubectl apply -f deploy-mariadb.yaml
  ```
  >Ket qua thuc hien <br/>
  ![alt text]( "")
  
  - Kiem tra trang thai cac pods 
    ```sh
    kubectl get pods 
    ```
    ![alt text]( "")
  - Descirbe pod mariadb
    ```sh
    kubectl describe pod mariadb
    ```
    ![alt text]( "")

2. Deploy wordpress
    - Tao PersistentVolumes (wordpress-pv) 
    - Tao PersistentVolumeClaims (wordpress-pvc)
    - Tao service ,deployment wordpress 
- Tao PersistentVolumes
    ```sh 
    touch wordpress_PV.yaml
    ```
    > file wordpress_PV.yaml
```sh
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: wordpress-pv
spec:
  storageClassName: manual
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```
- Tao PersistentVolumeClaim
  ```sh
  touch wordpress_PVC.yaml
  ```
> file wordpress_PVC.yaml
```sh
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wordpress-pvc
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```
- Deployment wordpress
  ```sh
  touch deploy-wordpress.yaml
  ```
> file deploy-wordpress.yaml
```sh
---
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  ports:
    - port: 8080
  selector:
    app: wordpress
  type: LoadBalancer
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      containers:
      - image: bitnami/wordpress:latest
        name: wordpress
        env:
        - name: WORDPRESS_DATABASE_USER
          value: bn_wordpress
        - name: WORDPRESS_DATABASE_PASSWORD
          value: bitnami
        - name: WORDPRESS_DATABASE_NAME
          value: bitnami_wordpress
        ports:
        - containerPort: 8080
          name: wordpress
        volumeMounts:
        - name: wordpress-data
          mountPath: /bitnami
      volumes:
      - name: wordpress-data
        persistentVolumeClaim:
          claimName: wordpress-pvc
```
- Deploy wordpress: 
  ```sh
  kubectl apply -f wordpress_PV.yaml
  kubectl apply -f wordpress_PVC.yaml
  kubectl apply -f deploy-wordpress.yaml
  ```
  >Ket qua thuc hien <br/>
  ![alt text]( "")
  
  - Open Kubernetes dashboard trong browser
    ```sh
    minikube dashboard
    ```
    
    ![alt](  "")
  - Kiem tra trang thai cac pods 
    ```sh
    kubectl get pods 
    ```
    ![alt text]( "")
  - Descirbe pod wordpress
    ```sh
    kubectl describe pod wordpress
    ```
    ![alt text]( "")
  - Get link trang ket qua 
    ```sh
    minikube service wordpress --url
    ```
    ![alt text]( "")
    > http://192.168.49.2:30521
1. Kiem tra ket qua 
   ```sh
   minikube ssh
   ```
   > ssh vao container minikube
   ```sh
   curl -k http://192.168.49.2:30521
   ```
   > ket qua 
   ![alt](  "")

### Tai lieu tham khao 
    https://minikube.sigs.k8s.io/docs/start/
    https://kubernetes.io/docs/tasks/run-application/run-single-instance-stateful-application/
    https://docs.bitnami.com/tutorials/work-with-non-root-containers/
    https://kubernetes.io/vi/docs/reference/kubectl/cheatsheet/

> kubectl cheat sheet

    kubectl get ...
    kubectl describe pods my-pod
    
    kubectl logs my_pod 
    minikube ssh
    minikube dashboard
   
   
