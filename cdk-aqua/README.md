# Canonical Kubernetes with Aqua Container Security Platform by Aqua Security

Aqua Container Security Platform has been tested and verified to work with Canonical Kubernetes and Ubuntu 18.04 and 16.04. 

The product provides security at all the stages of your container platform, from automated build pipeline through to container deployment and can be used either in a passive or enforcing mode.

Aqua is an excellent tool for day-2 operations of container platforms, including Docker and Kubernetes. 

Here are some of the features the product provides: 
- CVE Scanning of Host Operating System (Ubuntu) and running Containers
- Security scanning built into the CD/CI Pipeline 
- Container Drift Management (ensure they are not being modified from their original state)
- Improved Secret Injection for Containers
- CIS Benchmark Scanning of Containers
- Enforce that only compliant Containers are allowed to run
- Mapping of usual Network Traffic to check for breaches, I.E get alerts if traffic suddenly goes to obscure source. 
- Alerting of security issues

This list of features is constantly expanding as new functionality is added to the product. 

## CIS Auditing of Canonical Kubernetes with Kube Bench

Aqua Security provides a free, opensource tool which can be used to perform CIS Benchmark checks of Kubernetes clusters. 

Patches have been written for this tool so it can be used with snap package based Kubernetes installs, such as those provided by Canonical Kubernetes (CDK).

The repository for Kube Bench can be found here: [https://github.com/aquasecurity/kube-hunter](https://github.com/aquasecurity/kube-bench)

Inside the repository, full instructions for using and deploying Kube Bench can be found. 

## CIS Auditing of Canonical Kubernetes with Kube Hunter 

Aqua Security provides a free, opensource tool which can be used to perform penetration tests against a Kubernetes cluster. 

Patches have been written for this tool so it can be used with snap package based Kubernetes installs, such as those provided by Canonical Kubernetes (CDK).

The repository for Kube Hunter can be found here: [https://github.com/aquasecurity/kube-hunter](https://github.com/aquasecurity/kube-hunter)

Inside the repository, full instructions for using and deploying Kube Hunter can be found. 

## Contacting Aqua for more information 

As Aqua Container Security Platform is not an open-source product, we have chosen not to release any of their manuals, documentation or kubernetes manifests for deploying their product. 

We strongly recommend reaching directly to Aqua for a trial and demonstration of their product using their website: 

[https://www.aquasec.com/about-us/contact-us/](https://www.aquasec.com/about-us/contact-us/)

They will be able to provide you a trial and the full documentation for deploying Aqua on Kubernetes. 

## Useful Links
- [https://www.aquasec.com/products/aqua-container-security-platform/](https://www.aquasec.com/products/aqua-container-security-platform/)
- [https://www.aquasec.com/products/open-source-projects/](https://www.aquasec.com/products/open-source-projects/)
- [https://github.com/aquasecurity/kube-hunter](https://github.com/aquasecurity/kube-hunter)
- [https://github.com/aquasecurity/kube-bench](https://github.com/aquasecurity/kube-bench)
