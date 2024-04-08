# WIP
# Remote worker nodes HA configuration

## Introduction
The purpose of this repository is to demonstrate a potential solution to provide application level HA between two nodes in an OpenShift cluster.  Several telco application vendors and customers have expressed interest in this setup to provide highly available services in remote locations.  

## Architecture
TODO: Add arch diagram
This solution will work with either a Hypershift(hosted control plane) + RWN or SNO + worker configuration for the OCP cluster.  

DRBD is used to replicate data between the two nodes.  DRBD is installed and configured via the Piraeus operator.  By default, only RWO volumes are supported.  In order to support RWX volumes for multi-tier applications, a NFS Server pod is created to expose the RWO volume in RWX mode.  

## Installation and Usage
1.  Build drbd-module-loader image and push to registry.
2.  Install Piraeus Operator and configure DRBD service
3.  Create NFS Server pods and service
4.  Set up example application