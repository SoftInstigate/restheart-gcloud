# restheart-gcloud
Google Cloud Container Engine sample configuration files for RESTHeart.

![Gcloud logo](https://cloud.google.com/_static/images/new-gcp-logo.png)

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

```
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
```

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




