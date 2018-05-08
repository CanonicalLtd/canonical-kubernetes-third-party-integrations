# Canonical Kubernetes with Rancher (cdk-rancher)

This repository explains how to deploy Rancher 2.0alpha on Canonical Kubernetes.

These steps are currently in alpha/testing phase and will most likely change.

### Index

- [Deploying Canonical Kubernetes](https://github.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/tree/master/cdk-rancher#deploying-canonical-kubernetes)
- [Deploying Rancher](https://github.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/tree/master/cdk-rancher#deploying-rancher)
  - [Deploying Rancher with a nodeport](https://github.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/tree/master/cdk-rancher#deploying-rancher-with-a-nodeport)
  - [Deploying Rancher with an ingress rule](https://github.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/tree/master/cdk-rancher#deploying-rancher-with-an-ingress-rule)
  - [Removing Rancher](https://github.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/tree/master/cdk-rancher#removing-rancher)
- [Using Rancher](https://github.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/tree/master/cdk-rancher#using-rancher)
  - [The Rancher GUI](https://github.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/tree/master/cdk-rancher#the-rancher-gui)
  - [Deploying a Workload with Rancher](https://github.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/tree/master/cdk-rancher#deploying-a-workload-with-rancher)
  - [Deploying a Workload with Rancher Catalog](https://github.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/tree/master/cdk-rancher#deploying-a-workload-with-rancher-catalog)
  - [Adding additional Kubernetes Clusters to Rancher](https://github.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/tree/master/cdk-rancher#adding-additional-kubernetes-clusters-to-rancher)
  - [Troubleshooting Rancher](https://github.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/tree/master/cdk-rancher#troubleshooting-rancher)
- [Conclusion](https://github.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/tree/master/cdk-rancher#conclusion)
  - [Software versions](https://github.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/tree/master/cdk-rancher#software-versions)
  - [Getting Help & Support](https://github.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/tree/master/cdk-rancher#getting-help--support)
- [Useful Links](https://github.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/tree/master/cdk-rancher#useful-links)

## Deploying Canonical Kubernetes

This documentation already assumes that you have Canonical Kubernetes up and running and deployed in an enivornment where you can also deploy F5 Big-IP load-balancers, either physical or virtual instances. Right now you can also do this on public clouds such as AWS if required for testing.

If you need the instructions for deploying CDK, they can be found here: [https://kubernetes.io/docs/getting-started-guides/ubuntu/installation/](https://kubernetes.io/docs/getting-started-guides/ubuntu/installation/) and here [https://jujucharms.com/canonical-kubernetes/](https://jujucharms.com/canonical-kubernetes/).

## Deploying Rancher

To deploy Rancher, we just need to run the Rancher container workload on-top of Kubernetes. Rancher provides their containers through dockerhub ([https://hub.docker.com/r/rancher/server/tags/](https://hub.docker.com/r/rancher/server/tags/)) and can be downloaded freely from the internet. If you're running your own registry or have an offline deployment, the container should be downloaded and pushed to the private registry.  

### Deploying Rancher with a nodeport

The yaml file called cdk-rancher-nodeport.yaml inside this repository can be used to deploy Rancher with a nodeport for accessing the cluster. Once kubectl is running and working, run the following command to deploy Rancher:

```
  kubectl apply -f cdk-rancher-nodeport.yaml
```

Now we need to open this nodeport so we can access it. For that, we can use juju. We need to run the open-port command for each of the worker nodes in our cluster. Inside the cdk-rancher-nodeport.yaml file, the nodeport has been set to 30443. Below shows how to open the port on each of the worker nodes:

```
   # repeat this for each kubernetes worker in the cluster.
   juju run --unit kubernetes-worker/0 "open-port 30443"
   juju run --unit kubernetes-worker/1 "open-port 30443"
   juju run --unit kubernetes-worker/2 "open-port 30443"
```

Rancher can now be accessed on this port through a worker IP or DNS entries if you have created them. It is generally recommended that you create a DNS entry for each of the worker nodes in your cluster. For example, if you have three worker nodes and you own the domain example.com, you could create three A records, one for each worker in the cluster.

As creating DNS entries is outside of the scope of this document, we will use the freely available xip.io service which can return A records for an IP address which is part of the domain name. For example, if you have the domain rancher.35.178.130.245.xip.io, the xip.io service will automatically return the IP address 35.178.130.245 as an A record which is useful for testing purposes.  For your deployment, the IP address 35.178.130.245 should be replaced with one of your worker IP address, which can be found using Juju or AWS:

```
 calvinh@ubuntu-ws:~/Source/cdk-rancher$ juju status

# ... output omitted.

Unit                      Workload  Agent  Machine  Public address  Ports                     Message
easyrsa/0*                active    idle   0        35.178.118.232                            Certificate Authority connected.
etcd/0*                   active    idle   1        35.178.49.31    2379/tcp                  Healthy with 3 known peers
etcd/1                    active    idle   2        35.177.99.171   2379/tcp                  Healthy with 3 known peers
etcd/2                    active    idle   3        35.178.125.161  2379/tcp                  Healthy with 3 known peers
kubeapi-load-balancer/0*  active    idle   4        35.178.37.87    443/tcp                   Loadbalancer ready.
kubernetes-master/0*      active    idle   5        35.177.239.237  6443/tcp                  Kubernetes master running.
  flannel/0*              active    idle            35.177.239.237                            Flannel subnet 10.1.27.1/24
kubernetes-worker/0*      active    idle   6        35.178.130.245  80/tcp,443/tcp,30443/tcp  Kubernetes worker running.
  flannel/2               active    idle            35.178.130.245                            Flannel subnet 10.1.82.1/24
kubernetes-worker/1       active    idle   7        35.178.121.29   80/tcp,443/tcp,30443/tcp  Kubernetes worker running.
  flannel/3               active    idle            35.178.121.29                             Flannel subnet 10.1.66.1/24
kubernetes-worker/2       active    idle   8        35.177.144.76   80/tcp,443/tcp,30443/tcp  Kubernetes worker running.
  flannel/1               active    idle            35.177.144.76                        

# Note the IP addresses for the kubernetes-workers in the example above.  You should pick one of the public addresses.
```

Try opening up Rancher in your browser using the nodeport and the domain name or ip address:  

```
  # replace the IP address with one of your Kubernetes worker, find this from juju status command.
  wget https://35.178.130.245.xip.io:30443 --no-check-certificate

  # this should also work
  wget https://35.178.130.245:30443 --no-check-certificate
```

If you need to make any changes to the kubernetes configuration file, edit the yaml file and then just use apply again:

```
  kubectl apply -f cdk-rancher-nodeport.yaml
```

### Deploying Rancher with an ingress rule

The cdk-rancher-ingress.yaml yaml file contains an example Kubernetes configuration for deploying Rancher with an ingress rule. If you use an ingress rule, you don't need to use the nodeport or open additional ports using juju.


It is generally recommended that you create a DNS entry for each of the worker nodes in your cluster. For example, if you have three worker nodes and you own the domain example.com, you could create three A records, one for each worker in the cluster.

As creating DNS entries is outside of the scope of this document, we will use the freely available xip.io service which can return A records for an IP address which is part of the domain name. For example, if you have the domain rancher.35.178.130.245.xip.io, the xip.io service will automatically return the IP address 35.178.130.245 as an A record which is useful for testing purposes.  For your deployment, the IP address 35.178.130.245 should be replaced with one of your worker IP address, which can be found using Juju or AWS:

```
 calvinh@ubuntu-ws:~/Source/cdk-rancher$ juju status

# ... output omitted.

Unit                      Workload  Agent  Machine  Public address  Ports                     Message
easyrsa/0*                active    idle   0        35.178.118.232                            Certificate Authority connected.
etcd/0*                   active    idle   1        35.178.49.31    2379/tcp                  Healthy with 3 known peers
etcd/1                    active    idle   2        35.177.99.171   2379/tcp                  Healthy with 3 known peers
etcd/2                    active    idle   3        35.178.125.161  2379/tcp                  Healthy with 3 known peers
kubeapi-load-balancer/0*  active    idle   4        35.178.37.87    443/tcp                   Loadbalancer ready.
kubernetes-master/0*      active    idle   5        35.177.239.237  6443/tcp                  Kubernetes master running.
  flannel/0*              active    idle            35.177.239.237                            Flannel subnet 10.1.27.1/24
kubernetes-worker/0*      active    idle   6        35.178.130.245  80/tcp,443/tcp,30443/tcp  Kubernetes worker running.
  flannel/2               active    idle            35.178.130.245                            Flannel subnet 10.1.82.1/24
kubernetes-worker/1       active    idle   7        35.178.121.29   80/tcp,443/tcp,30443/tcp  Kubernetes worker running.
  flannel/3               active    idle            35.178.121.29                             Flannel subnet 10.1.66.1/24
kubernetes-worker/2       active    idle   8        35.177.144.76   80/tcp,443/tcp,30443/tcp  Kubernetes worker running.
  flannel/1               active    idle            35.177.144.76

# Note the IP addresses for the kubernetes-workers in the example above.  You should pick one of the public addresses.
```

Looking at the output from the juju status above, the Public Address (35.178.130.245) can be used to create a xip.io DNS entry (rancher.35.178.130.245.xip.io) whichs hould be placed into the cdk-rancher-ingress.yaml file. You could also create your own DNS entry as long as it resolves to each of the worker nodes or one of them it will work fine:

```
  # Edit this line inside the rancher ingress yaml file
  cat cdk-rancher-ingress.yaml | grep xip.io
  - host: rancher.35.178.130.245.xip.io
```

Once you've edited the ingress rule to reflect your DNS entries, run the kubectl apply -f cdk-rancher-ingress.yaml to deploy Kubernetes:

```
 kubectl apply -f cdk-rancher-ingress.yaml
```

Rancher can now be accessed on the regular 443 through a worker IP or DNS entries if you have created them. Try opening it up in your browser:

```
  # replace the IP address with one of your Kubernetes worker, find this from juju status command.
  wget https://35.178.130.245.xip.io:443 --no-check-certificate
```

If you need to make any changes to the kubernetes configuration file, edit the yaml file and then just use apply again:

```
  kubectl apply -f cdk-rancher-ingress.yaml
```

### Removing Rancher

If you wish to remove rancher from the cluster, we can do it using kubectl. Deleting constructs in Kubernetes is as simple as creating them:

```
  # If you used the nodeport example change the yaml filename if you used the ingress example.
  kubectl delete -f cdk-rancher-nodeport.yaml
```

## Using Rancher

Rancher can be accessed using it's web interface but a CLI client is planned in the future.

The best resource for using Rancher is the official manual found here: [http://rancher.com/docs/rancher/v2.0/en/quick-start-guide/](http://rancher.com/docs/rancher/v2.0/en/quick-start-guide/).

Hit the Rancher IP in your browser, you should see a login prompt:

![rancher login page](https://raw.githubusercontent.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/master/cdk-rancher/images/rancher-login.png "Rancher Web GUI Login Page")

The default username is admin and password is admin. You should be prompted to change them, it is recommended you use a strong password.

Once you've set the password, you should see a GUI like this:

![rancher gui page](https://raw.githubusercontent.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/master/cdk-rancher/images/rancher-gui.png "Rancher Web GUI Page")

### The Rancher GUI

The Rancher GUI has three main views, each of which has its own set of sub-menus.  

![rancher gui cluster page](https://raw.githubusercontent.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/master/cdk-rancher/images/rancher-gui-cluster.png "Rancher Web GUI Cluster")

These are the three main views which contain a set of sub-menus:

- Global View: Provides a view of each of the Kubernetes clusters under Rancher's control. This provides a break down of Nodes, Node Drivers, Catalogs, Users and Security Settings.   

- Cluster: This provides a view of of an individual Kubernetes cluster. It allows you to download the kubeconfig file for a cluster, check basic cluster stats, namespaces, projects, nodes and users for a particular cluster.

- Project: This provides a view for a project defined on a Kubernetes cluster. It allows you to configure workloads, DNS entries, ingress rules, volumes, secrets, registries, certificates and most importantly, the catalog.

Essentially each of these different views is a drill-down from the top level. The Global view provides a high-level overview for all of the clusters it has control over, where as the project view is the lowest level you can drill down into within a cluster.

### Deploying a Workload with Rancher

Once Rancher is up and running, we can launch a regular Kubernetes workload. First hover over the Global option in the top left-hand part of the menu. Once the menu appears, click Default at the bottom to select the Default project. Next, hit the Workloads Tab:

![rancher gui workloads](https://raw.githubusercontent.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/master/cdk-rancher/images/rancher-gui-workloads.png "Rancher Web GUI Workloads")

Hit the Deploy button at the Top Right and you should see an interface like this:

![rancher gui wkdeploy](https://raw.githubusercontent.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/master/cdk-rancher/images/rancher-gui-workloads-deploy-pod.png "Rancher Web GUI Deploy Pod")

Fill in details and then hit the Launch button. Congratulations, you've just ran your first Ubuntu workload on CDK using Rancher.

To interact with the new container, you can enter an interactive session using the following command:

```
# My workstation.
calvinh@ubuntu-ws:~/Source/cdk-rancher$ uname -a
Linux ubuntu-ws 4.13.0-17-generic #20-Ubuntu SMP Mon Nov 6 10:04:08 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux

# SSH into the Docker Container and check the container kernel version.
calvinh@ubuntu-ws:~/Source/cdk-rancher$ kubectl exec -it hello-ubuntu -- /bin/bash
root@hello-ubuntu:/# uname -a
Linux hello-ubuntu 4.4.0-1050-aws #59-Ubuntu SMP Tue Jan 30 19:57:10 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```

The workloads interface can also be used to destroy, reprovision, clone and perform many other tasks on workloads running on Kubernetes.

### Deploying a Workload with Rancher Catalog

Rancher can also launch a workload using the Rancher catalog. First hover over the Global option in the top left-hand part of the menu. Once the menu appears, click Default at the bottom to select the Default project.

First lets add an SSL certificate and private key for use within our default project. First generate your private key and certificate:

```
 calvinh@ubuntu-ws:~/Documents/Rancher$ openssl req -newkey rsa:4096 -nodes -keyout key.pem -x509 -days 365 -out certificate.pem
Generating a 4096 bit RSA private key
.....................................................................++
........................................................................................................++
writing new private key to 'key.pem'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:GB
State or Province Name (full name) [Some-State]:London
Locality Name (eg, city) []:London
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Canonical LTD
Organizational Unit Name (eg, section) []:CPE
Common Name (e.g. server FQDN or YOUR name) []:35.178.130.245.xip.io
Email Address []:calvin.hartwell@canonical.com
```

This will create two files, certificate.pem and key.pem. Back on the Rancher GUI, go to Resources -> and then Certificates. Give your certificate a name, and import your private key and certificate file either by selecting the files on Disk or by copying and pasting the content into the boxes manually. You can also choose which namespace it should be assigned. Hit the save button when you're done:

![rancher certs](https://raw.githubusercontent.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/master/cdk-rancher/images/rancher-gui-add-cert.png "Rancher Add Certs")

We also need to Enable the catalogs. To do this, go to the Global View and hit the Catalogs Button. You should see a screen like this:

![rancher enable catalog](https://raw.githubusercontent.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/master/cdk-rancher/images/rancher-gui-catalogs-enable.png "Rancher Catalogs Enable")

There are two buttons which read Disabled, one for Rancher certified Library and one for Community Contributed. Press both of these to Enable to continue.

Next, hit the Catalog Apps button, you should see a screen like this:

![rancher gui catalog](https://raw.githubusercontent.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/master/cdk-rancher/images/rancher-gui-catalog-launch.png "Rancher Catalog Launch")

Hit the Launch button and you should see the list of available app within the Catalogue, including Hadoop, Jenkins, Artifactory, F5 Load Balancer, Confluence, and many other applications. The default catalog is provided by Helm, but it is possible to create your own if required:

![rancher gui catalogapps](https://raw.githubusercontent.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/master/cdk-rancher/images/rancher-gui-catalog-apps.png "Rancher Catalog Launch")

Let's try and launch Artifactory. Find JFrog Artifactory in the list of apps and hit View Details. Use the default values but make sure you change the SSL Certificate option to reflect the newly created SSL Certificate and then hit Launch at the bottom of the page:

![rancher gui catalogapps](https://raw.githubusercontent.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/master/cdk-rancher/images/rancher-gui-artifactory.png "Rancher Artifactory Launch")

Rancher will now attempt to deploy Artifactory from the Helm Catalogue automatically on-top of Kubernetes.

### Adding additional Kubernetes Clusters to Rancher

It is also possible to add additional clusters to Rancher. This is done by providing Rancher with the kubectl configuration file, which Rancher uses to take control of the additional clusters. Highlight the Global menu in the top-left and press it, and then go to Clusters. You should see a menu like this below:

![rancher gui clusters](https://raw.githubusercontent.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/master/cdk-rancher/images/rancher-gui-add-cluster.png "Rancher Artifactory Clusters")

Hit the add Cluster button and you should be at another screen which asks you what kind of cluster you'd like to import:

![rancher gui rancher](https://raw.githubusercontent.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/master/cdk-rancher/images/rancher-gui-add-cluster-options.png "Rancher Artifactory Options")

Hit the Import an Existing Cluster option and move onto the next screen:

![rancher gui clusters](https://raw.githubusercontent.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/master/cdk-rancher/images/rancher-gui-add-cluster-kubeconf.png "Rancher Artifactory Kubectl")

This next screen allows you to specify a name for the cluster and provide the kubeconfig file (the same configuration file used for kubectl) which is used to control and authenticate with the cluster. Give the new cluster a name and hit the Read from a file button to import your kubeconfig file or manually copy and paste the contents of the kubeconfig from its file. Note you will nee to SCP this file from your newly provisioned CDK cluster.

### Troubleshooting Rancher

If for any reason the Rancher pod becomes unresponsive, you can bounce the pod using the following command:

```
  # Replace the apply with your rancher yaml file.
  kubectl delete po $(kubectl get po | grep rancher | cut -d ' ' -f1) && kubectl apply -f cdk-rancher-nodeport.yaml
```

If is possible to check the Rancher logs using the following command:

```
  kubectl logs -f $(kubectl get po | grep rancher | cut -d ' ' -f1)
```

## Conclusion

This documentation has explained how to configure and deploy Canonical Kubernetes with Rancher running on top. It also provided a short introduction on how to use Rancher to control and manage Canonical Kubernetes. Note that this documentation was written at the time of the Rancher 2.0 alpha but a beta release is due out very soon which would be a better release candidate for a production environment.

### Software Versions

The software versions used throughout this docunmentation are:
 - Kubernetes 1.9.3
 - Rancher v2.0.0-alpha16

### Getting Help & Support

This document was written by [Calvin Hartwell](https://www.linkedin.com/in/calvinhartwell), please feel to drop me an email on [calvin.hartwell@canonical.com](calvin.hartwell@canonical.com).

To open issues with CDK, please open an issue with the repository here [https://github.com/juju-solutions/bundle-canonical-kubernetes](https://github.com/juju-solutions/bundle-canonical-kubernetes). If you want professional support for Kubernetes please contact Canonical Sales [https://www.ubuntu.com/kubernetes](https://www.ubuntu.com/kubernetes).

Rancher Provides support through [Slack](https://slack.rancher.io/) and by paid support [https://rancher.com/support/](https://rancher.com/support/). Please contact Rancher directly to get more help with their product.

## Useful Links

- [http://rancher.com/docs/rancher/latest/en/quick-start-guide/](http://rancher.com/docs/rancher/latest/en/quick-start-guide/)
- [https://github.com/kubernetes/kubectl/issues/276](https://github.com/kubernetes/kubectl/issues/276)
- [http://rancher.com/docs/rancher/v2.0/en/quick-start-guide/](http://rancher.com/docs/rancher/v2.0/en/quick-start-guide/)
- [https://kubernetes.io/docs/getting-started-guides/ubuntu/installation/](https://kubernetes.io/docs/getting-started-guides/ubuntu/installation/)
- [https://github.com/rancher/rancher](https://github.com/rancher/rancher)
- [https://hub.docker.com/r/rancher/server/tags/](https://hub.docker.com/r/rancher/server/tags/)
- [https://github.com/juju-solutions/bundle-canonical-kubernetes](https://github.com/juju-solutions/bundle-canonical-kubernetes)
- [https://github.com/juju-solutions/bundle-canonical-kubernetes/wiki/Authorization-Mode-and-RBAC](https://github.com/juju-solutions/bundle-canonical-kubernetes/wiki/Authorization-Mode-and-RBAC)
