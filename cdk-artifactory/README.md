# Canonical Kubernetes with JFrog Artifactory

This document describes how to deploy and configure JFrog Artifactory on-top of Canonical Kubernetes to act as a private registry.

First we assume you have a Canonical Kubernetes cluster deployed with PersistantStorage configured and ready to use. I would recommend following [this guide](https://github.com/CanonicalLtd/canonical-kubernetes-demos/tree/master/cdk-ceph) which explains how to configure CDK with Ceph storage on AWS or Azure. If you have PersistantStorage provided by another mechanism, that can be used as well. I recommend adding several hundred gigabytes of storage instead of the default 10GB:

```
# commands for AWS
juju add-storage ceph-osd/0 osd-devices=ebs,200G,1
juju add-storage ceph-osd/1 osd-devices=ebs,200G,1
juju add-storage ceph-osd/2 osd-devices=ebs,200G,1

# commands for Azure
juju add-storage ceph-osd/0 osd-devices=azure,200G,1
juju add-storage ceph-osd/1 osd-devices=azure,200G,1
juju add-storage ceph-osd/2 osd-devices=azure,200G,1
```

In order to use Artifactory Pro you need a trial key or a paid subscription.

## Deployment with Rancher

Rancher can be used as a means to deploy Artifactory easily using its built in Helm and charts.

## Deployment with Helm
## Configuring CDK to Artifactory as a Private Registry
## Useful Links
