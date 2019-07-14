# OCP 4.1 on AWS (Official Documentation)

https://docs.openshift.com/container-platform/4.1/welcome/index.html


## OpenShift Container Platform architecture

### CUSTOM OPERATING SYSTEM

- OpenShift Container Platform uses Red Hat Enterprise Linux CoreOS (RHCOS), a new container-oriented operating system that combines some of the best features and functions of the CoreOS and Red Hat Atomic Host operating systems.

- RHCOS is specifically designed for running containerized applications from OpenShift Container Platform and works with new tools to provide fast installation, Operator-based management, and simplified upgrades.

- RHCOS includes:
	
	- Ignition, which is a firstboot system configuration for initially bringing up and configuring OpenShift Container Platform nodes.

	- cri-o, a Kubernetes native container runtime implementation that integrates closely with the operating system to deliver an efficient and optimized Kubernetes experience.

	- Kubelet, the primary node agent for Kubernetes that is responsible for launching and monitoring containers.


> In OpenShift Container Platform 4.1, you must use RHCOS for all control plane machines, but you can use Red Hat Enterprise Linux (RHEL) as the operating system for compute, or worker, machines. **If you choose to use RHEL workers, you must perform more system maintenance than if you use RHCOS for all of the cluster machines.**

### SIMPLIFIED INSTALLATION AND UPDATE PROCESS

For clusters that use RHCOS for all machines, updating, or upgrading, OpenShift Container Platform is a simple, highly-automated process. Because OpenShift Container Platform completely controls the systems and services that run on each machine, including the operating system itself, from a central control plane, upgrades are designed to become automatic events. **If your cluster contains RHEL worker machines, the control plane benefits from the streamlined update process, but you must perform more tasks to upgrade the RHEL machines.**

### OTHER KEY FEATURES

- Operators

- OLM (operator lifecycle manager)

- CRIO-O. Container Engine

- Quay. Red hat container registry


## Install

### Register a new domain on aws 

We are going to register the iaciscp.net

- On aws go to Route 53
- Register a new domain
- Fulfil all the information required.
- Accept received email confirmation
- wait for the domain to be available (15 mins)
- **Disable the automatic renovation**

> This process will automatically create a aws hosted zone

```
[sp81891@oc2157818656 ~]$ dig iaciscp.net

; <<>> DiG 9.9.4-RedHat-9.9.4-74.el7_6.1 <<>> iaciscp.net
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 36380
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;iaciscp.net.			IN	A

;; AUTHORITY SECTION:
iaciscp.net.		600	IN	SOA	ns-125.awsdns-15.com. awsdns-hostmaster.amazon.com. 1 7200 900 1209600 86400

;; Query time: 83 msec
;; SERVER: 80.58.61.254#53(80.58.61.254)
;; WHEN: Sun Jul 14 16:23:54 CEST 2019
;; MSG SIZE  rcvd: 121

```

### AWS account limits

> To avoid reaching the DEFAULT resources limit (per account) on eu-central-1 (Frankfurt) we are going to work on eu-west-2 (London).

- AZ's available on eu-west-2 region

```
[sp81891@oc2157818656 ~]$ aws ec2 describe-availability-zones --region eu-west-2
{
    "AvailabilityZones": [
        {
            "State": "available",
            "Messages": [],
            "RegionName": "eu-west-2",
            "ZoneName": "eu-west-2a",
            "ZoneId": "euw2-az2"
        },
        {
            "State": "available",
            "Messages": [],
            "RegionName": "eu-west-2",
            "ZoneName": "eu-west-2b",
            "ZoneId": "euw2-az3"
        },
        {
            "State": "available",
            "Messages": [],
            "RegionName": "eu-west-2",
            "ZoneName": "eu-west-2c",
            "ZoneId": "euw2-az1"
        }
    ]
}
```

### Resources to be deployed

- 1 VPC

- 2 NAT gateways

- 3 EIPs. One for every Nat Gateway attached to every private subnet in every Az.

- 1 External NLB

- 1 Internal NLB

- 1 ELB

- 27 ENIs for every AZ.

- 1 VPC Gateway for S3 access.

- 2 S3 buckets

- 10 Security Groups


### Required AWS permissions

When you attach the AdministratorAccess policy to the IAM user that you create, you grant that user all of the required permissions

### Installing a cluster on AWS with customizations

https://docs.openshift.com/container-platform/4.1/installing/installing_aws/installing-aws-customizations.html#installing-aws-customizations

In OpenShift Container Platform version 4.1, you can install a customized cluster on infrastructure that the installation program provisions on Amazon Web Services (AWS). To customize the installation, you modify some parameters in the install-config.yaml file before you install the cluster.

