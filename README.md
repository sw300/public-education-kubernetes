# Public Education Service Deploy on Kubernetes

## Prior knowledge
please see the tutorials if you're new to kubernetes and knative: 
  https://workflowy.com/s/msa/27a0ioMCzlpV04Ib 


## Building and Running on Kubernetes Manually
```
git clone https://github.com/sw300/public-education-core
cd public-education-core
docker build -t [YOUR_DOCKER_REGISTRY]/[PROJECT]/[ARTIFACT_ID]:[VERSION] .

kuberctl run public-education-core --image=[YOUR_DOCKER_REGISTRY]/[PROJECT]/[ARTIFACT_ID]:[VERSION]
```

## Multi-yaml version

getting the yaml

```
kubectl get deploy public-education-core -oyaml > deploy.yaml
```

delete the manually deployed version
```
kubectl delete deploy public-education-core
```

now, create deployment with file declaration
```
kubectl create -f deploy.yaml
```

## Helm-chart version (updating)

make directory structure as follow with files we've created so far:
```
templates
  deployment.yml
requirements.yml
```

once the directory created, you can deploy all the deployments and the dependency charts with one command:
```
helm install public-education .
```

## KNative version

- prerequites:  

Kafka installation with helm
```
helm install --name education-kafka incubator/kafka
```

KNative installation: 
see the instructions in the "Serverless and FaaS" section of https://workflowy.com/s/msa/27a0ioMCzlpV04Ib 


- build, deploy for core subdomain
```
kubectl create -f ksvc-core.yaml
kubectl create -f ksvc-marketing.yaml
kubectl create -f ksvc-dashboard.yaml
kubectl create -f job-dashboard-aggregator.yaml
```

- if everything goes well, you can check as follows:
```
$ kubectl get po
NAME                                     READY     STATUS     RESTARTS   AGE
education-kafka-0                        1/1       Running    0          14m
education-kafka-1                        1/1       Running    0          12m
education-kafka-2                        1/1       Running    0          11m
education-kafka-zookeeper-0              1/1       Running    0          14m
education-kafka-zookeeper-1              1/1       Running    0          14m
education-kafka-zookeeper-2              1/1       Running    0          13m
public-education-core-00001-vwzfk        0/1       Init:1/3   0          40s
public-education-marketing-00001-555r2   0/1       Init:2/3   0          1m

```

- wait for seconds
