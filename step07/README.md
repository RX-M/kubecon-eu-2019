![rx-m LLC][RX-M LLC]

# A Day in the life of a cloud native developer


## Step 07 - Logging with Elasticsearch, Fluentd, and Kibana (EFK) Stack

### 1. Adding Logging to the service

### 2. Deploying the Helm Charts

To deploy the EFK stack, you will use your own values.yaml files to provide your own values to each of the helm charts.
By passing the `-f` option, you can specify what values file to use. The Helm chart will parse the provided yaml file
and apply any settings in it that are used by its templates. Helm will use the chart defaults for any values that are
not provided in the user-defined values.yaml file.

You're now ready to deploy the helm charts. First, deploy Elasticsearch, providing the values.yaml file under this
step's elasticsearch directory:

```
ubuntu@labsys:~$ helm install elastic/elasticsearch \
-f kubecon-eu-2019/step07/elasticsearch/values.yaml  \
--name $INITIALS-efk-es

NAME:   cal-efk-es
LAST DEPLOYED: Mon May 20 11:18:26 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Pod(related)
NAME                    READY  STATUS    RESTARTS  AGE
elasticsearch-master-0  0/1    Init:0/1  0         0s

==> v1/Service
NAME                           TYPE       CLUSTER-IP      EXTERNAL-IP  PORT(S)            AGE
elasticsearch-master           ClusterIP  10.107.252.106  <none>       9200/TCP,9300/TCP  0s
elasticsearch-master-headless  ClusterIP  None            <none>       9200/TCP           0s

==> v1beta1/PodDisruptionBudget
NAME                      MIN AVAILABLE  MAX UNAVAILABLE  ALLOWED DISRUPTIONS  AGE
elasticsearch-master-pdb  N/A            1                0                    0s

==> v1beta1/StatefulSet
NAME                  READY  AGE
elasticsearch-master  0/1    0s


NOTES:
1. Watch all cluster members come up.
  $ kubectl get pods --namespace=default -l app=elasticsearch-master -w
2. Test cluster health using Helm test.
  $ helm test cal-efk-es

ubuntu@labsys:~$
```

Next, deploy Kibana. Kibana provides a visualization layer for Elasticsearch data, making it an excellent compliment to
any Elasticsearch deployment.

```
ubuntu@labsys:~$ helm install elastic/kibana \
-f kubecon-eu-2019/step07/kibana/values.yaml  \
--name $INITIALS-efk-kibana

NAME:   cal-efk-kibana
LAST DEPLOYED: Mon May 20 11:18:59 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Deployment
NAME                   READY  UP-TO-DATE  AVAILABLE  AGE
cal-efk-kibana-kibana  0/1    1           0          0s

==> v1/Pod(related)
NAME                                   READY  STATUS             RESTARTS  AGE
cal-efk-kibana-kibana-d78957468-dchcr  0/1    ContainerCreating  0         0s

==> v1/Service
NAME                   TYPE       CLUSTER-IP    EXTERNAL-IP  PORT(S)   AGE
cal-efk-kibana-kibana  ClusterIP  10.101.87.74  <none>       5601/TCP  0s

ubuntu@labsys:~$
```

Finally, deploy Fluentd. This will allow Kubernetes, and your service, to feed event data into Elasticsearch:

```
ubuntu@labsys:~$ helm install kiwigrid/fluentd-elasticsearch \
-f kubecon-eu-2019/step07/fluentd/values.yaml  \
--name $INITIALS-efk-fluentd

NAME:   cal-efk-fluentd
LAST DEPLOYED: Mon May 20 11:20:01 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ClusterRole
NAME                                   AGE
cal-efk-fluentd-fluentd-elasticsearch  0s

==> v1/ClusterRoleBinding
NAME                                   AGE
cal-efk-fluentd-fluentd-elasticsearch  0s

==> v1/ConfigMap
NAME                                   DATA  AGE
cal-efk-fluentd-fluentd-elasticsearch  6     0s

==> v1/DaemonSet
NAME                                   DESIRED  CURRENT  READY  UP-TO-DATE  AVAILABLE  NODE SELECTOR  AGE
cal-efk-fluentd-fluentd-elasticsearch  1        1        0      1           0          <none>         0s

==> v1/Pod(related)
NAME                                         READY  STATUS             RESTARTS  AGE
cal-efk-fluentd-fluentd-elasticsearch-g22c8  0/1    ContainerCreating  0         0s

==> v1/ServiceAccount
NAME                                   SECRETS  AGE
cal-efk-fluentd-fluentd-elasticsearch  1        0s


NOTES:
1. To verify that Fluentd has started, run:

  kubectl --namespace=default get pods -l "app.kubernetes.io/name=fluentd-elasticsearch,app.kubernetes.io/instance=cal-efk-fluentd"

THIS APPLICATION CAPTURES ALL CONSOLE OUTPUT AND FORWARDS IT TO elasticsearch . Anything that might be identifying,
including things like IP addresses, container images, and object names will NOT be anonymized.

ubuntu@labsys:~$
```

Wait a few minutes for all of your deployed sources to come online, and soon you'll be able to receive data from it.

### 3. Exploring the configuration

```
kubectl get cm cal-efk-fluentd-fluentd-elasticsearch -o yaml


```

### 4. Checking it out

First, cu

Kibana was deployed with a NodePort service, meaning that you can access it visiting the URL of any of the Kubernetes workers at your own port.

```
ubuntu@labsys:~$ kubectl get svc

NAME                            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
cal-efk-kibana-kibana           NodePort    10.109.242.62   <none>        5601:32536/TCP      64s
elasticsearch-master            ClusterIP   10.105.41.133   <none>        9200/TCP,9300/TCP   2m27s
elasticsearch-master-headless   ClusterIP   None            <none>        9200/TCP            2m27s
kubernetes                      ClusterIP   10.96.0.1       <none>        443/TCP             5d3h

ubuntu@labsys:~$
```

Here, you can see that Kibana has routed its port, 5601, to port 32536 on the cluster. In your browser window, enter one
of the external IP address or FQDN of one of the Kubernetes worker nodes with your port. You should be brought to the
Kibana welcome page.

Navigate to Kibana:

Go to Discover:

You should be prompted to create an index pattern. There should be an index starting with `logstash` already present.

Type "logstash" into the Index Pattern text box:

If Kibana reports "Success! Your index pattern matches ... index," click the `> Next step` button:

You will next be prompted to select a time filter. In the dropdown, you will find @timestamp available as a choice, so
select it:

Click `Create index pattern`:

Once it finishes, you should see an index with all the fields.

In the left menu bar, click `Discover` again:

You should now see that Kibana has data in it!

Click `Add a filter +`

Select the following:

- tag
- is
- oss.project

You should now see your GRPC service's logs in Kibana!

<br>

Congratulations, you have completed the tutorial step!

<br>

_Copyright (c) 2019 RX-M LLC, Cloud Native Consulting, all rights reserved_

[RX-M LLC]: http://rx-m.io/rxm-cnc.svg "RX-M LLC"
