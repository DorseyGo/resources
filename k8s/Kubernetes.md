# Kubernetes + Jenkins

# 1. Kubernetes

## 1.1 Installation



## 1.2 Remove

### 1.2.1 delete kubernetes dashboard

```shell
kubectl get secret,sa,role,rolebinding,services,deployments --namespace=kube-system | grep dashboard
kubectl delete service kubernetes-dashboard -n kube-system
```

## Appendix

## P1. kubeadm init failed

Required images can not be pulled, so change the location of repository:

- registry.cn-hangzhou.aliyuncs.com/google_containers/

# 2. Jenkins

## 2.1 Installation

To install Jenkins, following steps should be executed:

<b>steps 1</b>: <font color="#a00">Create NFS to persistent Jenkins data</font> (<b>optional</b>)

 To prevent the data lose, using <font color="#aa0"><b>NFS</b></font> to persist the data generated from Jenkins. Using the <font color="#a00">jenkins-pv-pvc.yaml</font> to achieve the target,  

```yam
apiVersion: v1
kind: PersistentVolume
metadata:
  name: jenkin
  labels:
    app: jenkin
spec:
  capacity:          
    storage: 20Gi
  accessModes:       
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain  
  mountOptions:   #NFS挂在选项
    - hard
    - nfsvers=4.1    
  nfs:            #NFS设置
    path: /home/kube/nfs/   
    server: 172.17.1.103
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: jenkin
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi 
  selector:
    matchLabels:
      app: jenkin
```

then, using command <font color="#a00">kubectl create -f jenkins-pv-pvc.yaml -n intelli-os</font> to create the persistent volume under the namespace <font color="#aa0">intelli-os</font>. 

> using command <font color="#a00">kubectl create namespace intelli-os</font> create the namespace first, otherwise result in errors.

<b>step 2</b>: <font color="#a00">Create service account</font>

Using the following yaml file to create the service account for Jenkins,

```yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins

---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: jenkins
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["create","delete","get","list","patch","update","watch"]
- apiGroups: [""]
  resources: ["pods/exec"]
  verbs: ["create","delete","get","list","patch","update","watch"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get","list","watch"]
- apiGroups: [""]
  resources: ["events"]
  verbs: ["watch"]
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get"]

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: jenkins
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: jenkins
subjects:
- kind: ServiceAccount
  name: jenkins
```

Using the command <font color="#a00">kubectl create -f ./service-account-yaml -n intelli-os</font>  to create the corresponding service account.

<b>step 3</b>: <font color="#a00">Create RBAC</font>

Using following yaml file to create role based access control rule,

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins-admin       #ServiceAccount名
  namespace: intelli-os     #指定namespace，一定要修改成你自己的namespace
  labels:
    name: jenkin
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: jenkins-admin
  labels:
    name: jenkin
subjects:
  - kind: ServiceAccount
    name: jenkins-admin
    namespace: intelli-os
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

Using command <font color="#a00">kubectl create -f jenkins-rbac.yaml -n intelli-os</font> to create the rule.

<b>step 4</b>: <font color="#a00">Deploy the Jenkins</font>

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: jenkin
  labels:
    app: jenkin
spec:
  type: NodePort
  ports:
  - name: http
    port: 8080                      #服务端口
    targetPort: 8080
    nodePort: 32001                 #NodePort方式暴露 Jenkins 端口
  - name: jnlp
    port: 50000                     #代理端口
    targetPort: 50000
    nodePort: 32002
  selector:
    app: jenkin
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkin
  labels:
    app: jenkin
spec:
  selector:
    matchLabels:
      app: jenkin
  replicas: 1
  template:
    metadata:
      labels:
        app: jenkin
    spec:
      serviceAccountName: jenkins-admin
      containers:
      - name: jenkins
        image: 172.17.1.103:5000/jenkins/jenkins:latest
        securityContext:                     
          runAsUser: 0                      #设置以ROOT用户运行容器
          privileged: true                  #拥有特权
        ports:
        - name: http
          containerPort: 8080
        - name: jnlp
          containerPort: 50000
        resources:
          limits:
            memory: 2Gi
            cpu: "1000m"
          requests:
            memory: 2Gi
            cpu: "1000m"
        env:
        - name: LIMITS_MEMORY
          valueFrom:
            resourceFieldRef:
              resource: limits.memory
              divisor: 1Mi
        - name: "JAVA_OPTS"                 #设置变量，指定时区和 jenkins slave 执行者设置
          value: " 
                   -Xmx$(LIMITS_MEMORY)m 
                   -XshowSettings:vm 
                   -Dhudson.slaves.NodeProvisioner.initialDelay=0
                   -Dhudson.slaves.NodeProvisioner.MARGIN=50
                   -Dhudson.slaves.NodeProvisioner.MARGIN0=0.85
                   -Dhudson.model.DownloadService.noSignatureCheck=true
                   -Duser.timezone=Asia/Shanghai
                 "    
        - name: "JENKINS_OPTS"
          value: "--prefix=/jenkins"         #设置路径前缀加上 Jenkins
        volumeMounts:                        #设置要挂在的目录
        - name: data
          mountPath: /var/jenkins_home
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: jenkin                 #设置PVC
```

using command <font color="#a00">kubectl create -f ./jenkins-deployment.yaml -n intelli-os</font> to deploy the Jenkins to Kubernetes.

## Appendix

Jenkins update site not changed after modify it on the screen web site, such as followings,

![image-20200923112918247](C:\Users\Dorsey\AppData\Roaming\Typora\typora-user-images\image-20200923112918247.png)

<font color="#a00">To guarantee</font> that will take effect, please do such followings as well,

```shell
sed -i 's/https:\/\/updates.jenkins.io\/download/http:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' ./updates/default.json
```

<font color="#a00">It is a MUST</font> to execute the above command, otherwise, no changes made.