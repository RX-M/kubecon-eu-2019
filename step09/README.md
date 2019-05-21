![rx-m LLC][RX-M LLC]


# A Day in the life of a cloud native developer


## Step 09 - Telepresence

Telepresence is an open source tool that lets you run a service locally, while connecting that service into a remote
Kubernetes cluster.

In this step we're going to debug a simple service with telepresence.


### 1. A simple service example

> Starting telepresence the first time may take a minute or two because Kubernetes needs to download the server-side
image.

Imagine you want to switch to running a cluster deployed service locally so that you can debug it. We can replace the
version of the service running on cluster with the telepresence proxy, which will interact with the cluster as if it
were the service. However it will actually just forward traffic to our dev box, effectively plugging us into the
cluster.

To keep thing simple, in this step we'll debug a trivial HTTP server. Start a basic python server with a message file:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ echo "hello from your laptop" > file.txt
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ python3 -m http.server 8001 &
[1] 28887
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ Serving HTTP on 0.0.0.0 port 8001 ...
127.0.0.1 - - [21/May/2019 05:01:08] "GET /file.txt HTTP/1.1" 200 -

```

Now in another (or the same) shell curl the server:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ curl http://localhost:8001/file.txt

hello from your laptop

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

Great, now imagine that we want to expose this webserver to a Kubernetes cluster. Kill your test service and read on!

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ kill %1

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ jobs

[1]+  Terminated              python3 -m http.server 8001

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```


### 2. Installing telepresence

Install telepresence:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ curl -s https://packagecloud.io/install/repositories/datawireio/telepresence/script.deb.sh | sudo bash

Detected operating system as Ubuntu/xenial.
Checking for curl...
Detected curl...
Checking for gpg...
Detected gpg...
Running apt-get update... done.
Installing apt-transport-https... done.
Installing /etc/apt/sources.list.d/datawireio_telepresence.list...done.
Importing packagecloud gpg key... done.
Running apt-get update... done.
The repository is setup! You can now install packages.

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ sudo apt install --no-install-recommends telepresence

Reading package lists... Done
Building dependency tree
Reading state information... Done
The following additional packages will be installed:
  conntrack sshfs torsocks
Recommended packages:
  tor
The following NEW packages will be installed:
  conntrack sshfs telepresence torsocks
0 upgraded, 4 newly installed, 0 to remove and 81 not upgraded.
Need to get 732 kB of archives.
After this operation, 1,165 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://eu-central-1.ec2.archive.ubuntu.com/ubuntu xenial/main amd64 conntrack amd64 1:1.4.3-3 [27.3 kB]
Get:2 http://eu-central-1.ec2.archive.ubuntu.com/ubuntu xenial/universe amd64 sshfs amd64 2.5-1ubuntu1 [41.7 kB]
Get:3 http://eu-central-1.ec2.archive.ubuntu.com/ubuntu xenial/universe amd64 torsocks amd64 2.1.0-2 [56.2 kB]
Get:4 https://packagecloud.io/datawireio/telepresence/ubuntu xenial/main amd64 telepresence amd64 0.99 [607 kB]
Fetched 732 kB in 0s (750 kB/s)
Selecting previously unselected package conntrack.
(Reading database ... 57076 files and directories currently installed.)
Preparing to unpack .../conntrack_1%3a1.4.3-3_amd64.deb ...
Unpacking conntrack (1:1.4.3-3) ...
Selecting previously unselected package sshfs.
Preparing to unpack .../sshfs_2.5-1ubuntu1_amd64.deb ...
Unpacking sshfs (2.5-1ubuntu1) ...
Selecting previously unselected package torsocks.
Preparing to unpack .../torsocks_2.1.0-2_amd64.deb ...
Unpacking torsocks (2.1.0-2) ...
Selecting previously unselected package telepresence.
Preparing to unpack .../telepresence_0.99_amd64.deb ...
Unpacking telepresence (0.99) ...
Processing triggers for man-db (2.7.5-1) ...
Setting up conntrack (1:1.4.3-3) ...
Setting up sshfs (2.5-1ubuntu1) ...
Setting up torsocks (2.1.0-2) ...
Setting up telepresence (0.99) ...

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```


### 3. Running Telepresence

Check your deployments:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ kubectl get deploy

NAME                                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
ossp                                 2         2         2            2           28m
wra-172-31-30-5-metrics-prometheus   1         1         1            1           78m

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

Imagine we want to replace ossp with the local service. We can swap them like this:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ telepresence --swap-deployment ossp --expose 50088 --run python3 -m http.server 50088 &

[1] 30371

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ T: Starting proxy with method 'vpn-tcp', which has the following limitations: All processes are affected, only one telepresence can run per machine, and you
T: can't use other VPNs. You may need to add cloud hosts and headless services with --also-proxy. For a full list of method limitations see
T: https://telepresence.io/reference/methods.html
T: Volumes are rooted at $TELEPRESENCE_ROOT. See https://telepresence.io/howto/volumes.html for details.
T: Starting network proxy to cluster by swapping out Deployment ossp with a proxy
T: Forwarding remote port 50088 to local port 50088.

T: Guessing that Services IP range is 10.100.0.0/16. Services started after this point will be inaccessible if are outside this range; restart telepresence if
T:  you can't access a new Service.
T: Setup complete. Launching your command.
Serving HTTP on 0.0.0.0 port 50088 ...
127.0.0.1 - - [21/May/2019 05:11:21] code 404, message File not found
127.0.0.1 - - [21/May/2019 05:11:21] "GET /metrics HTTP/1.1" 404 -
127.0.0.1 - - [21/May/2019 05:11:35] code 404, message File not found
127.0.0.1 - - [21/May/2019 05:11:35] "GET /metrics HTTP/1.1" 404 -
127.0.0.1 - - [21/May/2019 05:11:51] code 404, message File not found
127.0.0.1 - - [21/May/2019 05:11:51] "GET /metrics HTTP/1.1" 404 -
127.0.0.1 - - [21/May/2019 05:12:05] code 404, message File not found
127.0.0.1 - - [21/May/2019 05:12:05] "GET /metrics HTTP/1.1" 404 -
127.0.0.1 - - [21/May/2019 05:12:21] code 404, message File not found
127.0.0.1 - - [21/May/2019 05:12:21] "GET /metrics HTTP/1.1" 404 -
127.0.0.1 - - [21/May/2019 05:12:35] code 404, message File not found
127.0.0.1 - - [21/May/2019 05:12:35] "GET /metrics HTTP/1.1" 404 -
127.0.0.1 - - [21/May/2019 05:12:51] code 404, message File not found
127.0.0.1 - - [21/May/2019 05:12:51] "GET /metrics HTTP/1.1" 404 -

```

> Note the hits are coming from Prometheus trying to scrape the metrics from the pods.

This does three things:

- Starts a VPN-like process that sends queries to the appropriate DNS and IP ranges to the cluster
- --swap-deployment tells Telepresence to replace the existing ossp pod with one running the Telepresence proxy (On
    exit, the old pod will be restored)
- --run tells Telepresence to run the local web server and hook it up to the networking proxy

As long as you leave the HTTP server running inside telepresence it will be accessible from inside the Kubernetes
cluster.

Run a curl client to test the replacement:

```
ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ kubectl get service

NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP                                                                 PORT(S)                         AGE
release-name-ossp                    LoadBalancer   10.100.221.157   a446d6c137b7611e98f95020deaa3a09-659563283.eu-central-1.elb.amazonaws.com   50088:30613/TCP                 122m
wra-172-31-30-5-metrics-prometheus   NodePort       10.100.52.191    <none>                                                                      9090:31345/TCP,3000:30218/TCP   88m

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$ curl a446d6c137b7611e98f95020deaa3a09-659563283.eu-central-1.elb.amazonaws.com:50088/file.txt

hello from your laptop

ubuntu@ip-172-31-30-5:~/kubecon-eu-2019$
```

Magic!!

If you made it this far you are something special, congratulations. Hope you had fund exploring the wide range of
powerful projects under the CNCF umbrella.

<br>

Congratulations, you have completed the tutorial step!

<br>

_Copyright (c) 2019 RX-M LLC, Cloud Native Consulting, all rights reserved_

[RX-M LLC]: http://rx-m.io/rxm-cnc.svg "RX-M LLC"
