# Add Persistent storage with OpenEBS

Following are the basic steps to setup OpenEBS as a Persistent Volume provider for Kubernetes.  Much more info can be found on the [official documentation](https://docs.openebs.io/).

## iSCSI
OpenEBS provides block volume support through the iSCSI protocol. Therefore, the iSCSI client (initiator) presence on all Kubernetes nodes is required. Choose the platform below to find the steps to verify if the iSCSI client is installed and running or to find the steps to install the iSCSI client.

```bash
sudo apt-get install -y open-iscsi
sudo systemctl enable iscsid
sudo systemctl start iscsid
```

Verify that the initiator name is configured and the iSCSI services is running.
```bash
sudo cat /etc/iscsi/initiatorname.iscsi
systemctl status iscsid
```
## Install OpenEBS

OpenEBS can be installed on an existing Kubernetes cluster by applying the openebs-operator.yaml file.
```bash
kubectl apply -f https://openebs.github.io/charts/openebs-operator-1.9.0.yaml
```

We can verify the status of tho OpenEBS pods by looking at the pods in the openebs namespace.
```bash
kubectl get pods -n openebs -o wide
```

## Usage

Within the labs we will focus on using the simple `openebs-hostpath` storageClass for persistentStorage.  To do this make sure that PersistentVolumeClaims specs contain `storageClassName: openebs-hostpath` or set a label on the PersistentVolumeClaim to `volume.beta.kubernetes.io/storage-class: openebs-hostpath`.  I have included two example deployments utilizing OpenEBS hostpath persistent volumes.


###
jenkins.yaml
```yaml
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: jenkins-claim
  annotations:
    volume.beta.kubernetes.io/storage-class: openebs-hostpath
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5G
---
apiVersion: apps/v1
kind: Deployment
metadata:
 name: jenkins
spec:
 replicas: 1
 selector:
   matchLabels:
     app: jenkins-app
 template:
  metadata:
   labels:
    app: jenkins-app
  spec:
   securityContext:
     fsGroup: 1000
   containers:
   - name: jenkins
     imagePullPolicy: IfNotPresent
     image: jenkins/jenkins:lts
     ports:
     - containerPort: 8080
     volumeMounts:  
       - mountPath: /var/jenkins_home
         name: jenkins-home
   volumes:
     - name: jenkins-home
       persistentVolumeClaim:
         claimName: jenkins-claim
---
apiVersion: v1
kind: Service
metadata:
  name: jenkins-svc
spec:
  ports:
    - port: 80
      targetPort: 8080
  selector:
    app: jenkins-app
  type: NodePort
```

_Note: You can connect to Jenkins via the NodePort.  The initial password can be found in the standard output of the jenkins pod.  Use `kubectl logs` to get the password._ 

### percona-mysql-pvc.yaml
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: percona
  labels:
    name: percona
spec:
  replicas: 1
  selector:
    matchLabels:
      name: percona
  template:
    metadata:
      labels:
        name: percona
    spec:
      securityContext:
        fsGroup: 999
      tolerations:
      - key: "ak"
        value: "av"
        operator: "Equal"
        effect: "NoSchedule"
      containers:
        - resources:
            limits:
              cpu: 0.5
          name: percona
          image: percona
          args:
            - "--ignore-db-dir"
            - "lost+found"
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: k8sDem0
          ports:
            - containerPort: 3306
              name: percona
          volumeMounts:
            - mountPath: /var/lib/mysql
              name: demo-vol1
      volumes:
        - name: demo-vol1
          persistentVolumeClaim:
            claimName: demo-vol1-claim
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: demo-vol1-claim
spec:
  storageClassName: openebs-hostpath
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5G
---
apiVersion: v1
kind: Service
metadata:
  name: percona-mysql
  labels:
    name: percona-mysql
spec:
  ports:
    - port: 3306
      targetPort: 3306
  selector:
      name: percona
```
