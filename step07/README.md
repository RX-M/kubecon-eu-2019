![rx-m LLC][RX-M LLC]


# A Day in the life of a cloud native developer


## Step 07 - Logging with Fluentd

Fluentd is a cross platform open-source data collection tool, having recently graduated it is now a top level CNCF
project. Fluentd is written primarily in Ruby with C in places where performance is critical.

In this step we'll use Fluentd to take the log events from our microservice and forward them to Elasticsearch where we
can view them with Kibana (this combination of tools is often known as the EFK stack).


### 1. Deploying the Helm Charts

The lab repo includes helm charts that will deploy the EFK stack for you in your personal namespace.

First, deploy Elasticsearch, using the helm chart under this step's elasticsearch directory:

```
ubuntu@ip-172-31-18-59:~/kubecon-eu-2019$ helm template ./step07/elasticsearch/. | kubectl apply -f -

poddisruptionbudget.policy/elasticsearch-master-pdb created
service/elasticsearch-master created
service/elasticsearch-master-headless created
statefulset.apps/elasticsearch-master created

ubuntu@ip-172-31-18-59:~/kubecon-eu-2019$
```

Next, deploy Kibana. Kibana provides a visualization layer for Elasticsearch data, making it an excellent compliment to
any deployment of Elasticsearch. This time, pass the `--name` flag and use the name you've used for the other Helm
deployments:

```
ubuntu@ip-172-31-18-59:~/kubecon-eu-2019$ helm template ./step07/kibana/. --name release-name  | kubectl apply -f -

service/release-name-kibana created
deployment.apps/release-name-kibana created

ubuntu@ip-172-31-18-59:~/kubecon-eu-2019$
```

Finally, deploy the Fluentd helm chart provided in this repo. This will allow Kubernetes, and your service, to feed
event data into Elasticsearch:

```
ubuntu@ip-172-31-18-59:~/kubecon-eu-2019$ helm template ./step07/fluentd/. | kubectl apply -f -

configmap/fluentd-sidecar created

ubuntu@ip-172-31-18-59:~/kubecon-eu-2019$
```

Wait a few minutes for all of your deployed sources to come online, and soon you'll be able to receive data from it.

```
ubuntu@ip-172-31-18-59:~/kubecon-eu-2019$ kubectl get all,cm

NAME                                           READY   STATUS    RESTARTS   AGE
pod/elasticsearch-master-0                     1/1     Running   0          118s
pod/release-name-kibana-6877866855-vcfpv       1/1     Running   0          106s
pod/release-name-ossp-6984b7d78-7xmln          1/1     Running   0          4m20s
pod/release-name-prometheus-5c4c95954d-snb6x   4/4     Running   0          3m9s

NAME                                    TYPE            CLUSTER-IP       EXTERNAL-IP                                                                 PORT(S)                         AGE
service/elasticsearch-master            ClusterIP       10.106.185.211   <none>                                                                      9200/TCP,9300/TCP               118s
service/elasticsearch-master-headless   ClusterIP       None             <none>                                                                      9200/TCP                        118s
service/release-name-kibana             NodePort        10.99.53.87      <none>                                                                      5601:30235/TCP                  106s
service/release-name-ossp               LoadBalancer    10.101.68.99     a806b89387ba211e98f95020deaa3a09-548583198.eu-central-1.elb.amazonaws.com   50088:31811/TCP                 4m20s
service/release-name-prometheus         NodePort        10.110.114.179   <none>                                                                      9090:31901/TCP,3000:30861/TCP   3m9s

NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/release-name-kibana       1/1     1            1           106s
deployment.apps/release-name-ossp         1/1     1            1           4m20s
deployment.apps/release-name-prometheus   1/1     1            1           3m9s

NAME                                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/release-name-kibana-6877866855       1         1         1       106s
replicaset.apps/release-name-ossp-6984b7d78          1         1         1       4m20s
replicaset.apps/release-name-prometheus-5c4c95954d   1         1         1       3m9s

NAME                                    READY   AGE
statefulset.apps/elasticsearch-master   1/1     118s

ubuntu@ip-172-31-18-59:~/kubecon-eu-2019$
```

Your helm charts are deployed, but you're not done setting up Fluentd.

### 2. Enabling ossp application logging

Before you can use logging, you first need to enable logging on the ossp pod. Currently, the ossp service outputs
directly to STDOUT, which is redirected to a series of logs found in /var/log. As a developer, you may not have access
to these logs in that directory. There are two options: You can change your code for the ossp service to log to a file,
or use shell redirection. In the interest of time, go for the shell redirection.

In order to enable that, however, you will need to create a container that has a shell. If you recall from step 2, you
built the ossp service with a scratch base image - one with no filesystem, tools or other programs - including a shell.
In order to use basic shell redirection, you will need to change the base image to something that includes a shell.

Open your Dockerfile and change the isolated container from the scratch base image to Alpine, the same version you built
the ossp binary on originally:

```
ubuntu@ip-172-31-18-59:~/kubecon-eu-2019$ vim Dockerfile

ubuntu@ip-172-31-18-59:~/kubecon-eu-2019$ cat Dockerfile

FROM golang:1.12.5-alpine3.9 as server-build
WORKDIR /go/src/app
COPY . .
RUN apk add --no-cache git
RUN go get -u google.golang.org/grpc
RUN go get -u github.com/golang/protobuf/protoc-gen-go
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo server.go

# Package the binary in its own isolated container
FROM alpine:3.9
COPY --from=server-build /go/src/app/server /server
EXPOSE 50088
ENTRYPOINT [ "/server" ]

ubuntu@ip-172-31-18-59:~/kubecon-eu-2019$
```

Now, build the Dockerfile, tagging this image _v0.2_:

```
ubuntu@ip-172-31-18-59:~/kubecon-eu-2019$ sudo docker build -t ossp:v0.2 .

Sending build context to Docker daemon    148MB
Step 1/11 : FROM golang:1.12.5-alpine3.9 as server-build
 ---> c7330979841b
Step 2/11 : WORKDIR /go/src/app
 ---> Using cache
 ---> 72987044156e
Step 3/11 : COPY . .
 ---> b7be8e21780b
Step 4/11 : RUN apk add --no-cache git
 ---> Running in b3f7c1c448dd
fetch http://dl-cdn.alpinelinux.org/alpine/v3.9/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.9/community/x86_64/APKINDEX.tar.gz
(1/6) Installing nghttp2-libs (1.35.1-r0)
(2/6) Installing libssh2 (1.8.2-r0)
(3/6) Installing libcurl (7.64.0-r1)
(4/6) Installing expat (2.2.6-r0)
(5/6) Installing pcre2 (10.32-r1)
(6/6) Installing git (2.20.1-r0)
Executing busybox-1.29.3-r10.trigger
OK: 20 MiB in 21 packages
Removing intermediate container b3f7c1c448dd
 ---> 607ccbff9a7d
Step 5/11 : RUN go get -u google.golang.org/grpc
 ---> Running in 743565195e1f
Removing intermediate container 743565195e1f
 ---> 613be503ca0b
Step 6/11 : RUN go get -u github.com/golang/protobuf/protoc-gen-go
 ---> Running in 768458c49a53
Removing intermediate container 768458c49a53
 ---> b35282cfcbc8
Step 7/11 : RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo server.go
 ---> Running in 3f2ea0708e0c
Removing intermediate container 3f2ea0708e0c
 ---> 8a5dd3ff8ea8
Step 8/11 : FROM alpine:3.9
 ---> 055936d39205
Step 9/11 : COPY --from=server-build /go/src/app/server /server
 ---> d346fbecac6a
Step 10/11 : EXPOSE 50088
 ---> Running in 8bde5c86e6c8
Removing intermediate container 8bde5c86e6c8
 ---> 543d96e0b8a9
Step 11/11 : ENTRYPOINT [ "/server" ]
 ---> Running in afa3baff8c55
Removing intermediate container afa3baff8c55
 ---> ea1ead255d91
Successfully built ea1ead255d91
Successfully tagged ossp:v0.2

ubuntu@ip-172-31-18-59:~/kubecon-eu-2019$
```

Before pushing it up to Harbor, give it a tag that conforms with all of the image name requirements:

```
ubuntu@ip-172-31-18-59:~$ sudo docker image tag ossp:v0.2 reg.rx-m.net/cndev/wra.ossp:v0.2

ubuntu@ip-172-31-18-59:~$
```

Now push it!

```
ubuntu@ip-172-31-18-59:~$ sudo docker image push reg.rx-m.net/cndev/wra.ossp:v0.2

The push refers to repository [reg.rx-m.net/cndev/wra.ossp]
1d20e816dced: Pushed
v0.2: digest: sha256:1d20e816dcedef3c63ac3b4ee3cdc8875f8aa34a3cef67bdf7b87a52c7b297 size: 528

ubuntu@ip-172-31-18-59:~$
```

Great, the v0.2 of your ossp service container is now up on Harbor.


### 3. Deploying the Fluentd sidecar

The Fluentd helm chart you deployed only deployed a configmap that will support the actual Fluentd operation. The
original helm chart it is based on actually deploys a daemonset, which ensures that there is a pod available on every
single node in the cluster - something that an SRE, not a Cloud Native Dev, should be concerned about.

In order to deploy the actual Fluentd instance that will serve logs to Elasticsearch, you will need to add a Fluentd
sidecar container to your ossp service pod. This will ensure that a Fluentd container will always be deployed
alongside your ossp service container.

This will be done by adding the following yaml block to your existing ossp deployment:

```yaml
        command:
        - "/bin/sh"
        - "-c"
        - "/server 2>&1 | tee /tmp/log/ossp.log"  
        volumeMounts:
        - name: varlog
          mountPath: /tmp/log
      - name: fluentd
        image: gcr.io/fluentd-elasticsearch/fluentd:v2.5.2
        env:
          - name: FLUENTD_ARGS
            value: -c /fluentd/etc/fluentd-sidecar.conf
        volumeMounts:
        - name: fluentd-sidecar
          mountPath: /fluentd/etc
        - name: varlog
          mountPath: /tmp/log
      volumes:
      - name: varlog
        emptyDir: {}
      - name: fluentd-sidecar
        configMap:
          name: fluentd-sidecar          
```

- A command that will override the ossp container's entrypoint, forcing the ossp service to redirect logs to a file at
`/tmp/log/ossp.log`
- A new container named Fluentd will be added to the ossp pod, running a Fluentd image from that is already set up to
forward events to Elasticsearch
- Both containers will mount a directory, varlog, to their filesystems to share logs, so an ephimeral `varlog` emptyDir
volume is declared. This directory shares the lifespan of the pod, and will be deleted if the pod dies.
- The Fluentd container also needs to mount its configuration file which is stored inside the `fluentd-sidecar` configmap.
The configmap is declared as a configMap volume and fluentd-sidecar.conf is mounted in the Fluentd container under
/fluentd/etc/

Use `kubectl edit` to edit the ossp deployment, making sure to do the following:

- Update the ossp container image to use the `v0.2` tag
- Insert the yaml block above just after the `terminationMessagePolicy: File` line of the original ossp container

```
ubuntu@ip-172-31-18-59:~/kubecon-eu-2019$ kubectl edit deploy release-name-ossp

kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2019-05-21T19:09:51Z"
  generation: 1
  labels:
    app: ossp
  name: release-name-ossp
  namespace: wra-172-31-30-5
  resourceVersion: "2531"
  selfLink: /apis/extensions/v1beta1/namespaces/wra-172-31-30-5/deployments/release-name-ossp
  uid: 29bd7922-8633-11e9-a3cc-000c29057440
  spec:
    progressDeadlineSeconds: 600
    replicas: 1
    revisionHistoryLimit: 10
    selector:
      matchLabels:
        app: ossp
    strategy:
      rollingUpdate:
        maxSurge: 25%
        maxUnavailable: 25%
      type: RollingUpdate
    template:
      metadata:
        creationTimestamp: null
        labels:
          app: ossp
      spec:
        containers:
  # Update the image to use the new v0.2 tag        
        - image: reg.rx-m.net/cndev/wra.ossp:v0.2
          imagePullPolicy: Never
          name: ossp
          ports:
          - containerPort: 50088
            protocol: TCP
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
  # Insert the updated container OSSP container spec and new Fluentd container spec here
          command:
          - "/bin/sh"
          - "-c"
          - "/server 2>&1 | tee /tmp/log/ossp.log"
          volumeMounts:
          - name: varlog
            mountPath: /tmp/log
        - name: fluentd
          image: gcr.io/fluentd-elasticsearch/fluentd:v2.5.2
          env:
            - name: FLUENTD_ARGS
              value: -c /fluentd/etc/fluentd-sidecar.conf
          volumeMounts:
          - name: fluentd-sidecar
            mountPath: /fluentd/etc
          - name: varlog
            mountPath: /tmp/log
        volumes:
        - name: varlog
          emptyDir: {}
        - name: fluentd-sidecar
          configMap:
            name: fluentd-sidecar
        dnsPolicy: ClusterFirst
...

:wq

deployment.extensions/release-name-ossp edited

ubuntu@ip-172-31-18-59:~/kubecon-eu-2019$
```

Check your pods now:

```
ubuntu@ip-172-31-18-59:~/kubecon-eu-2019$ kubectl get pods

NAME                                       READY   STATUS    RESTARTS   AGE
elasticsearch-master-0                     1/1     Running   0          6m45s
release-name-kibana-6877866855-vcfpv       1/1     Running   0          6m33s
release-name-ossp-bc7574b99-5tnfc          2/2     Running   0          11s
release-name-prometheus-5c4c95954d-snb6x   4/4     Running   0          7m56s

ubuntu@ip-172-31-18-59:~/kubecon-eu-2019$
```

Your ossp pod is now much younger than the rest of your pods, and now reports 2/2 ready containers, indicating that
Fluentd was deployed alongside the ossp container.

Check the Fluentd container logs with `kubectl logs`, using `-c fluentd` to specify which container to take logs from:

```
ubuntu@ip-172-31-18-59:~/kubecon-eu-2019$ kubectl logs release-name-ossp-bc7574b99-5tnfc -c fluentd

2019-06-07 17:12:29 +0000 [info]: parsing config file is succeeded path="/fluentd/etc/fluentd-sidecar.conf"
...
2019-06-07 17:12:29 +0000 [info]: #0 starting fluentd worker pid=10 ppid=1 worker=0
2019-06-07 17:12:29 +0000 [info]: #0 following tail of /tmp/log/ossp.log
2019-06-07 17:12:29 +0000 [info]: #0 fluentd worker is now running worker=0

```

Fluentd found its configuration under /fluentd/etc/ and is now tailing the ossp.log file under /tmp/log/, it should now
be ready to send events to Elasticsearch!


### 4. Generating Events

Use your `client.go` program to send some events to the ossp service:

```
ubuntu@ip-172-31-18-59:~/kubecon-eu-2019$ go run client.go a806b89387ba211e98f95020deaa3a09-548583198.eu-central-1.elb.amazonaws.com:31811 fluentd

2019/06/07 17:13:39 Projects: name:"fluentd" custodian:"cncf"

ubuntu@ip-172-31-18-59:~/kubecon-eu-2019$ go run client.go a806b89387ba211e98f95020deaa3a09-548583198.eu-central-1.elb.amazonaws.com:31811 containerd

2019/06/07 17:13:44 Projects: name:"containerd" custodian:"cncf"

ubuntu@ip-172-31-18-59:~/kubecon-eu-2019$ go run client.go a806b89387ba211e98f95020deaa3a09-548583198.eu-central-1.elb.amazonaws.com:31811 kubernetes

2019/06/07 17:13:54 Projects: name:"kubernetes" custodian:"cncf"

ubuntu@ip-172-31-18-59:~/kubecon-eu-2019$ go run client.go a806b89387ba211e98f95020deaa3a09-548583198.eu-central-1.elb.amazonaws.com:31811 prometheus

2019/06/07 17:14:01 Projects: name:"prometheus" custodian:"cncf"

ubuntu@ip-172-31-18-59:~/kubecon-eu-2019$
```

After generating some traffic, examine the logs from your ossp deployment pod:

```
ubuntu@ip-172-31-18-59:~/kubecon-eu-2019$ kubectl logs release-name-ossp-bc7574b99-5tnfc -c ossp

2019/06/07 02:11:48 Received: fluentd
2019/06/07 02:11:53 Received: containerd
2019/06/07 02:12:02 Received: kubernetes
2019/06/07 02:12:35 Received: prometheus

ubuntu@ip-172-31-18-59:~/kubecon-eu-2019$
```

Keep these events in mind, you will see them again in a more colorful way in the next step.


### 5. Visualizing log data

With events sent Elasticsearch through Fluentd, you can now use Kibana to check the events that are coming in from all
parts of your Kubernetes cluster (within your namespace).

Kibana was deployed with a NodePort service, meaning that you can access it visiting the URL of any of the Kubernetes
workers at your own port.

Retrieve the nodeport for Kibana with `kubectl get svc`:

```
ubuntu@ip-172-31-18-59:~/kubecon-eu-2019$ kubectl get svc

NAME                            TYPE            CLUSTER-IP       EXTERNAL-IP                                                                 PORT(S)                         AGE
elasticsearch-master            ClusterIP       10.106.185.211   <none>                                                                      9200/TCP,9300/TCP               2m
elasticsearch-master-headless   ClusterIP       None             <none>                                                                      9200/TCP                        2m
release-name-kibana             NodePort        10.99.53.87      <none>                                                                      5601:30235/TCP                  2m
release-name-ossp               LoadBalancer    10.101.68.99     a806b89387ba211e98f95020deaa3a09-548583198.eu-central-1.elb.amazonaws.com   50088:31811/TCP                 6m
release-name-prometheus         NodePort        10.110.114.179   <none>                                                                      9090:31901/TCP,3000:30861/TCP   5m

ubuntu@ip-172-31-18-59:~/kubecon-eu-2019$
```

Here, you can see that Kibana has routed its port, 5601, to port 30892 on the cluster.

Pick one of the External IP addresses from the list of available worker nodes retrieved by `kubectl get nodes -o wide`:

```
ubuntu@ip-172-31-18-59:~/kubecon-eu-2019$ kubectl get nodes -o wide

NAME                                               STATUS   ROLES    AGE   VERSION   INTERNAL-IP       EXTERNAL-IP      OS-IMAGE         KERNEL-VERSION                CONTAINER-RUNTIME
ip-192-168-104-49.eu-central-1.compute.internal    Ready    <none>   12h   v1.12.7   192.168.104.49    35.156.190.95    Amazon Linux 2   4.14.106-97.85.amzn2.x86_64   docker://18.6.1
ip-192-168-117-8.eu-central-1.compute.internal     Ready    <none>   12h   v1.12.7   192.168.117.8     35.159.40.214    Amazon Linux 2   4.14.106-97.85.amzn2.x86_64   docker://18.6.1
ip-192-168-141-101.eu-central-1.compute.internal   Ready    <none>   10h   v1.12.7   192.168.141.101   18.195.161.16    Amazon Linux 2   4.14.106-97.85.amzn2.x86_64   docker://18.6.1
ip-192-168-142-13.eu-central-1.compute.internal    Ready    <none>   10h   v1.12.7   192.168.142.13    54.93.170.136    Amazon Linux 2   4.14.106-97.85.amzn2.x86_64   docker://18.6.1
ip-192-168-158-118.eu-central-1.compute.internal   Ready    <none>   12h   v1.12.7   192.168.158.118   54.93.252.48     Amazon Linux 2   4.14.106-97.85.amzn2.x86_64   docker://18.6.1
...

ubuntu@ip-172-31-18-59:~/kubecon-eu-2019$
```

In your browser window, enter one of the external IP address or FQDN of one of the Kubernetes worker nodes with your
Kibana port, like: `http://35.156.190.95:30235`

You should be brought to the Kibana welcome page.

![Welcome to Kibana](./images/kibana-welcome.PNG)

Click "Explore on my Own":

![Welcome to Kibana](./images/kibana-explore.PNG)

Go to Discover:

![Welcome to Kibana](./images/kibana-discover.PNG)

You will be prompted to create an index pattern. Kibana needs to be aware of all indices within Elasticsearch so it can
formulate effective queries against them to better visualize data. There should be some indices starting with `logstash`
already present.

The EFK stack is actually an alternative to the ELK (Elasticsearch, Logstash, and Kibana). The only difference is that
Fluentd takes Logstash's place. By default, Fluentd's Elasticsearch plugin will make it act identical to Logstash, thus
causing Fluentd to send all events in the same manner as Logstash would.

Type "logstash" into the Index Pattern text box:

![Creating the logstash index pattern](./images/kibana-create-index-1.PNG)

If Kibana reports "Success! Your index pattern matches ...," click the `> Next step` button:

You will next be prompted to select a time filter. This allows Kibana to designate a field within a Fluentd event as a
sortable time field.

In the dropdown, you will find @timestamp available as a choice, so select it:

![Creating the logstash index pattern](./images/kibana-create-index-2.PNG)

That's it. Click `Create index pattern`. Once it finishes, you should see an index with all the fields:

![Continuing to create the logstash index pattern](./images/kibana-created-index.PNG)

In the left menu bar, click `Discover` again:

![The events you sent to the ossp service in step 2](./images/ossp-event.PNG)

You should now see your ossp service's logs in Kibana! Take a minute to look around; with its current setting Fluentd is
capture all activity coming to all containers within your namespace.

One more piece of the observability triad to go, tracing application calls with Istio:
[../step08/README.md](../step08/README.md)



<br>

Congratulations, you have completed the tutorial step!

<br>

_Copyright (c) 2019 RX-M LLC, Cloud Native Consulting, all rights reserved_

[RX-M LLC]: http://rx-m.io/rxm-cnc.svg "RX-M LLC"
