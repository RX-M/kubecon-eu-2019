![rx-m LLC][RX-M LLC]

# Step 7. Track It!

### 1. Adding Logging to the service

### 2. Adding the Helm Repositories

The Helm charts you'll be using to deploy the EFK stack are found on separate GitHub repos, so you'll need to add those
repos before.

Add the Elastic Helm Chart repository to your Helm:

```
$ helm repo add elastic https://helm.elastic.co


```

The Fluentd Helm chart on the official Helm repo has been depricated and moved to the maintainer's own Helm Repository
at Kiwigrid.

Now, add the Kiwigrid Helm Chart repository to your Helm:

```
$ helm repo add kiwigrid https://kiwigrid.github.io


```

### 3. Preparing your host

The Elasticsearch helm chart you'll be using requires a persistent volume.

```
ubuntu@labsys:~$ mkdir /tmp/efk/
```

Create the following persistent volume spec:

```
ubuntu@labsys:~$ vim kubecon-eu-2019/step07/efk-pv.yaml

ubuntu@labsys:~$ cat kubecon-eu-2019/step07/efk-pv.yaml

kind: PersistentVolume
apiVersion: v1
metadata:
  name: <YOUR NAME>-efk-pv-volume
  labels:
    type: local
spec:
  storageClassName: standard
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/tmp/efk"

ubuntu@labsys:~$
```

Then apply it:

```
ubuntu@labsys:~$ kubectl apply -f kubecon-eu-2019/step07/efk-pv.yaml

persistentvolume/efk-pv-volume created

ubuntu@labsys:~$
```

Check that the Persistent volume is created.

```
ubuntu@labsys:~$ kubectl get pv
NAME            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
<YOUR NAME>-efk-pv-volume   1Gi        RWO            Retain           Available           standard                18s

ubuntu@labsys:~$
```



### 4. Deploying the Helm Charts

You're now ready to deploy the helm charts:

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

### 5. Checking it out

Navigate to Kibana

Go to Discover

You should be prompted to create an index pattern. There should be an index starting with `logstash` already present. Type "logstash" into the Index Pattern text box.

If Kibana reports "Success! Your index pattern matches ... index" Then click the `> Next step` button.

You will next be prompted to select a time filter. In the dropdown, you will find @timestamp available as a choice, so select it.

Click `Create index pattern`

Once it finishes, you should see an index with all the fields.

In the left menu bar, click `Discover` again.

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
