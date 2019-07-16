# OCP 4.1 on AWS (Official Documentation)

https://docs.openshift.com/container-platform/4.1/welcome/index.html


## 0. Cluster info

- Ocp4.0 running cluster : https://console-openshift-console.apps.ocp4-tst-001.iaciscp.net

- Identity providers configured:

	- OAuth github.ibm.com
		- every member of the d-lab org can access the cluster as admin role (project manager)

	- Htpasswd
		- admin user (cluster-admin)





## 1. OpenShift Container Platform architecture

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


## 2. Red Hat Enterprise Linux CoreOS (RHCOS)

- RHCOS is the only supported operating system for OpenShift Container Platform control plane, or master, machines. While RHCOS is the default operating system for all cluster machines, you can create compute, or worker, machines that use RHEL as their operating system.

- Management is performed remotely from the OpenShift Container Platform cluster. When you set up your RHCOS machines, you can modify only a few system settings

- Although RHCOS contains features for running the OCI- and libcontainer-formatted containers that Docker requires, it incorporates the CRI-O container engine instead of the Docker container engine

- At the moment, CRI-O is only available as a container engine within OpenShift Container Platform clusters.

- For tasks such as building, copying, and otherwise managing containers, RHCOS replaces the Docker CLI tool with a compatible set of container tools. While direct use of these tools in RHCOS is discouraged, you can use them for debugging purposes.

- RHCOS features transactional upgrades and rollbacks using the rpm -ostree upgrade system.

- Ignition is the utility that is used by RHCOS to manipulate disks during initial configuration. It completes common disk tasks, including partitioning disks, formatting partitions, writing files, and configuring users. On first boot, Ignition reads its configuration from the installation media or the location that you specify and applies the configuration to the machines.

- 

## 3. Install OCP 4.1 on AWS

### 3.1 Register a new domain on aws 

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

### 3.2 AWS account limits

> To avoid reaching the DEFAULT resources limit (per account) on eu-central-1 (Frankfurt) we are going to work on eu-west-2 (London).

- AZ's available on eu-west-2 region

- In this case we are going to use eu-west-2a and eu-west-2b for (Masters) and eu-west-2c (Compute)

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

### 3.3 Resources to be deployed

- 1 VPC

- 2 NAT gateways

- 2 EIPs. One for every Nat Gateway attached to every private subnet in every Az.

- 1 External NLB

- 1 Internal NLB

- 1 ELB

- 27 ENIs for every AZ.

- 1 VPC Gateway for S3 access.

- 2 S3 buckets

- 10 Security Groups


### 3.4 Required AWS permissions

When you attach the AdministratorAccess policy to the IAM user that you create, you grant that user all of the required permissions

### 3.5 Installing a cluster on AWS with customizations

https://docs.openshift.com/container-platform/4.1/installing/installing_aws/installing-aws-customizations.html#installing-aws-customizations

In OpenShift Container Platform version 4.1, you can install a customized cluster on infrastructure that the installation program provisions on Amazon Web Services (AWS). To customize the installation, you modify some parameters in the install-config.yaml file before you install the cluster.

#### 3.5.1 Obtaining the installation program

- go to https://cloud.redhat.com/openshift/install/aws/installer-provisioned

- Then download installer & Copy the pull secret 

	https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-install-linux-4.1.4.tar.gz

	```
	tar xvf openshift-install-linux-4.1.4.tar.gz

	sudo mv openshift-install /usr/local/bin

	[sp81891@oc2157818656 ~]$ openshift-install version
	openshift-install v4.1.4-201906271212-dirty
	built from commit bf47826c077d16798c556b1bd143a5bbfac14271
	release image quay.io/openshift-release-dev/ocp-release@sha256:a6c177eb007d20bb00bfd8f829e99bd40137167480112bd5ae1c25e40a4a163a

	```


#### 3.5.2 Generating an SSH private key and adding it to the agent

> You can use this key to SSH into the master nodes as the user **core**. When you deploy the cluster, the key is added to the core user’s ~/.ssh/authorized_keys list.


- If you do not have an SSH key that is configured for password-less authentication on your computer, create one.

```
ssh-keygen -t rsa -b 4096 -N '' -f <path>/<file_name>

```

- Start the ssh-agent process as a background task

```
eval "$(ssh-agent -s)"
```

- Add the ssh private key to the ssh-agent

```
# add 
ssh-add ~/.ssh/id_rsa

# list

ssh-add -l

2048 SHA256:GJnFpJMvKyXvCAVWy3TvJQJJdI5RE9BCoIuUAIoPiso /home/sp81891/.ssh/id_rsa (RSA)

```

#### 3.5.3 Creating the installation configuration file

- This process will use the [default] section on your aws credentials file.

- If you don't have a [default] section or the credential file, the installer will ask you access and secret key.

- At the end, the installer will create a [default] section on the aws credential file. **(careful)**

- The Base Domain will be automatically discovered, you can't type!!

```
openshift-install create install-config --dir=install_files

? SSH Public Key /home/sp81891/.ssh/id_rsa.pub
? Platform aws
? AWS Access Key ID AKIAY7EHYMAAOXFCCR2B
? AWS Secret Access Key [? for help] ****************************************
INFO Writing AWS credentials to "/home/sp81891/.aws/credentials" (https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html) 
? Region eu-west-2
? Base Domain iaciscp.net
? Cluster Name ocp4-tst-001
? Pull Secret [? for help] ******************
```

- When the installation finished a **install_files** directory will be created with this content:

```
drwxrwxr-x. 4 sp81891 sp81891  4096 Jul 14 18:48 ..
-rw-rw-r--. 1 sp81891 sp81891  4453 Jul 14 19:03 .openshift_install.log
-rw-r--r--. 1 sp81891 sp81891 13250 Jul 14 19:03 .openshift_install_state.json
-rw-r--r--. 1 sp81891 sp81891  3709 Jul 14 19:03 install-config.yaml
```

#### 3.5.4 customize your install-config.yaml

- 3 masters
	- az: 2a and 2b

- 2 compute
	- az: 2c

- Note that we are hiding the SECRET texts.

```
apiVersion: v1
baseDomain: iaciscp.net
controlPlane:
  hyperthreading: Enabled
  name: master
  platform: 
    aws:
      zones:
      - eu-west-2a
      - eu-west-2b
      rootVolume:
        iops: 2000
        size: 200
        type: gp2
      type: m5.xlarge
  replicas: 3
compute:
- hyperthreading: Enabled
  name: worker
  platform: 
    aws:
      zones:
      - eu-west-2c
      rootVolume:
        iops: 1000
        size: 200
        type: gp2
      type: m5.large
  replicas: 2
metadata:
  creationTimestamp: null
  name: ocp4-tst-001
networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  machineCIDR: 10.0.0.0/16
  networkType: OpenShiftSDN
  serviceNetwork:
  - 172.30.0.0/16
platform:
  aws:
    region: eu-west-2
    userTags:
      iacContact: soranno
      env: tst
pullSecret: 'PULL SECRET JSON'
sshKey: |
  ssh-rsa SECRET
```

#### 3.5.5 Deploy the cluster

- **WARNING**: this will be the last chance to backup your **install-config.yaml** because it will be consumed and transformed on terraform files.

- At the end the bootstrap configuration (node, bucket, etc) will be destroyed automatically

```
openshift-install create cluster --dir=install_files --log-level debug
```
- After 30 mins you will see

```
DEBUG Cluster is initialized                       
INFO Waiting up to 10m0s for the openshift-console route to be created... 
DEBUG Route found in openshift-console namespace: console 
DEBUG Route found in openshift-console namespace: downloads 
DEBUG OpenShift console route is created           
INFO Install complete!                            
INFO To access the cluster as the system:admin user when using 'oc', run 'export KUBECONFIG=/home/sp81891/PycharmProjects/githubIBM/D-CLOUD/ocp-4.1-aws/install_files/auth/kubeconfig' 
INFO Access the OpenShift web-console here: https://console-openshift-console.apps.ocp4-tst-001.iaciscp.net 
INFO Login to the console with user: kubeadmin, password: Tr7gv-indjp-pM6Zw-79x2o 
```

- after installation these are the files and directories created

```
[sp81891@oc2157818656 ocp-4.1-aws]$ tree
.
├── install-config.yaml.backup
├── install_files
│   ├── auth
│   │   ├── kubeadmin-password
│   │   └── kubeconfig
│   ├── metadata.json
│   ├── terraform.aws.auto.tfvars
│   ├── terraform.tfstate
│   ├── terraform.tfvars
│   └── tls
│       ├── journal-gatewayd.crt
│       └── journal-gatewayd.key
└── README.md

```


#### 3.5.7 Destroy the cluster

- **WARNING** This will remove all resources previously created and will delete all the configuration files. So, in case you want the create the cluster again a new **install-config.yaml** need to be generated again or restored from backup.

```
openshift-install destroy cluster --dir=install_files --log-level debug
```


## 4. Installing the OpenShift Command-line Interface

- Download the last oc version from here https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/

	- This version https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-client-linux-4.1.4.tar.gz

```
tar xvf openshift-client-linux-4.1.4.tar.gz
sudo mv oc /usr/local/bin

oc version
Client Version: version.Info{Major:"4", Minor:"1+", GitVersion:"v4.1.4-201906271212+6b97d85-dirty", GitCommit:"6b97d85", GitTreeState:"dirty", BuildDate:"2019-06-27T18:11:21Z", GoVersion:"go1.11.6", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"13+", GitVersion:"v1.13.7-eks-c57ff8", GitCommit:"c57ff8e35590932c652433fab07988da79265d5b", GitTreeState:"clean", BuildDate:"2019-06-07T20:43:03Z", GoVersion:"go1.11.5", Compiler:"gc", Platform:"linux/amd64"}

```

## 5. Logging into the cluster

```
export KUBECONFIG=<install path>/install_files/auth/kubeconfig
oc whoami
```

- all pods created

```
oc get pods --all-namespaces
NAMESPACE                                               NAME                                                                 READY   STATUS      RESTARTS   AGE
openshift-apiserver-operator                            openshift-apiserver-operator-5c6755499b-g5lvx                        1/1     Running     1          100m
openshift-apiserver                                     apiserver-8t7mm                                                      1/1     Running     0          93m
openshift-apiserver                                     apiserver-q2vhh                                                      1/1     Running     0          90m
openshift-apiserver                                     apiserver-sjbsx                                                      1/1     Running     0          92m
openshift-authentication-operator                       authentication-operator-647879c74-fgnd8                              1/1     Running     0          95m
openshift-authentication                                oauth-openshift-7b859cbcbd-44plp                                     1/1     Running     0          92m
openshift-authentication                                oauth-openshift-7b859cbcbd-wsvvh                                     1/1     Running     0          92m
openshift-cloud-credential-operator                     cloud-credential-operator-6685d5bfbb-w4qq9                           1/1     Running     0          100m
openshift-cluster-machine-approver                      machine-approver-5cdc9fb764-5mwjm                                    1/1     Running     0          100m
openshift-cluster-node-tuning-operator                  cluster-node-tuning-operator-7d5749d66d-9th8b                        1/1     Running     0          95m
openshift-cluster-node-tuning-operator                  tuned-6bcdq                                                          1/1     Running     0          94m
openshift-cluster-node-tuning-operator                  tuned-d4ntl                                                          1/1     Running     0          95m
openshift-cluster-node-tuning-operator                  tuned-mhz4t                                                          1/1     Running     0          94m
openshift-cluster-node-tuning-operator                  tuned-qqmjz                                                          1/1     Running     0          95m
openshift-cluster-node-tuning-operator                  tuned-tz44n                                                          1/1     Running     0          95m
openshift-cluster-samples-operator                      cluster-samples-operator-7b99cfcc9b-kzh88                            1/1     Running     0          93m
openshift-cluster-storage-operator                      cluster-storage-operator-5c7cd6d54f-pv4d6                            1/1     Running     0          94m
openshift-cluster-version                               cluster-version-operator-857c4f9697-cpb66                            1/1     Running     0          100m
openshift-console-operator                              console-operator-67c5744676-7z85x                                    1/1     Running     0          94m
openshift-console                                       console-6b877c86f7-74fb5                                             1/1     Running     0          93m
openshift-console                                       console-6b877c86f7-vtb4s                                             1/1     Running     0          90m
openshift-console                                       downloads-9cc78db84-62r56                                            1/1     Running     0          94m
openshift-console                                       downloads-9cc78db84-lvqpx                                            1/1     Running     0          94m
openshift-controller-manager-operator                   openshift-controller-manager-operator-cd87b4d9c-fjqsd                1/1     Running     1          100m
openshift-controller-manager                            controller-manager-55wlp                                             1/1     Running     0          90m
openshift-controller-manager                            controller-manager-wzpbw                                             1/1     Running     0          92m
openshift-controller-manager                            controller-manager-z8gx4                                             1/1     Running     0          93m
openshift-dns-operator                                  dns-operator-76bff8b7d9-l4rwl                                        1/1     Running     0          96m
openshift-dns                                           dns-default-8k6h6                                                    2/2     Running     0          94m
openshift-dns                                           dns-default-bxrpp                                                    2/2     Running     0          99m
openshift-dns                                           dns-default-hz2jd                                                    2/2     Running     0          99m
openshift-dns                                           dns-default-qks7w                                                    2/2     Running     0          94m
openshift-dns                                           dns-default-xzfcx                                                    2/2     Running     0          99m
openshift-etcd                                          etcd-member-ip-10-0-139-59.eu-west-2.compute.internal                2/2     Running     0          98m
openshift-etcd                                          etcd-member-ip-10-0-141-15.eu-west-2.compute.internal                2/2     Running     0          98m
openshift-etcd                                          etcd-member-ip-10-0-150-10.eu-west-2.compute.internal                2/2     Running     0          99m
openshift-image-registry                                cluster-image-registry-operator-c7d86746-8pqf9                       1/1     Running     0          94m
openshift-image-registry                                image-registry-5cc84f8c4d-rzcvs                                      1/1     Running     0          93m
openshift-image-registry                                node-ca-42jld                                                        1/1     Running     0          93m
openshift-image-registry                                node-ca-6b7jd                                                        1/1     Running     0          93m
openshift-image-registry                                node-ca-kf8zh                                                        1/1     Running     0          93m
openshift-image-registry                                node-ca-qbrdh                                                        1/1     Running     0          93m
openshift-image-registry                                node-ca-wh5m7                                                        1/1     Running     0          93m
openshift-ingress-operator                              ingress-operator-657754d884-x7s4n                                    1/1     Running     0          94m
openshift-ingress                                       router-default-6494b4999f-mxg88                                      1/1     Running     0          93m
openshift-ingress                                       router-default-6494b4999f-tzgh2                                      1/1     Running     0          93m
openshift-kube-apiserver-operator                       kube-apiserver-operator-575b4dbdb-xm6q5                              1/1     Running     1          100m
openshift-kube-apiserver                                installer-2-ip-10-0-139-59.eu-west-2.compute.internal                0/1     Completed   0          96m
openshift-kube-apiserver                                installer-2-ip-10-0-141-15.eu-west-2.compute.internal                0/1     Completed   0          97m
openshift-kube-apiserver                                installer-2-ip-10-0-150-10.eu-west-2.compute.internal                0/1     Completed   0          98m
openshift-kube-apiserver                                installer-3-ip-10-0-139-59.eu-west-2.compute.internal                0/1     Completed   0          95m
openshift-kube-apiserver                                installer-3-ip-10-0-150-10.eu-west-2.compute.internal                0/1     Completed   0          94m
openshift-kube-apiserver                                installer-5-ip-10-0-139-59.eu-west-2.compute.internal                0/1     Completed   0          89m
openshift-kube-apiserver                                installer-5-ip-10-0-141-15.eu-west-2.compute.internal                0/1     Completed   0          90m
openshift-kube-apiserver                                installer-5-ip-10-0-150-10.eu-west-2.compute.internal                0/1     Completed   0          92m
openshift-kube-apiserver                                kube-apiserver-ip-10-0-139-59.eu-west-2.compute.internal             2/2     Running     0          88m
openshift-kube-apiserver                                kube-apiserver-ip-10-0-141-15.eu-west-2.compute.internal             2/2     Running     0          90m
openshift-kube-apiserver                                kube-apiserver-ip-10-0-150-10.eu-west-2.compute.internal             2/2     Running     0          92m
openshift-kube-apiserver                                revision-pruner-2-ip-10-0-139-59.eu-west-2.compute.internal          0/1     Completed   0          95m
openshift-kube-apiserver                                revision-pruner-2-ip-10-0-141-15.eu-west-2.compute.internal          0/1     Completed   0          95m
openshift-kube-apiserver                                revision-pruner-2-ip-10-0-150-10.eu-west-2.compute.internal          0/1     Completed   0          95m
openshift-kube-apiserver                                revision-pruner-3-ip-10-0-139-59.eu-west-2.compute.internal          0/1     Completed   0          94m
openshift-kube-apiserver                                revision-pruner-3-ip-10-0-150-10.eu-west-2.compute.internal          0/1     Completed   0          92m
openshift-kube-apiserver                                revision-pruner-5-ip-10-0-139-59.eu-west-2.compute.internal          0/1     Completed   0          87m
openshift-kube-apiserver                                revision-pruner-5-ip-10-0-141-15.eu-west-2.compute.internal          0/1     Completed   0          89m
openshift-kube-apiserver                                revision-pruner-5-ip-10-0-150-10.eu-west-2.compute.internal          0/1     Completed   0          90m
openshift-kube-controller-manager-operator              kube-controller-manager-operator-5d8b79f6f6-k2c9h                    1/1     Running     1          100m
openshift-kube-controller-manager                       installer-4-ip-10-0-150-10.eu-west-2.compute.internal                0/1     Completed   0          98m
openshift-kube-controller-manager                       installer-5-ip-10-0-139-59.eu-west-2.compute.internal                0/1     Completed   0          95m
openshift-kube-controller-manager                       installer-5-ip-10-0-141-15.eu-west-2.compute.internal                0/1     Completed   0          96m
openshift-kube-controller-manager                       installer-5-ip-10-0-150-10.eu-west-2.compute.internal                0/1     Completed   0          98m
openshift-kube-controller-manager                       installer-6-ip-10-0-139-59.eu-west-2.compute.internal                0/1     Completed   0          91m
openshift-kube-controller-manager                       installer-6-ip-10-0-141-15.eu-west-2.compute.internal                0/1     Completed   0          92m
openshift-kube-controller-manager                       installer-6-ip-10-0-150-10.eu-west-2.compute.internal                0/1     Completed   0          93m
openshift-kube-controller-manager                       kube-controller-manager-ip-10-0-139-59.eu-west-2.compute.internal    2/2     Running     0          91m
openshift-kube-controller-manager                       kube-controller-manager-ip-10-0-141-15.eu-west-2.compute.internal    2/2     Running     0          92m
openshift-kube-controller-manager                       kube-controller-manager-ip-10-0-150-10.eu-west-2.compute.internal    2/2     Running     0          93m
openshift-kube-controller-manager                       revision-pruner-4-ip-10-0-150-10.eu-west-2.compute.internal          0/1     Completed   0          98m
openshift-kube-controller-manager                       revision-pruner-5-ip-10-0-139-59.eu-west-2.compute.internal          0/1     Completed   0          94m
openshift-kube-controller-manager                       revision-pruner-5-ip-10-0-141-15.eu-west-2.compute.internal          0/1     Completed   0          95m
openshift-kube-controller-manager                       revision-pruner-5-ip-10-0-150-10.eu-west-2.compute.internal          0/1     Completed   0          96m
openshift-kube-controller-manager                       revision-pruner-6-ip-10-0-139-59.eu-west-2.compute.internal          0/1     Completed   0          91m
openshift-kube-controller-manager                       revision-pruner-6-ip-10-0-141-15.eu-west-2.compute.internal          0/1     Completed   0          91m
openshift-kube-controller-manager                       revision-pruner-6-ip-10-0-150-10.eu-west-2.compute.internal          0/1     Completed   0          92m
openshift-kube-scheduler-operator                       openshift-kube-scheduler-operator-6f8769ff74-4lc92                   1/1     Running     1          100m
openshift-kube-scheduler                                installer-2-ip-10-0-150-10.eu-west-2.compute.internal                0/1     Completed   0          98m
openshift-kube-scheduler                                installer-3-ip-10-0-150-10.eu-west-2.compute.internal                0/1     Completed   0          98m
openshift-kube-scheduler                                installer-4-ip-10-0-150-10.eu-west-2.compute.internal                0/1     Completed   0          98m
openshift-kube-scheduler                                installer-5-ip-10-0-139-59.eu-west-2.compute.internal                0/1     Completed   0          95m
openshift-kube-scheduler                                installer-5-ip-10-0-141-15.eu-west-2.compute.internal                0/1     Completed   0          96m
openshift-kube-scheduler                                installer-6-ip-10-0-139-59.eu-west-2.compute.internal                0/1     Completed   0          91m
openshift-kube-scheduler                                installer-6-ip-10-0-141-15.eu-west-2.compute.internal                0/1     Completed   0          90m
openshift-kube-scheduler                                installer-6-ip-10-0-150-10.eu-west-2.compute.internal                0/1     Completed   0          93m
openshift-kube-scheduler                                openshift-kube-scheduler-ip-10-0-139-59.eu-west-2.compute.internal   1/1     Running     0          91m
openshift-kube-scheduler                                openshift-kube-scheduler-ip-10-0-141-15.eu-west-2.compute.internal   1/1     Running     0          90m
openshift-kube-scheduler                                openshift-kube-scheduler-ip-10-0-150-10.eu-west-2.compute.internal   1/1     Running     0          93m
openshift-kube-scheduler                                revision-pruner-2-ip-10-0-150-10.eu-west-2.compute.internal          0/1     Completed   0          98m
openshift-kube-scheduler                                revision-pruner-3-ip-10-0-150-10.eu-west-2.compute.internal          0/1     Completed   0          98m
openshift-kube-scheduler                                revision-pruner-4-ip-10-0-150-10.eu-west-2.compute.internal          0/1     Completed   0          96m
openshift-kube-scheduler                                revision-pruner-5-ip-10-0-139-59.eu-west-2.compute.internal          0/1     Completed   0          93m
openshift-kube-scheduler                                revision-pruner-5-ip-10-0-141-15.eu-west-2.compute.internal          0/1     Completed   0          95m
openshift-kube-scheduler                                revision-pruner-6-ip-10-0-139-59.eu-west-2.compute.internal          0/1     Completed   0          90m
openshift-kube-scheduler                                revision-pruner-6-ip-10-0-141-15.eu-west-2.compute.internal          0/1     Completed   0          88m
openshift-kube-scheduler                                revision-pruner-6-ip-10-0-150-10.eu-west-2.compute.internal          0/1     Completed   0          91m
openshift-machine-api                                   cluster-autoscaler-operator-6844d54489-9sk6r                         1/1     Running     0          100m
openshift-machine-api                                   machine-api-controllers-5dbfc7b969-qq4w6                             3/3     Running     0          99m
openshift-machine-api                                   machine-api-operator-5db898985f-mj9qw                                1/1     Running     0          100m
openshift-machine-config-operator                       etcd-quorum-guard-6cc5dd7676-4ssxm                                   1/1     Running     0          98m
openshift-machine-config-operator                       etcd-quorum-guard-6cc5dd7676-7fzp9                                   1/1     Running     0          98m
openshift-machine-config-operator                       etcd-quorum-guard-6cc5dd7676-wj7x9                                   1/1     Running     0          98m
openshift-machine-config-operator                       machine-config-controller-69676779f4-4tngn                           1/1     Running     0          99m
openshift-machine-config-operator                       machine-config-daemon-f5wzf                                          1/1     Running     0          98m
openshift-machine-config-operator                       machine-config-daemon-gx9dw                                          1/1     Running     0          94m
openshift-machine-config-operator                       machine-config-daemon-jn6dt                                          1/1     Running     0          98m
openshift-machine-config-operator                       machine-config-daemon-wmvf2                                          1/1     Running     0          94m
openshift-machine-config-operator                       machine-config-daemon-z9n7l                                          1/1     Running     0          98m
openshift-machine-config-operator                       machine-config-operator-58dd66c9c-qd6sj                              1/1     Running     0          100m
openshift-machine-config-operator                       machine-config-server-7826q                                          1/1     Running     0          98m
openshift-machine-config-operator                       machine-config-server-jcsxr                                          1/1     Running     0          98m
openshift-machine-config-operator                       machine-config-server-t8rtx                                          1/1     Running     0          98m
openshift-marketplace                                   certified-operators-5f6b544444-dpgqt                                 1/1     Running     0          93m
openshift-marketplace                                   community-operators-f4c4b85c5-pwfs4                                  1/1     Running     0          93m
openshift-marketplace                                   marketplace-operator-7f844fc74-cj6h8                                 1/1     Running     0          94m
openshift-marketplace                                   redhat-operators-76dcfbbcfc-qpx8f                                    1/1     Running     0          93m
openshift-monitoring                                    alertmanager-main-0                                                  3/3     Running     0          92m
openshift-monitoring                                    alertmanager-main-1                                                  3/3     Running     0          92m
openshift-monitoring                                    alertmanager-main-2                                                  3/3     Running     0          92m
openshift-monitoring                                    cluster-monitoring-operator-bdfff7b89-ts4wn                          1/1     Running     0          94m
openshift-monitoring                                    grafana-d667f6d7b-pfwrp                                              2/2     Running     0          92m
openshift-monitoring                                    kube-state-metrics-794d7ffd-gkrgj                                    3/3     Running     0          93m
openshift-monitoring                                    node-exporter-4zkdn                                                  2/2     Running     0          93m
openshift-monitoring                                    node-exporter-dlg2n                                                  2/2     Running     0          93m
openshift-monitoring                                    node-exporter-mhpkk                                                  2/2     Running     0          93m
openshift-monitoring                                    node-exporter-pwt2t                                                  2/2     Running     0          93m
openshift-monitoring                                    node-exporter-wt5xg                                                  2/2     Running     0          93m
openshift-monitoring                                    prometheus-adapter-674448f46b-kxf2j                                  1/1     Running     0          91m
openshift-monitoring                                    prometheus-adapter-674448f46b-rjrjh                                  1/1     Running     0          91m
openshift-monitoring                                    prometheus-k8s-0                                                     6/6     Running     1          92m
openshift-monitoring                                    prometheus-k8s-1                                                     6/6     Running     1          92m
openshift-monitoring                                    prometheus-operator-b7d9db9d4-xv7t5                                  1/1     Running     0          91m
openshift-monitoring                                    telemeter-client-6445c9d89-w6vlf                                     3/3     Running     0          93m
openshift-multus                                        multus-2gv2p                                                         1/1     Running     0          94m
openshift-multus                                        multus-bl6fq                                                         1/1     Running     0          100m
openshift-multus                                        multus-gzw4k                                                         1/1     Running     0          100m
openshift-multus                                        multus-pjfpd                                                         1/1     Running     0          94m
openshift-multus                                        multus-z2bzd                                                         1/1     Running     0          99m
openshift-network-operator                              network-operator-7859cc7c8c-d4bqg                                    1/1     Running     0          100m
openshift-operator-lifecycle-manager                    catalog-operator-6f8b655bf4-v4682                                    1/1     Running     0          100m
openshift-operator-lifecycle-manager                    olm-operator-69bccbdcdc-528jb                                        1/1     Running     0          100m
openshift-operator-lifecycle-manager                    olm-operators-gsbbv                                                  1/1     Running     0          98m
openshift-operator-lifecycle-manager                    packageserver-69f7f6c9d8-hnz84                                       1/1     Running     0          97m
openshift-operator-lifecycle-manager                    packageserver-69f7f6c9d8-jjh4g                                       1/1     Running     0          97m
openshift-sdn                                           ovs-2hz9j                                                            1/1     Running     0          99m
openshift-sdn                                           ovs-g9c8f                                                            1/1     Running     0          94m
openshift-sdn                                           ovs-hvlcd                                                            1/1     Running     0          99m
openshift-sdn                                           ovs-swr59                                                            1/1     Running     0          99m
openshift-sdn                                           ovs-t2hj6                                                            1/1     Running     0          94m
openshift-sdn                                           sdn-54z5k                                                            1/1     Running     0          94m
openshift-sdn                                           sdn-75zxz                                                            1/1     Running     0          99m
openshift-sdn                                           sdn-controller-69nh8                                                 1/1     Running     0          99m
openshift-sdn                                           sdn-controller-6njhs                                                 1/1     Running     0          99m
openshift-sdn                                           sdn-controller-lblx9                                                 1/1     Running     0          99m
openshift-sdn                                           sdn-g5qrj                                                            1/1     Running     0          94m
openshift-sdn                                           sdn-qbglz                                                            1/1     Running     0          99m
openshift-sdn                                           sdn-rq9pk                                                            1/1     Running     0          99m
openshift-service-ca-operator                           service-ca-operator-84b4f66d79-qtxmd                                 1/1     Running     0          100m
openshift-service-ca                                    apiservice-cabundle-injector-9cbfd8669-2898c                         1/1     Running     0          99m
openshift-service-ca                                    configmap-cabundle-injector-6c9b5cd9b9-5chlw                         1/1     Running     0          99m
openshift-service-ca                                    service-serving-cert-signer-6b78d4df7f-tkc27                         1/1     Running     0          99m
openshift-service-catalog-apiserver-operator            openshift-service-catalog-apiserver-operator-7dc6c4cf5d-kjz8r        1/1     Running     0          95m
openshift-service-catalog-controller-manager-operator   openshift-service-catalog-controller-manager-operator-79d9gwtsw      1/1     Running     0          95m

```


## 6.  identity providers configuration

- The OpenShift Container Platform master includes a built-in OAuth server. Developers and administrators obtain OAuth access tokens to authenticate themselves to the API.

- By default, only a kubeadmin user exists on your cluster. To specify an identity provider, you must create a Custom Resource (CR) that describes that identity provider and add it to the cluster.

- OpenShift Container Platform user names containing /, :, and % are not supported.


### 6.1 Configuring a GitHub or GitHub Enterprise identity provider

- For GitHub Enterprise, go to your GitHub Enterprise home page and then click Settings → Developer settings → Register a new application.
	- ocp4-tst-001

- Home page URL: https://oauth-openshift.apps.ocp4-tst-001.iaciscp.net

- Authorization call back: https://oauth-openshift.apps.ocp4-tst-001.iaciscp.net/oauth2callback/githubidp/
	- IMPORTANT: githubidp must match with the identity provider name of the custom resource

- click Register application. And this will return a valid **client ID** and **client secret**

```
Client ID
5e469bcb2fe54c9e6fd1
Client Secret
0b9f6a17c17f2521c51f414f65ddac9568f2ac2e
```

#### 6.1.1 Creating the secret

- Identity providers use OpenShift Container Platform Secrets in the openshift-config namespace to contain the client secret, client certificates, and keys.

```
oc create secret generic githubclientsecret --from-literal=clientSecret=0b9f6a17c17f2521c51f414f65ddac9568f2ac2e -n openshift-config
```

#### 6.1.2 Creating a configmap

- Identity providers use OpenShift Container Platform ConfigMaps in the openshift-config namespace to contain the certificate authority bundle. These are primarily used to contain certificate bundles needed by the identity provider.

- Export certificate from github.com 

- In this case we don't have certificates on our openshift platform, but in case we do: (THIS STEP IS NOT NECESSARY)

```
oc create configmap ca-github-config-map --from-file=ca.crt=github_ca.crt -n openshift-config
```

#### 6.1.2 Creating a github Custom Resource

- clientID: is the client returned when configuring github identity provider
- clientSecret: is the secret text returned when configuring github identity provider

> With this OAuth configuration every user with access to the d-lab github org can access the ocp cluster as admin default role.

```
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
    - challenge: false
      github:
        clientID: 5e469bcb2fe54c9e6fd1
        clientSecret:
          name: githubclientsecret
        hostname: github.ibm.com
        organizations:
          - d-lab
      login: true
      mappingMethod: claim
      name: githubidp
      type: GitHub
```


### 6.1 Configuring a HTPasswd identity provider

This step was done with the UI, by loading a .htpasswd file.

> Administration - cluster settings - global configuration - OAuth - Identity providers - Add


## 7.  RBAC

- Create a cluster role binding for github-cluster-admin-group. This group is for cluster-admin only.

```
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: github-cluters-admin
subjects:
  - kind: Group
    apiGroup: rbac.authorization.k8s.io
    name: github-cluster-admin-group
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
```

- New group and user added
```
oc adm groups new github-cluster-admin-group Miguel-Angel-Soranno
```

- Add more users to group

```
oc adm groups add-users <group> <user>
```

- Remove member from group

```
oc adm groups remove-users github-cluster-admin-group miguel.angel.soranno@ibm.com
```

## 8. Removing the kubeadmin user

After you define an identity provider and create a new cluster-admin user, you can remove the kubeadmin to improve cluster security.

```
oc delete secrets kubeadmin -n kube-system
```
