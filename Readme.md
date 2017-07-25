---
# Introduction to BOSH

## 

### Table Of Contents

- What is BOSH
- BOSH Architecture
  - Releases
- Deploying BOSH
- BOSH CLI
- BOSH Manifest Overview
  - BOSH State
  - Cloud Config
  - Deployment Manifest
- BOSH Lifecycle Management
  - Persistent & Ephemeral Storage
- Overview of how to deploy a BOSH Release: "KUBO"


### What is BOSH

BOSH is an open source tool that enables deployment and lifecycle management of distributed system.   It is the primary method to deploy Cloud Foundry and is contributed to by many key members of the Cloud Foundry Foundation such as Google, Pivotal, & VMware.  It can support deployment across many different IaaS providers. Some of these providers are:

- VMware vSphere
- Google Compute Platform
- Amazon Web Services EC2
- Microsoft Azure
- Openstack


![BOSH-Simple](images/OmniGraffle-All-Images/BOSH-Simple.png)

BOSH accomplishes this by creating some major abstraction objects to make it easy and repeatable to deploy complex systems.

1. **CPI**:  The CPI is the executable library that BOSH uses to interact with any given IaaS.  There is a CPI available per each BOSH supported IaaS, and when you deploy the BOSH instance(s) you can define which one(s) it will use.   In the image, the vSphere CPI is shown.  It allows BOSH to perform all the required IaaS actions such as creating a VM or instance as well as various other primitives to instantiate a deployment.
2. **BOSH Stemcell**: A Stemcell is a versioned base operating system image built for each CPI that BOSH supports.  Its commonly based on Canonical's ubuntu distribution,  but is also available in RHEL and even Windows image ports.  The Stemcell typically is a hardened base OS image with a BOSH agent deployed
3. **BOSH Release**: A bosh release is a versioned tarball containing all source code as well as job definitions to describe to BOSH how that release of software should be deployed on a VM/instance based on a stemcell.  An Example is the [kubo release] (https://github.com/cloudfoundry-incubator/kubo-release) that includes all the required packages and detail to allow BOSH to deploy a fully functional Kubernetes cluster.
4. **BOSH Deployment Manifest**: BOSH needs to have some declarative information to actually deploy something.  This is provided by an operator via a manifest.  A manifest defines one or more __*releases*__ and __*stemcells*__ to be used in a deployments and provides some key variables like IPstack info, instance count, and advanced configuration of the given release you want to deploy.  This manifest is typically written in a YAML format.
5. **BOSH Deployment Manifest**: A BOSH deployment is a given instantiation of a BOSH deployment manifest.  BOSH will deploy the versioned releases & stemcells defined in the manifest yaml and will maintain the health and availability through the lifetime of the VM/instances it deploys.  Each VM/instance will be given one or more 'jobs' defined in the BOSH release and are therefore sometimes referred to as job instances. 




### BOSH Architecture

![BOSH-Components](images/OmniGraffle-All-Images/BOSH-Components.png)

BOSH is typically deployed as a single VM or Instance.  That VM/instance has many components that perform vital roles in how BOSH manages deployments at scale:

- **NATS**: Provides a message bus for the various services of BOSH to interact
- **POSTGRESQL**:  BOSH writes all of its state into a database.  Typically that database is internal to a single VM BOSH deployment and provided by Postgres.  This can however be modified to use an external data source so that  BOSH VM could be rebuilt and reconnect to the database to reload its persistent state.
- **BLOBSTORE**: Each stemcell and release uploaded to BOSH is stored in a blobstore.  Default deployments of BOSH use an internal store (webdav), but like the Postgresql database,  this can also be externalized.
- **Director**: Is the main API that the BOSH CLI will interface with to allow an operator to create and manage BOSH deployments.
- **Health Monitor**:  BOSH requires that each VM it deploys have an agent that it can communicate with to assign and deploy jobs from BOSH releases defined in a deployment manifest. It will also maintain the health of each VM/instance it has deployed.  The agent will report vitals back to BOSH and in cases where Services are faulted or the agent is unreachable, the HealthMonitor can use plugins to restart services and even rebuild VMs/instances.
- **CPI**:  The CPI is the executable library that BOSH uses to interact with any given IaaS
- **UAA**: Or User Access & Authentication allows BOSH to authenticate operators via SAML or LDAP backends
- **CREDHUB**: Manages credentials like passwords, certificates, certificate authorities, ssh keys, rsa keys and arbitrary values (strings and JSON blobs).  BOSH will leverage credhub to create and store key credentials for deployments, like public certificates & keys.



