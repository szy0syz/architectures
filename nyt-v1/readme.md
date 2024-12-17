# nyt architecture v1

## notes

### docker

```bash
sudo apt-get -y install --no-install-recommends wget gnupg ca-certificates

wget -O - https://mirrors.tencentyun.com/docker-ce/linux/debian/gpg | sudo apt-key add -

codename=`grep -Po 'VERSION="[0-9]+ \(\K[^)]+' /etc/os-release`
echo "deb https://mirrors.tencentyun.com/docker-ce/linux/debian $codename docker-ce" \
    | sudo tee /etc/apt/sources.list.d/docker-ce.list

sudo apt-get update
sudo apt-get -y install docker

# vim /etc/docker/daemon.json
{
   "registry-mirrors": [
       "https://mirror.ccs.tencentyun.com"
  ]
}

sudo systemctl daemon-reload
sudo systemctl restart docker

# check validation
sudo docker info
```

- k3s (single node)

```bash
sudo curl -sfL https://get.k3s.io | sh -s - --docker

# check validation
sudo k3s kubectl get pods --all-namespaces
sudo docker ps
sudo k3s kubectl get nodes -o wide

# new file nginx-deployment.yaml
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
        image: nginx:1.26

# test deploy
k3s kubectl apply -f nginx-deployment.yaml
k3s kubectl get deployments
k3s kubectl get pods
k3s kubectl delete deployment nginx-deployment
```

### Kuboard

> k8s集群管理

```bash
# vim new kuboard-install.sh
docker run -d \
  --restart=unless-stopped \
  --name=kuboard \
  -p 10082:80/tcp \
  -p 10081:10081/tcp \
  -e KUBOARD_ENDPOINT="{内网IP}:80" \
  -e KUBOARD_AGENT_SERVER_TCP_PORT="10081" \
  -v /root/kuboard-data:/data \
  eipwork/kuboard:v3

chmod +x ./kuboard-install.sh
bash ./kuboard-install.sh
```

- 登录账号admin 密码Kuboard123
- 添加集群`.kubeconfig`方式连接
- 然后将`/etc/rancher/k3s/k3s.yaml`复制到配置文本框里
- 注意内容里有一行`server: https://127.0.0.1:6443`，填到Kuboard以后确认前把127.0.0.1修改为你的内网地址

### Smoke test

```bash
# 1. Nginx Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-test
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-test
  template:
    metadata:
      labels:
        app: nginx-test
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 50m
            memory: 64Mi
          limits:
            cpu: 100m
            memory: 128Mi
---
# 2. Service
apiVersion: v1
kind: Service
metadata:
  name: nginx-test-svc
  namespace: default
spec:
  selector:
    app: nginx-test
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
---
# 3. Ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-test-ingress
spec:
  ingressClassName: traefik
  rules:
  - host: tt.wwmm.cc
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-test-svc
            port:
              number: 80
```
