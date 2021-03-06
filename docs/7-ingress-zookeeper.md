1. 创建PV、PVC
```
# cat zk-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
    name: pv-zk1
    annotations:
      volume.beta.kubernetes.io/storage-class: "anything"     #对应的pv class 名
    labels:
      type: local
spec:
    capacity:
      storage: 2Gi
    accessModes:
      - ReadWriteOnce
    hostPath:
      path: "/opt/data/zookeeper"
    persistentVolumeReclaimPolicy: Recycle
---
apiVersion: v1
kind: PersistentVolume
metadata:
    name: pv-zk2
    annotations:
      volume.beta.kubernetes.io/storage-class: "anything"
    labels:
      type: local
spec:
    capacity:
      storage: 2Gi
    accessModes:
      - ReadWriteOnce
    hostPath:
      path: "/opt/data/zookeeper"
    persistentVolumeReclaimPolicy: Recycle
---
apiVersion: v1
kind: PersistentVolume
metadata:
    name: pv-zk3
    annotations:
      volume.beta.kubernetes.io/storage-class: "anything"
    labels:
      type: local
spec:
    capacity:
      storage: 2Gi
    accessModes:
      - ReadWriteOnce
    hostPath:
      path: "/opt/data/zookeeper"
    persistentVolumeReclaimPolicy: Recycle
    
# kubectl apply -f zk-pv.yaml
```
2. 创建zookeeper
```
#cat zookeeper.yaml
apiVersion: v1
kind: Service
metadata:
  name: pvc-zk-hs
  labels:
    app: zk
spec:
  ports:
  - port: 2888
    name: server
  - port: 3888
    name: leader-election
  clusterIP: None
  selector:
    app: zk
---
apiVersion: v1
kind: Service
metadata:
  name: svc-zk-cs
  labels:
    app: zk
spec:
  ports:
  - port: 2181
    name: client
  selector:
    app: zk
---
apiVersion: policy/v1beta1
kind: PodDisruptionBudget
metadata:
  name: zk-pdb
spec:
  selector:
    matchLabels:
      app: zk
  maxUnavailable: 1
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: zk
spec:
  selector:
    matchLabels:
      app: zk
  serviceName: pvc-zk-hs
  replicas: 3
  updateStrategy:
    type: RollingUpdate
  podManagementPolicy: OrderedReady   #注意这里使用OrderedReady顺序启动方式，Parallel方式可能会报连接错误的问题
  template:
    metadata:
      labels:
        app: zk
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: "app"
                    operator: In
                    values:
                    - zk
              topologyKey: "kubernetes.io/hostname"
      containers:
      - name: kubernetes-zookeeper
        imagePullPolicy: Always
        image: "k8s.gcr.io/kubernetes-zookeeper:1.0-3.4.10"
        resources:
          requests:
            memory: "1Gi"
            cpu: "0.5"
        ports:
        - containerPort: 2181
          name: client
        - containerPort: 2888
          name: server
        - containerPort: 3888
          name: leader-election
        command:
        - sh
        - -c
        - "start-zookeeper \
          --servers=3 \
          --data_dir=/var/lib/zookeeper/data \
          --data_log_dir=/var/lib/zookeeper/data/log \
          --conf_dir=/opt/zookeeper/conf \
          --client_port=2181 \
          --election_port=3888 \
          --server_port=2888 \
          --tick_time=2000 \
          --init_limit=10 \
          --sync_limit=5 \
          --heap=512M \
          --max_client_cnxns=1024 \
          --snap_retain_count=3 \
          --purge_interval=12 \
          --max_session_timeout=40000 \
          --min_session_timeout=4000 \
          --log_level=INFO"
        readinessProbe:
          exec:
            command:
            - sh
            - -c
            - "zookeeper-ready 2181"
          initialDelaySeconds: 10
          timeoutSeconds: 5
        livenessProbe:
          exec:
            command:
            - sh
            - -c
            - "zookeeper-ready 2181"
          initialDelaySeconds: 10
          timeoutSeconds: 5
        volumeMounts:
        - name: datadir
          mountPath: /var/lib/zookeeper
  volumeClaimTemplates:
  - metadata:
      name: datadir
      annotations:
        volume.beta.kubernetes.io/storage-class: "anything"   #这个要和创建的pv对应
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 2Gi
kubectl apply -f zookeeper.yaml
```
3. 添加ingress tcp转发
```
#修改 mandatory.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---

kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-configuration
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---
kind: ConfigMap
apiVersion: v1
metadata:
  name: tcp-services
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
data:                                 #添加
  2181: "default/svc-zk-cs:2181"      #添加
##格式:
<ZK port>: <namespace/service name>:<service port>:[PROXY]:[PROXY]
#kubectl apply -f mandatory.yaml
```
4.修改service-nodeport.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  type: NodePort
  ports:
    - name: http
      port: 80
      targetPort: 80
      protocol: TCP
      nodePort: 8480
    - name: https
      port: 443
      targetPort: 443
      protocol: TCP
      nodePort: 8443
    - name: zk-tcp       #新加
      port: 2181         #新加
      targetPort: 2181   #新加
      protocol: TCP      #新加
      nodePort: 8421     #新加
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
#kubectl apply -f service-nodeport.yaml
```
