# Canonical Kubernetes with NetApp Tridnet for PersistantStorage

This documentation explains how to integrate Canonical Kubernetes with NetApp Trident as a mechanism to provide PersistantStorage to your cluster.

### Index

## Deploying Canonical Kubernetes

This documentation already assumes that you have Canonical Kubernetes up and running. Right now you can also do this on public clouds such as AWS if required for testing. You can deploy CDK using Juju or Conjure-up.

If you need the instructions for deploying CDK, they can be found here: [https://kubernetes.io/docs/getting-started-guides/ubuntu/installation/](https://kubernetes.io/docs/getting-started-guides/ubuntu/installation/) and here [https://jujucharms.com/canonical-kubernetes/](https://jujucharms.com/canonical-kubernetes/). There are also instructions in the Canonical Kubernetes Demos repository for deploying CDK on AWS or Azure.

## Deploying NetApp OnTap on AWS

For the purposes of this documentation, the integration has been tested using NetApp OnTap through AWS. However, the instructions for deploying Trident with OnTap should be very similar for physical devices with SolidFire (Element), ONTAP (AFF/FAS/Select/Cloud) and SANtricity (E/EF-Series).

I assume you are already familiar with AWS and have an account. First sign into the console and pick the same region where your Kubernetes Cluster resides. In my case I am using eu-west-1 which is the Ireland region. Go to AWS and then EC2, hit Launch Instance and in the market place find the NetApp OnCommand Cloud Manager (for ONTAP Cloud). Hit select and use all the default values and deploy it, you may have to accept the terms and conditions of NetApp OnTAP.

![netapp cloud manager provisioning](https://raw.githubusercontent.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/master/cdk-netapp-trident/images/NetApp-OnCommand-CloudManager.png "Provisiong NetApp Cloud Manager")

To deploy OnTAP we will use the NetApp OnCommand Cloud Manager which simplifies the management of NetApp OnTAP and other storage products on public cloud. I believe you can also just deploy OnTAP devices and configure them manually using SSH or scripts if you are familiar with the OnTAP commands. Once the cloud manager is deployed, you can access the web interface using the public IP address:

![netapp cloud manager ip](https://raw.githubusercontent.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/master/cdk-netapp-trident/images/NetApp-OnCommand-CloudManager-Provisioned.png "NetApp Cloud Manager Public IP")

When you hit the public IP address in a browser you should be greeted with an interface which looks like this. If it asks you to sign up, create an account and continue from there:

![netapp cloud manager interface](https://raw.githubusercontent.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/master/cdk-netapp-trident/images/NetApp-OnCommand-CloudManagerGUI-Loading.png "NetApp Cloud Manager Interface")

![netapp cloud manager gui](https://raw.githubusercontent.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/master/cdk-netapp-trident/images/NetApp-OnCommand-CloudManager-GUI.png "NetApp Cloud Manager GUI")

Next we put in a site name, I used the value cdk, but you could use something else. Type in a name and then hit 'Lets Start' this will cause the system to do some more loading. Eventually you should end up with an interface that looks like this:

![netapp cloud manager ready](https://raw.githubusercontent.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/master/cdk-netapp-trident/images/NetApp-OnCommand-CloudManagerGUI-ready.png "NetApp Cloud Manager Interface")

Now we will hit the Create button. The cloud manager will automatically create and configure an OnTAP instance for us to use with Kubernetes. This will require you to generate an AWS Secret Key and Access Key or use an existing one. The credentials can be found within the IAM section of AWS:

![netapp cloud manager create key](https://raw.githubusercontent.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/master/cdk-netapp-trident/images/NetApp-OnCommand-CloudManagerGUI-Create-Key.png "NetApp Cloud Create Key")

Put in your details and hit the button to show that you have verified the IAM policy is associated with your user. If you have errors with your IAM credentials it is likely that the account you hvae used does not have the correct permissions to provision the OnTAP storage devices. Now we put in storage details, I used the Working Environment Name CDK (cluster name) and the username admin with password Kubernetes123. I would recommend setting a strong username/password. After that, hit next:

![netapp cloud user](https://raw.githubusercontent.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/master/cdk-netapp-trident/images/NetApp-OnCommand-CloudManagerGUI-credentials.png "NetApp Cloud Create User")

Next we select a location. My cluster is in eu-west-1, Ireland, so I will select that. Make sure you select the same subnet as your kubernetes cluster so your instances have network connectivity to the OnTAP instances you deploy. Also make sure you use an SSH Key Pair on your account you have full access to use, as we will need to jump onto the OnTAP instance to check some information on the device after deployment:

![netapp cloud location](https://raw.githubusercontent.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/master/cdk-netapp-trident/images/NetApp-OnCommand-CloudManagerGUI-Location.png "NetApp Cloud Create Location")

The next option allows you to enable encryption out of the box for your storage. This is quite useful, especially if you need to meet GDPR requirements. However, for this configuration I do not enable the encyrption, so I selected the None option:

![netapp cloud encryption](https://raw.githubusercontent.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/master/cdk-netapp-trident/images/NetApp-OnCommand-CloudManagerGUI-Encryption.png "NetApp Cloud Create Encryption")

The next stage asks you if you'd like to use your existing OnTAP Licence with the cloud manager. I do not have one of these so I hit the No option, you may be able to get a trial licence from NetApp for this. The next section asks you which package you'd like to configure. For this documentation I use the POC/Small workloads package, but for production you may want to consider one of the other package types. Depending on your workload you should choose a package, for example the highest performance production workloads package has high performance and can handle up to 36TB of storage. It is also possible to create your own package based on your own requirements:

![netapp cloud licence](https://raw.githubusercontent.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/master/cdk-netapp-trident/images/NetApp-OnCommand-CloudManager-Licence.png "NetApp Cloud Create Licence")

The next section asks you about support credentials. If you don't have any you can skip this section and leave the options blank. The next section is where you create the volume. You can set the details of the volume, its name, size, protocol, you can even set thin provisioning, deduplication and compression here. I gave the volume the name cdkontap with a size of 200GB and the protocol NFS, the other options I left to the default options:

![netapp cloud Volume](https://raw.githubusercontent.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/master/cdk-netapp-trident/images/NetApp-OnCommand-CloudManagerGUI-CreateVolume.png "NetApp Cloud Create Volume")

The next screen will give you an Overview of all the options you have entered, once you are happy, hit the GO button to start the provisioning of the storage. Make sure you tick both of the boxes before you press it.

![netapp cloud Overview](https://raw.githubusercontent.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/master/cdk-netapp-trident/images/NetApp-OnCommand-CloudManagerGUI-OverView.png "NetApp Cloud Overview")

The next stage will take up to 25 minutes as it provisions and automatically configures the OnTAP storage for us, so go grab a coffee and come back:

![netapp cloud Deploying](https://raw.githubusercontent.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/master/cdk-netapp-trident/images/NetApp-OnCommand-CloudManagerGUI-Deploying.png "NetApp Cloud Deploying OnTAP")

While your waiting, you can move onto the Deploying NetApp Trident section in preperation for integrating Canonical Kubernetes and the storage. Eventually you should side a screen like this:

![netapp cloud wkenv](https://raw.githubusercontent.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/master/cdk-netapp-trident/images/NetApp-OnCommand-CloudManagerGUI-wkenv.png "NetApp Cloud Working Environment")

## Accessing your NetApp OnTap Instance

Your storage should now be accessible but by default it is deployed onto a private IP range, you will need to SSH into the storage using an existing machine which has an interface on that network or temporarily expose the management interface to the internet using a public IP address. Be careful when doing this, it is not a good idea to directly expose your services to the internet, especially if they contain production data. This could potentially breach GDPR.

If you want to assign a public IP to your newly provisioned storage, head on over to AWS, go to EC2 and find the region you provisioned your Kubernetes cluster and storage into. Once your storage has been deployed, you should see a machine named after whatever you called the storage at the start of these steps. For me, its called cdk:

![netapp accessing ontap](https://raw.githubusercontent.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/master/cdk-netapp-trident/images/NetApp-accessing-OnTap.png "NetApp Accessing OnTAP")

If you right click the instance with the name of your storage backend, hit Networking and then go to Manage IP addresses. From here, you can see that the instance has two networking interfaces. One is for Management of the storage which we will connect to through SSH, the other interface is used for Cluster Management, Inter-cluster Communication and for the Data Network, but it has 4 IP addresses and we are not sure which one is used for which purpose.

To find out which one we should use, we will SSH to the instance and use the NetApp CLI tool to interact with it. You can either SSH onto a node on the same network, I.E:

```
 juju ssh kubernetes-worker/0
```

or assign a public interface described. To do this, create a new Elastic IP and assign it to the first interface on the storage machine you created. You will need to create a new Elastic IP on the left-hand side of EC2 and then assign it to the correct interface, you can find this from the network interfaces section as previously discussed. For me it was the following interface:

```
eth0: eni-c3cb92c7 - Interface for Node Management - 172.31.0.0/20
```

During a default deploy, OnTAP has several interfaces with different IP addresses for different purposes. They are explained quite well in the [manual here](http://docs.netapp.com/occm/index.jsp?topic=%2Fcom.netapp.doc.onc-cloud-mgr-concepts-320%2FGUID-2185A005-16F6-44A6-90D0-B95267289C09.html). Once you are able to SSH to the machine, you should be created with the NetApp OnTap CLI:

```
calvinh@ubuntu-ws:~/Source/canonical-kubernetes-third-party-integrations/cdk-netapp-trident$ ssh admin@34.241.55.211
The authenticity of host '34.241.55.211 (34.241.55.211)' can't be established.
ECDSA key fingerprint is SHA256:wrSxiXsXc3WW7f8s1JbD0FqmLs5zJ9uuK2fpvidjq/I.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '34.241.55.211' (ECDSA) to the list of known hosts.
cdk::>
cdk::>
cdk::>
cdk::> ?
  application>                Display and manage applications
  cluster>                    Manage clusters
  event>                      Manage system events
  exit                        Quit the CLI session
  history                     Show the history of commands for this CLI session
  job>                        Manage jobs and job schedules
  lun>                        Manage LUNs
  man                         Display the on-line manual pages
  metrocluster>               Manage MetroCluster
  network>                    Manage physical and virtual network connections
  qos>                        QoS settings
  redo                        Execute a previous command
  rows                        Show/Set the rows for this CLI session
  run                         Run interactive or non-interactive commands in the nodeshell
  security>                   The security directory
  set                         Display/Set CLI session settings
  snaplock>                   Manages SnapLock attributes in the system
  snapmirror>                 Manage SnapMirror
  statistics>                 Display operational statistics
  statistics-v1>              The statistics-v1 directory
  storage>                    Manage physical storage, including disks, aggregates, and failover
  system>                     The system directory
  top                         Go to the top-level directory
  up                          Go up one directory
  volume>                     Manage virtual storage, including volumes, snapshots, and mirrors
  vserver>                    Manage Vservers
```

Using this CLI tool, we can check the which interface is used for management LIF and data LIF which is required for the next stage:

```
cdk::> network

cdk::network> interface

cdk::network interface> show
            Logical    Status     Network            Current       Current Is
Vserver     Interface  Admin/Oper Address/Mask       Node          Port    Home
----------- ---------- ---------- ------------------ ------------- ------- ----
cdk
            cdk-01_mgmt1 up/up    172.31.8.21/20     cdk-01        e0a     true
            cluster-mgmt up/up    172.31.6.15/20     cdk-01        e0b     true
            intercluster up/up    172.31.12.133/20   cdk-01        e0b     true
svm_cdk
            iscsi        up/up    172.31.15.137/20   cdk-01        e0b     true
            svm_cdk_data_lif
                         up/up    172.31.9.246/20    cdk-01        e0b     true
            svm_cdk_mgmt_lif
                         up/up    172.31.15.48/20    cdk-01        e0b     true
6 entries were displayed.
```

The management interface is marked cdk-01_mgmt, with IP 172.31.8.21 and the data lif is marked too, with the IP address 172.31.9.246. Finally we can also use the CLI to check the svm name:

```
cdk::> volume

cdk::volume> show
Vserver   Volume       Aggregate    State      Type       Size  Available Used%
--------- ------------ ------------ ---------- ---- ---------- ---------- -----
cdk-01    vol0         aggr0        online     RW      69.72GB    65.57GB    5%
svm_cdk   cdkontap     aggr1        online     RW        200GB    190.0GB    5%
svm_cdk   svm_cdk_root aggr1        online     RW          1GB    972.5MB    5%
3 entries were displayed.
```

## Deploying NetApp Trident

Now we've provisioned our volume and we have a CDK cluster, we can start to use Trident. Trident allows you to configure and consume storage with Kubernetes in a similar way to helm. By default the Netapp CloudManager will provision our storage on the private networks, so it will not be internet accessible. Therefore, we must deploy Trident onto one of the nodes in our cluster. Using Juju, we can SSH to one of the kubernetes-workers or the kubernetes-master server to utilise Trident:

```
 juju ssh kubernetes-worker/0
```

Trident itself is shipped as a cli tool. To install the latest version, grab the latest release from their github page here: [https://github.com/NetApp/trident/releases/latest](https://github.com/NetApp/trident/releases/latest) or use the command below:

```
  # Download the latest release of trident, replace the URL from the release page
  wget -O trident-installer.tar.gz https://github.com/NetApp/trident/releases/download/v18.04.0/trident-installer-18.04.0.tar.gz
  # and extract it
  tar -xvf trident-installer.tar.gz
```

Once extracted, we are ready to utilise Trident. If you get stuck doing this, I would recommend reading the Trident manual in the Useful links section at the end of this document. Using your console, go into the Trident directory, the most important file inside here is the tridentctl command line tool, this tool is used to both install trident and manage it. Before we install it, we need to create a yaml file and place it into the setup directory. This file contains information about our storage mechanism which trident uses to configure itself.

The file we need to create is called backend.json, an example is included inside this repository, but here is its contents:

```
{
    "version": 1,
    "storageDriverName": "ontap-nas",
    "backendName": "cdk",
    "managementLIF": "172.31.8.21",
    "dataLIF": "172.31.9.246",
    "svm": "svm_cdk",
    "username": "admin",
    "password": "Kubernetes123"
}
```

This file is based on one of the examples provided with Trident itself. Inside the directory is a folder called sample-input. This includes all a bunch of sample files for configuring things like PVC/PV, storage classes and the various forms of storage Trident can utilise, like OnTap. The yaml above is based on the file backend-ontap-nas.json file.

The Trident Manual provides an excellent explanation of all of these fields within the yaml file [here](https://netapp-trident.readthedocs.io/en/stable-v18.04/kubernetes/operations/tasks/backends/ontap.html). However, lets examine the ones in this file and I will explain where the values come from:

- Version: at the moment, this value should always be 1, I assume this is for future version compatibility.
- storageDriverName: this describes the type of ontap-nas you wish to use.
- backendName: this is the name we set earlier for the working environment for me it is cdk, in lowercase.
- managementLIF: an IP address of a logical interface for management of the storage
- dataLIF: an IP address of a logical interface for data
- svm: the name of our storage virtual machine, in this case it is svm_ appended to the backendName. We can also check this by SSHing into the OnTap device.
- username: this is admin by default
- password: this is the password we set earlier

The managementLIF should be the IP of the first interface on the OnTAP instance, the same IP you use to SSH to the instance in order to use the CLI. The Data LIF can be found using the CLI tool but it generally seems to be the third IP address on the second interface. We can confirm this using the steps in the previous section.

Once we have these values, we can create the file backend.json in the setup directory within the trident installer directory. Next, run the tridentcli tool:

```
ubuntu@ip-172-31-20-236:~/trident-installer$ ./tridentctl install
WARN For maximum security, we recommend running Trident in its own namespace.  example="./tridentctl install -n trident"
INFO Starting storage driver.                      backend=/home/ubuntu/trident-installer/setup/backend.json
INFO Controller serial numbers.                    serialNumbers=90283656803828725674
WARN Aggregate has unknown media type.             aggregate=aggr1 mediaType=vmdisk
INFO Storage driver loaded.                        driver=ontap-nas
INFO Starting Trident installation.                namespace=default
INFO Created service account.                     
INFO Created cluster role.                        
INFO Created cluster role binding.                
INFO Created PVC.                                 
INFO Created PV.                                   pv=trident
INFO Waiting for PVC to be bound.                  pvc=trident
INFO Created Trident deployment.       
```

Sometimes the install will hang because the containers cannot mount the newly created NFS volume it created because it is not exposed by NFS by default. In order to fix this, go back to the NetApp cloud manager web interface, go to Workign Environments -> your environment (in my case, CDK). You should see who new volumes here, one called trident_trident.

![netapp missing env issue](https://raw.githubusercontent.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/master/cdk-netapp-trident/images/NetApp-OnCommand-CloudManagerGUI-wkenv-NOMOUNT.png "NetApp No NFS Issue")

You can see in the picture below that the Mount Command is darked out and unavailable, this is because NetApp trident has created the storage but not actually exposed it by default through NFS, so we need to adjust it:

![netapp fixing nfs](https://raw.githubusercontent.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/master/cdk-netapp-trident/images/NetApp-OnCommand-CloudManagerGUI-wkenv-fix.png "NetApp Fixing NFS)

The way to fix it is to hit the Edit button on the trident_trident volume. On the edit volume screen, change the Access Control settings by adding a custom export policy, this will automatically expose the volume through NFS. Now we can re-run the install, but first, we need to run the uninstall command:

```
ubuntu@ip-172-31-20-236:~/trident-installer$ ./tridentctl uninstall
INFO Deleted Trident deployment.                  
INFO Deleted cluster role binding.                
INFO Deleted cluster role.                        
INFO Deleted service account.                     
INFO The uninstaller did not delete the Trident's namespace, PVC, and PV in case they are going to be reused. Please use the --all option if you need the PVC and PV deleted.
INFO Trident uninstallation succeeded.      
```

And then re-run the installer:

```
ubuntu@ip-172-31-20-236:~/trident-installer$ ./tridentctl install
WARN For maximum security, we recommend running Trident in its own namespace.  example="./tridentctl install -n trident"
INFO Starting Trident installation.                namespace=default
INFO Created service account.                     
INFO Created cluster role.                        
INFO Created cluster role binding.                
INFO Created Trident deployment.                  
INFO Waiting for Trident pod to start.            
INFO Trident pod started.                          namespace=default pod=trident-5fd99d494d-ts8wx
INFO Waiting for Trident REST interface.          
INFO Trident REST interface is up.                 version=18.04.0
INFO Trident installation succeeded.    
```   

After the installation, we need to run another command to allow us to use the storage for our own containers. The first install command just setups the trident container (similar to the Tiller in Helm), the second is used to setup a backend storage mechanism for us to use with our regular containers:

```
ubuntu@ip-172-31-20-236:~/trident-installer$ ./tridentctl create backend -f setup/backend.json
+------+----------------+--------+---------+
| NAME | STORAGE DRIVER | ONLINE | VOLUMES |
+------+----------------+--------+---------+
| cdk  | ontap-nas      | true   |       0 |
```

Once this is complete, you're ready to create a storage class and consume storage using containers. The official documentation suggests that you should run both the tridentctl install command and the create backend command with the -n flag which is used to specify a namespace. This is good practice and will cause trident to remain isolated in its own namespace, I.E:

```
# deployment steps, but using a custom namespace for added security
ubuntu@ip-172-31-20-236:~/trident-installer$ ./tridentctl install -n trident
ubuntu@ip-172-31-20-236:~/trident-installer$ ./tridentctl create backend -f setup/backend.json -n trident
```

## Removing Trident

If for any reason you wish to remove Trident from your kubernetes cluster, just run the following command:

```
ubuntu@ip-172-31-20-236:~/trident-installer$ ./tridentctl uninstall
INFO Deleted Trident deployment.                  
INFO Deleted cluster role binding.                
INFO Deleted cluster role.                        
INFO Deleted service account.                     
INFO The uninstaller did not delete the Trident's namespace, PVC, and PV in case they are going to be reused. Please use the --all option if you need the PVC and PV deleted.
INFO Trident uninstallation succeeded.  
```    

## Testing our Storage on Kubernetes

Once Trident is deployed correctly and working, it is time for us to configure a default StorageClass and to try to create a PVC. When we create a PVC, a PV should be automatically created based on the StorageClass we have specified, usually this is the default storageclass. Included in this repository is a default storage class based on the OnTap instance we just created. First create this storage class:

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: default
provisioner: netapp.io/trident
parameters:
  backendType: "ontap-nas"
```

Next use kubectl to apply it:

```
ubuntu@ip-172-31-20-236:~/trident-installer$ kubectl apply -f storageclass.yaml
storageclass.storage.k8s.io "default" created

ubuntu@ip-172-31-20-236:~/trident-installer$ kubectl get storageclass
NAME      PROVISIONER         AGE
default   netapp.io/trident   1m
```

Once created, we can then create a pvc. One is is included within this repository:

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: netapp-ontap-pvc
spec:
  storageClassName: default
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```

Apply the file to the cluster:

```
ubuntu@ip-172-31-20-236:~/trident-installer$ kubectl apply -f pvc.yaml
persistentvolumeclaim "netapp-ontap-pvc" created
```

Finally, you should see the PVC has been created, along with the PV:

```
ubuntu@ip-172-31-20-236:~/trident-installer$ kubectl get pvc
NAME               STATUS    VOLUME                           CAPACITY   ACCESS MODES   STORAGECLASS   AGE
netapp-ontap-pvc   Bound     default-netapp-ontap-pvc-4372b   2Gi        RWO            default        1m
trident            Bound     trident                          2Gi        RWO                           49m

ubuntu@ip-172-31-20-236:~/trident-installer$ kubectl get pv
NAME                             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS    CLAIM                      STORAGECLASS   REASON    AGE
default-netapp-ontap-pvc-4372b   2Gi        RWO            Delete           Bound     default/netapp-ontap-pvc   default                  1m
trident                          2Gi        RWO            Retain           Bound     default/trident  
```

You should be able to see this inside the CloudManager as well:

![netapp mounted](https://raw.githubusercontent.com/CanonicalLtd/canonical-kubernetes-third-party-integrations/master/cdk-netapp-trident/images/NetApp-OnCommand-ontapmounted.png "NetApp Mounted)

If you're interested in doing some proper testing, try to deploy some containers which require PV. One example is Minio Storage, and [a demo workload exists for this here](https://github.com/CanonicalLtd/canonical-kubernetes-demos/tree/master/cdk-minio). Everytime you create a new PV it should also be created inside the NetApp cloud manager, so you can manage the volume, back it up, etc using this tool as well.  

## Known Issues and Workarounds

If you're seeing an error within the NetApp Cloud Manager that a stack already exists even though it is deleted it is most likely because it still exists in CloudFormation. Go to CloudFormation and delete the stack to remove the error:

```
An error occurred while creating working environment: Stack cdk already exists
```

You can of course delete Environments from within NetApp Cloud Manager as well which is the recommended route. Only do this as a last resort, as there may still be other things lingering around AWS created by the cloud manager.

If you need to troubleshoot Trident, the following command is quite useful:

```
./tridentctl -n trident logs
```

## Conclusion

This documentation has explained how to configure and deploy Canonical Kubernetes with NetApp OnTAP Storage to provide PersistantStorage to our Kubernetes cluster. We have deployed NetApp OnTap storage on AWS and then connected it to our cluster using NetApp Trident.

### Getting Help & Support

This document was written by [Calvin Hartwell](https://www.linkedin.com/in/calvinhartwell), please feel to drop me an email on [calvin.hartwell@canonical.com](calvin.hartwell@canonical.com).

To open issues with CDK, please open an issue with the repository here [https://github.com/juju-solutions/bundle-canonical-kubernetes](https://github.com/juju-solutions/bundle-canonical-kubernetes). If you want professional support for Kubernetes please contact Canonical Sales [https://www.ubuntu.com/kubernetes](https://www.ubuntu.com/kubernetes).

If you find issues or bugs with Trident, you should raise a support tick on the Github repository [https://github.com/NetApp/trident](https://github.com/NetApp/trident). NetApp sells support contracts for all its software and hardware, so for commercial support options it would be a good idea to speak to them directly.

## Useful Links
- [https://github.com/NetApp/trident](https://github.com/NetApp/trident)
- [http://netapp-trident.readthedocs.io/en/stable-v18.04/kubernetes/deploying.html](http://netapp-trident.readthedocs.io/en/stable-v18.04/kubernetes/deploying.html)
- [http://netapp-trident.readthedocs.io/en/stable-v18.04/](http://netapp-trident.readthedocs.io/en/stable-v18.04/)
- [https://media.readthedocs.org/pdf/netapp-trident/stable-v18.04/netapp-trident.pdf](https://media.readthedocs.org/pdf/netapp-trident/stable-v18.04/netapp-trident.pdf)
- [https://netapp.io/2016/12/23/introducing-trident-dynamic-persistent-volume-provisioner-kubernetes/](https://netapp.io/2016/12/23/introducing-trident-dynamic-persistent-volume-provisioner-kubernetes/)
- [https://netapp.io/2017/03/21/trident-part-2-installing-and-configuring-trident/](https://netapp.io/2017/03/21/trident-part-2-installing-and-configuring-trident/)
-[Using NetApp OnTAP with OpenShift and Kubernetes](http://www.dburkland.com/how-to-deploy-kubernetes-with-netapp-trident-persistent-storage/) -[Netapp Tridnet Manual with OnTAP](https://netapp-trident.readthedocs.io/en/stable-v18.04/kubernetes/operations/tasks/backends/ontap.html#ontap-nas-and-ontap-nas-economy)
- [https://www.youtube.com/watch?v=Epz3N-VFKRY](https://www.youtube.com/watch?v=Epz3N-VFKRY)
