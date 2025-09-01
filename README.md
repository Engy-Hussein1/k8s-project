
# Kubernetes Project

## Steps Performed

1. **Pod Creation**
   - Command: `kubectl run mynginx --image=nginx --restart=Never`
   - Output: pod created
   - Screenshot: `screenshots/pod_creation.png`

2. **ReplicaSet**
   - Applied `replicaset.yaml`
   - Command: `kubectl apply -f replicaset.yaml`
   - Output: 3 pods running
   - Screenshot: `screenshots/replicaset.png`

3. **Deployments**
   - HTTPD Deployment: `kubectl apply -f deployment-httpd.yaml`
   - WebApp Deployment: `kubectl apply -f webapp-deployment.yaml`
   - Screenshot: `screenshots/deployment.png`


