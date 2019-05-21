![rx-m LLC][RX-M LLC]


# A Day in the life of a cloud native developer


## Step 08 - Istio

Istio helps reduce the complexity of microservice application deployments, and eases the strain on development teams. It
is an open source service mesh that layers transparently onto existing distributed application platforms. It includes
APIs that let it integrate into any logging platform, or telemetry or policy system. Istio’s diverse feature set lets
you successfully, and efficiently, run a distributed microservice architecture, and provides a uniform way to secure,
connect, and monitor microservices.

In this step we're going to explore Istio features from the vantage of our ossproject service.


### 1. Accessing Istio

Navigate to the Istio console (known as Kiali): http://kiali.rx-m.net:15029/kiali/console/

Login as admin/admin.

[login](./images/login.png)

Locate you namespace and drill down to you application:

[no sidecar](./images/nos.png)

Notice that Istio complains that there is no sidecar. This is because namespaces configured to use Istio will
automatically have an envoy sidecars injected into every pod to create the routing mesh.


### 2. Enabling Istio

To enable Istio for our application we need to set the `istio-injection=enabled` label on our namespace. First delete
the old deployment without side cars:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ kubectl get deploy

NAME                                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
release-name-ossp                    1         1         1            1           80m
wra-172-31-30-5-metrics-prometheus   1         1         1            1           46m

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ kubectl delete deploy release-name-ossp

deployment.extensions "release-name-ossp" deleted

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

Now label your namespace (be sure to enter your namespace name):

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ kubectl config view | grep namespace

    namespace: wra-172-31-30-5

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ kubectl label namespace wra-172-31-30-5 istio-injection=enabled

namespace/wra-172-31-30-5 labeled

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

Now redeploy your pods:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ kubectl create -f ossp-dep.yaml

deployment.extensions/ossp created

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

List out the pods until you see all containers running:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ kubectl get po

NAME                                                  READY   STATUS            RESTARTS   AGE
ossp-67469746c-k6vgj                                  0/2     PodInitializing   0          10s
ossp-67469746c-qjlwp                                  1/2     Running           0          10s
wra-172-31-30-5-metrics-prometheus-66b7bd6568-qqx7p   4/4     Running           0          50m

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ kubectl get po

NAME                                                  READY   STATUS            RESTARTS   AGE
ossp-67469746c-k6vgj                                  0/2     PodInitializing   0          18s
ossp-67469746c-qjlwp                                  2/2     Running           0          18s
wra-172-31-30-5-metrics-prometheus-66b7bd6568-qqx7p   4/4     Running           0          50m

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ kubectl get po

NAME                                                  READY   STATUS    RESTARTS   AGE
ossp-67469746c-k6vgj                                  2/2     Running   0          27s
ossp-67469746c-qjlwp                                  2/2     Running   0          27s
wra-172-31-30-5-metrics-prometheus-66b7bd6568-qqx7p   4/4     Running   0          50m

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

Now describe one of the new pods:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ kubectl describe pod ossp-67469746c-k6vgj
Name:               ossp-67469746c-k6vgj
Namespace:          wra-172-31-30-5
Priority:           0
PriorityClassName:  <none>
Node:               ip-192-168-104-49.eu-central-1.compute.internal/192.168.104.49
Start Time:         Tue, 21 May 2019 04:37:10 +0000
Labels:             app=ossp
                    pod-template-hash=67469746c
Annotations:        sidecar.istio.io/status:
                      {"version":"f5ccf91fad7f2d056f204a075aff1128670ca6997fecdeccaf7a2af6a498558c","initContainers":["istio-init"],"containers":["istio-proxy"]...
Status:             Running
IP:                 192.168.65.41
Controlled By:      ReplicaSet/ossp-67469746c
Init Containers:
  istio-init:
    Container ID:  docker://166c1625011190101d5f574a55d7eb0cdf9e673e0cde76a21c8200abee74af99
    Image:         docker.io/istio/proxy_init:1.1.7
    Image ID:      docker-pullable://istio/proxy_init@sha256:9056ebb0757be99006ad568331e11bee99ae0daaa4459e7c15dfaf0e0cba2f48
    Port:          <none>
    Host Port:     <none>
    Args:
      -p
      15001
      -u
      1337
      -m
      REDIRECT
      -i
      *
      -x

      -b
      50088
      -d
      15020
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Tue, 21 May 2019 04:37:19 +0000
      Finished:     Tue, 21 May 2019 04:37:20 +0000
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     100m
      memory:  50Mi
    Requests:
      cpu:        10m
      memory:     10Mi
    Environment:  <none>
    Mounts:       <none>
Containers:
  ossp:
    Container ID:   docker://4e962d33140ffb6023c021cac1d0b1fc1023cda6266e0e2f2b5805f473e9147a
    Image:          reg.rx-m.net/cndev/wra.ossp:latest
    Image ID:       docker-pullable://reg.rx-m.net/cndev/wra.ossp@sha256:cc8ae1c9e75007ef3c63ac3b4ee3cdc8875f8aa34a3cef67bdf7b87a52c7b297
    Port:           50088/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 21 May 2019 04:37:20 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-72tx9 (ro)
  istio-proxy:
    Container ID:  docker://97fd54221ad3dd0c916359c4c52e84cd8114b1d4bd7d811a2426cccaa0215d4a
    Image:         docker.io/istio/proxyv2:1.1.7
    Image ID:      docker-pullable://istio/proxyv2@sha256:e6f039115c7d5ef9c8f6b049866fbf9b6f5e2255d3a733bb8756b36927749822
    Port:          15090/TCP
    Host Port:     0/TCP
    Args:
      proxy
      sidecar
      --domain
      $(POD_NAMESPACE).svc.cluster.local
      --configPath
      /etc/istio/proxy
      --binaryPath
      /usr/local/bin/envoy
      --serviceCluster
      ossp.$(POD_NAMESPACE)
      --drainDuration
      45s
      --parentShutdownDuration
      1m0s
      --discoveryAddress
      istio-pilot.istio-system:15010
      --zipkinAddress
      zipkin.istio-system:9411
      --connectTimeout
      10s
      --proxyAdminPort
      15000
      --concurrency
      2
      --controlPlaneAuthPolicy
      NONE
      --statusPort
      15020
      --applicationPorts
      50088
    State:          Running
      Started:      Tue, 21 May 2019 04:37:28 +0000
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     2
      memory:  1Gi
    Requests:
      cpu:      100m
      memory:   128Mi
    Readiness:  http-get http://:15020/healthz/ready delay=1s timeout=1s period=2s #success=1 #failure=30
    Environment:
      POD_NAME:                      ossp-67469746c-k6vgj (v1:metadata.name)
      POD_NAMESPACE:                 wra-172-31-30-5 (v1:metadata.namespace)
      INSTANCE_IP:                    (v1:status.podIP)
      ISTIO_META_POD_NAME:           ossp-67469746c-k6vgj (v1:metadata.name)
      ISTIO_META_CONFIG_NAMESPACE:   wra-172-31-30-5 (v1:metadata.namespace)
      ISTIO_META_INTERCEPTION_MODE:  REDIRECT
      ISTIO_METAJSON_LABELS:         {"app":"ossp","pod-template-hash":"67469746c"}

    Mounts:
      /etc/certs/ from istio-certs (ro)
      /etc/istio/proxy from istio-envoy (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-72tx9 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-72tx9:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-72tx9
    Optional:    false
  istio-envoy:
    Type:    EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:  Memory
  istio-certs:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  istio.default
    Optional:    true
QoS Class:       Burstable
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From                                                      Message
  ----    ------     ----  ----                                                      -------
  Normal  Scheduled  46s   default-scheduler                                         Successfully assigned wra-172-31-30-5/ossp-67469746c-k6vgj to ip-192-168-104-49.eu-central-1.compute.internal
  Normal  Pulling    45s   kubelet, ip-192-168-104-49.eu-central-1.compute.internal  pulling image "docker.io/istio/proxy_init:1.1.7"
  Normal  Pulled     38s   kubelet, ip-192-168-104-49.eu-central-1.compute.internal  Successfully pulled image "docker.io/istio/proxy_init:1.1.7"
  Normal  Created    37s   kubelet, ip-192-168-104-49.eu-central-1.compute.internal  Created container
  Normal  Started    37s   kubelet, ip-192-168-104-49.eu-central-1.compute.internal  Started container
  Normal  Pulled     36s   kubelet, ip-192-168-104-49.eu-central-1.compute.internal  Successfully pulled image "reg.rx-m.net/cndev/wra.ossp:latest"
  Normal  Pulling    36s   kubelet, ip-192-168-104-49.eu-central-1.compute.internal  pulling image "reg.rx-m.net/cndev/wra.ossp:latest"
  Normal  Created    36s   kubelet, ip-192-168-104-49.eu-central-1.compute.internal  Created container
  Normal  Started    36s   kubelet, ip-192-168-104-49.eu-central-1.compute.internal  Started container
  Normal  Pulling    36s   kubelet, ip-192-168-104-49.eu-central-1.compute.internal  pulling image "docker.io/istio/proxyv2:1.1.7"
  Normal  Pulled     29s   kubelet, ip-192-168-104-49.eu-central-1.compute.internal  Successfully pulled image "docker.io/istio/proxyv2:1.1.7"
  Normal  Created    29s   kubelet, ip-192-168-104-49.eu-central-1.compute.internal  Created container
  Normal  Started    28s   kubelet, ip-192-168-104-49.eu-central-1.compute.internal  Started container

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

As you can see, Istio injected an Envoy sidecar into the pod to establish it in the service mesh.


### 3. Monitoring calls

Use the client to generate some traffic against your service:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ kubectl get service

NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP                                                                 PORT(S)                         AGE
release-name-ossp                    LoadBalancer   10.100.221.157   a446d6c137b7611e98f95020deaa3a09-659563283.eu-central-1.elb.amazonaws.com   50088:30613/TCP                 88m
wra-172-31-30-5-metrics-prometheus   NodePort       10.100.52.191    <none>                                                                      9090:31345/TCP,3000:30218/TCP   54m

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ go run client.go a446d6c137b7611e98f95020deaa3a09-659563283.eu-central-1.elb.amazonaws.com fluentd

2019/05/21 04:41:48 Projects: name:"fluentd" custodian:"cncf"

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ go run client.go a446d6c137b7611e98f95020deaa3a09-659563283.eu-central-1.elb.amazonaws.com fluentd

2019/05/21 04:41:51 Projects: name:"fluentd" custodian:"cncf"

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ go run client.go a446d6c137b7611e98f95020deaa3a09-659563283.eu-central-1.elb.amazonaws.com fluentd

2019/05/21 04:41:52 Projects: name:"fluentd" custodian:"cncf"

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ go run client.go a446d6c137b7611e98f95020deaa3a09-659563283.eu-central-1.elb.amazonaws.com fluentd

2019/05/21 04:41:54 Projects: name:"fluentd" custodian:"cncf"

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

Now return to the Istio dashboard for your application, you will see that the pods no longer report the sidecar missing.
Take a moment to explore the data provided by Istio. Generate more client traffic to see load statistics.

As observable as our app now is, we may still need to debug it on occasion. In a cloud native setting this is not easy,
unless you have telepresence: [../step09/README.md](../step09/README.md)


<br>

Congratulations, you have completed the tutorial step!

<br>

_Copyright (c) 2019 RX-M LLC, Cloud Native Consulting, all rights reserved_

[RX-M LLC]: http://rx-m.io/rxm-cnc.svg "RX-M LLC"
