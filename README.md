# Canonical Kubernetes Third-party Integrations

This repository contains source-code and documentation for various third-party integrations with Canonical Kubernetes (CDK). The documentation here is designed to help deployment of CDK in the field.

## Deploying Canonical Kubernetes

Most of the documentation in this repository assumes you already have a Canonical Kubernetes cluster up and running. If you need the steps to deploy a cluster, they can be found here: [Canonical Kubernetes Deployment Guide](https://kubernetes.io/docs/getting-started-guides/ubuntu/installation/).

Additional steps can be found on the Juju store for Canonical Kubernetes: [https://jujucharms.com/canonical-kubernetes/](https://jujucharms.com/canonical-kubernetes/). There is also some documentation in the demos repository here: [https://github.com/CanonicalLtd/canonical-kubernetes-demos](https://github.com/CanonicalLtd/canonical-kubernetes-demos).  

## Third-party Product List

This repository currently contains documentation for integrating the following products with Canonical Kubernetes:

- Rancher 2.x
- JFrog Artifactory
- F5 Big-IP Load Balancers
- NetApp Trident
- SonaType Nexus

If you would like to see your product integrated with Kubernetes or are having difficulty in doing so, please create an issue on this repository.

## Demo Repository

For demo workloads and for non-commercial product integrations, we have another repository called Canonical Kubernetes demos which can be found here: [https://github.com/CanonicalLtd/canonical-kubernetes-demos](https://github.com/CanonicalLtd/canonical-kubernetes-demos).

## Getting Help

If your issue is regarding a bug in the Canonical Kubernetes distribution itself, you can raise them here: [Canonical Kubernetes Bundle Builder](https://github.com/juju-solutions/bundle-canonical-kubernetes/issues).

Support for Canonical Kubernetes can be purchased here: [https://www.ubuntu.com/kubernetes](https://www.ubuntu.com/kubernetes).

Support for Ceph storage can be purchased here: [https://www.ubuntu.com/cloud/storage](https://www.ubuntu.com/cloud/storage)

## Licence and Contributing

The assets in this repository are distributed under the MIT licence, please feel free to re-use and modify our code. Corrections and new demos are always welcome, Pull Requests are always welcome.

If you wish to contribute your own integration, please try to the follow the structure of the other integrations. Generally this should include:

- README.MD containing all of the steps for deploying the integration, any configuration, caveats and useful links.
- Any scripts or yaml files used to deploy and configure the integration.
- Any associated licence with any re-used code, I.E if you fork code make sure it includes original licence.

## Useful Links
- [Canoical Kubernetes - Product Information](https://www.ubuntu.com/kubernetes)
- [Canonical Kubernetes Installation Guide](https://kubernetes.io/docs/getting-started-guides/ubuntu/installation/)
- [Canonical Kubernetes Juju Bundle on Juju Charms Store](https://jujucharms.com/canonical-kubernetes/)
- [Canonical Kubernetes Demos Repository](https://github.com/CanonicalLtd/canonical-kubernetes-demos)
- [Canonical Kubernetes Third-party Integration Documentation](https://github.com/CanonicalLtd/canonical-kubernetes-third-party-integrations)
- [Canonical Kubernetes Helm Charts](https://github.com/CanonicalLtd/canonical-kubernetes-helm-charts)
