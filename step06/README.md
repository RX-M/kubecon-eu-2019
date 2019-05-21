![rx-m LLC][RX-M LLC]


# A Day in the life of a cloud native developer


## Step 06 - Prometheus and Grafana

Prometheus is an open source monitoring and alerting application, often employed to provide application- and node-level
metrics monitoring to Cloud Native application deployments. When paired with Grafana, a powerful visualization tool that
meshes well with Promtheus, users are given deep insight on the health of their applications.

Prometheus works by continually collecting, or scraping, open metrics endpoints made available by applications, looking
at any exposed endpoint at /metrics to collect any pertinent data. Data can then be sorted inside the Prometheu GUI,
where the user can design queries that can aggregate more useful metrics like averages, sums and even histograms.


### 1. Render the Helm Charts

In the previous step, you will once again render the contents of a Helm chart. Be sure to have

```
ubuntu@labsys:~/kubecon-eu-2019/step06$ helm template ~/kubecon-eu-2019/step06/prometheus/.  \
-f ~/kubecon-eu-2019/step06/prometheus/values.yaml \
--set metadata.namespace=$INITIALS --output-dir ~/kubecon-eu-2019/step06/ --name $INITIALS-metrics

wrote /home/ubuntu/kubecon-eu-2019/step06//prometheus-demo/templates/grafana-secret.yaml
wrote /home/ubuntu/kubecon-eu-2019/step06//prometheus-demo/templates/grafana-dashboard-provider-configMap.yaml
wrote /home/ubuntu/kubecon-eu-2019/step06//prometheus-demo/templates/grafana-datasource-configMap.yaml
wrote /home/ubuntu/kubecon-eu-2019/step06//prometheus-demo/templates/grafana-kubernetes-pod-overview-dashboard-configMap.yaml
wrote /home/ubuntu/kubecon-eu-2019/step06//prometheus-demo/templates/prometheus-configMap.yaml
wrote /home/ubuntu/kubecon-eu-2019/step06//prometheus-demo/templates/prometheus-jmx-exporter-configMap.yaml
wrote /home/ubuntu/kubecon-eu-2019/step06//prometheus-demo/templates/prometheus-serviceAccount.yaml
wrote /home/ubuntu/kubecon-eu-2019/step06//prometheus-demo/templates/prometheus-role.yaml
wrote /home/ubuntu/kubecon-eu-2019/step06//prometheus-demo/templates/prometheus-roleBinding.yaml
wrote /home/ubuntu/kubecon-eu-2019/step06//prometheus-demo/templates/prometheus-service.yaml
wrote /home/ubuntu/kubecon-eu-2019/step06//prometheus-demo/templates/prometheus-deployment.yaml

ubuntu@labsys:~/kubecon-eu-2019/step06$
```

You render the helm chart with the following options:

  - `-f ~/kubecon-eu-2019/step06/prometheus/values.yaml` - uses the values.yaml file found inside the Prometheus Helm chart directory
  - `--set metadata.namespace="$INITIALS"`- Tells Helm to render each manifest with your namespace
  - `--output-dir ~/kubecon-eu-2019/step06/` - Tells Helm to output all manifests inside this directory
  - `--name $INITIALS-metrics` - Names the metrics deployment with your initials suffixed by "-metrics"


### 2. Deploy your Metrics

The previous step creates several static manifests using that you can apply to your Kubernetes cluster using `kubectl
apply -f`.

```
ubuntu@labsys:~/kubecon-eu-2019/step06$ kubectl apply -f ~/kubecon-eu-2019/step06/prometheus-demo/templates/.

configmap/cal-metrics-grafana-dashboard-provider created
configmap/cal-metrics-grafana-datasource created
configmap/cal-metrics-kubernetes-pod-overview-dashboard created
secret/cal-metrics-grafana created
configmap/cal-metrics-prometheus-config created
deployment.apps/cal-metrics-prometheus created
configmap/cal-metrics-prometheus-jmx-exporter created
role.rbac.authorization.k8s.io/cal-metrics-prometheus created
rolebinding.rbac.authorization.k8s.io/cal-metrics-prometheus created
service/cal-metrics-prometheus created
serviceaccount/cal-metrics-prometheus created

ubuntu@labsys:~/kubecon-eu-2019/step06$
```

Once deployed, ensure that the Helm chart elements were deployed successfully using `kubectl get all`:

```
ubuntu@labsys:~/kubecon-eu-2019/step06$ kubectl get all -n cal -l release=$INITIALS-metrics

NAME                                          READY   STATUS    RESTARTS   AGE
pod/cal-metrics-prometheus-7cdff57674-kxrsl   4/4     Running   0          14m

NAME                             TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)                         AGE
service/cal-metrics-prometheus   NodePort   10.100.43.188   <none>        9090:31116/TCP,3000:30199/TCP   14m

NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cal-metrics-prometheus   1/1     1            1           14m

NAME                                                DESIRED   CURRENT   READY   AGE
replicaset.apps/cal-metrics-prometheus-7cdff57674   1         1         1       14m

ubuntu@labsys:~/kubecon-eu-2019/step06$
```

By using the `-l release=$INITIALS-prometheus` flag, you can filter your results so they only show for the most recently
deployed Helm chart.

Wait a moment while the metrics resources come up and online.


### 3. Accessing Your Metrics

Take a look at the service deployed with your Prometheus Helm chart. It presents four ports numbers in two mappings.
Port 9090 in this example is mapped to 31116, while Port 3000 is mapped to 30199,

```
ubuntu@labsys:~/kubecon-eu-2019/step06$ kubectl get svc -n $INITIALS

NAME                     TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)                         AGE
cal-metrics-prometheus   NodePort   10.100.43.188   <none>        9090:31116/TCP,3000:30199/TCP   22m

ubuntu@labsys:~/kubecon-eu-2019/step06$
```

To facilitate access to your metrics, the Prometheus service was set to open a NodePort. This means that you can access
your metrics by entering the external IP address or fully qualified domain name combined with one of the external ports.
Accessing a worker node on port 31116 in this example will take you to the Prometheus GUI, while accessing port 30199
takes you to Grafana.

In a browser, enter one of the IP addresses or FQDNs of the Kubernetes worker machines:

![Grafana Login Screen](./images/grafana.PNG)

Login to Grafana. The default credentials, which you can set in the Helm chart's Values.yaml file, are as follows:

  - Username: admin
  - Password: kubecon2019eu

![Grafana Dashboard ](./images/grafana-ui.PNG)

Once logged into Grafana, click "Home" to open the Dashboard search. You should see a pre-populated dashboard,
**Kubernetes: POD Overview**:

![Grafana Selecting Dashboard ](./images/grafana-dashboard.PNG)

Select **Kubernetes: POD Overview**:

![Grafana Pod Status](./images/grafana-pod-status.PNG)

You will be taken to a very simple dashboard that reports the amount of Pods currently reporting Up status to the
cluster's master Prometheus instance.


<br>

Congratulations, you have completed the tutorial step!

<br>

_Copyright (c) 2019 RX-M LLC, Cloud Native Consulting, all rights reserved_

[RX-M LLC]: http://rx-m.io/rxm-cnc.svg "RX-M LLC"
