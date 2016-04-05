# SIT113 - Week 4 - Cloud Mechanism

#### Usage Monitoring 
 
There are a few agents that take the responsibility of monitoring

Resource Agents: Events driven 
Polling Agents: Collects meta-data, uptime vs. downtime etc...

### Specialized Cloud Mechanisms

#### Automated Scaling 

*Typically deployed near firewall*
Automatically listen to communication and activities to dynamically scale in or out resources 

#### Load Balancer 

Attempts to balance the workload/resources based on priority so no one source of processing power is utilised

Allocate better processing capabilities to higher priority/heavier workloads  

#### SLA monitor

Monitor quality of service based on aggreed upon metrics. Information are logged and **repair/modifications are made accordingly**

#### Pay-per-use monitor 

Includes request/response messages based on data volume & bandwidth consumption

#### Failover system 

Takes charge of switching resources when one resource becomes available using two ways: 

+ Active: Actively use load balancing and monitoring and make changes as needed 
+ Passive: Only becomes active when resources fail 

#### Multi-Device broker

Different gateways are used for different types of devices which allows for universal access

+ XML gateways
+ Mobile Device Gateways
+ Cloud Storage Gateways

#### State database 

Allow for persisting/pausing a state of a cloud resource and resuming later instead of caching it


### Management Mechanisms 

#### Primarily two types of portal:

– **Usage and Administration Portal**: general purpose portal centralising management controls for cloud-based IT resources
– **Self-Service Portal**: effectively a “shopping portal” allowing cloud consumers to choose cloud-based IT resources for provisioning

#### SLA management

+ Relies on SLA monitoring agents
+ Takes charges of investigating quality/log of the service to ensure satisfaction 

#### Billing

+ Pay-per-use
+ Custom models/cloud consumer

### Security Mechanisms

+ Encryption
+ Public Keys
+ Hashing 
+ Access Management 
+ Digital Signature 
+ SSO (Single Sign On) - Similar to Token Authentication 
+ Hardening Image
    * Closing unused ports
    * Removing redundancy/unnecessary programs
    * Disable internal root access/unused services/guest access
