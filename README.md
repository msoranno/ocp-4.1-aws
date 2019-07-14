# OCP 4.1 on AWS (Official Documentation)

https://docs.openshift.com/container-platform/4.1/installing/installing_aws/installing-aws-account.html

## Configuring Route53

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

## AWS account limits

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

## Resources to be deployed

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

