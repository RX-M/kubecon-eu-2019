![rx-m LLC][RX-M LLC]


# A Day in the life of a cloud native developer


## Step 05 - Helm

Helm is a tool that streamlines installing and managing Kubernetes applications. Think of it like apt/yum/homebrew for
Kubernetes. Tiller is an optional component of helm that runs inside of your Kubernetes cluster, and manages releases
(installations) of your charts.

In this step we're going to deploy our ossproject app using helm.


### 1. Install Helm

There are two parts to Helm: The Helm client (helm) and the Helm server (Tiller), the latter of which is not required.
The Helm client can be downloaded from the releases page of the Helm repo: (https://github.com/helm/helm/releases) but
the repo also holds an installer script that will automatically grab the latest version of the Helm client and install
it locally.

Install helm:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ curl https://raw.githubusercontent.com/helm/helm/master/scripts/get > get_helm.sh

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  7028  100  7028    0     0  63846      0 --:--:-- --:--:-- --:--:-- 63890

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

Make it executable and run it:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ chmod 700 get_helm.sh

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ ./get_helm.sh

Downloading https://kubernetes-helm.storage.googleapis.com/helm-v2.14.0-linux-amd64.tar.gz
Preparing to install helm and tiller into /usr/local/bin
helm installed into /usr/local/bin/helm
tiller installed into /usr/local/bin/tiller
Run 'helm init' to configure helm.

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

**DO NOT** follow the final instruction to configure helm, which will launch the Tiller server as a pod on our cluster,
which we _will not_ be using.

We can use the `version` command to make sure the client is working:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ helm version -c

Client: &version.Version{SemVer:"v2.14.0", GitCommit:"05811b84a3f93603dd6c2fcfe57944dfa7ab7fd0", GitTreeState:"clean"}

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

This command will hang without specifying the client only flag (`-c`) (it will try to reach Tiller).


### 2. Create a helm chart

Now we'll create a simple helm chart that deploys our entire application (deployment and service). Helm can generate a
chart template for us to work from:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ helm create ossp

Creating ossp

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

Take a look at the directory structure of the chart generated:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ sudo apt install tree

Reading package lists... Done
Building dependency tree
Reading state information... Done
The following NEW packages will be installed:
  tree
0 upgraded, 1 newly installed, 0 to remove and 81 not upgraded.
Need to get 40.6 kB of archives.
After this operation, 138 kB of additional disk space will be used.
Get:1 http://eu-central-1.ec2.archive.ubuntu.com/ubuntu xenial/universe amd64 tree amd64 1.7.0-3 [40.6 kB]
Fetched 40.6 kB in 0s (0 B/s)
Selecting previously unselected package tree.
(Reading database ... 57069 files and directories currently installed.)
Preparing to unpack .../tree_1.7.0-3_amd64.deb ...
Unpacking tree (1.7.0-3) ...
Processing triggers for man-db (2.7.5-1) ...
Setting up tree (1.7.0-3) ...

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ tree ossp

ossp
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml

3 directories, 8 files

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```


### 3. Create a service template

The templates/ directory houses YAML definitions for Services, Deployments and other resources you might need. We can delete the definitions we do not need and overwrite the others with our configs from the last step.

Take a look at the service template:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ cat ossp/templates/service.yaml

apiVersion: v1
kind: Service
metadata:
  name: {{ include "ossp.fullname" . }}
  labels:
{{ include "ossp.labels" . | indent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app.kubernetes.io/name: {{ include "ossp.name" . }}
    app.kubernetes.io/instance: {{ .Release.Name }}

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

As you can see helm augments the basic K8s config syntax with parameters in the form of double curly brace enclosed
variable names. To try out variables we'll leave the `name: {{ include "ossp.fullname" . }}` in place but replace everything else with our service definition from the last step.

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ vim ossp/templates/service.yaml

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ cat ossp/templates/service.yaml

apiVersion: v1
kind: Service
metadata:
  name: {{ include "ossp.fullname" . }}
spec:
  selector:
    app: ossp
  type: LoadBalancer
  ports:
  - protocol: TCP
    port: 50088
    targetPort: 50088

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```


### 4. Create a deployment template

Next create a deployment template with variables for the name and the replica count:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ vim ossp/templates/deployment.yaml

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ cat ossp/templates/deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "ossp.fullname" . }}
  labels:
    app: ossp
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: ossp
  template:
    metadata:
      labels:
        app: ossp
    spec:
      containers:
      - image: reg.rx-m.net/cndev/wra.ossp:latest
        imagePullPolicy: Always
        name: ossp
        ports:
        - containerPort: 50088
          protocol: TCP

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

The `ossp.` variables come from the chart project itself but the `.Values.` variables come from the values file.


### 5. Setup the values file

Remove everything from the values file except the replica count:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ cat ossp/values.yaml

# Default values for ossp.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

image:
  repository: nginx
  tag: stable
  pullPolicy: IfNotPresent

imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: false
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: chart-example.local
      paths: []

  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local

resources: {}
  # We usually recommend not to specify default resources and to leave this as a conscious
  # choice for the user. This also increases chances charts run on environments with little
  # resources, such as Minikube. If you do want to specify resources, uncomment the following
  # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
  # limits:
  #   cpu: 100m
  #   memory: 128Mi
  # requests:
  #   cpu: 100m
  #   memory: 128Mi

nodeSelector: {}

tolerations: []

affinity: {}

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ vim ossp/values.yaml

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ cat ossp/values.yaml

replicaCount: 1

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```


### 6. Clean up unused templates

Now remove all of the template directory files you do not need:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ rm ossp/templates/ingress.yaml

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ rm -rf ossp/templates/tests

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ rm ossp/templates/NOTES.txt

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

Now try running the template engine on the chart:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ helm template ./ossp

---
# Source: ossp/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: release-name-ossp
spec:
  selector:
    app: ossp
  type: LoadBalancer
  ports:
  - protocol: TCP
    port: 50088
    targetPort: 50088

---
# Source: ossp/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: release-name-ossp
  labels:
    app: ossp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ossp
  template:
    metadata:
      labels:
        app: ossp
    spec:
      containers:
      - image: reg.rx-m.net/cndev/wra.ossp:latest
        imagePullPolicy: Always
        name: ossp
        ports:
        - containerPort: 50088
          protocol: TCP

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

Pretty slick, a nicely generated set of K8s resource configs.


### 7. Create the template resources

Now we can use the template engine to emit resource configs directly to kubectl:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ helm template ./ossp | kubectl create -f -

service/release-name-ossp created
deployment.apps/release-name-ossp created

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ kubectl get all

NAME                                    READY   STATUS    RESTARTS   AGE
pod/release-name-ossp-67469746c-4dbjh   1/1     Running   0          2m38s

NAME                        TYPE           CLUSTER-IP       EXTERNAL-IP                                                                 PORT(S)           AGE
service/release-name-ossp   LoadBalancer   10.100.221.157   a446d6c137b7611e98f95020deaa3a09-659563283.eu-central-1.elb.amazonaws.com   50088:30613/TCP   2m38s

NAME                                DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/release-name-ossp   1         1         1            1           2m38s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/release-name-ossp-67469746c   1         1         1       2m38s

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

Voila, a complete application up and running with configurable parameters for use by different teams or different
regions and clouds.

Our app is really getting easy to deploy, however we are falling down on the observability front, we need to be able to
monitor, track and trace our applications activity: [../step06/README.md](../step06/README.md)


<br>

Congratulations, you have completed the tutorial step!

<br>

_Copyright (c) 2019 RX-M LLC, Cloud Native Consulting, all rights reserved_

[RX-M LLC]: http://rx-m.io/rxm-cnc.svg "RX-M LLC"
