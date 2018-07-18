# registry create
az acr create -n devopscourseregistry -g devopscourse -l westeurope
az ad sp create-for-rbac --scopes subscroption --role Owner --password myPassword

docker login devopscourseregistry-on.azurecr.io -u 13b58060-07b4-482d-96d2-6dcda38309a7 -p password

# list containers
az acr repository list -n devopscourseregistry -o json -u 13b58060-07b4-482d-96d2-6dcda38309a7 -p password

# get node info
kubectl get nodes
kubectl api-versions

# run similar containers
kubectl run nginx --image nginx
kubectl expose deployments nginx --port=80 --type=LoadBalancer

# weave scope
kubectl apply -f 'https://cloud.weave.works/launch/k8s/weavescope.yaml'
kubectl port-forward $(kubectl get pod --selector=weave-scope-component=app -o jsonpath='{.items..metadata.name}') 4040

# run first application
nano Dockerfile

### Dockerfile
FROM php:apache
COPY src/ /var/www/html
EXPOSE 80

# build image
docker build -t first-docker .

# pull image to windows azure registry
docker tag first-docker devopscourse-on.azurecr.io/first-docker:v1
docker push devopscourse-on.azurecr.io/first-docker:v1

# check container images on registry
az acr repository list -n devopscourse -o json -u fc24afb5-bbe5-4bb2-9f86-8c6e7f2f7422 -p password

# create myregistrykey
kubectl create secret docker-registry myregistrykey \
--docker-server=https://devopscourseregistry-on.azurecr.io \
--docker-username=13b58060-07b4-482d-96d2-6dcda38309a7 \
--docker-password=password \
--docker-email=vigh.zoltan@codefactory.hu

# first-docker-app.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-svc
  labels:
    app: my-app
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: my-app
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: my-app-deployment
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: devopscourse-on.azurecr.io/first-docker:v1
        ports:
        - containerPort: 80
      imagePullSecrets:
        - name: myregistrykey

# A Deployment provides declarative updates for Pods and Replica Sets (the next-generation Replication Controller).
kubectl create -f first-docker-app.yaml --record
kubectl get deployments
kubectl get pods
kubectl get rs
kubectl get service

# check service yaml
kubectl edit service/first-docker-app

# edit index.html Dockerfile!!
docker build -t devopscourse-on.azurecr.io/first-docker:v2 .

# vagy
docker build -t first-docker .
docker tag first-docker devopscourse-on.azurecr.io/first-docker:v2

# push new image
docker push devopscourse-on.azurecr.io/first-docker:v2

# update nginx
kubectl set image deployment/my-app-deployment my-app=devopscourse-on.azurecr.io/first-docker:v2

# or
kubectl edit deployment/first-docker-deployment

# rollout status
kubectl rollout status deployment/first-docker-deployment

# undo deployment
kubectl rollout undo

# delete resource group
az group delete -n devopscourse
