---
title: "Persistent Volume and Storage Class"
keywords: "storage, volume, pv, pvc, storage class, csi, Ceph RBD, Glusterfs, QingCloud, "
description: "Persistent Volume and Storage Class Management"

linkTitle: "Persistent Volume and Storage Class"
weight: 100
---
## Introduction
This section describes how to manage storage classes and persistent volumes in KubeSphere.

A PersistentVolume (PV) is a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes. PVs are volume plugins like Volumes, but have a lifecycle independent of any individual Pod that uses the PV.

A PersistentVolumeClaim (PVC) is a request for storage by a user. It is similar to a Pod. Pods consume node resources and PVCs consume PV resources.

There are two ways PVs may be provisioned: [statically](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#static) or [dynamically](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#dynamic).

KubeSphere supports [dynamic volume provisioning](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/) 
based on storage class to create PVs.

A [StorageClass](https://kubernetes.io/docs/concepts/storage/storage-classes) provides a way for administrators to describe the "classes" of storage they offer. Different classes might map to quality-of-service levels, or to backup policies, or to arbitrary policies determined by the cluster administrators. Each StorageClass has a provisioner that determines what volume plugin is used for provisioning PVs. This field must be specified. For which value to use, please read [here](https://kubernetes.io/docs/concepts/storage/storage-classes/#provisioner) or check with your storage administrator.

Below table summaries the common volume plugins for various provisioners(storage systems).
| Type | Description |
| :---- | :---- |  
| In-tree | Built-in and run as part of Kubernetes, like [RBD](https://kubernetes.io/docs/concepts/storage/storage-classes/#ceph-rbd) & [Glusterfs](https://kubernetes.io/docs/concepts/storage/storage-classes/#glusterfs) and [more](https://kubernetes.io/docs/concepts/storage/storage-classes/#provisioner)| 
| External-provisioner| Deployed independently from Kubernetes, but work like in-tree plugin, like [nfs-client](https://github.com/kubernetes-retired/external-storage/tree/master/nfs-client) and [more](https://github.com/kubernetes-retired/external-storage) | 
| CSI | Container Storage Interface, a standard for exposing storage resources to workloads on COs like Kubernetes, like [QingCloud-csi](https://github.com/yunify/qingcloud-csi) & [Ceph-CSI](https://github.com/ceph/ceph-csi) and [more](https://kubernetes-csi.github.io/docs/drivers.html)|                          

## Manage Storage Class
You can list, create, update and delete a storage class in KubeSphere console via the `Storage` -> `Storage Classes` panel.

KubeSphere supports you to create storage classes for `QingCloud-CSI`, `Glusterfs` and `Ceph RBD` directly, you can also create custom storage classes for other storage systems by your needs.

![Create Storage Class: Basic Info](/images/storage/create-storage-class-basic-info.png)

![Create Storage Class: Storage System](/images/storage/create-storage-class-storage-system.png)

![Create Storage Class: Settings](/images/storage/create-storage-class-settings.png)

### Common Settings
Some settings are commonly used and shared among storage classes. 
You can find them as dashboard properties on the console, which are also indicated by fields or annotations in the StorageClass manifest. 
You can see the manifest file in YAML format by enabling **Edit Mode** in the top right corner. 
Here are property descriptions of some commonly used fields in KubeSphere.

| Setting | Description |
| :---- | :---- |  
| Allow Volume Expansion | Filed [`allowVolumeExpansion`](https://kubernetes.io/docs/concepts/storage/storage-classes/#allow-volume-expansion). When set to true, PVs can be configured to be expandable. | 
| Reclaiming Policy | Filed [`reclaimPolicy`](https://kubernetes.io/docs/concepts/storage/storage-classes/#reclaim-policy). It could be set as `Delete` or `Retain` (default) | 
| Storage System | Filed [`provisioner`](https://kubernetes.io/docs/concepts/storage/storage-classes/#provisioner). It determines what volume plugin is used for provisioning PVs| 
| Supported Access Mode | Annotation `metadata.annotations[storageclass.kubesphere.io/supported-access-modes]`.It tells KubeSphere which [`Access Mode`](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes) supported| 

Other settings are always customized for the special plugin which are always defined in the field of [`parameter`](https://kubernetes.io/docs/concepts/storage/storage-classes/#parameters) in storage class.
They will be described in the detailed section of volume plugin.

### QingCloud CSI
QingCloud CSI is a CSI plugin on Kubernetes for the disk of QingCloud. Storage classes of QingCloud CSI could be created on KubeSphere console.

#### Prerequisite
- QingCloud CSI can be used on both public cloud and private cloud of QingCloud. Therefore, make sure KubeSphere has been installed on either of them so that you can use cloud disks.
- QingCloud CSI Plugin has been installed on the KubeSphere cluster. See [QingCloud-CSI Installation](https://github.com/yunify/qingcloud-csi#installation) for more information.

#### Setting
| Setting | Description |
| :---- | :---- |  
| type     | In QingCloud platform, 0 represents high performance volume. 2 represents high capacity volume. 3 represents super high performance volume. 5 represents Enterprise Server SAN. 6 represent NeonSan HDD. 100 represents standard volume. 200 represents enterprise SSD | 
| maxSize  | Volume size upper limit | 
| stepSize | Volume size increment | 
| minSize  | Volume size lower limit | 
| fsType   | Filesystem type of the volume: ext3, ext4 (default), xfs | 
| tags     | The ID of QingCloud Tag resource, split by commas | 
 
More storage class Parameters could be seen in [QingCloud-CSI user guide](https://github.com/yunify/qingcloud-csi/blob/master/docs/user-guide.md#set-storage-class).

### Glusterfs
Glusterfs is an in-tree storage plugin on Kubernetes, that means you don't need to install volume plugin additionally. 
The Glustefs storage system must be installed before creating storage classes for it.

#### Prerequisite
- Glusterfs storage system has already been installed. See [Glusterfs Installation Documentation](https://www.gluster.org/install/) for more information.

#### Setting
| Setting | Description |
| :---- | :---- |  
| resturl  | Gluster REST service/Heketi service url which provision gluster volumes on demand | 
| clusterid | The ID of the cluster which will be used by Heketi when provisioning the volume | 
| restauthenabled | Gluster REST service authentication boolean that enables authentication to the REST server | 
| restuser | Glusterfs REST service/Heketi user who has access to create volumes in the Glusterfs Trusted Pool | 
| secretNamespace, secretName | Identification of Secret instance that contains user password to use when talking to Gluster REST service| 
| gidMin, gidMax | The minimum and maximum value of GID range for the StorageClass| 
| volumetype |  The volume type and its parameters can be configured with this optional value | 

For more information about StorageClass parameters, see [Glusterfs in Kubernetes Documentation](https://kubernetes.io/docs/concepts/storage/storage-classes/#glusterfs).

### Ceph RBD
Ceph RBD is also an in-tree storage plugin on Kubernetes. Volume plugin is already in Kubernetes, 
but storage server should be installed ahead for creating storage classes of Ceph RBD.

As **hyperkube** images were [deprecated since 1.17](https://github.com/kubernetes/kubernetes/pull/85094), in-tree Ceph RBD may not work without **hyperkube**.
Nevertheless, you can use [rbd provisioner](https://github.com/kubernetes-incubator/external-storage/tree/master/ceph/rbd) as a substitute, 
whose format is the same as in-tree Ceph RBD. The only different parameter is *provisioner* (i.e **Storage System** in KubeSphere console). 
If you want to use rbd provisioner, the value of *provisioner* should be **ceph.com/rbd** (Input this value in **Storage System** in the image below).
If you use in-tree Ceph RBD, the value should be *kubernetes.io/rbd*. 

![Storage System](/images/storage/storage-system.png)

#### Prerequisite
- Ceph server has already installed. See [Ceph Installation Documentation](https://docs.ceph.com/en/latest/install/) for more information.
- Install plugin if you choose to use *rbd provisioner*. Community developers provide [charts for rbd provisioner](https://github.com/kubesphere/helm-charts/tree/master/src/test/rbd-provisioner)  that you cloud use to install *rbd-provisioner* by helm. 

#### Setting
| Setting | Description |
| :---- | :---- |  
| monitors| Ceph monitors, comma delimited | 
| adminId| Ceph client ID that is capable of creating images in the pool | 
| adminSecretName| Secret Name for *adminId* | 
| adminSecretNamespace|  The namespace for *adminSecretName* | 
| pool | Ceph RBD pool | 
| userId | Ceph client ID that is used to map the RBD image | 
| userSecretName | The name of Ceph Secret for userId to map RBD image | 
| userSecretNamespace | The namespace for *userSecretName* | 
| fsType | fsType that is supported by kubernetes | 
| imageFormat | Ceph RBD image format, "1" or "2" | 
| imageFeatures| This parameter is optional and should only be used if you set imageFormat to “2”| 

For more information about StorageClass parameters, see [Ceph RBD in Kubernetes Documentation](https://kubernetes.io/docs/concepts/storage/storage-classes/#ceph-rbd).

### Custom Storage Class
You can create custom storage classes for your storage systems if they are not directly supported by KubeSphere.

The following example shows you how to create a storage class for NFS in KubeSphere console.

#### NFS Introduction
NFS (Net File System) is widely used on Kubernetes with the external-provisioner volume plugin 
[nfs-client](https://github.com/kubernetes-retired/external-storage/tree/master/nfs-client).
You can create storage classes of nfs-client by the way of custom storage class.

![Create Custom Storage Class](/images/storage/create-storage-class-storage-system-custom.png)

#### Prerequisite
- A available NFS server 
- Volume plugin nfs-client has already installed. Community developers provide [charts for nfs-client](https://github.com/kubesphere/helm-charts/tree/master/src/main/nfs-client-provisioner) that you cloud use to install *nfs-client* by helm. 

#### Common Setting
| Setting | Description |
| :---- | :---- |  
| Storage System | Provisioner of the plugin. If installed by [charts for nfs-client](https://github.com/kubesphere/helm-charts/tree/master/src/main/nfs-client-provisioner), it would be `cluster.local/nfs-client-nfs-client-provisioner`  | 
| Allow Volume Expansion | `No` | 
| Reclaiming Policy | `Delete` | 
| Supported Access Mode | `ReadWriteOnce` & `ReadOnlyMany` & `ReadWriteMany` | 

#### Parameters
| Key| Description | Value |
| :---- | :---- |  :----| 
| archiveOnDelete | Archive pvc when deleting | `true` | 

## Manage Volumes
Once the storage class was created, you can create volumes with it.

You can list, create, update and delete volumes in KubeSphere console via the `Storage` -> `Volumes` panel. For more details, please see [Volume Management](/docs/project-user-guide/storage/volumes/).

