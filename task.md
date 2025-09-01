Part1:

1- create pod nginx with name my nginx direct from command don't use yaml file 

### explanation:
kubectl run mynginx → creates a Pod named mynginx.
--image=nginx → tries to pull the image named nginx from Docker Hub 
--restart=Never → ensures the object is a Pod and not a Deployment.
### Command used :
kubectl run mynginx --image=nginx --restart=Never
kubectl get pods
### Output of the command :
pod/mynginx created


2- create pod nginx with name my nginx command and use Image nginx123  direct from command don't use yaml file

### explanation:
kubectl run mynginx → creates a Pod named mynginx.
--image=nginx123 → tries to pull the image named nginx123 from Docker Hub (but this image doesn’t exist, so it will likely fail).
--restart=Never → ensures the object is a Pod and not a Deployment.
### Command used :
kubectl delete pod mynginx
kubectl run mynginx --image=nginx123 --restart=Never
kubectl get pod
### Output of the command :
pod/mynginx created

NAME      READY   STATUS         RESTARTS   AGE
mynginx   0/1     ErrImagePull   0          12s


3- check the status and why it doesn't work 

### explanation:
- I checked the Pod status using `kubectl get pods`.  
- The Pod shows `ErrImagePull` / `ImagePullBackOff` because Kubernetes cannot find the image `nginx123`.  
- To confirm, I described the Pod using `kubectl describe pod mynginx`, and the error clearly states that the image does not exist in Docker Hub.  

### Command used :
kubectl get pods
kubectl describe pod mynginx

### Output of the command :
- kubectl get pods (Output)
NAME      READY   STATUS             RESTARTS   AGE
mynginx   0/1     ImagePullBackOff   0          4m18s
---
kubectl describe pod mynginx (Output ):Show in screenshot

4- I need to know node name - IP - Image Of the POD

### explanation:
I checked the Pod details using `kubectl get pod -o wide` and `kubectl describe pod`.  
This shows the node name where the Pod is scheduled, the Pod IP, and the image used.

### Command used :
kubectl get pod mynginx -o wide
kubectl describe pod mynginx

### Output of the command :
NAME      READY   STATUS             RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
mynginx   0/1     ImagePullBackOff   0          11m   10.244.0.46   minikube   <none>           <none>

---
5- delete the pod 
### explanation:
kubectl delete pod <pod-name> removes the Pod from the cluster.
Once deleted, the Pod no longer appears in kubectl get pods.
Since we created it with --restart=Never, Kubernetes won’t try to recreate it.
### Command used :
kubectl delete pod mynginx
### Output of the command :
pod "mynginx" deleted from default namespace

6- create another one with yaml file and use label

### explanation:
I created a new Pod using a YAML file instead of a direct command.  
In the YAML, I added labels (`app=nginx` and `env=dev`) to the Pod.  
Labels are useful for organizing and selecting Pods.

### Command used :
kubectl apply -f mynginx.yaml
kubectl get pods --show-labels
### Output of the command :
kubectl apply -f mynginx.yaml(OUTPUT): pod/mynginx-yaml created
kubectl get pods --show-labels (OUTPUT) :
NAME           READY   STATUS    RESTARTS   AGE   LABELS
mynginx-yaml   1/1     Running   0          73s   app=nginx,env=dev
---

7-create Riplicaset with 3 replicas using nginx Image 

### explanation:
I created a ReplicaSet named `nginx-rs` using a YAML file.  
The ReplicaSet ensures that 3 Pods are always running with the nginx image.  
The `selector` matches Pods with the label `app=nginx`.

### Command used :
kubectl apply -f replicaset.yaml
kubectl get rs
kubectl get pods -l app=nginx
### Output of the command :
 kubectl apply -f replicaset.yaml(OUTPUT):replicaset.apps/nginx-rs created
 kubectl get rs(OUTPUT):
 NAME       DESIRED   CURRENT   READY   AGE
nginx-rs   3         3         2       9s
kubectl get pods -l app=nginx(OUTPUT):
NAME             READY   STATUS    RESTARTS   AGE
mynginx-yaml     1/1     Running   0          6m24s
nginx-rs-27mnp   1/1     Running   0          23s
nginx-rs-8wr5j   1/1     Running   0          23s
---
8-scale the replicas to 5 without edit in the Yaml file
### explanation:
I used the `kubectl scale` command to increase the number of replicas from 3 to 5.  
This immediately created 2 additional Pods, so the ReplicaSet now maintains 5 Pods running the nginx image.

### Command used :
kubectl scale rs nginx-rs --replicas=5
kubectl get rs
kubectl get pods -l app=nginx
### Output of the command :
kubectl scale rs nginx-rs --replicas=5(OUTPUT):
replicaset.apps/nginx-rs scaled
kubectl get rs(OUTPUT):
NAME       DESIRED   CURRENT   READY   AGE
nginx-rs   5         5         3       8m55s
kubectl get pods -l app=nginx(OUTPUT):
NAME             READY   STATUS    RESTARTS   AGE
mynginx-yaml     1/1     Running   0          15m
nginx-rs-27mnp   1/1     Running   0          9m7s
nginx-rs-8wr5j   1/1     Running   0          9m7s
nginx-rs-qk9b2   1/1     Running   0          22s
nginx-rs-rh4vx   1/1     Running   0          22s
---

9-Delete any one of the 5 pods and check what happen and explain 
### explanation:
The ReplicaSet automatically maintains the number of replicas.  
When I deleted one Pod, the ReplicaSet detected that only 4 Pods were running while the desired state was 5.  
It immediately created a new Pod with a different name to restore the total back to 5 Pods.  
This shows how ReplicaSet provides self-healing and ensures high availability.
### Command used :
kubectl delete pod nginx-rs-abc12
kubectl get rs
kubectl get pods -l app=nginx
### Output of the command :
kubectl delete pod nginx-rs-abc12(OUTPUT):
pod "nginx-rs-rh4vx" deleted from default namespace
kubectl get rs(OUTPUT):
NAME       DESIRED   CURRENT   READY   AGE
nginx-rs   5         5         5       14m
kubectl get pods -l app=nginx(OUTPUT):
NAME             READY   STATUS    RESTARTS   AGE
mynginx-yaml     1/1     Running   0          20m
nginx-rs-27mnp   1/1     Running   0          14m
nginx-rs-8wr5j   1/1     Running   0          14m
nginx-rs-qk9b2   1/1     Running   0          5m26s
nginx-rs-w27rt   1/1     Running   0          16s
---
?10-Scale down the pods aging to 2 without scale command use terminal 
### explanation:
Instead of using the `kubectl scale` command, I reduced the replicas by directly editing the ReplicaSet.  
I opened it with `kubectl edit`, changed the value of `replicas` from 5 to 2, and Kubernetes automatically terminated the extra Pods until only 2 remained.
### Command used :
kubectl edit rs nginx-rs
kubectl get rs
kubectl get pods -l app=nginx
### Output of the command :
---kubectl edit rs nginx-rs(OUTPUT):
replicaset.apps/nginx-rs edited
kubectl get rs(OUTPUT):
NAME                        DESIRED   CURRENT   READY   AGE
httpd-frontend-59cf8fd4c8   0         0         0       102m
httpd-frontend-7686c4d745   3         3         3       103m
nginx-rs                    2         2         2       10m
webapp-deploy-78ff84ffc7    3         3         3       28m
webserver-78df96f968        7         7         7       124m
kubectl get pods -l app=nginx(OUTPUT):
NAME             READY   STATUS    RESTARTS   AGE
nginx-rs-cgwwr   1/1     Running   0          11m
nginx-rs-xlrlv   1/1     Running   0          11m

---
11- find out the issue in the below Yaml (don't use AI)

apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-2
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: nginx
    spec:
      containers:
      - name: nginx
        image: nginx

### Issue: 
Labels mismatch → selector uses tier: frontend but template uses tier: nginx.

### explanation:
ReplicaSet won’t find Pods to manage because labels don’t match.
### Fix

apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-2
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx

---
12- find out the issue in the below Yaml (don't use AI)

apiVersion: apps/v1
kind: deployment
metadata:
  name: deployment-1
spec:
  replicas: 2
  selector:
    matchLabels:
      name: busybox-pod
  template:
    metadata:
      labels:
        name: busybox-pod
    spec:
      containers:
      - name: busybox-container
        image: busybox
        command:
        - sh
        - "-c"
        - echo Hello Kubernetes! && sleep 3600

### Issue: 
kind written as deployment (small letters).

### explanation:
Kubernetes is case-sensitive; it must be Deployment.
### Fix 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-1
spec:
  replicas: 2
  selector:
    matchLabels:
      name: busybox-pod
  template:
    metadata:
      labels:
        name: busybox-pod
    spec:
      containers:
      - name: busybox-container
        image: busybox
        command:
        - sh
        - "-c"
        - echo Hello Kubernetes! && sleep 3600
---
13- find out the issue in the below Yaml (don't use AI)

apiVersion: v1
kind: Deployment
metadata:
  name: deployment-1
spec:
  replicas: 2
  selector:
    matchLabels:
      name: busybox-pod
  template:
    metadata:
      labels:
        name: busybox-pod
    spec:
      containers:
      - name: busybox-container
        image: busybox
        command:
        - sh
        - "-c"
        - echo Hello Kubernetes! && sleep 3600

### Issue: 
Wrong apiVersion → written as v1 instead of apps/v1.
### explanation: 
Deployment resource only works under apps/v1.
### Fix

apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-1
spec:
  replicas: 2
  selector:
    matchLabels:
      name: busybox-pod
  template:
    metadata:
      labels:
        name: busybox-pod
    spec:
      containers:
      - name: busybox-container
        image: busybox
        command:
        - sh
        - "-c"
        - echo Hello Kubernetes! && sleep 3600

14- what's command you use to know what Image name that running the deployment 
### explanation:
I use kubectl get with -o jsonpath to extract the image name directly, or kubectl describe to check all details of the deployment.

### Command used:
kubectl get deployment webserver -o=jsonpath='{.spec.template.spec.containers[*].image}'
OR
kubectl describe deployment webserver 


15- create deployment using following data :
Name: httpd-frontend;
Replicas: 3;
Image: httpd:2.4-alpine
### explanation:
create a Deployment with 3 replicas running the httpd:2.4-alpine image.Normally, this can be done using a YAML file or kubectl create deployment.
### Command used:
kubectl apply -f httpd-deployment.yml
kubectl get deployments
kubectl get pods -l app=httpd
kubectl describe deploy httpd-frontend

16- replace the image to nginx777 with command directly 
### expalantion:
to update the container image of the existing Deployment directly from the command line.
This allows Kubernetes to perform a rolling update without editing the YAML file.
### Command used:
kubectl set image deployment/httpd-frontend httpd-container=nginx777

### Output of the command :
kubectl set image deployment/httpd-frontend httpd-container=nginx777(OUTPUT):
deployment.apps/httpd-frontend image updated
kubectl describe deployment httpd-frontend(OUTPUT): httpd-container:
    Image:         nginx777

17- rollback to pervious version
### explanation:
Update the container image of the existing Deployment directly from the command line.
Kubernetes will perform a rolling update automatically.
### Command used:
kubectl rollout undo deployment/httpd-frontend
kubectl get deploymentkubectl describe deployment httpd-frontend
### Output of the command :
 kubectl rollout undo deployment/httpd-frontend(OUTPUT):
 deployment.apps/httpd-frontend rolled back
kubectl get deployment httpd-frontend(OUTPUT):
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
httpd-frontend   0/3     3            0           4m30s
kubectl describe deployment httpd-frontend(OUTPUT):
Containers:
   httpd-container:
    Image:         httpd:2.4-alpine
---

18- Create a Simple Web Application:
* Use a Dockerfile to create a simple web application (e.g., an Nginx server serving an HTML page).
* Build the Docker image and push it to DockerHub your private Account.
Step 1: Create Dockerfile and HTML
### explanation: 
Create index.html and Dockerfile to serve the page via Nginx.
### Command Used:
nano index.html
nano Dockerfile

### Output: Files ready (screenshot optional).

Step 2: Build Docker Image

### explanation:
 Build image engy11/webapp:latest.
### Command used:
docker build -t engy11/webapp:latest .
### Output: Successfully built <IMAGE_ID> (screenshot optional).

Step 3: Login to DockerHub

### explanation:
 Authenticate to push image.
### Command used:
docker login -u engy11
### Output: Login Succeeded

Step 4: Push Image
### explanation:
 Push image to DockerHub.
### Command used:
docker push engy11/webapp:latest
### Output: Image pushed successfully (screenshot optional).

Step 5: Run Locally
### explanation: 
 Verify the HTML page.
### Command used:
docker run -d -p 8080:80 engy11/webapp:latest
Open browser: http://localhost:8080 
---

19- Create a Deployment Using This Image:
* Deploy the Docker image from DockerHub to Kubernetes with a Deployment that has 3 replicas.

### explanation:
Create a Deployment in Kubernetes using the image engy11/webapp:latest from DockerHub.
Set 3 replicas.
### Command used:
kubectl create deployment webapp-deploy --image=engy11/webapp:latest --replicas=3
kubectl get deployments
kubectl get pods -l app=webapp-deploy
### Output of the command :
kubectl create deployment webapp-deploy --image=engy11/webapp:latest --replicas=3(OUTPUT):
deployment/apps/webapp-deploy created
kubectl get deployments(OUTPUT):
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
httpd-frontend   0/3     3            0           83m
webapp-deploy    0/3     3            0           7m53s
webserver        0/7     7            0           103m
kubectl get pods -l app=webapp-deploy(OUTPUT):
NAME                             READY   STATUS              RESTARTS   AGE
webapp-deploy-78ff84ffc7-q6797   0/1     ContainerCreating   0          5m25s
webapp-deploy-78ff84ffc7-xbfv9   0/1     ContainerCreating   0          5m27s
webapp-deploy-78ff84ffc7-zw4sc   0/1     ContainerCreating   0          5m25s
