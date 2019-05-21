![rx-m LLC][RX-M LLC]


# A Day in the life of a cloud native developer


## Step 04 - Kubernetes

Kubernetes is an open-source container-orchestration system for automating application deployment, scaling, and
management. It was originally designed by Google, and is now maintained by the Cloud Native Computing Foundation.

Amazon Elastic Container Service for Kubernetes (Amazon EKS) makes it easy to deploy, manage, and scale containerized
applications using Kubernetes on AWS. Amazon EKS runs the Kubernetes management infrastructure for you across multiple
AWS availability zones to eliminate a single point of failure. Amazon EKS is certified Kubernetes conformant so you can
use existing tooling and plugins from partners and the Kubernetes community. Applications running on any standard
Kubernetes environment are fully compatible and can be easily migrated to Amazon EKS.

In this step we're going to deploy our ossproject app to a hosted EKS Kubernetes cluster.


### 1. Install AWS support

Cloud providers all have their custom command line tools used for gaining secure access to resources in their respective
clouds. To use our EKS cluster we need to install the AWS CLI, a Python app. Install pip3 the Python 3 package manager:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ sudo apt-get -y install python3-pip

Reading package lists... Done
Building dependency tree
Reading state information... Done
The following additional packages will be installed:
  binutils build-essential cpp cpp-5 dpkg-dev fakeroot g++ g++-5 gcc gcc-5 libalgorithm-diff-perl libalgorithm-diff-xs-perl libalgorithm-merge-perl libasan2
  libatomic1 libc-dev-bin libc6 libc6-dev libcc1-0 libcilkrts5 libdpkg-perl libexpat1-dev libfakeroot libfile-fcntllock-perl libgcc-5-dev libgomp1 libisl15
  libitm1 liblsan0 libmpc3 libmpx0 libpython3-dev libpython3.5-dev libquadmath0 libstdc++-5-dev libtsan0 libubsan0 linux-libc-dev make manpages-dev
  python-pip-whl python3-dev python3-setuptools python3-wheel python3.5-dev

...

Setting up python3.5-dev (3.5.2-2ubuntu0~16.04.5) ...
Setting up python3-dev (3.5.1-3) ...
Setting up python3-pip (8.1.1-2ubuntu0.4) ...
Setting up python3-setuptools (20.7.0-1) ...
Setting up python3-wheel (0.29.0-1) ...
Processing triggers for libc-bin (2.23-0ubuntu10) ...

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

Now install the aws cli:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ pip3 install awscli --upgrade --user

Collecting awscli
  Downloading https://files.pythonhosted.org/packages/04/4f/0423b693415c5693ce4f1dcdbeffc7d02a6270e156d641b8a95176252eb8/awscli-1.16.162-py2.py3-none-any.whl (1.6MB)
    100% |████████████████████████████████| 1.6MB 986kB/s
Collecting rsa<=3.5.0,>=3.1.2 (from awscli)
  Downloading https://files.pythonhosted.org/packages/e1/ae/baedc9cb175552e95f3395c43055a6a5e125ae4d48a1d7a924baca83e92e/rsa-3.4.2-py2.py3-none-any.whl (46kB)
    100% |████████████████████████████████| 51kB 12.2MB/s
Collecting PyYAML<=3.13,>=3.10 (from awscli)
  Downloading https://files.pythonhosted.org/packages/9e/a3/1d13970c3f36777c583f136c136f804d70f500168edc1edea6daa7200769/PyYAML-3.13.tar.gz (270kB)
    100% |████████████████████████████████| 276kB 5.2MB/s
Collecting colorama<=0.3.9,>=0.2.5 (from awscli)
  Downloading https://files.pythonhosted.org/packages/db/c8/7dcf9dbcb22429512708fe3a547f8b6101c0d02137acbd892505aee57adf/colorama-0.3.9-py2.py3-none-any.whl
Collecting docutils>=0.10 (from awscli)
  Downloading https://files.pythonhosted.org/packages/36/fa/08e9e6e0e3cbd1d362c3bbee8d01d0aedb2155c4ac112b19ef3cae8eed8d/docutils-0.14-py3-none-any.whl (543kB)
    100% |████████████████████████████████| 552kB 2.7MB/s
Collecting botocore==1.12.152 (from awscli)
  Downloading https://files.pythonhosted.org/packages/06/ab/22ef182a80d085a1af91980881884c83ab2b5b98ebd578ed47f4efb23367/botocore-1.12.152-py2.py3-none-any.whl (5.4MB)
    100% |████████████████████████████████| 5.4MB 284kB/s
Collecting s3transfer<0.3.0,>=0.2.0 (from awscli)
  Downloading https://files.pythonhosted.org/packages/d7/de/5737f602e22073ecbded7a0c590707085e154e32b68d86545dcc31004c02/s3transfer-0.2.0-py2.py3-none-any.whl (69kB)
    100% |████████████████████████████████| 71kB 12.2MB/s
Collecting pyasn1>=0.1.3 (from rsa<=3.5.0,>=3.1.2->awscli)
  Downloading https://files.pythonhosted.org/packages/7b/7c/c9386b82a25115cccf1903441bba3cbadcfae7b678a20167347fa8ded34c/pyasn1-0.4.5-py2.py3-none-any.whl (73kB)
    100% |████████████████████████████████| 81kB 12.5MB/s
Collecting urllib3<1.25,>=1.20; python_version >= "3.4" (from botocore==1.12.152->awscli)
  Downloading https://files.pythonhosted.org/packages/01/11/525b02e4acc0c747de8b6ccdab376331597c569c42ea66ab0a1dbd36eca2/urllib3-1.24.3-py2.py3-none-any.whl (118kB)
    100% |████████████████████████████████| 122kB 10.8MB/s
Collecting jmespath<1.0.0,>=0.7.1 (from botocore==1.12.152->awscli)
  Downloading https://files.pythonhosted.org/packages/83/94/7179c3832a6d45b266ddb2aac329e101367fbdb11f425f13771d27f225bb/jmespath-0.9.4-py2.py3-none-any.whl
Collecting python-dateutil<3.0.0,>=2.1; python_version >= "2.7" (from botocore==1.12.152->awscli)
  Downloading https://files.pythonhosted.org/packages/41/17/c62faccbfbd163c7f57f3844689e3a78bae1f403648a6afb1d0866d87fbb/python_dateutil-2.8.0-py2.py3-none-any.whl (226kB)
    100% |████████████████████████████████| 235kB 6.0MB/s
Collecting six>=1.5 (from python-dateutil<3.0.0,>=2.1; python_version >= "2.7"->botocore==1.12.152->awscli)
  Downloading https://files.pythonhosted.org/packages/73/fb/00a976f728d0d1fecfe898238ce23f502a721c0ac0ecfedb80e0d88c64e9/six-1.12.0-py2.py3-none-any.whl
Building wheels for collected packages: PyYAML
  Running setup.py bdist_wheel for PyYAML ... done
  Stored in directory: /home/ubuntu/.cache/pip/wheels/ad/da/0c/74eb680767247273e2cf2723482cb9c924fe70af57c334513f
Successfully built PyYAML
Installing collected packages: pyasn1, rsa, PyYAML, colorama, docutils, urllib3, jmespath, six, python-dateutil, botocore, s3transfer, awscli
Successfully installed PyYAML-3.11 awscli botocore colorama docutils jmespath pyasn1-0.1.9 python-dateutil rsa s3transfer six-1.10.0 urllib3-1.13.1
You are using pip version 8.1.1, however version 19.1.1 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

Before we can use the aws cli we need to setup auth keys and zone defaults. Download the creds here:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ wget https://s3.eu-central-1.amazonaws.com/rx-m-kubecon-barcelona/kubecon-barcelona

--2019-05-21 02:42:14--  https://s3.eu-central-1.amazonaws.com/rx-m-kubecon-barcelona/kubecon-barcelona
Resolving s3.eu-central-1.amazonaws.com (s3.eu-central-1.amazonaws.com)... 52.219.72.8
Connecting to s3.eu-central-1.amazonaws.com (s3.eu-central-1.amazonaws.com)|52.219.72.8|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 116 [binary/octet-stream]
Saving to: ‘kubecon-barcelona’

kubecon-barcelona                       100%[==============================================================================>]     116  --.-KB/s    in 0s

2019-05-21 02:42:14 (5.88 MB/s) - ‘kubecon-barcelona’ saved [116/116]

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ cat kubecon-barcelona

[default]
aws_access_key_id = XXXXXXXXXXXXXXXX
aws_secret_access_key = XXXXXXXXXXXXXXXX

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

Now run aws configure and use the creds from the file downloaded:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ aws configure

AWS Access Key ID [None]: XXXXXXXXXXXXXXXX
AWS Secret Access Key [None]: XXXXXXXXXXXXXXXXXXxx
Default region name [None]: eu-central-1
Default output format [None]: json

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

To test the setup try checking your identity with the AWS Security Token service (STS):

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ aws sts get-caller-identity

{
    "UserId": "XXXXXXXXXXXXXXX",
    "Arn": "arn:aws:iam::433017611331:user/kubecon-barcelona-eks",
    "Account": "433017611331"
}

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

With AWS configured we can setup our Kubernetes client, kubectl!


### 2. Install and configure kubectl

Google hosts a package repo with DEB packages for all of the key Kubernetes components. Install the cloud package repo
key:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

OK

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

Now add the google cloud repo to the apt repo lists:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" \
> | sudo tee -a /etc/apt/sources.list.d/kubernetes.list

deb http://apt.kubernetes.io/ kubernetes-xenial main

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

Update the local package indexes:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ sudo apt-get update

Hit:1 http://eu-central-1.ec2.archive.ubuntu.com/ubuntu xenial InRelease
Get:2 http://eu-central-1.ec2.archive.ubuntu.com/ubuntu xenial-updates InRelease [109 kB]
Get:3 http://eu-central-1.ec2.archive.ubuntu.com/ubuntu xenial-backports InRelease [107 kB]
Get:4 http://security.ubuntu.com/ubuntu xenial-security InRelease [109 kB]
Hit:5 https://download.docker.com/linux/ubuntu xenial InRelease
Get:6 http://eu-central-1.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 Packages [957 kB]
Get:7 https://packages.cloud.google.com/apt kubernetes-xenial InRelease [8,993 B]
Get:8 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 Packages [26.0 kB]
Fetched 1,317 kB in 0s (1,810 kB/s)
Reading package lists... Done

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

Our EKS cluster is running K8s 1.12 so we'll install the same kubectl version:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ sudo apt-get install -y kubectl=1.12.6-00

Reading package lists... Done
Building dependency tree
Reading state information... Done
The following NEW packages will be installed:
  kubectl
0 upgraded, 1 newly installed, 0 to remove and 80 not upgraded.
Need to get 9,589 kB of archives.
After this operation, 57.4 MB of additional disk space will be used.
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubectl amd64 1.12.6-00 [9,589 kB]
Fetched 9,589 kB in 2s (4,227 kB/s)
Selecting previously unselected package kubectl.
(Reading database ... 57068 files and directories currently installed.)
Preparing to unpack .../kubectl_1.12.6-00_amd64.deb ...
Unpacking kubectl (1.12.6-00) ...
Setting up kubectl (1.12.6-00) ...

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

Now as a final step we can use the AWS CLI to generate a .kube/config file with creds that will point kubectl at the EKS
cluster we will deploy to:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ aws eks --region eu-central-1 update-kubeconfig --name eks-cloud-native-dev

Added new context arn:aws:eks:eu-central-1:433017611331:cluster/eks-cloud-native-dev to /home/ubuntu/.kube/config

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

Looks good! Let's check the cluster:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ kubectl cluster-info

Kubernetes master is running at https://05E2CBDCDB969EFF27C916E1660F7AF8.yl4.eu-central-1.eks.amazonaws.com
CoreDNS is running at https://05E2CBDCDB969EFF27C916E1660F7AF8.yl4.eu-central-1.eks.amazonaws.com/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

List the nodes available:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ kubectl get nodes

NAME                                               STATUS   ROLES    AGE     VERSION
ip-192-168-104-49.eu-central-1.compute.internal    Ready    <none>   5h18m   v1.12.7
ip-192-168-117-8.eu-central-1.compute.internal     Ready    <none>   5h18m   v1.12.7
ip-192-168-141-101.eu-central-1.compute.internal   Ready    <none>   172m    v1.12.7
ip-192-168-142-13.eu-central-1.compute.internal    Ready    <none>   172m    v1.12.7
ip-192-168-158-118.eu-central-1.compute.internal   Ready    <none>   5h18m   v1.12.7
ip-192-168-170-113.eu-central-1.compute.internal   Ready    <none>   172m    v1.12.7
ip-192-168-198-68.eu-central-1.compute.internal    Ready    <none>   5h18m   v1.12.7
ip-192-168-244-176.eu-central-1.compute.internal   Ready    <none>   5h18m   v1.12.7
ip-192-168-249-81.eu-central-1.compute.internal    Ready    <none>   172m    v1.12.7
ip-192-168-99-172.eu-central-1.compute.internal    Ready    <none>   172m    v1.12.7

...

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

Connectivity!


### 3. Setup a namespace

To keep your services separate from others (other users or other teams) K8s provides namespaces, a means of multitenancy
in containerized environments. Create a namespace using your initials and your lab system IP address, something like
this:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ kubectl create namespace wra-172-31-30-5

namespace/wra-172-31-30-5 created

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

Now change contexts to that namespace so that all of your work happens there:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ kubectl config set-context --current --namespace=wra-172-31-30-5

Context "arn:aws:eks:eu-central-1:433017611331:cluster/eks-cloud-native-dev" modified.

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ kubectl config current-context

arn:aws:eks:eu-central-1:433017611331:cluster/eks-cloud-native-dev

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ kubectl config current-context

arn:aws:eks:eu-central-1:433017611331:cluster/eks-cloud-native-dev
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://05E2CBDCDB969EFF27C916E1660F7AF8.yl4.eu-central-1.eks.amazonaws.com
  name: arn:aws:eks:eu-central-1:433017611331:cluster/eks-cloud-native-dev
contexts:
- context:
    cluster: arn:aws:eks:eu-central-1:433017611331:cluster/eks-cloud-native-dev
    namespace: wra-172-31-30-5
    user: arn:aws:eks:eu-central-1:433017611331:cluster/eks-cloud-native-dev
  name: arn:aws:eks:eu-central-1:433017611331:cluster/eks-cloud-native-dev
current-context: arn:aws:eks:eu-central-1:433017611331:cluster/eks-cloud-native-dev
kind: Config
preferences: {}
users:
- name: arn:aws:eks:eu-central-1:433017611331:cluster/eks-cloud-native-dev
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1alpha1
      args:
      - --region
      - eu-central-1
      - eks
      - get-token
      - --cluster-name
      - eks-cloud-native-dev
      command: aws
      env: null

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ kubectl get pod

No resources found.

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

Great, we have an empty namespace ready to use. Now to deploy our service.


### 4. Deploy your containerized service

Now that we have access and a namespace we can create a K8s deployment to startup some pods to run our service in. K8s
pods package small sets of containers (one often enough) together to be deployed as a unit.

Here's what our k8s deployment will look like:

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: ossp
  name: ossp
spec:
  replicas: 2
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
```

In K8s, deployments own a time series of replicasets, each replicaset owns a set of pods with the same creation
template. Updating the deployment creates a new replicaset and the deployment (by default) performs a rolling upgrade to
the new version. If problems occur you can rollback to the previous replicaset.

Create the deployment file above and deploy it with kubectl:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ vim ossp-dep.yaml

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ cat ossp-dep.yaml

apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: ossp
  name: ossp
spec:
  replicas: 2
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

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ kubectl create -f ossp-dep.yaml

deployment.extensions/ossp created

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

Check the status of the deployment and the initial replicaset it created along with the 2 pods created by the
replicaset:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ kubectl get deploy,rs,po

NAME                         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/ossp   2         2         2            2           70s

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.extensions/ossp-67469746c   2         2         2       70s

NAME                       READY   STATUS    RESTARTS   AGE
pod/ossp-67469746c-bjkb7   1/1     Running   0          70s
pod/ossp-67469746c-dm4lg   1/1     Running   0          70s

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

As you can imagine, in a production setting all of the things we have created thus far would be kept in source code
control. Each change we made to the Go code of the service, the Dockerfile packaging code, or the K8s deployment code
would trigger a CI event and, if successful, eventually a new deployment.


### 5. Create a service to provide access to the pods

If we want to access our services at present we have to lookup the IP of one of the pods and gain access to it. Because
most K8s clusters run on a SDN network that is not directly reachable from outside the cluster we need another approach.
Fortunately we can create a service in K8s. A service is an abstraction that gives us a virtual IP address that load
balances over whichever pods are implementing it.

Because we gave our pods a label of "app: ossp" we can use that label in a selector to associate a service with just
those pods. Here's the spec for our service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ossp-service
spec:
  selector:
    app: ossp
  type: LoadBalancer
  ports:
  - protocol: TCP
    port: 50088
    targetPort: 50088
```

The service type "loadbalancer" will cause the cloud to provision a loadbalancer and a public IP for us to use.

Create the service:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ vim ossp-svc.yaml

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ cat ossp-svc.yaml

apiVersion: v1
kind: Service
metadata:
  name: ossp-service
spec:
  selector:
    app: ossp
  type: LoadBalancer
  ports:
  - protocol: TCP
    port: 50088
    targetPort: 50088

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ kubectl create -f ossp-svc.yaml

service/ossp-service created

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

Check your service:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ kubectl get service

NAME           TYPE           CLUSTER-IP      EXTERNAL-IP                                                                  PORT(S)           AGE
ossp-service   LoadBalancer   10.100.83.253   a03552f567b6c11e981790ae78cda592-1560382816.eu-central-1.elb.amazonaws.com   50088:30016/TCP   60s

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ kubectl describe service ossp-service

Name:                     ossp-service
Namespace:                wra-172-31-30-5
Labels:                   <none>
Annotations:              <none>
Selector:                 app=ossp
Type:                     LoadBalancer
IP:                       10.100.83.253
LoadBalancer Ingress:     a03552f567b6c11e981790ae78cda592-1560382816.eu-central-1.elb.amazonaws.com
Port:                     <unset>  50088/TCP
TargetPort:               50088/TCP
NodePort:                 <unset>  30016/TCP
Endpoints:                192.168.199.57:50088,192.168.70.73:50088
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
  Type    Reason                Age   From                Message
  ----    ------                ----  ----                -------
  Normal  EnsuringLoadBalancer  103s  service-controller  Ensuring load balancer
  Normal  EnsuredLoadBalancer   101s  service-controller  Ensured load balancer

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

Great we have two pods behind our service and a loadbalancer listening on
"a03552f567b6c11e981790ae78cda592-1560382816.eu-central-1.elb.amazonaws.com"


### 6. Test the service

To test our service in the cloud, from your lab system rerun your client app with the loadbalancer host name:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ go run client.go a03552f567b6c11e981790ae78cda592-1560382816.eu-central-1.elb.amazonaws.com fluentd

2019/05/21 02:02:52 Projects: name:"fluentd" custodian:"cncf"

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

That is pretty dang cool. Now delete it all ... :-)

We can remove our service and our deployment using the delete command. By deleting the deployment we will automatically
remove the replicaset and pods (trying to delete the pods or replicasets is futile because the deployment will just
recreate them).

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ kubectl delete deploy ossp

deployment.extensions "ossp" deleted

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ kubectl delete service ossp-service

service "ossp-service" deleted

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

Our app runs in the cloud but we have to maintain various resources to roll it out. Wouldn't it be nice if we
could package everything up and deploy the entire app in one fell swoop? Helm to the rescue:
[../step05/README.md](../step05/README.md)


<br>

Congratulations, you have completed the tutorial step!

<br>

_Copyright (c) 2019 RX-M LLC, Cloud Native Consulting, all rights reserved_

[RX-M LLC]: http://rx-m.io/rxm-cnc.svg "RX-M LLC"
