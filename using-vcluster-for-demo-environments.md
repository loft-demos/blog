---
type: post
title: Not Using vClusters for Kubernetes Demo Environments, Then You Are Doing It Wrong!
slug: using-vcluster-for-demo-environments
authors:
  - kurt-madel
tags:
  - vcluster
  - cost-optimization
  - enterprise
  - use-cases
  - demos
metaTitle: Using vClusters for Kubernetes Demo Environments
metaDescription: "vClusters for Kubernetes Demo Environments: ."
comments: true
---
If you are demoing Kubernetes based applications and not using [vCluster](https://www.vcluster.com/docs/what-are-virtual-clusters), that is virtual Kubernetes clusters, then you are doing it wrong. That's it. The end of this article...

Okay, just kidding, there is a bit more to this article than that and actually too much for one article.

This series of articles will explore a bit of my history with pre-sales demo environments and will then explore the benefits of using vClusters to quickly spin up shareable, ephemeral demo environments for Kubernetes based applications.

## Demo Environments the 'Old Way'

Back in 2015 I was just starting in a pre-sales engineering role at CloudBees. Docker containers were all the rage (Kubernetes was already a thing, but I hadn't heard of it yet). I was tasked with creating easily shareable, and *mostly* ephemeral demo environments for CloudBees' enterprise Jenkins product. So, Docker containers seemed like the obvious approach to take versus a more *traditional* VM based install. And guess where we ran Docker for the demo environment containers - on VMs, or to be more precise on EC2 instances. Managing everything was not trivial, it was time consuming and very manual. But running containers with Docker did make it easier. And it allowed using tools like Docker Compose to coordinate a multi-container install.

Furthermore, the architecture of CloudBees' enterprise Jenkins product did not lend itself to running on one VM. Talk about Operations Center and managed controllers in Docker containers.

One of CloudBees' main value adds for their enterpirse Jenkins product was (and still is) something they called Operations Center which is basically a special instance of Jenkins that allows you to easily manage multiple instances of Jenkins. Think of it as a Jenkins controller for Jenkins instances.

## Enter Kubernetes

Eventually CloudBees updated their enterprise Jenkins product to run on Kubernetes (after going with Mesos initially, but who knew :). The new support for running on Kubernetes allowed us to start using Kubernetes for demo environm.              ents, semi-ephemeral workshop environments and trial sandbox environments; with each install being its own Google Cloud Platform GKE cluster. In some ways it was certainly an improvement over using Docker containers on EC2 instances or other VMs. But it was far from perfect for multiple reasons to include:
- Time: Spinning up these environmens was definitely not instant. And although GKE clusters typically spin up much faster than AWS EKS or Azure AKS clusters, it typically still takes more than 10 minutes and sometimes more than 20 minutes to create a new GKE cluster, and that is before even installig the apps to be demoed.
- Money: these environments ran for long periods of time, and although CloudBees' does have a cool feature that 'hiberantes' individual controllers, the Operations Center did not hibernate. So, having a GKE cluster for multiple environments for each use case (demos, workshops, trial sandboxes) is fairly expensive.
- Security: Kubernetes `Namespaces` were used to for multi-tenancy, to include workshop environments where we gave access to multiple prospects.

Talk about Operations Center and managed controllers in the context of multi-tenancy for the presales team and workshop attendees. Use and limitations of namespaces.

## 8 Years Later and Kubernetes Has Become a Thing

So, running CloudBees CI demo environments on Kubernetes made it easier to manage, but it still had its drawbacks. 

Enter one of the most exciting Kubernetes projects in the last few year: vCluster. If you don't already know, vClusters are virtual Kubernetes clusters that run in a `Namespace` on a host cluster. They spin up in seconds versus 10+ minutes - so much faster than the GKE clusters I was using at CloudBees. 

## Demo Environments at Loft Labs

At Loft Labs we use the vCluster Platform to create and manage demo environments. That is, we are installing our own product into a vCluster that is running on top of our product - or more correctly, a vCluster being managed by our product. So, although our vCluster demo setup has a certain "Inception-ish" feel to it - our setup actually illustrates just how awesomely like-real Kubernetes vClusters are ... but better, especially for Kubernetes based demo environments.

Sleep mode and auto-delet makes it easier to manage unused demo environments.

vCluster templates allows us to manage the demo vClusters at scale. 
Loft/vCluster.Pro Apps
Additional support Apps/integrations
