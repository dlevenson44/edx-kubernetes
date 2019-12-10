# Chapter 11: Kubernetes Volume Management
- Volumes: directory backed by a storage medium-- storage medium, content, and access mode are determined by volume type
- Data stored in a container is deleted if it crashes, and `kubelet` will restart it with a clean start, which means it won't have any of the old data
-- Volumes corrects this issue and allows us to retrieve deleted data (check back on this)
- Directory mounted in a Pod is backed by the underlying Volume Type
- Volume Type decides properties of directory, such as size, content, default access mode etc... Volume Types include:
1. `emptyDir`: created for Pod as soon as it's scheduled on worker node-- volume's life is coupled with the Pod-- if Pod is deleted, so is `emptyDir` forever
2. `hostPath`: allows us to share a directory from the host to the Pod-- if Pod is terminated, content of Volume is still available on the host
3. `gcePersistentDisk`: can mount a Google Compute Engine(GCE) persistent disk into a Pod
4. `awsElasticBlockStore`: can mount AWS EBS Volume into a Pod
5. `azureDisk`: can mount an MSFT Azure Data Disk into a Pod
6. `azureFile`: can mount a MSFT Azure File Volume into a Pod
7. `cephfs`: an existing CephFS volume can be mounted into a Pod-- when Pod terminates, volume is unmounted and its contents are preserved
8. `nfs`: can mount an NFS share into a pod
9. `iscsi`: can mount an iSCSI into a Pod
10. `secret`: can pass sensitive info, such as passwords, to Pods
11. `configMap`: objects that provide config data, or shell commands and arguments, into a Pod
12. `persistentVolumeClaim`: can attach PersistentVolume to a pod


- PersistentVolume: subsystem that provides APIs to users and admins to manage and consume persistent storage
- it uses PerssitentVolume API resource type to manage Volume-- uses PersistentVolumeClaim API resource type to consume it
- In containerized world, storage is managed by storage/sys admins-- EU receives instructions to use storage
- PersistentVolumes helps us follow these rules accross the many VolumeTypes
- PV is network-attached storage in cluster, provisioned by admin
-- Can be dynamically provisioned based on StorageClass resource
- StorageClass contains pre-defined provisioners and parameters to create a PV... using PersistentVolumeClaims, user sends request for dynamic PV creation, which gets wired to StorageClass
- Some VolumeTypes that support managing storage using PV are GCEPersistentDisk, AWSElasticBlockStore, AzureFile, AzureDisk, CephFS, NFS, iSCSI

- PersistentVolumeClaim: request for storage by a user-- requested based on type, access mode, and size
- Three access modes are ReadWriteOnce(read-write by a single node), ReadOnlyMany(read-only by many nodes), and ReadWRiteMany (read-write many nodes)
- Once suitable PV is found, it is bound to a PVC... after successful bound, PVC resource can be used in a Pod
- Once user finishes work, attached PV can be released-- underlying PV can be reclaimed (for admin to verify/aggregate data), deleted (both data and volume are deleted), or recycled for future usage
- Container Storage Interface (CSI) makes installing new CSI-compliant volume plugins easier
