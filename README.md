![rx-m LLC][RX-M LLC]


# kubecon-eu-2019


## A day in the life of a Cloud Native Dev


You made it! Good. Now get to work!!!

> N.B. This tutorial is designed to run on an Ubuntu 16.04 cloud instance. Pretty much any cloud instance will do and
if you know what you are doing you can probably get everything done on a Mac or a local VM running under windows.


### 1. SSH into the cloud instance assigned to you when you arrived

Something like this:

```
$ ssh -i k8scon-eu-2019.pem ubuntu@4.4.4.4

ubuntu@ip-172-31-45-121:~$
```

User is ubuntu, ip is the one assigned to you (not 4.4.4.4). If you are using putty you will need to convert the key
into ppk format.


### 2. Clone this repo

Clone and change into the root of the `kubecon-eu-2019` dir:

```
ubuntu@ip-172-31-45-121:~$ git clone https://github.com/RX-M/kubecon-eu-2019.git

Cloning into 'kubecon-eu-2019'...
remote: Enumerating objects: 8, done.
remote: Counting objects: 100% (8/8), done.
remote: Compressing objects: 100% (7/7), done.
remote: Total 8 (delta 1), reused 4 (delta 1), pack-reused 0
Unpacking objects: 100% (8/8), done.
Checking connectivity... done.

ubuntu@ip-172-31-45-121:~$ cd kubecon-eu-2019/

ubuntu@ip-172-31-45-121:~/kubecon-eu-2019$
```

Steps 1-9 are found in the respective subdirectory (step01, step02, step03, ...). Each step will follow on the heals of
the last so you can keep all of your work in this directory if you like. Each step has a `starter` directory you can use
to begin at that step if you didn't have time to finish the last step.

> For example, to begin at step 6, from the repo root simply run the copy command, `cp step06/starter/* .`, to copy all
of the starter files into the repo root. You should then be able to pickup at lab step 6 and jump right in.


### 3. Get started!

You ready to start your cloud native project exploration. Open up the step01/README.md and get coding!


_Copyright (c) 2019 RX-M LLC, Cloud Native Consulting, all rights reserved_

[RX-M LLC]: http://rx-m.io/rxm-cnc.svg "RX-M LLC"
