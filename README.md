#### 1.1.2 kubectl run으로 pod 생성
```
kubectl run test-nginx --image=nginx:1.19
```

#### 1.1.3 yaml 형식으로 pod 생성
```
apiVersion: v1
kind: Pod
metadata:
  name: test-nginx2
spec:
  containers:
  - name: test-nginx2
    image: nginx:1.19
    ports:
    - containerPort: 80
```

### 1.2 ReplicaSet
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
  labels:
    app: nginx-rs
    tier: nginx-rs
spec:
  # modify replicas according to your case
  replicas: 3
  selector:
    matchLabels:
      tier: nginx-rs
  template:
    metadata:
      labels:
        tier: nginx-rs
    spec:
      containers:
      - name: nginx-rs
        image: nginx
```

### 1.3 Deployment
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.19
        ports:
        - containerPort: 80
```

### 2.1 Request / Limit
```
---
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: app
    image: images.my-company.example/app:v4
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
  - name: log-aggregator
    image: images.my-company.example/log-aggregator:v6
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

### 2.2 ResourceQuota
```
# quota-test.yaml
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: default
spec:
  hard:
    cpu: "1000"
    memory: 200Gi
    pods: "10"
#  scopeSelector:
#    matchExpressions:
#    - operator : In
#      scopeName: PriorityClass
#      values: ["high"]
```
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.19
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
```

### 2.3 LimitRange
LimitRange.yaml
```
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-min-max-demo-lr
  namespace: default
spec:
  limits:
  - max:
      cpu: "800m"
    min:
      cpu: "200m"
    type: Pod #Container
```
nginx-deploy.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.19
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "1000m"
          limits:
            memory: "128Mi"
            cpu: "1500m"
```

### 2.2 Pod Selector
pod-label.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    svc: web
```

### 3.2 PV 생성
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: test-pv
  labels:
    name: test-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteMany
  hostPath:
    path: "/test/volume"
```

### 3.3 PVC 생성
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
     storage: 1Gi
```

### 3.4 PVC를 사용할 Pod 생성
```
apiVersion: v1
kind: Pod
metadata:
  name: test-nginx2
spec:
  containers:
  - name: test-nginx2
    image: nginx:1.19
    ports:
    - containerPort: 80
    volumeMounts:
      - mountPath: "/test/volume"
        name: test-pv
  volumes:
  - name: test-pv
    persistentVolumeClaim:
      claimName: test-pvc
```

#### 4.1.1 컨피그맵을 생성합니다
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: prod-account
  namespace: default
data:
  ID: prod-user
  PASSWORD: prod-pass
  LOG_LEVEL: info
```

#### 4.1.2 컨테이너에서 prod-account 컨피그맵의 LOG_LEVEL값을 가지고 오도록 설정합니다
```
piVersion: apps/v1
kind: Deployment
metadata:
  name: configmap-test
  labels:
    app: prod
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prod
  template:
    metadata:
      labels:
        app: prod
    spec:
      containers:
      - name: testapp
        image: nginx:1.19
        ports:
        - containerPort: 8080
        env:
          - name: LOG_LEVEL
            valueFrom:
              configMapKeyRef:
                 name: prod-account
                 key: LOG_LEVEL
```

#### 5.1.2 yaml 파일로 시크릿 생성
```
apiVersion: v1
kind: Secret
metadata:
  name: prod-user
type: Opaque
data:
  username: cHJvZC11c2Vy
  password: cHJvZC1wYXNz
```

#### 5.1.3 시크릿 사용
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secretapp
  labels:
    app: secretapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: secretapp
  template:
    metadata:
      labels:
        app: secretapp
    spec:
      containers:
      - name: testapp
        image: nginx:1.19
        ports:
        - containerPort: 8080
        env:
          - name: SECRET_USERNAME
            valueFrom:
              secretKeyRef:
                name: prod-user
                key: username
          - name: SECRET_PASSWORD
            valueFrom:
              secretKeyRef:
                name: prod-user
                key: password
```

# 03

#### 1.2.1 ClusterIP
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app.kubernetes.io/name: proxy
spec:
  containers:
  - name: nginx
    image: nginx:stable
    ports:
      - containerPort: 80
        name: http-web-svc

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app.kubernetes.io/name: proxy
  ports:
  - name: name-of-service-port
    protocol: TCP
    port: 80
    targetPort: http-web-svc
```

#### 1.2.2 NodePort
```
---                                                                                           
apiVersion: apps/v1                                                                           
kind: Deployment                                                                              
metadata:                                                                                     
  name: nginx-deployment                                                                      
  labels:                                                                                     
    app: nginx                                                                                
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
      containers:                                                                             
      - name: nginx                                                                           
        image: nginx                                                                          
        ports:                                                                                
        - containerPort: 80                                                                   
---                                                                                           
apiVersion: v1                                                                                
kind: Service                                                                                 
metadata:                                                                                     
  name: my-service-nodeport                                                                   
spec:                                                                                         
  type: NodePort                                                                              
  selector:                                                                                   
    app: nginx                                                                                
  ports:                                                                                      
      # 기본적으로 그리고 편의상 `targetPort` 는 `port` 필드와 동일한 값으로 설정된다.                           
    - port: 80                                                                                
      targetPort: 80                                                                          
      # 선택적 필드                                                                               
      # 기본적으로 그리고 편의상 쿠버네티스 컨트롤 플레인은 포트 범위에서 할당한다(기본값: 30000-32767)                   
      nodePort: 30007
```

#### 1.2.3 LoadBalancer
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx2-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx2-service
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 8088
      targetPort: 80
```

## 2. Ingress
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  name: rewrite
  namespace: default
spec:
  ingressClassName: nginx
  rules:
  - host: sllee.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
```
