![rx-m LLC][RX-M LLC]


# kubecon-eu-2019


## A Day in the life of a cloud native developer


This guide will detail the steps needed to deploy the EKS environment used in the course of this demo.


### 1. EKS Setup

#### 1a. Launch the cluster

The AWS CLI is a unified tool to manage AWS services. With just one tool to download and configure, you can control AWS
services from the command line and automate them with scripts. Other tooling we will use throughout our EKS labs will
also leverage the AWS CLI or its configs. The AWS CLI is a Python application and can be installed with the Python
package manager. Open a terminal in your lab system and update the system package indexes:

```
ubuntu@labsys:~$ sudo apt-get update

Hit:1 http://security.ubuntu.com/ubuntu xenial-security InRelease
Hit:2 http://us.archive.ubuntu.com/ubuntu xenial InRelease
Hit:3 http://us.archive.ubuntu.com/ubuntu xenial-updates InRelease
Hit:4 http://us.archive.ubuntu.com/ubuntu xenial-backports InRelease
Reading package lists... Done

ubuntu@labsys:~$
```

Now install the Python pip package manager for Python 3:

```
ubuntu@labsys:~$ sudo apt-get -y install python3-pip

Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  build-essential cpp-5 g++ g++-5 gcc-5 gcc-5-base libasan2 libatomic1 libcc1-0 libcilkrts5 libexpat1 libexpat1-dev
  libgcc-5-dev libgomp1 libitm1 liblsan0 libmpx0 libpython3-dev libpython3.5 libpython3.5-dev libpython3.5-minimal
  libpython3.5-stdlib libquadmath0 libstdc++-5-dev libstdc++6 libtsan0 libubsan0 python-pip-whl python3-dev
  python3-setuptools python3-wheel python3.5 python3.5-dev python3.5-minimal
Suggested packages:
  gcc-5-locales g++-multilib g++-5-multilib gcc-5-doc libstdc++6-5-dbg gcc-5-multilib libgcc1-dbg libgomp1-dbg
  libitm1-dbg libatomic1-dbg libasan2-dbg liblsan0-dbg libtsan0-dbg libubsan0-dbg libcilkrts5-dbg libmpx0-dbg
  libquadmath0-dbg libstdc++-5-doc python-setuptools-doc python3.5-venv python3.5-doc binfmt-support
The following NEW packages will be installed:
  build-essential g++ g++-5 libexpat1-dev libpython3-dev libpython3.5-dev libstdc++-5-dev python-pip-whl
  python3-dev python3-pip python3-setuptools python3-wheel python3.5-dev
The following packages will be upgraded:
  cpp-5 gcc-5 gcc-5-base libasan2 libatomic1 libcc1-0 libcilkrts5 libexpat1 libgcc-5-dev libgomp1 libitm1 liblsan0
  libmpx0 libpython3.5 libpython3.5-minimal libpython3.5-stdlib libquadmath0 libstdc++6 libtsan0 libubsan0
  python3.5 python3.5-minimal
22 upgraded, 13 newly installed, 0 to remove and 448 not upgraded.
Need to get 74.5 MB of archives.

...

ubuntu@labsys:~$
```

Finally install the AWS CLI with pip:

```
ubuntu@labsys:~$ python3 -m pip install awscli --upgrade --user

Collecting awscli
  Downloading https://files.pythonhosted.org/packages/65/c8/052a27efb2f19172fe7fc17d409e54b72eee805843cc7049e258f3eaa91c/awscli-1.16.164-py2.py3-none-any.whl (1.6MB)
    100% |████████████████████████████████| 1.6MB 881kB/s
...
Installing collected packages: pyasn1, rsa, six, python-dateutil, jmespath, urllib3, docutils, botocore, s3transfer, PyYAML, colorama, awscli
Successfully installed PyYAML awscli botocore colorama docutils jmespath pyasn1 python-dateutil rsa s3transfer six-1.10.0 urllib3-1.13.1

ubuntu@labsys:~$
```

Verify the AWS CLI installation:

```
ubuntu@labsys:~$ aws --version

aws-cli/1.16.164 Python/3.5.2 Linux/4.4.0-31-generic botocore/1.12.154

ubuntu@labsys:~$
```

Our AWS CLI version is compliant (we needed the minimum version 1.16.156 for interoperability with eks tooling).

Download the latest release of eksctl from Github and move it to `/usr/local/bin`:

> The -s flag passed to the uname command simply prints the kernel name

```
ubuntu@labsys:~$ curl --silent --location \
"https://github.com/weaveworks/eksctl/releases/download/latest_release/eksctl_$(uname -s)_amd64.tar.gz" \
| tar xz -C /tmp

ubuntu@labsys:~$ sudo mv /tmp/eksctl /usr/local/bin

ubuntu@labsys:~$ eksctl version

[ℹ]  version.Info{BuiltAt:"", GitCommit:"", GitTag:"0.1.32"}

ubuntu@labsys:~$
```

We will need to have AWS API credentials configured before we can use eksctl.

eksctl will use the ~/.aws/credentials file or environment variables to talk to the AWS API. Use the `aws configure`
command to create a profile:

```
ubuntu@labsys:~$ aws configure

AWS Access Key ID [None]: XXXXXXXXXXXXXXXXXXXX
AWS Secret Access Key [None]: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
Default region name [None]: us-east-2
Default output format [None]: json

ubuntu@labsys:~$
```

Run `aws sts get-caller-identity` and validate that your Arn contains `eks-sre`:

```
ubuntu@labsys:~$ aws sts get-caller-identity

{
    "UserId": "AIDAWJUO4MRB2ETRY566R",
    "Account": "433017611331",
    "Arn": "arn:aws:iam::433017611331:user/eks-sre"
}

ubuntu@labsys:~$
```

You may have performed these steps previously, but they are here for completeness; skip this step if you have kubectl
installed.

Some apt package repos use the aptitude protocol however the kubernetes packages are served of https so we need to add
the apt https transport:

```
ubuntu@labsys:~$ sudo apt-get update && sudo apt-get install -y apt-transport-https

Hit:1 http://us.archive.ubuntu.com/ubuntu xenial InRelease
Get:2 http://security.ubuntu.com/ubuntu xenial-security InRelease [102 kB]
Hit:3 https://download.docker.com/linux/ubuntu xenial InRelease                   
Get:4 http://us.archive.ubuntu.com/ubuntu xenial-updates InRelease [102 kB]       
Get:5 http://us.archive.ubuntu.com/ubuntu xenial-backports InRelease [102 kB]                 
Get:6 http://us.archive.ubuntu.com/ubuntu xenial-updates/main amd64 Packages [699 kB]
Get:7 http://us.archive.ubuntu.com/ubuntu xenial-updates/main i386 Packages [653 kB]
Fetched 1,659 kB in 3s (423 kB/s)                        
Reading package lists... Done
Reading package lists... Done
Building dependency tree       
Reading state information... Done
apt-transport-https is already the newest version (1.2.29).
0 upgraded, 0 newly installed, 0 to remove and 256 not upgraded.

ubuntu@labsys:~$
```

`apt-transport-https` was installed earlier, but here for completeness. Now add the Google cloud packages repo key so
that we can install packages hosted by Google:

```
ubuntu@labsys:~$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

OK

ubuntu@labsys:~$
```

Next add a repository list file with an entry for Ubuntu Xenial apt.kubernetes.io packages. The following command copies
the text you enter to the "kubernetes.list" file, up to but not including the string "EOF".

```
ubuntu@labsys:~$ echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" \
| sudo tee -a /etc/apt/sources.list.d/kubernetes.list

deb http://apt.kubernetes.io/ kubernetes-xenial main

ubuntu@labsys:~$
```

Update the package indexes to add the Kubernetes packages from apt.kubernetes.io:

```
ubuntu@labsys:~$ sudo apt-get update

...       
Get:3 https://packages.cloud.google.com/apt kubernetes-xenial InRelease [8,972 B]           
Get:7 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 Packages [12.4 kB]
Fetched 226 kB in 1s (151 kB/s)     
Reading package lists... Done

ubuntu@labsys:~$
```

Notice the new packages.cloud.google.com repository above. If you _do not_ see it in your terminal output, you must fix
the entry in `/etc/apt/sources.list.d/kubernetes.list` before moving on!

Check the EKS platform versions available, and go with the latest. They're behind a couple of versions.

https://docs.aws.amazon.com/eks/latest/userguide/platform-versions.html

Use the aptitude package manager to install the desired packages; EKS is on version 1.12, so install a version of
kubectl to match:

```
ubuntu@labsys:~$ sudo apt-get install -y kubectl=1.12.8-00 jq

Reading package lists... Done

...

Setting up kubectl (1.12.8-00) ...

...

ubuntu@labsys:~$
```

Create SSH keys so that we can access the worker nodes from our local terminal:

```
ubuntu@labsys:~$ aws ec2 create-key-pair --key-name eks-key-ditl --query 'KeyMaterial' --output text \
> | tee -a eks-key-ditl.pem

-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAkOjyRr0f9KGtZvjfIEG9RSXk8Nv8MDnBZusje1LCzJ96+DkvQ5mhpz4KyEN1
hkW8Uiww1eAW1xV+8O5ISHlSO+ncjAWCyI9ZaIQ3g32ayj1nnQUJ4MAEnN4v9T0GIybTkkZ4aUi+
jHpGlu9K244HoBl+uRUFXuozmMfgMhwA+iQip5bGAZiEHRe9iFiRM91blKiUyU/02n/3Pj1eYqF/
654xGSifitf06SSV0fvVsU+tn/4U1oP6uyQ8ddglj7u5t4G1jNqu+KQkF9kKjdPl1PALANs/7DLQ
e3hJLH4GuPmQ29c16v1uGmr6u/fY9ZrDMp1rq470Xp25p37sKl/BEwIDAQABAoIBAQCESoXUGphv
xqH9XbqlQh6+X+fwE4TZqgBsKsJLtbRtBjNHJT2G41x2x+ckCKHkIQnZoso6lseDN/aZkY+fylJO
rCNSGT3aRzQCfKIJgsOrWf+bk5v++I29gAIcSsetk6aW5YrL40NCD+cdp/uZEMLZYC0WXqB2lCzi
j6aXWIVz1eW5KhuU8FJ5y7aNJsjOgK9la9hbhUto9Z7ohMhU61NnfSxntKCsWp6a8XiDFfWsdh9A
/1UenEx/KImvt2KkCajgRbNvWmaDPPUmUbhMfLiUB6ShbPHyacmG6o3k6bAaUpgfPyTOPsZNalJO
07kvin1GLb/X8BTHKbngdTmTG+PxAoGBAMCjzJr+3a73bCrJj8FmcZRMXdc3gC8Mn7fVooIc4dWX
e5pmrxbgVmZjW6mWUqKvkZncXFIZJIv94PNYZP7zBw4LGN3wSSQGbv+V5A5N6GlO0DhrQJPry65O
6iZzhKyGF4V9v1TWcW13YhxTsWudN6K+OZa29uKBLWDgZXeDHSmfAoGBAMCSTvG4xF5F9ziCq5Z4
PbRyWw+NzLJPF0yWMUX8SrmBnb3CPe/vSU/s/1whLFdFo6KwJv4/2BkoQyu1yYm+3/fRsUQrJaY8
3+7hYPq3jOus1EyToRNc3mAERApneTK/jtziKmFnXnK5xyxlbOBT+0BavQ/IFhx/KpkasZeZ4NwN
AoGAL+RJCLuWF1qRvK1xnM5ALHMz3T6CErBbwNNO3HQbvQM1CnS+0LwjHr9S2X1yu9lUJGFBXnO7
v0X1t+ng6fU3asldfEexl1A2Jjp4gQnjXtLmNzCK1HuJnqMl2Ttc35tSm7BgcdICTwmgDZTNBgkG
/OG35X1FMZiV1IDVGPoytNcCgYEAqB5TO2aiUOdmKHiz0n6Q6Ds50n9qKHUyExPAWqgimIdXLjYp
GpJd/6AZY9Y2Ps62SC7fK/KS94uV8NAY7d+s6k6wIqJEkTfuDD/JCbk7FvlgsqXj5uKZ5Vt0B11E
ixB/acktVLII28Hi55h4j/PhktJk4iU9YI2Io/eQ+ZhGnfUCgYBGx3z1rQ9Wmgf3LHW0jf82zDKv
2VvWyz+EqRkxgZN+1mDoFuHW6qc45nfGkDFQr+Zfd8kDk0wMFCRmbwulman42zHtWtUSEiby845F
5nerqN0Qe1ZLqMswCldftOyX1mF6ld58Lpz2M+mb8mDrjYczHiJvzcM7ZZJQRTi9fUhLxw==
-----END RSA PRIVATE KEY-----

ubuntu@labsys:~$
```

Save this key somewhere and distribute it to the team, in case ssh access is needed for cluster machines.

Now we can create an EKS cluster and a worker node group with the eksctl command line utility. Cluster provisioning
usually takes between 10 and 15 minutes so be patient!

Make sure to adjust the node-type (image) and min/max counts according to the demo needs:

```
ubuntu@labsys:~$ eksctl create cluster \
--asg-access \
--color fabulous \
--name eks-ditl \
--nodegroup-name eks-ditl-workers \
--node-type t3.medium \
--nodes-min 3 \
--nodes-max 6 \
--node-ami auto \
--region us-east-2 \
--ssh-access \
--ssh-public-key eks-key-ditl \
--version 1.12

[ℹ]  using region us-east-2
[ℹ]  setting availability zones to [us-east-2b us-east-2c us-east-2a]
[ℹ]  subnets for us-east-2b - public:192.168.0.0/19 private:192.168.96.0/19
[ℹ]  subnets for us-east-2c - public:192.168.32.0/19 private:192.168.128.0/19
[ℹ]  subnets for us-east-2a - public:192.168.64.0/19 private:192.168.160.0/19
[ℹ]  nodegroup "eks-ditl-workers" will use "ami-04ea7cb66af82ae4a" [AmazonLinux2/1.12]
[ℹ]  using EC2 key pair "eks-key-ditl"
[ℹ]  creating EKS cluster "eks-ditl" in "us-east-2" region
[ℹ]  will create 2 separate CloudFormation stacks for cluster itself and the initial nodegroup
[ℹ]  if you encounter any issues, check CloudFormation console or try 'eksctl utils describe-stacks --region=us-east-2 --name=eks-ditl'
[ℹ]  2 sequential tasks: { create cluster control plane "eks-ditl", create nodegroup "eks-ditl-workers" }
[ℹ]  building cluster stack "eksctl-eks-ditl-cluster"
[ℹ]  deploying stack "eksctl-eks-ditl-cluster"
[ℹ]  building nodegroup stack "eksctl-eks-ditl-nodegroup-eks-ditl-workers"
[ℹ]  deploying stack "eksctl-eks-ditl-nodegroup-eks-ditl-workers"    
[✔]  all EKS cluster resource for "eks-ditl" had been created
[✔]  saved kubeconfig as "/home/ubuntu/.kube/config"
[ℹ]  adding role "arn:aws:iam::433017611331:role/eksctl-eks-ditl-nodegroup-eks-dit-NodeInstanceRole-1L6XRXIA6PSFI" to auth ConfigMap
[ℹ]  nodegroup "eks-ditl-workers" has 0 node(s)
[ℹ]  waiting for at least 3 node(s) to become ready in "eks-ditl-workers"
[ℹ]  nodegroup "eks-ditl-workers" has 3 node(s)
[ℹ]  node "ip-192-168-28-253.us-east-2.compute.internal" is ready
[ℹ]  node "ip-192-168-46-157.us-east-2.compute.internal" is ready
[ℹ]  node "ip-192-168-80-17.us-east-2.compute.internal" is ready
[✖]  neither aws-iam-authenticator nor heptio-authenticator-aws are installed
[ℹ]  cluster should be functional despite missing (or misconfigured) client binaries
[✔]  EKS cluster "eks-ditl" in "us-east-2" region is ready
ubuntu@labsys:~$
```

Configure kubectl to use the cluster:

```
ubuntu@labsys:~$ aws eks --region us-east-2 update-kubeconfig --name eks-ditl

Added new context arn:aws:eks:us-east-2:433017611331:cluster/eks-ditl to /home/ubuntu/.kube/config

ubuntu@labsys:~$
```


#### 1b. Set the security group for the EKS cluster

Participants need to interact with a service exposed by various LoadBalancer and Nodeport services.

Open up the cluster's worker security group to TCP traffic on all ports to allow participants to interact with the OSSP
service deployed:

```
ubuntu@labsys:~$ STACK_NAME=$(aws cloudformation describe-stacks | \
jq -r '.Stacks[].StackName' | grep eksctl-eks-ditl-nodegroup) && echo $STACK_NAME

eksctl-eks-ditl-nodegroup-eks-ditl-workers

ubuntu@labsys:~$ SG_ID=$(aws cloudformation describe-stack-resources --stack-name $STACK_NAME \
--logical-resource-id SG | jq -r '.StackResources[].PhysicalResourceId') && echo $SG_ID

sg-014178436bc5787e8

ubuntu@labsys:~$ aws ec2 authorize-security-group-ingress \
--group-id $SG_ID \
--ip-permissions '[{"IpProtocol": "tcp", "FromPort": 0, "ToPort": 65535, "IpRanges": [{"CidrIp": "0.0.0.0/0", "Description": "EKS cluster"}]}]'

ubuntu@labsys:~$
```


### 2. Launch step-specific masters

#### 2a. Harbor

Step two requires an instance of Harbor to be up and running in order to allow users access their images for Kubernetes.

**Input from Randy on Harbor deployment required**

#### 2b. Prometheus Master

To get metrics for Step 06, a Prometheus Master instance needs to be available.

Apply the static spec files to deploy the Prometheus master instance in the kube-system namespace:

```
ubuntu@labsys:~$ kubectl -n kube-system apply -f ~/kubecon-eu-2019/setup/prometheus-master/.

clusterrole.rbac.authorization.k8s.io/prometheus-master-server created
clusterrolebinding.rbac.authorization.k8s.io/prometheus-master-server created
configmap/prometheus-master-server created
deployment.extensions/prometheus-master-server created
service/prometheus-master-server created
serviceaccount/prometheus-master-server created

ubuntu@labsys:~$
```

The spec files are already pre-configured to retrieve information about pods and other Kubernetes elements. The
Prometheus helm charts participants will deploy will retrieve information on resources deployed in their personal
namespaces.


### 3. Enable Istio on EKS for Kiali

**Input from Chris on Istio deployment required**

_Copyright (c) 2019 RX-M LLC, Cloud Native Consulting, all rights reserved_

[RX-M LLC]: http://rx-m.io/rxm-cnc.svg "RX-M LLC"
