---

# Introduction to BOSH


### Table Of Contents

- What is BOSH
  - Overview
  - What Problems does BOSH solve?
  - Use Case(s)
- Deploying BOSH
  - BOSH Architecture
  - BOSH References
  - CookBook: how to deploy "KUBO"


### What is BOSH


#### BOSH Overview 

BOSH is an open source tool that enables deployment and lifecycle management of distributed systems.   It is the primary method to deploy Cloud Foundry and is contributed to by many key members of the Cloud Foundry Foundation such as Google, Pivotal, & VMware.  It can support deployments across many different IaaS providers. Some of these providers are:

- VMware vSphere
- Google Compute Platform
- Amazon Web Services EC2
- Microsoft Azure
- Openstack


![BOSH-Simple](images/OmniGraffle-All-Images/BOSH-Simple.png)

BOSH accomplishes deployments by creating some major abstraction objects to make it easy and repeatable to deploy complex systems.

1. **CPI**:  The CPI is the executable library that BOSH uses to interact with any given IaaS.  There is a CPI available per each BOSH supported IaaS, and when you deploy the BOSH instance(s) you can define which one(s) it will use.   In the image above, the vSphere CPI is shown.  It allows BOSH to perform all the required IaaS actions such as creating a VM or instance as well as various other instance, network, and storage primitives required to instantiate a deployment.
2. **BOSH Stemcell**: A Stemcell is a versioned base operating system image built for each CPI that BOSH supports.  Its commonly based on Canonical's ubuntu distribution,  but is also available in RHEL and even Windows image ports.  The Stemcell typically is a hardened base OS image with a BOSH agent pre deployed.  BOSH will use this agent to install and manage the lifecycle of software on that VM or instance.
3. **BOSH Release**: A bosh release is a versioned tarball containing all source code as well as job definitions to describe to BOSH how that release of software should be deployed on a VM or instance provisioned from a stemcell.  An Example is the [kubo release] (https://github.com/cloudfoundry-incubator/kubo-release) that includes all the required packages and detail to allow BOSH to deploy a fully functional Kubernetes cluster.
4. **BOSH Deployment Manifest**: BOSH needs to have some declarative information to actually deploy something.  This is provided by an operator via a manifest.  A manifest defines one or more __*releases*__ and __*stemcells*__ to be used in a deployment and provides some key variables like IPstack info, instance count, and advanced configuration of the given release(s) you want to deploy.  This manifest is typically written in a YAML format.
5. **BOSH Deployment**: A BOSH deployment is a given instantiation of a BOSH deployment manifest.  BOSH will deploy the versioned releases & stemcells defined in the manifest yaml and will maintain the health and availability through the lifetime of the VMs or instances it deploys.  Each VM or instance will be given one or more 'jobs' defined in the BOSH release and are therefore sometimes referred to as job instances. 

#### What Problems does BOSH solve?

BOSH allows release developers to easily version, package and deploy software in a reproducible manner.  Operators can consume BOSH releases and be guaranteed that deployments are repeatable with predictable results across environments.   To accomplish this , BOSH release developers focus on providing some key abilities when building a release:


- **Identifiability**

An operator needs to be able to document the deployment of software and its versions.   A BOSH release ,by design, requires the developer to declare and package everything in the release.   The release itself must also be versioned.  This allows an Operator to fully understand what is deployed as well as consistently upgrade or downgrade versions of software in a release.
	
	
![BOSH-Components](images/OmniGraffle-All-Images/BOSH-Solves.png)

__For Example__: In the image above,  an operator defining a deployment, can refer to one or more versioned releases in a deployment manifest.  This gives an identifiable pattern to determine versions of software used in a deployment.  In the image above, BOSH has two versions of the Kubo release available, versions 0.0.5 & 0.0.6.    The operator has defined the use of version 0.0.5 of the release in the deployment manifest,  which will enforce the use of Kubernetes version 1.6.6 across the deployment called mykubo-deployment.

- **Reproducibility**

Another key tenant in releasing software that BOSH addresses is reproducibility.  To an operator, this means that software should be deployed exactly the same across multiple environments in order to guarantee operational stability.

![BOSH-Components](images/OmniGraffle-All-Images/BOSH-Solves2.png)	
__For Example__: In the image above, a single manifest can deploy something like Kubernetes in a consistent way, providing the same functional deployment with the same releases across multiple environments.  Those environments can even cross multiple IaaS providers by using the CPI abstraction.   The simplified and partial deployment manifest in the image above is declaring which __BOSH stemcell__ , __BOSH Release__ , and config properties to use to deploy functionally identical Kubernetes clusters in two different environments.


- **Consistency**

BOSH also enforces consistency in BOSH release development to ensure that virtually any software can be packaged, versioned and deployed in a similar pattern.  This also provides operational stability.


#### BOSH Use Cases

The primary value of bosh is to simplify the deployment of and day 2 lifecycle management of complex systems.  It was primarily developed to deploy Cloud Foundry but has been extended to deploy many simple and complex environments by developers publishing BOSH releases.   These systems that BOSH can deploy can be found in two primary locations.  The first location is [Pivotal Network](https://network.pivotal.io/), where Pivotal curates commercial BOSH releases of Pivotal Cloud Foundry as well as Pivotal Services that are typically driven by Pivotal Operations Manager + BOSH.  The second location is [BOSH.io](http://bosh.io/releases), which hosts an OSS community repo of various systems that can be deployed.  An example of a prime BOSH use case is:

##### Kubernetes powered by BOSH aka KUBO

![BOSH-Components](images/OmniGraffle-All-Images/BOSH-UseCase-KUBO.png)    

BOSH can be used to deploy and simplify management of complex systems like [Kubernetes via KUBO](https://github.com/cloudfoundry-incubator/kubo-deployment).  Referencing the image above we can see some key benefits BOSH has provided the Operator.

1.  **Repeatability**:  In a cloud native development environment, the Operator can generate 2 or more similar deployment manifests to deploy 2 or more unique but functionally identical Kubernetes deployments to meet the needs of multiple Developer consumers.  
2. **Day 2 Operations**:  BOSH lifecycle management makes keeping all of the Kubernetes deployments healthy easy.  
    - **Maintain Health** Each VM or instance deployed by BOSH also deploys an agent that communicates health back to BOSH.  In the event a KUBO node is unhealthy,  BOSH will automatically try to repair and or rebuild the affected node.  This improves uptime.
    - **Increase Uptime** Each release job instance type can have multiple VMs or instances distributed across availability zones to ensure services provided are not affected by physical faults in a given availability zone.   Availability zones are only supported on certain CPIs, such as the vSphere CPI where availability zones map to vCenter clusters.
    - **Patching** Because BOSH uses versioned releases,  its trivial for an operator to upgrade the Kubernetes KUBO release and apply it to all running deployments with little to no interruption of service.   BOSH will update each deployment as well as maintain the state of each deployment by: (1) detaching persistent disks, (2) rebuilding the affected VMs or instances, and then (3) re-attaching persistent disks.

### Deploying BOSH

#### BOSH Architecture

![BOSH-Components](images/OmniGraffle-All-Images/BOSH-Components.png)

BOSH is typically deployed as a single VM or Instance.  That VM/instance has many components that perform vital roles in how BOSH manages deployments at scale:

- **NATS**: Provides a message bus for the various services of BOSH to interact
- **POSTGRESQL**:  BOSH writes all of its state into a database.  Typically that database is internal to a single VM BOSH deployment and provided by Postgres.  This can however be modified to use an external data source so that  the BOSH VM can be rebuilt and reconnect to the database to reload its persistent state.
- **BLOBSTORE**: Each stemcell and release uploaded to BOSH is stored in a blobstore.  Default deployments of BOSH use an internal store (webdav), but like the Postgresql database,  this can also be externalized.
- **Director**: Is the main API that the BOSH CLI will interface with to allow an operator to create and manage BOSH deployments.
- **Health Monitor**:  BOSH requires that each VM it deploys have an agent that it can communicate with to assign and deploy jobs from BOSH releases that are defined in a deployment manifest. It will also maintain the health of each VM or instance it has deployed.  The agent will report vitals back to BOSH and in cases where services in the VM are faulted, or the agent is unreachable, the HealthMonitor can use plugins to restart services and even rebuild the VM or instance.
- **CPI**:  The CPI is the IaaS specific executable binary that BOSH uses to interact with the defined IaaS in its deployment yaml.
- **UAA**: Provides User Access & Authentication that allows BOSH to authenticate operators via SAML or LDAP backends.
- **CREDHUB**: Manages credentials like passwords, certificates, certificate authorities, ssh keys, rsa keys and arbitrary values (strings and JSON blobs).  BOSH will leverage credhub to create and store key credentials for deployments, like public certificates & keys.

Full reference on BOSH can be found here: [BOSH.io](http://bosh.io)

#### CookBook: How to deploy "KUBO" on vSphere

BOSH is deployed by using the [BOSH CLI](http://bosh.io/docs/cli-v2.html),   passing the correct cmd line arguments or storing those arguments as variable data within additional yaml files to define how BOSH itself will be deployed.  This 'CookBook' section will assist in deploying BOSH and then a basic Kubo deployment.

Prereqs include:

- vSphere vCenter 6.x
- 1 x vSphere Datastore with adequate space for deployment
- 1 x vCenter Resource pool for your Kubo deployment

##### Steps To deploy BOSH (Mac OSX) ...
Operating System Specific Installation of CLI is documented [here](http://bosh.io/docs/cli-v2.html).

**Get the BOSH CLI ...**

- This cookbook uses version 2 of the BOSH CLI.  Its a go compiled binary,  but does have some os dependancies

`1. sudo wget -O /usr/local/bin/bosh https://s3.amazonaws.com/bosh-cli-artifacts/bosh-cli-2.0.28-darwin-amd64 && sudo chmod 755 /usr/local/bin/bosh`

**Prepare MAC OS requirements like Ruby ...**

- Other OS Specific Instructions can be found [here](http://bosh.io/docs/cli-v2.html) & [here](http://bosh.io/docs/cli-env-deps.html)

```
2. gem update --system
3. xcode-select --install
4. brew install openssl
```

**Use GIT Client to Clone the BOSH Deployment repo ...**

- This cookbook assumes u have the git client already installed.  This repo contains everything you will need to deploy a BOSH instance.

```
5. git clone https://github.com/cloudfoundry/bosh-deployment
6. cd bosh-deployment
```

**Deploy BOSH ...**

- This single cli command will deploy the BOSH instance.  -o flags are one or more yaml files that are composed to form a single manifest.  -v flags are the variables that we assign to variable markers in each yaml.  The BOSH int or interpolate command will compose the manifest from the yaml stubs and the variables. 


```
7. /usr/local/bin/bosh create-env bosh.yml \
    --state=mystate.json \
    --vars-store=mycreds.yml \
    -o vsphere/cpi.yml \
    -o uaa.yml \
    -o misc/powerdns.yml \
    -o credhub.yml \
    -v director_name=kubobosh \
    -v internal_cidr=[[CIDR-OF-NETWORK-FOR-BOSH-VM]] \
    -v internal_gw=[[GATEWAY-OF-NETWORK-FOR-BOSH-VM]] \
    -v internal_ip=[[IP-OF-NETWORK-FOR-BOSH-VM]] \
    -v network_name=[[VCENTER-PG-NAME-NETWORK-FOR-BOSH-VM]] \
    -v vcenter_dc=[[VCENTER-DC-NAME]] \
    -v vcenter_ds=[[VCENTER-DATASTORE-NAME]] \
    -v vcenter_ip=[[VCENTER-IP]] \
    -v vcenter_user=[[VCENTER-USER]] \
    -v vcenter_password=[[VCENTER-PASSWD]] \
    -v vcenter_templates=kubobosh-templates \
    -v vcenter_vms=kubobosh-vms \
    -v vcenter_disks=kubobosh-disks \
    -v vcenter_cluster=[[VCENTER-CLUSTER]] \
    -v dns_recursor_ip=[[YOUR-NETWORK-DNS]]
    
8. /usr/local/bin/bosh alias-env kubobosh -e [[IP-OF-NETWORK-FOR-BOSH-VM]] --ca-cert <(/usr/local/bin/bosh int ./mycreds.yml --path /director_ssl/ca)
9. export BOSH_CLIENT=admin
10. export BOSH_CLIENT_SECRET=$(/usr/local/bin/bosh int ./mycreds.yml --path /admin_password)
11. /usr/local/bin/bosh -e kubobosh env

```

##### Steps To Deploy Kubo on BOSH ...

**Use GIT Client to Clone the BOSH Deployment repo...**

- This cookbook assumes u have the git client already installed.  This repo contains everything you will need to deploy a Kubo with your BOSH instance.

```
1. git clone https://github.com/cloudfoundry-incubator/kubo-deployment
2. cd kubo-deployment
```

**Generate BOSH cloud-config manifest & update it...**


- A cloud-config manifest is specific to each CPI and will map BOSH constructs (like availability zones) to vSphere ones.  A deployment manifest will reference constructs from the cloud-config.

```
3. /usr/local/bin/bosh int configurations/vsphere/cloud-config.yml \
    -o manifests/ops-files/k8s_master_static_ip_vsphere.yml \
    -v director_name=bosh \
    -v internal_cidr=[[CIDR-OF-NETWORK-FOR-KUBO-CAN-BE-SAME-AS-BOSH]] \
    -v internal_gw=[[GAETWAY-OF-NETWORK-FOR-KUBO-CAN-BE-SAME-AS-BOSH]] \
    -v internal_ip=[[DNS-OF-NETWORK-FOR-KUBO-CAN-BE-SAME-AS-BOSH]] \
    -v kubernetes_master_host=[[IP-FOR-KUBO-MASTER-VIP]] \
    -v reserved_ips=[[IP-RANGE-YOU-DONT-WANT-BOSH-TO-USE ex: 192.168.100.10-192.168.100.20]] \
    -v network_name=[[VCENTER-PG-NAME-NETWORK-FOR-KUBO-CAN-BE-SAME-AS-BOSH]] \
    -v deployments_network=[[VCENTER-PG-NAME-NETWORK-FOR-KUBO-CAN-BE-SAME-AS-BOSH]] \
    -v vcenter_cluster=[[VCENTER-CLUSTER-FOR-KUBO]] \
    -v vcenter_rp="[[VCENTER-RESOURCE-POOL-FOR-KUBO]]" > mycloudconfig.yml
4. bosh -e kubobosh update-cloud-config mycloudconfig.yml
```

**Generate KUBO deployment manifest...**

```
5. /usr/local/bin/bosh int manifests/kubo.yml \
     -o manifests/ops-files/master-haproxy-vsphere.yml \
     -o manifests/ops-files/worker-haproxy.yml \
     -v deployments_network=[[VCENTER-PG-NAME-NETWORK-FOR-KUBO-CAN-BE-SAME-AS-BOSH]] \
     -v kubo-admin-password="mykubopasswd" \
     -v kubelet-password="mykubopasswd" \
     -v kubernetes_master_port=443 \
     -v kubernetes_master_host=[[IP-FOR-KUBO-MASTER-VIP]] \
     -v deployment_name=mykubocluster \
     -v worker_haproxy_tcp_frontend_port=1234 \
     -v worker_haproxy_tcp_backend_port=4231 > mykubo.yml
```

**Upload latest BOSH Stemcell...**

```
6. /usr/local/bin/bosh -e kubobosh upload-stemcell https://bosh.io/d/stemcells/bosh-vsphere-esxi-ubuntu-trusty-go_agent
```

**Upload BOSH Kubo Release...**

```
7. wget https://github.com/cloudfoundry-incubator/kubo-release/releases/download/v0.0.5/kubo-release-0.0.5.tgz
8. /usr/local/bin/bosh -e kubobosh upload-release kubo-release-0.0.5.tgz
```


**Deploy Kubo :)**

```
9. /usr/local/bin/bosh -e kubobosh -d mykubocluster deploy ~/kubo-deployment/mykubo.yml
```

##### Steps to Test Kubo Deployment

```
*******************
**Install Credhub**
*******************

credhub_user_password=$(bosh -e kubobosh int "../bosh-deployment/mycreds.yml" --path "/credhub_cli_password")
credhub_api_url="https://10.190.64.10:8844"

bosh -e kubobosh int "../bosh-deployment/mycreds.yml" --path="/uaa_ssl/ca" > credhubca.crt
bosh -e kubobosh int "../bosh-deployment/mycreds.yml" --path="/credhub_tls/ca" > credhub.crt

credhub login -u credhub-cli -p "${credhub_user_password}" -s "${credhub_api_url}" --ca-cert credhubca.crt --ca-cert credhub.crt
bosh int <(credhub get -n "/kubobosh/mykubocluster/tls-kubernetes" --output-json) --path=/value/ca > mykubecert.crt
endpoint="10.190.64.11"
port="443"
address="https://${endpoint}:${port}"
admin_password="mykubopasswd"
context_name="mykubocluster"

kubectl config set-cluster "mykubocluster" --server="$address" --certificate-authority=mykubecert.crt --embed-certs=true
kubectl config set-credentials "mykubocluster-admin" --token="${admin_password}"
kubectl config set-context "mykubocluster" --cluster="mykubocluster" --user="mykubocluster-admin"
kubectl config use-context "mykubocluster"
kubectl get pods --namespace=kube-system
```