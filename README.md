# Public Education Service Deploy on Kubernetes

## Prior knowledge
please see the tutorials if you're new to kubernetes and knative: 
  https://workflowy.com/s/msa/27a0ioMCzlpV04Ib 

## Multi-yaml version

```
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
