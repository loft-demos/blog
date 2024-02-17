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
metaTitle: vClusters for Kubernetes Demo Environments
metaDescription: "vClusters for Kubernetes Demo Environments: ."
comments: true
---
If you are demoing Kubernetes based applications and not using vCluster, virtual Kubernetes clusters, then you are doing it wrong. That's it. The end of this article...

Okay, just kidding, there is a bit more to this article than that.

This article explores a bit of my history with pre-sales demo environments and will then explore the benefits of using vClusters to quickly spin up shareable, ephemeral demo environments for Kubernetes based applications.

## Demo Environments the 'Old Way'

Back in 2015 I was just starting in a pre-sales engineering role at CloudBees. Docker containers were all the rage (Kubernetes was already a thing, but I hadn't heard of it yet). I was tasked with creating easily shared, and *somewhat* ephemeral demo environments for CloudBees' enterprise Jenkins product. So, Docker containers seemed like the obvious approach to take versus a more *traditional* VM based install. And guess where we ran Docker for the demo environment containers - on VMs, or to be more precise on EC2 instances. Managing everything was not trivial work, it was time consuming and fairly manual. But running containers with Docker did make it easier. And it allowed using tools like Docker Compose to coordinate a multi-container install.

Furthermore, the architecture of CloudBees' enterprise Jenkins product did not lend itself to running on one VM. Talk about Operations Center and managed controllers in Docker containers.

### Brief Foray with Docker Swarm

Using Docke Swarm

## Enter Kubernetes

Eventually CloudBees updated their enterprise Jenkins product to run on Kubernetes (after going with Mesos initially, but who knew :). So, with the added support for running on Kubernetes, we began using Kubernetes for demo environments, ephemeral workshop environments and trial environments; each being its own GKE cluster. In some ways it was certainly an improvement over using Docker containers on EC2 instances or other VMs. But it wasn't perfect. Kubernetes `Namespaces` were used to for multi-tenancy, to include workshop environments where we gave access to multiple prospects.

Talk about Operations Center and managed controllers in the context of multi-tenancy for the presales team and workshop attendees. Use and limitations of namespaces.

## 8 Years Later and Kubernetes has Become a Thing

Enter one of the most exciting Kubernetes projects in the last few year: vCluster. If you don't already know, vClusters are virtual Kubernetes clusters that run in a `Namspace` on a host cluster.

## Demo Environments at Loft Labs

At Loft Labs the Loft platform is used to manage Loft demo environment. That is, we are installing our own product into a vCluster that is running on top of our product - or more correctly, a vCluster being managed by our product. So, although our vCluster demo setup has a certain "Inception-ish" feel to it - our setup actually illustrates just how awesomely like-real Kubernetes vClusters are ... but better, especially for Kubernetes based demo environments.

vCluster templates
Loft/vCluster.Pro Apps
Additional support Apps/integrations
