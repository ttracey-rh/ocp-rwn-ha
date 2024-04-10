# WIP
# Remote worker nodes HA configuration

## Introduction
The purpose of this repository is to demonstrate a potential solution to provide application level HA between two nodes in an OpenShift cluster.  Several telco application vendors and customers have expressed interest in this setup to provide highly available services in remote locations.  

## Architecture
TODO: Add arch diagram

This solution utilizes Hypershift(hosted control plane) and two Remote Worker Nodes for the OCP cluster.  A SNO + Worker architecture is under development. 

DRBD is used to replicate data between the two nodes.  DRBD is installed and configured via the [Piraeus operator](https://github.com/piraeusdatastore/piraeus-operator/tree/v2).  By default, only RWO volumes are supported.  In order to support RWX volumes for multi-tier applications, a NFS Server pod is created to expose the RWO volume in RWX mode.  

Application pods will connect to a statically defined NFS-backed PV that utilizes the IP of the NFS Kubernetes service.  

## Installation and Usage
### 1.  Build drbd-module-loader image and push to registry.
- The DRBD Kernel module is built and installed by a `drbd-module-loader` init container in the `linstor-satellite-*` pods.  
- The operator provided drbd-module-loader image does not work with RHCOS nativley and must be built before setting up the operator.  
- A Dockerfile and the necessary installation script are provided in the `00-drbd-module-image` directory.
- Build the image using podman and push to a registry that is accessible from the OCP cluster.  NOTE: The image must be built on a properly subscribed RHEL system due to dependencies not being available in the standard UBI repositories.
### 2.  Install Piraeus Operator and configure DRBD service
- Utilize the `operator_install.sh` script to install the Piraeus operator.  Some operator pods will not start properly due to their default configurations being incorrect for OCP.  
- Two customizations are required to the the `02-satellite-config.yaml` file:
      - Edit the `image:` field for the `kernel-header-copy` init container.  Obtain the proper image ID for the specific version of OCP in your cluster by executing `oc adm release info --image-for=driver-toolkit`.  
      - Edit the `image:` field for the `drbd-module-loader` init container.  Use the image URL of the `drbd-module-loader` image created earlier.  
- Apply the `01-linstor-cluster.yaml`, `02-satellite-config.yaml` and `03-storage-class.yaml` to the OCP cluster.  All operator pods should now be running.  
### 3.  Create NFS Server pods and service
### 4.  Set up example application