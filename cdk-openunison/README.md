# Canonical Kubernetes with OpenUnison

This repository explains how to deploy OpenUnison on-top of Canonical Kubernetes to provide authentication and authorization to the cluster. 

## Index

Todo. 

## Deploying Canonical Kubernetes

This documentation already assumes that you have Canonical Kubernetes up and running. Right now you can also do this on public clouds such as AWS if required for testing. You can deploy CDK using Juju or Conjure-up.

If you need the instructions for deploying CDK, they can be found here: [https://kubernetes.io/docs/getting-started-guides/ubuntu/installation/](https://kubernetes.io/docs/getting-started-guides/ubuntu/installation/) and here [https://jujucharms.com/canonical-kubernetes/](https://jujucharms.com/canonical-kubernetes/). There are also instructions in the Canonical Kubernetes Demos repository for deploying CDK on AWS or Azure.

## Deploying and Configuring OpenUnison

OpenUnison provides open source identity management with all the power of Unison, the commercial supported release of OpenUnison. It supports the following authentication types:
- Username and Password
- SAML2
- OpenID Connect
- TOTP (Google Authentication)
- U2F
- Secret Questions
- OAuth2 Bearer Tokens
- Kerberos
- One-Time-Password over SMS
- Certificates/PIV CaC

and the following DataStores:

- Active Directory
- LDAP
- RDBMS
- Web Services
- MongoDB
- Kubernetes
- OpenStack Keystone

There are also several deployment options, including the following: 
- Docker Container
- Embedded Undertow Sertver
- J2EE (Tomcat, Wildfly, Jetty etc)

An open-source vanilla kubernetes installation does not include integration with some of the common authentication mechanisms, I.E active directory, SAML or OAUTH, so we will use OpenUnison for that purpose. 

To deploy OpenUnison on our cluster, we first run 

## Conclusion 

We have covered a simple deployment of OpenUnison on-top of Canonical Kubernetes. We have configured and tested authentication of CDK with OpenUnison with the TOTP authentication mechanism, but many others could be used. 

### Getting Help & Support

OpenUnison is an open-source and free release of the Unison product, if you want to get commercial support you should [contact Tremolo](https://www.tremolosecurity.com/tremolosecurity-scalejs/contactus/index.html).

It is also possible to create issues on Github as OpenUnison is an open-source release this is strongly recommended, the repository can be found here: [https://github.com/TremoloSecurity/OpenUnison](https://github.com/TremoloSecurity/OpenUnison).

Canonical Kubernetes (CDK) is an open-source distribution and tickets can be opened directly against it on Github here: [https://github.com/juju-solutions/bundle-canonical-kubernetes](https://github.com/juju-solutions/bundle-canonical-kubernetes). 

Canonical sells enterprise support subscriptions for Ubuntu, Kubernetes, Openstack and for tools to manage and support those products. Please contact Canonical Sales for more information, here: [https://www.ubuntu.com/kubernetes](https://www.ubuntu.com/kubernetes). 

### Useful Links
- OpenUnison and Unison documentation [https://www.tremolosecurity.com/documentation/](https://www.tremolosecurity.com/documentation/)
- OpenUnison Github Repository [https://github.com/TremoloSecurity/OpenUnison](https://github.com/TremoloSecurity/OpenUnison)
- Canonical Kubernetes Bundle [https://github.com/juju-solutions/bundle-canonical-kubernetes](https://github.com/juju-solutions/bundle-canonical-kubernetes)
- OpenUnison Kubernetes Quickstart [https://github.com/TremoloSecurity/openunison-qs-kubernetes](https://github.com/TremoloSecurity/openunison-qs-kubernetes)