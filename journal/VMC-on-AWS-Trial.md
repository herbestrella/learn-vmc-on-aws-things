# VMC-on-AWS-Trial

## Goals
- Successfully sign up for VMC on AWS Trial
- Create (VMC) SDDC

## Pre-requisites
- VMWare Customer Connect Account 
    - [VMware Customer Connect](https://customerconnect.vmware.com/home)
- AWS Account
    - [AWS Free Tier](https://aws.amazon.com/free/?trk=fce796e8-4ceb-48e0-9767-89f7873fac3d&sc_channel=ps&ef_id=CjwKCAjwysipBhBXEiwApJOcu3i2DNJ0GD1-VOYKmPYjSW09SKIUfNGo7eccOgxgDD952ar4f-1n4RoC1AEQAvD_BwE:G:s&s_kwcid=AL!4422!3!592542020599!e!!g!!aws!1644045032!68366401852&all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc&awsf.Free%20Tier%20Types=*all&awsf.Free%20Tier%20Categories=*all)

## Trial Period (30 Days)

Sign up for the free trial, my assumption is that you will need a justifiable use-case. In my case I have a registered company name with a matching domain name. When VMware reached out I gave them my justificaiton and it was accepted!

You can signup for the trial period here: [VMC on AWS Trial Signup](https://www.vmware.com/products/vmc-on-aws/free-trial.html?src=ps_39szbtx12scfm&cid=7012H000000sxcfQAA&gclid=CjwKCAjwp8OpBhAFEiwAG7NaEqMhZTF94TyH7LWbsnJhaKYg30NIKXjPENGrWWMo7aRUP-5kpaEcxRoCv50QAvD_BwE&gclsrc=aw.ds)

Vendor Documentation regarding the trial: [VMC on AWS Trial Official Documentation](https://docs.vmware.com/en/VMware-Cloud-on-AWS/services/com.vmware.vmc-aws.getting-started/GUID-2844809E-3C4A-49D9-9068-191E6985CE5C.html)

Reference Architecture: [Getting Started with VMC on AWS Reference Architecture](https://www.vmware.com/content/dam/digitalmarketing/vmware/en/pdf/docs/vmw-getting-started-with-vmc-on-aws-reference-architecture.pdf)

Super helpful YouTube "Quick Start Series" [VMWare Cloud on AWS Quick Start](https://youtube.com/playlist?list=PLNOz1mVhDkG5JH3JqN1yPSfenGdizXqwh&si=sUJ7oUPVq6URgx_z)

If you have never used VMC on AWS and would like a primer, I highly suggest trying out the [VMware Cloud on AWS - Fundamentals - Hands-on Lab](https://customerconnect.vmware.com/en/evalcenter?p=vmc-aws-hol-gen-23)

![VMC on AWS Trial Email Response](/assets/VMC-on-AWS-Trial-images/vmc-trial.png)

## Networking

### On-prem Networks

| Name | Subnet | VLAN ID | Usable Addresses |
| ----- | ----- | ----- | ----- |
| Management | 10.120.0.0/16 | 10 | 10.120.0.1 - 10.120.255.254 |
| user-vlan-11 | 192.168.250.0/24 | 11 | 192.168.250.1 - 192.168.250.254 |

### VMC on AWS (example)

| Component | Value | Usable Addresses |
| ---- | ---- | ---- |
| AWS Region | us-east-1 | n/a |
| VPC Name | VMC-VPC | n/a |
| VPC CIDR Block | 10.100.0.0/21 | 10.100.0.1 - 10.100.7.254 |
| SDDC Name | SDDC-1-DAF1 | n/a |
| SDDC Management CIDR Block (default) | 10.2.0.0/16 | 10.2.0.1 - 10.2.255.254 |

### Create VPC Subnets as needed (example)

| Subnet Purpose | Availability Zone A | Availability Zone B |
| ---- | ---- | ---- |
| Private Subnet | 10.100.**0**.0/23 | 10.100.**2**.0/23 |
| Connected SDDC | 10.100.**4**.0/24 | 10.100.**5**.0/24 |
| Public Subnet | 10.100.**6**.0/24 | 10.100.**7**.0/24 |

## Create VPC

## Create SDDC
