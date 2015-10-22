# restheart-gcloud

![Gcloud logo](https://cloud.google.com/_static/images/new-gcp-logo.png)

Google Cloud Container Engine sample configuration files for RESTHeart API Server.

## Introduction

This repo contains a set of files which allows to deploy a very basic RESTHeart and MongoDB configuration on the Google Cloud Container Engine. The purpose is double: to test Google Cloud services and to provide a way to deploy a RESTHeart demo environments in few minutes. This setup could be the foundation for a more robust and scalable configuration.

## What is Google Container Engine?

> Google Container Engine is a powerful cluster manager and orchestration system for running your Docker containers. Container Engine schedules your containers into the cluster and manages them automatically based on requirements you define (such as CPU and memory). It's built on the open source Kubernetes system, giving you the flexibility to take advantage of on-premises, hybrid, or public cloud infrastructure.
> - https://cloud.google.com/container-engine/

## Setup

The first step is to read carefully the official Google documentation, starting from the "[Before you begin](https://cloud.google.com/container-engine/docs/before-you-begin)" section.

At the end of this section you should have installed and configured the `gcloud` and `kubectl` command line tools. (note these are not available for Windows).

To view the final glocud configuration run:

    $ gcloud config list

You'll have different values depending on your project-id, zone, account, etc. Our own output is the following:

    [app]
    use_appengine_api = True
    [compute]
    zone = europe-west1-c
    [container]
    cluster = rh-cluster-1
    [core]
    account = <Google_account>
    disable_usage_reporting = False
    project = <project-id>
    [meta]
    active_config = <config_name>

## Next steps

> **Note**: this is not a tutorial for the Google Cloud Platform, you need a minimal working knowledge of the platform and its tools to understand the following steps.

### 1) Download the configuration files

Clone this repo, you'll see four JSON files:

* mongo-master-controller.json
* mongo-master-service.json
* restheart-controller.json
* restheart-service.json

These will configure one Replication Controller for MongoDB and one for RESTHeart, with the related services. Maybe you want to overview some of these [Kubernetes concepts](http://kubernetes.io/v1.0/docs/user-guide/overview.html) before going forward.

### 2) Create a gcloud cluster

We create a new cluster, for example `rh-cluster-1`

    $ gcloud container clusters create rh-cluster-1

You can list clusters for in your project or get the details for a single cluster:

    $ gcloud container clusters list
    $ gcloud container clusters describe rh-cluster-1

### 3) start the MongoDB master

The first thing that we're going to do is start up a pod for the MongoDB master. We'll use a replication controller to create the pod—even though it's a single pod, the controller is still useful for monitoring health and restarting the pod if required.

We'll use this config file: `mongo-master-controller.json`.

Start up the pod:

    $ kubectl create -f mongo-master-controller.json

Verify that the master is running:

    kubectl get pods -l name=mongo-master
    NAME                 READY     STATUS    RESTARTS   AGE
    mongo-master-htjg6   1/1       Running   0          1d

### 4) Look at your Docker containers (optional)

Find the node's name, listed in the NODE column in response to `kubectl get`:

    $ kubectl get pods -l name=mongo-master -o wide
    NAME                 READY     STATUS    RESTARTS   AGE       NODE
    mongo-master-htjg6   1/1       Running   0          1d        gke-rh-cluster-1-54767033-node-42aq

SSH into the node:

    $ gcloud compute ssh mongo-master-htjg6

List the running Docker containers using the sudo docker ps command:

    $ sudo docker ps

### 5) Start the MongoDB service

A service is an abstraction which defines a logical set of pods and a policy by which to access them. It is effectively a named load balancer that proxies traffic to one or more pods.

When you set up a service, you tell it the pods to proxy based on pod labels. The pod created in step one has the label name=mongo-master.

We'll use the file `mongo-master-service.json` to create a service for the MongoDB master:

    $ kubectl create -f mongo-master-service.json

To view service's details:

    $ kubectl get services -l name=mongo-master
    NAME      LABELS              SELECTOR            IP(S)            PORT(S)
    mongodb   name=mongo-master   name=mongo-master   10.231.252.199   27017/TCP

### 6) Create RESTHeart's Pods

Use this config file: `restheart-controller.json`.

    $ kubectl create -f restheart-controller.json

Verify that RESTHeart is running:

    $ kubectl get pods -l name=restheart
    NAME              READY     STATUS    RESTARTS   AGE
    restheart-vpque   1/1       Running   0          1d

### 7) Create RESTHeart's Service

Use this config file: `restheart-service.json`:

    $ kubectl create -f restheart-service.json
    NAME        LABELS           SELECTOR         IP(S)            PORT(S)
    restheart   name=restheart   name=restheart   10.231.250.241   80/TCP
                                                  146.148.xxx.xxx  

> **Note**: as we are using a load balancer, you'll see two IPs: one internal and the other public (in this example `146.148.xxx.xxx`). It takes a few minutes for the LB to be provisioned and started, so you might not see the public address at first.

At this stage you'll have a fully functional RESTHeart + MongoDB server, running on Docker containers orchestrated by Kubernetes.

### 8) Display all RC, Pods and Services

To view all Replication Controllers:

    $ kubectl get rc
    CONTROLLER     CONTAINER(S)     IMAGE(S)                  SELECTOR            REPLICAS
    mongo-master   mongo-master     mongo                     name=mongo-master   1
    restheart      restheart-demo   softinstigate/restheart   name=restheart      1

To view all Pods:

    $ kubectl get pods
    NAME                 READY     STATUS    RESTARTS   AGE
    mongo-master-htjg6   1/1       Running   0          1d
    restheart-vpque      1/1       Running   0          1d

To view all Services:

    $ kubectl get services
    NAME         LABELS                                    SELECTOR            IP(S)            PORT(S)
    kubernetes   component=apiserver,provider=kubernetes   <none>              10.231.240.1     443/TCP
    mongodb      name=mongo-master                         name=mongo-master   10.231.252.199   27017/TCP
    restheart    name=restheart                            name=restheart      10.231.250.241   80/TCP
                                                                               146.148.xxx.xxx    

### 9) Access RESTHeart

Point your browser to `http://146.148.xxx.xxx/browser` to access RESTHeart's embedded HAL Browser.

> **WARNING**: as we didn't attach to MongoDB any permanent filesystem, that pod is ephemeral and will loose all data if restarted. Please have a look at [Create your persistent disks](https://cloud.google.com/container-engine/docs/tutorials/persistent-disk/#create_your_persistent_disks) for an example of allowing the application to preserve its state across pod shutdown and startup. This is not a problem for the RESTHeart Pod, as the RESTHeart process is fully stateless.

