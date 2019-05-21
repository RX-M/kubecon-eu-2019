![rx-m LLC][RX-M LLC]

# A Day in the life of a cloud native developer


## Step 07 - Logging with Fluentd

Fluentd is a cross platform open-source data collection software project recently graduated and a top level project at
the CNCF. Fluentd is written primarily in Ruby with C in places where performance is critical.

In this step we'll use Fluentd to take the log events from our microservice and forward them to Elasticsearch where we
can view them with Kibana (the infamous EFK stack).


### 1. Deploying the Helm Charts

The lab repo includes helm charts that will deploy the EFK stack for you in your personal namespace.

First, deploy Elasticsearch, providing the values.yaml file under this step's elasticsearch directory:

```
ubuntu@ip-172-31-30-255:~/kubecon-eu-2019$ helm template ./step07/elasticsearch/. | kubectl apply -f -

poddisruptionbudget.policy/elasticsearch-master-pdb created
service/elasticsearch-master created
service/elasticsearch-master-headless created
statefulset.apps/elasticsearch-master created

ubuntu@ip-172-31-30-255:~/kubecon-eu-2019$
```

Next, deploy Kibana. Kibana provides a visualization layer for Elasticsearch data, making it an excellent compliment to
any Elasticsearch deployment. This time, pass the --name flag and put the name of your namespace:

```
ubuntu@ip-172-31-30-255:~/kubecon-eu-2019$ helm template ./step07/kibana/. --name cal-172-31-30-255  | kubectl apply -f -

service/cal-172-31-30-255-kibana created
deployment.apps/cal-172-31-30-255-kibana created

ubuntu@ip-172-31-30-255:~/kubecon-eu-2019$
```

Finally, deploy Fluentd. This will allow Kubernetes, and your service, to feed event data into Elasticsearch For this
step, use the `--set metadata.namespace` to your namespace to ensure that all the Fluentd instances start within your
own namespace:

```
ubuntu@ip-172-31-30-255:~/kubecon-eu-2019$ helm template ./step07/fluentd/. --name cal-172-31-30-255 --set metadata.namespace=cal-172-31-30-255  | kubectl apply -f -

configmap/cal-172-31-30-255-fluentd-elasticsearch created
serviceaccount/cal-172-31-30-255-fluentd-elasticsearch created
clusterrole.rbac.authorization.k8s.io/cal-172-31-30-255-fluentd-elasticsearch created
clusterrolebinding.rbac.authorization.k8s.io/cal-172-31-30-255-fluentd-elasticsearch created
daemonset.apps/cal-172-31-30-255-fluentd-elasticsearch created

ubuntu@ip-172-31-30-255:~/kubecon-eu-2019$
```

Wait a few minutes for all of your deployed sources to come online, and soon you'll be able to receive data from it.

```
ubuntu@ip-172-31-30-255:~/kubecon-eu-2019$ kubectl get all

NAME                                                       READY   STATUS    RESTARTS   AGE
pod/cal-172-31-30-255-fluentd-elasticsearch-8sgn5          1/1     Running   0          19s
pod/cal-172-31-30-255-fluentd-elasticsearch-9n5r5          1/1     Running   0          19s
pod/cal-172-31-30-255-fluentd-elasticsearch-cflt7          1/1     Running   0          19s
pod/cal-172-31-30-255-fluentd-elasticsearch-d6qlj          1/1     Running   0          19s
pod/cal-172-31-30-255-fluentd-elasticsearch-d8ln4          1/1     Running   0          19s
pod/cal-172-31-30-255-fluentd-elasticsearch-gm7lx          1/1     Running   0          19s
pod/cal-172-31-30-255-fluentd-elasticsearch-gpdw7          1/1     Running   0          19s
pod/cal-172-31-30-255-fluentd-elasticsearch-pcnlf          1/1     Running   0          19s
pod/cal-172-31-30-255-fluentd-elasticsearch-pfskf          1/1     Running   0          19s
pod/cal-172-31-30-255-fluentd-elasticsearch-wv7xm          1/1     Running   0          19s
pod/cal-172-31-30-255-kibana-f6bd4dd5f-8b29c               0/1     Running   0          45s
pod/cal-172-31-30-255-metrics-prometheus-676449cdb-rrw5k   4/4     Running   0          63m
pod/elasticsearch-master-0                                 1/1     Running   0          70s
pod/release-name-ossp-67469746c-8lp4g                      1/1     Running   0          16m
pod/release-name-ossp-67469746c-dmbxc                      1/1     Running   0          16m

NAME                                           TYPE           CLUSTER-IP       EXTERNAL-IP                                                                 PORT(S)                         AGE
service/cal-172-31-30-255-kibana               NodePort       10.100.150.42    <none>                                                                      5601:30305/TCP                  45s
service/cal-172-31-30-255-metrics-prometheus   NodePort       10.100.231.121   <none>                                                                      9090:31267/TCP,3000:30242/TCP   63m
service/elasticsearch-master                   ClusterIP      10.100.184.240   <none>                                                                      9200/TCP,9300/TCP               70s
service/elasticsearch-master-headless          ClusterIP      None             <none>                                                                      9200/TCP                        70s
service/release-name-ossp                      LoadBalancer   10.100.67.130    a6f463ffd7b9a11e981790ae78cda592-171018466.eu-central-1.elb.amazonaws.com   50088:31114/TCP                 16m

NAME                                                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/cal-172-31-30-255-fluentd-elasticsearch   10        10        10      10           10          <none>          19s

NAME                                                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cal-172-31-30-255-kibana               1         1         1            0           45s
deployment.apps/cal-172-31-30-255-metrics-prometheus   1         1         1            1           63m
deployment.apps/release-name-ossp                      2         2         2            2           16m

NAME                                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/cal-172-31-30-255-kibana-f6bd4dd5f               1         1         0       45s
replicaset.apps/cal-172-31-30-255-metrics-prometheus-676449cdb   1         1         1       63m
replicaset.apps/release-name-ossp-67469746c                      2         2         2       16m

NAME                                    DESIRED   CURRENT   AGE
statefulset.apps/elasticsearch-master   1         1         70s

ubuntu@ip-172-31-30-255:~/kubecon-eu-2019$
```


### 2. Generating Events

Using your `client.go` program to hit the gRPC microservice, send some events corresponding to some of the growing number
of graduated CNCF projects:

```
ubuntu@ip-172-31-30-255:~/kubecon-eu-2019$ go run client.go a6f463ffd7b9a11e981790ae78cda592-171018466.eu-central-1.elb.amazonaws.com fluentd

2019/05/21 07:48:22 Projects: name:"fluentd" custodian:"cncf"

ubuntu@ip-172-31-30-255:~/kubecon-eu-2019$ go run client.go a6f463ffd7b9a11e981790ae78cda592-171018466.eu-central-1.elb.amazonaws.com containerd

2019/05/21 07:48:27 Projects: name:"containerd" custodian:"cncf"

ubuntu@ip-172-31-30-255:~/kubecon-eu-2019$ go run client.go a6f463ffd7b9a11e981790ae78cda592-171018466.eu-central-1.elb.amazonaws.com prometheus

2019/05/21 07:48:31 Projects: name:"prometheus" custodian:"cncf"

ubuntu@ip-172-31-30-255:~/kubecon-eu-2019$ go run client.go a6f463ffd7b9a11e981790ae78cda592-171018466.eu-central-1.elb.amazonaws.com kubernetes

2019/05/21 07:48:35 Projects: name:"kubernetes" custodian:"cncf"

ubuntu@ip-172-31-30-255:~/kubecon-eu-2019$

```

Once you're done sending them, grab the logs from each of your ossp deployment pods:

```
ubuntu@ip-172-31-30-255:~/kubecon-eu-2019$ kubectl logs release-name-ossp-67469746c-8lp4g

2019/05/21 07:33:56 Received: fluentd
2019/05/21 07:40:52 Received: containerd
2019/05/21 07:40:57 Received: prometheus
2019/05/21 07:48:31 Received: prometheus

ubuntu@ip-172-31-30-255:~/kubecon-eu-2019$ kubectl logs release-name-ossp-67469746c-dmbxc

2019/05/21 07:34:06 Received: fluentd
2019/05/21 07:34:21 Received: prometheus
2019/05/21 07:40:48 Received: fluentd
2019/05/21 07:41:03 Received: kubernetes
2019/05/21 07:48:22 Received: fluentd
2019/05/21 07:48:27 Received: containerd
2019/05/21 07:48:35 Received: kubernetes

ubuntu@ip-172-31-30-255:~/kubecon-eu-2019$
```

Keep these events in mind, you will see them again in a more colorful way in the next step.


### 3. Visualizing log data

With events sent Elasticsearch through Fluentd, you can now use Kibana to check the events that are coming in from all
parts of your Kubernetes cluster (within your namespace).

Kibana was deployed with a NodePort service, meaning that you can access it visiting the URL of any of the Kubernetes
workers at your own port.

Retrieve the nodeport for Kibana with `kubectl get svc`:

```
ubuntu@ip-172-31-30-255:~/kubecon-eu-2019$ kubectl get svc

NAME                                   TYPE           CLUSTER-IP       EXTERNAL-IP                                                                 PORT(S)                         AGE
cal-172-31-30-255-metrics-prometheus   NodePort       10.100.231.121   <none>                                                                      9090:31267/TCP,3000:30242/TCP   58m
elasticsearch-master                   ClusterIP      10.100.232.4     <none>                                                                      9200/TCP,9300/TCP               3m35s
elasticsearch-master-headless          ClusterIP      None             <none>                                                                      9200/TCP                        3m35s
release-name-kibana                    NodePort       10.100.17.189    <none>                                                                      5601:32400/TCP                  2m50s
release-name-ossp                      LoadBalancer   10.100.67.130    a6f463ffd7b9a11e981790ae78cda592-171018466.eu-central-1.elb.amazonaws.com   50088:31114/TCP                 10m

ubuntu@ip-172-31-30-255:~/kubecon-eu-2019$
```

Here, you can see that Kibana has routed its port, 5601, to port 32188 on the cluster.

In your browser window, enter one of the external IP address or FQDN of one of the Kubernetes worker nodes with your
port. You should be brought to the Kibana welcome page.

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
causing Fluentd to send all events in the same manner as logstash would.

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

![Kibana sees data!](./images/kibana-discover-populated.PNG)

You should now see that Kibana has data in it! It's quite a lot, so try to filter it down. Try to find the events you
sent to the OSSP program earlier.

Click `Add a filter +`:

![Adding Event Filters to Kibana](./images/kibana-addfilter.PNG)

Select the following:

- kubernetes.container_name is ossp
- kubernetes.namespace_name is `<Your Namespace>`

And in the top search bar, look for **received**:

![The events you sent to the OSSP service in step 2](./images/ossp-event.PNG)

You should now see your GRPC service's logs in Kibana! Take a minute to look around; with its current setting Fluentd is
capture all activity coming to all containers within your namespace.

One more piece of the observability triad to go, tracing application calls with Istio:
[../step08/README.md](../step08/README.md)



<br>

Congratulations, you have completed the tutorial step!

<br>

_Copyright (c) 2019 RX-M LLC, Cloud Native Consulting, all rights reserved_

[RX-M LLC]: http://rx-m.io/rxm-cnc.svg "RX-M LLC"
