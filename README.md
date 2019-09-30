# Public Education Service Deploy on Kubernetes

## Prior knowledge
please see the tutorials if you're new to kubernetes and knative: 
  https://workflowy.com/s/msa/27a0ioMCzlpV04Ib 

## Building and Running on Local Dev Env

clone the repos first:
```
git clone https://github.com/sw300/public-education-core
git clone https://github.com/sw300/public-education-marketing
git clone https://github.com/sw300/public-education-dashboard
git clone https://github.com/sw300/public-education-dashboard-aggregator
git clone https://github.com/sw300/public-education-api-gw
git clone https://github.com/sw300/public-education-frontend
```

download the kafka, and run the event-platform:
```
(download kafka kafka_2.12-2.1.0 version)
(new shell)
cd ~/Downloads/kafka_2.12-2.1.0
bin/zookeeper-server-start.sh config/zookeeper.properties

(new shell)
cd ~/Downloads/kafka_2.12-2.1.0
bin/kafka-server-start.sh config/server.properties

(set your 'hosts' file pointing to education-kafka to your localhost as noted here:)
127.0.0.1	education-kafka-zookeeper
127.0.0.1	education-kafka

```


run each microservices (core, marketing, dashboard):
```
(new shell)
cd public-education-core
mvn spring-boot:run -Dspring.profiles.active=event-driven -Dserver.port=8086
(check the service is up with command: http localhost:8086)

(new shell)
cd public-education-marketing
mvn spring-boot:run -Dserver.port=8087
(check the service is up with command: http localhost:8087)

(new shell)
cd public-education-dashboard
mvn spring-boot:run -Dserver.port=8088
(check the service is up with command: http localhost:8088)
```

[IMPORTANT] You have to give new port numbers for each microservices

run api-gateway for single entry point:
```
(new shell)
cd public-education-api-gw
mvn spring-boot:run
```

(check the api-gw is properly working with command: http localhost:8080/courses, http localhost:8080/mailLogs, http localhost:8080/dashboards)

run the dashboard-aggregator that is normal java application:
```
(new shell)
cd public-education-dashboard-aggregator
mvn install
mvn org.codehaus.mojo:exec-maven-plugin:1.5.0:java -Dexec.mainClass="com.sw300.streams.EnrollmentAggregate"
```

run the front-end:
```
(new shell)
cd public-education-frontend

npm install
npm run dev
```

set your hosts file points to the backend:
```
127.0.0.1       backend.public-education.com
```

you can now access to the frontend:
```
http://localhost:8081 or 8082
```

create a new course and class with Admin UI:

![image](https://user-images.githubusercontent.com/487999/52828095-c8371d80-310a-11e9-95bc-36700d9006e8.png)
![image](https://user-images.githubusercontent.com/487999/52828137-f4529e80-310a-11e9-9e61-002db9637f6f.png)
![image](https://user-images.githubusercontent.com/487999/52828159-046a7e00-310b-11e9-9c3c-41e7604ec1b9.png)


create a customer and enroll in the class with API:
```json
$ http backend.public-education.com:8086/clazzes
HTTP/1.1 200 
Content-Type: application/hal+json;charset=UTF-8
Date: Fri, 15 Feb 2019 01:21:32 GMT
Transfer-Encoding: chunked
X-Application-Context: application:8086

{
    "_embedded": {
        "clazzes": [
            {
                "_links": {
                    "clazz": {
                        "href": "http://localhost:8086/clazzes/6"
                    }, 
                    "clazzDayList": {
                        "href": "http://localhost:8086/clazzes/6/clazzDayList"
                    }, 
                    "course": {
                        "href": "http://localhost:8086/clazzes/6/course"
                    }, 
                    "self": {
                        "href": "http://localhost:8086/clazzes/6"        <--- note this uri for clazz
                    }
                }, 
                "evaluationRate": 0.0, 
                "price": 0.0, 
                "status": "CREATED"
            }
        ]
    }, 
    "_links": {
        "profile": {
            "href": "http://localhost:8086/profile/clazzes"
        }, 
        "self": {
            "href": "http://localhost:8086/clazzes{?page,size,sort}", 
            "templated": true
        }
    }, 
    "page": {
        "number": 0, 
        "size": 20, 
        "totalElements": 1, 
        "totalPages": 1
    }
}


$ http PATCH backend.public-education.com:8086/clazzes/6 price=500  # set the price
HTTP/1.1 200 OK
...

$ http backend.public-education.com:8086/customers firstName="Rick" lastName="Jang"
HTTP/1.1 201 
Content-Type: application/json;charset=UTF-8
Date: Fri, 15 Feb 2019 01:21:59 GMT
Location: http://localhost:8086/customers/7
Transfer-Encoding: chunked
X-Application-Context: application:8080

{
    "_links": {
        "customer": {
            "href": "http://localhost:8086/customers/7"
        }, 
        "paymentMethod": {
            "href": "http://localhost:8086/customers/7/paymentMethod"
        }, 
        "self": {
            "href": "http://localhost:8086/customers/7"                 <--- note this uri for customer
        }
    }, 
    "email": null, 
    "firstName": "Rick", 
    "industry": null, 
    "job": null, 
    "lastName": "Jang", 
    "membership": false, 
    "phone": null
}

$ http backend.public-education.com:8086/enrollments customer="http://localhost:8086/customers/7" clazz="http://localhost:8086/clazzes/6"
HTTP/1.1 201 
Content-Type: application/json;charset=UTF-8
Date: Fri, 15 Feb 2019 01:23:07 GMT
Location: http://localhost:8086/enrollments/8
Transfer-Encoding: chunked
X-Application-Context: application:8080

{
    "_links": {
        "clazz": {
            "href": "http://localhost:8086/enrollments/8/clazz"
        }, 
        "customer": {
            "href": "http://localhost:8086/enrollments/8/customer"
        }, 
        "enrollment": {
            "href": "http://localhost:8086/enrollments/8"
        }, 
        "payments": {
            "href": "http://localhost:8086/enrollments/8/payments"
        }, 
        "self": {
            "href": "http://localhost:8086/enrollments/8"
        }
    }, 
    "date": null, 
    "grade": 0, 
    "status": null
}

```

You may check the logs are out at the public-education-marketing service:
```
<이메일 발송>
제목: Rick Jang 님의 강의 수강이 신청되었습니다.
수강신청과목명: BPM and MSA Course

```
Also you can find the dashboard service has updated the total enrollment time and price for each customers:
```
http backend.public-education.com:8080/dashboards
```

[TIP] to watch the kafka events, you can use these scripts:
```
(new shell)
$ cd ~/Downloads/kafka_2.12-2.1.0
$ bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic class.topic --from-beginning --formatter kafka.tools.DefaultMessageFormatter --property print.key=true --property print.value=true --property key.deserializer=org.apache.kafka.common.serialization.StringDeserializer --property value.deserializer=org.apache.kafka.common.serialization.StringDeserializer

3	{"classId":"2","courseTitle":"SW 300","customerId":"3","customerName":"장 진영","hour":0,"price":50.0}
3	{"classId":"2","courseTitle":"SW 300","customerId":"3","customerName":"장 진영","hour":0,"price":50.0}
3	{"classId":"2","courseTitle":"SW 300","customerId":"3","customerName":"장 진영","hour":0,"price":50.0}
3	{"classId":"2","courseTitle":"SW 300","customerId":"3","customerName":"장 진영","hour":0,"price":50.0}
7	{"classId":"6","courseTitle":"BPM and MSA Course","customerId":"7","customerName":"Rick Jang","hour":0,"price":0.0}
7	{"classId":"6","courseTitle":"BPM and MSA Course","customerId":"7","customerName":"Rick Jang","hour":0,"price":0.0}
7	{"classId":"6","courseTitle":"BPM and MSA Course","customerId":"7","customerName":"Rick Jang","hour":0,"price":500.0}
7	{"classId":"6","courseTitle":"BPM and MSA Course","customerId":"7","customerName":"Rick Jang","hour":0,"price":500.0}
7	{"classId":"6","courseTitle":"BPM and MSA Course","customerId":"7","customerName":"Rick Jang","hour":0,"price":500.0}
7	{"classId":"6","courseTitle":"BPM and MSA Course","customerId":"7","customerName":"Rick Jang","hour":0,"price":500.0}
7	{"classId":"6","courseTitle":"BPM and MSA Course","customerId":"7","customerName":"Rick Jang","hour":0,"price":500.0}
7	{"classId":"6","courseTitle":"BPM and MSA Course","customerId":"7","customerName":"Rick Jang","hour":0,"price":500.0}
...

(new shell)
$ cd ~/Downloads/kafka_2.12-2.1.0
$ bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic enrollment-total-output --from-beginning --formatter kafka.tools.DefaultMessageFormatter --property print.key=true --property print.value=true --property key.deserializer=org.apache.kafka.common.serialization.StringDeserializer --property value.deserializer=org.apache.kafka.common.serialization.StringDeserializer

3	{"count":10,"totalPrice":500,"totalTime":0}
3	{"count":13,"totalPrice":650,"totalTime":0}
7	{"count":1,"totalPrice":0,"totalTime":0}
7	{"count":2,"totalPrice":0,"totalTime":0}
7	{"count":4,"totalPrice":1000,"totalTime":0}
7	{"count":5,"totalPrice":1500,"totalTime":0}
7	{"count":6,"totalPrice":2000,"totalTime":0}
7	{"count":12,"totalPrice":5000,"totalTime":0}
....

```


## Building and Running on Kubernetes Manually

- prerequites:  

Kafka installation with helm
```
helm install --name education-kafka incubator/kafka
```

- building, containerizing and run on kubernetes
```
cd public-education-core
docker build -t [YOUR_DOCKER_REGISTRY]/[PROJECT]/[ARTIFACT_ID]:[VERSION] .

kuberctl run public-education-core --image=[YOUR_DOCKER_REGISTRY]/[PROJECT]/[ARTIFACT_ID]:[VERSION]

(do this for each microservices as well)
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

## Zero-down time deploy and AutoScale with KNative


KNative installation: 
see the instructions in the "Serverless and FaaS" section of https://workflowy.com/s/msa/27a0ioMCzlpV04Ib 

- clone the repo first
```
git clone https://github.com/sw300/public-education-kubernetes
cd public-education-kubernetes
```

- build and deploy applications
```
kubectl apply -f ksvc-core.yaml
kubectl apply -f ksvc-marketing.yaml
kubectl apply -f ksvc-dashboard.yaml
```

- if everything goes fine, pods look like this:
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

- change the source code:
```java

(from v1)

@RepositoryRestResource(collectionResourceRel = "v1/mailLogs", path = "v1/mailLogs")
public interface MailLogRepository extends PagingAndSortingRepository<MailLog, Long>{

}

(to v2)

@RepositoryRestResource(collectionResourceRel = "v2/mailLogs", path = "v2/mailLogs")
public interface MailLogRepository extends PagingAndSortingRepository<MailLog, Long>{

}

```

- commit and push 
```
git commit
git push
```

- build and deploy again:  change the ksvc-marketing.yaml
```
nano kvsc-marketing.yaml
v1
(to)
v2
(save)

kubectl apply -f ksvc-marketing.yaml   # will trigger build and deploy for version 2
```

- see the detail build logs:
```
kubectl logs  public-education-marketing-00001-555r2 -c build-step-build-and-push
```

- wait while watching the version:
```
$ run.sh

{
  "_links" : {
    "v1/mailLogs" : {
      "href" : "http://public-education-marketing.default.svc.cluster.local/v1/mailLogs{?page,size,sort}",
      "templated" : true
    },
    "profile" : {
      "href" : "http://public-education-marketing.default.svc.cluster.local/profile"
    }
  }
}
.... after deployment is done, the version has been changed to v2 WITHOUT DOWN-TIME! ....

{
  "_links" : {
    "v2/mailLogs" : {
      "href" : "http://public-education-marketing.default.svc.cluster.local/v2/mailLogs{?page,size,sort}",
      "templated" : true
    },
    "profile" : {
      "href" : "http://public-education-marketing.default.svc.cluster.local/profile"
    }
  }
}
```

- 
