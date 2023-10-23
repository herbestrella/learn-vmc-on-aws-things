# VMC-on-AWS VPN Connectivity Options

## Goals
- Configure VPN connection from SDDC to On-Prem Datacenter
- Import a VM from On-prem vCenter to SDDC vCenter

[Overview of Network Connectivity Options](https://youtu.be/y-Likfr6mxM?si=8bvBGtts6ArUsDRL)

## VPN Options
- **Route Based VPN with BGP**
    - [VMware Documentation](https://docs.vmware.com/en/VMware-Cloud-on-AWS/services/com.vmware.vmc-aws-networking-security/GUID-5AF45CE6-FA53-45C0-83E5-25F8E3A055E9.html)
- Policy Based VPN
    - [VMware Documentation](https://docs.vmware.com/en/VMware-Cloud-on-AWS/services/com.vmware.vmc-aws-networking-security/GUID-586C053D-9553-461E-B6A8-FF508C8F091C.html)
- L2VPN Client using the "NSX Autonomous Edge"
    - [Using NSX Autonomous Edge to extend L2 networks from on-prem to VMC on AWS](https://jonamiki.com/2021/09/03/using-nsx-autonomous-edge-to-extend-l2-networks-to-the-cloud-without-vds-vmware-vsphere-enterprise-plus-licenses/)

Summary: I ended up going with a Route Based VPN with BGP for my setup, I discovered some limitations with the Palo Alto I was using and going with Policy Based, I believe I have to use ProxyIDs if I want it to work but I stopped short there and switched to BGP which was a few additional configurations.

## Configure Palo Alto

### Create IPSec Crypto Profile on Palo Alto
![Route-Based VPN](/assets/VMC-on-AWS-Trial-images/Route-based-vpn-1.png)

### Create IKE Crypto Profile on Palo Alto
![Route-Based VPN](/assets/VMC-on-AWS-Trial-images/Route-based-vpn-2.png)

### Create IKE Gateway Profile on Palo Alto
![Route-Based VPN](/assets/VMC-on-AWS-Trial-images/Route-based-vpn-3.png)

![Route-Based VPN](/assets/VMC-on-AWS-Trial-images/Route-based-vpn-4.png)

### Create Tunnel Interface
![Route-Based VPN](/assets/VMC-on-AWS-Trial-images/Route-based-vpn-5.png)

![Route-Based VPN](/assets/VMC-on-AWS-Trial-images/Route-based-vpn-6.png)

### Create IPSec Tunnel
![Route-Based VPN](/assets/VMC-on-AWS-Trial-images/Route-based-vpn-7.png)

### Create Firewall Rules
![Route-Based VPN](/assets/VMC-on-AWS-Trial-images/Route-based-vpn-8.png)

![Route-Based VPN](/assets/VMC-on-AWS-Trial-images/Route-based-vpn-9.png)

## BGP Configurations

### (default) Virtual Router BGP Settings
- Reject Default Route
- AS Number
- Router ID
    - not essential for the VPN process but should have a Router ID defined
- Auth Profile
    - Create one as you'll need this for the SDDC and the Peer Group

![Route-Based VPN](/assets/VMC-on-AWS-Trial-images/Route-based-vpn-10.png)

### Create Peer Group

![Route-Based VPN](/assets/VMC-on-AWS-Trial-images/Route-based-vpn-11.png)

- Define the name of the Peer Group
- Peer AS on the SDDC is 6500 by default
- Local Address = your tunnel interface you created
    - ex. 1.1.1.3/29 (Prefix Length)
- Peer Address = the address of your SDDC
    - ex. 1.1.1.2 (IP only defined)

![Route-Based VPN](/assets/VMC-on-AWS-Trial-images/Route-based-vpn-12.png)

- Select the AUTH Profile

![Route-Based VPN](/assets/VMC-on-AWS-Trial-images/Route-based-vpn-13.png)

Route Redistribution - it is necessary if you're using multiple VPN connections in your SDDC to not select "Allow Redistribute Default Route" in my case - I have a static route going to my ISP (ex. 0.0.0.0/0 -> eth1) when this is selected it will redistribute that to the SDDC and may cause issues with other VPN connections - my on-prem NSX had to modified as well to add a static route (see section below)

![Route-Based VPN](/assets/VMC-on-AWS-Trial-images/Route-based-vpn-14.png)

![Route-Based VPN](/assets/VMC-on-AWS-Trial-images/Route-based-vpn-15.png)

## Configure NSX on SDDC
### Configure Route Based IPSEC VPN

- Name
- Local IP Address
- Remote Public IP
- BGP Local IP/Prefix Length
- BGP Remote IP
- BGP Neigbor ASN
- Preshared Key
- Secret


The rest of the settings can be left default if you used the settings as described in the Palo Alto section above


![Route-Based VPN](/assets/VMC-on-AWS-Trial-images/Route-based-vpn-16.png)

Create a Management Group with IP addresses you want to have access to Management Assests in your SDDC

![Route-Based VPN](/assets/VMC-on-AWS-Trial-images/Route-based-vpn-17.png)

Create a Compute Group with IP addresses you want to have access to Compute Assests in your SDDC

![Route-Based VPN](/assets/VMC-on-AWS-Trial-images/Route-based-vpn-18.png)

Using the Compute Group you created create a Gateway Rule under the compute section to allow VPN access; defining the "BGP Subnet" will allow BGP traffic to your On-Prem along with the Policy created on the Palo Alto. Ensure to change the "Applied To" 

![Route-Based VPN](/assets/VMC-on-AWS-Trial-images/Route-based-vpn-19.png)

From here if you have all your configurations in place your Tunnel will be available and BGP will be established.

![Route-Based VPN](/assets/VMC-on-AWS-Trial-images/Route-based-vpn-20.png)

Tunnel is active on the Palo Alto

![Route-Based VPN](/assets/VMC-on-AWS-Trial-images/Route-based-vpn-21.png)

Learned Routes on the SDDC show networks from the On-Prem

![Route-Based VPN](/assets/VMC-on-AWS-Trial-images/Route-based-vpn-22.png)

## (Optional) On-prem NSX-T Static Route
In my setup my on-prem Palo’s BGP settings had “Allow Redistribute Default Route” as well as a static route of 0.0.0.0/0 to my eth1 (Untrust) ISP connection. 

When this was enabled 0.0.0.0/0 was present on the SDDC learned routes which caused access interruption on another VPN connection we had set up on the SDDC.

The fix was to not allow redistribute default route and set up a static route on the on-prem T0 gateway to another gateway (on the Palo) that could reach the internet. In my case, my “Edge Uplink” network gateways (which are sub-interfaces on the Palo) have access to the internet. 

![on-prem-nsx](/assets/VMC-on-AWS-Trial-images/on-prem-static-1.png)

![on-prem-nsx](/assets/VMC-on-AWS-Trial-images/on-prem-static-1.png)

## Helpful Links
[VMware VMC on AWS Policy Based VPN Example](https://youtu.be/XZ3ra2YbanA?si=JEv_iXXqWEEJN1xE) - Part of the VMC on AWS quickstart series - dives into the configuration of the Policy Based VPN setup from the SDDC side in NSX.

[Palo Alto How to Configure IPSEC VPN](https://knowledgebase.paloaltonetworks.com/KCSArticleDetail?id=kA10g000000ClGkCAK) - Generic Palo Alto configuration of IPSEC VPN

[How to check Status, Clear, Restore, and Monitor an IPSEC VPN Tunnel](https://knowledgebase.paloaltonetworks.com/KCSArticleDetail?id=kA10g000000ClVGCA0) - Post Palo Alto IPSEC VPN creation to check status and monitor the newly established VPN Tunnel.

[VMC on AWS to Palo Alto - Route based IPSEC VPN](https://cloudadvisors.net/2023/01/27/vmc-on-aws-to-palo-alto-route-based-ipsec-vpn/) - Great article and guide for setting up the VPN Tunnel - the key piece was allowing the BGP network on the SDDC NSX side and 

[Palo Alto BGP over IPSEC Route Based VPN Tunnel](https://www.youtube.com/watch?v=xU708Go_Sz4) - more information on BGP over IPSEC