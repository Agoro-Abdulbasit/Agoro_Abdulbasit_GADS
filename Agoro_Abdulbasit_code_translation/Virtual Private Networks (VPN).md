# Virtual Private Networks (VPN)

## Overview

In this lab, you establish VPN tunnels between two networks in separate regions such that a VM in one network can ping a VM in the other network over its internal IP address.

## Objectives

In this lab, you learn how to perform the following tasks:

- Create VPN gateways in each network

- Create VPN tunnels between the gateways

- Verify VPN connectivity

  

## Task 1: Explore the networks and instances

Two custom networks with VM instances have been configured for you. For the purposes of the lab, both networks are VPC networks within a Google Cloud project. However, in a real-world application, one of these networks might be in a different Google Cloud project, on-premises, or in a different cloud.

1. Verify that vpn-network-1 and vpn-network-2 have been created with subnets in separate regions:

```
gcloud compute networks list
gcloud compute networks subnets list
```

2. Explore the firewall rules:

```
gcloud compute firewall-rules list
```

3. Explore the instances :

```
gcloud compute instances list
```

* Note both instances external and internal IP address

4. Explore connectivity between the instances

   * SSH into server-1 and connect to it:

     ```
     gcloud compute ssh server-1 --zone=us-central1-a
     ```

   * To test connectivity to server-2's external IP address, run the following command, replacing server-2's external IP address with the value noted earlier:

     ```
     ping -c 3 <Enter server-2's external IP address here>
     ```

   ​      This works because the VM instances can communicate over the internet.

   * To test connectivity to server-2's internal IP address, run the following command, replacing server-2's internal IP address with the value noted earlier:

     ```
     ping -c 3 <Enter server-2's internal IP address here>
     ```

     You should see 100% packet loss when pinging the internal IP address because you don't have VPN connectivity yet.

   * Exit the SSH terminal:

     ```
     exit
     ```

   * SSH into server-2 and connect to it:

     ```
     gcloud compute ssh server-2 --zone=europe-west1-c
     ```

   * To test connectivity to server-2's external IP address, run the following command, replacing server-2's external IP address with the value noted earlier:

   ```
       ping -c 3 <Enter server-1's external IP address here>
   ```

   ​      This works because the VM instances can communicate over the internet.

   * To test connectivity to server-2's internal IP address, run the following command, replacing server-2's internal IP address with the value noted earlier:

     ```
     ping -c 3 <Enter server-1's internal IP address here>
     ```

     You should see 100% packet loss when pinging the internal IP address because you don't have VPN connectivity yet.

   * Exit the SSH terminal:

     ```
     exit
     ```

   

## Task 2: Create the VPN gateways and tunnels

Establish private communication between the two VM instances by creating VPN gateways and tunnels between the two networks.

1. ### **Reserve two static IP addresses**

   Reserve one static IP address for each VPN gateway.

   * reserve static address vpn-1-static-ip with the following properties:

     | Property   | Value           |
     | ---------- | --------------- |
     | Name       | vpn-1-static-ip |
     | IP version | IPv4            |
     | Region     | us-central1     |

     ```
     gcloud compute addresses create vpn-1-static-ip --project=qwiklabs-gcp-01-948fe2c90113 --region=us-central1
     ```

   * Reserve static address vpn-1-static-ip with the following properties:

     | Property   | Value           |
     | ---------- | --------------- |
     | Name       | vpn-2-static-ip |
     | IP version | IPv4            |
     | Region     | europe-west-1   |

     ```
     gcloud compute addresses create vpn-2-static-ip --project=qwiklabs-gcp-01-948fe2c90113 --region=europe-west1
     ```

     Note both IP addresses for the next step. They will be referred to us `[VPN-1-STATIC-IP]` and `[VPN-2-STATIC-IP]`.

2. ### Create the vpn-1 gateway and tunnel1to2

   * Create Vpn-1 gateway with the following properties:

     | Property   | Value           |
     | ---------- | --------------- |
     | Name       | vpn-1           |
     | Network    | vpn-network-1   |
     | Region     | us-central1     |
     | IP address | vpn-1-static-ip |

     ```
     gcloud compute --project "Project_id" target-vpn-gateways create "vpn-1" --region "us-central1" --network "vpn-network-1"
     
     gcloud compute --project "Project_id" forwarding-rules create "vpn-1-rule-esp" --region "us-central1" --address "[VPN-1-STATIC-IP]" --ip-protocol "ESP" --target-vpn-gateway "vpn-1"
     
     gcloud compute --project "Project_id" forwarding-rules create "vpn-1-rule-udp500" --region "us-central1" --address "[VPN-1-STATIC-IP]" --ip-protocol "UDP" --ports "500" --target-vpn-gateway "vpn-1"
     
     gcloud compute --project "Project_id" forwarding-rules create "vpn-1-rule-udp4500" --region "us-central1" --address "[VPN-1-STATIC-IP]" --ip-protocol "UDP" --ports "4500" --target-vpn-gateway "vpn-1"
     ```

   * Create tunnel1to2 with the following properties:

     | Property                 | Value             |
     | ------------------------ | ----------------- |
     | Name                     | tunnel1to2        |
     | Remote peer IP address   | [VPN-2-STATIC-IP] |
     | IKE pre-shared key       | gcprocks          |
     | Routing options          | Route-based       |
     | Remote network IP ranges | 10.1.3.0/24       |

     ```
     gcloud compute --project "Project_id" vpn-tunnels create "tunnel1to2" --region "us-central1" --peer-address "[VPN-2-STATIC-IP]" --shared-secret "gcprocks" --ike-version "2" --local-traffic-selector "0.0.0.0/0" --target-vpn-gateway "vpn-1"
     
     gcloud compute --project "Project_id" routes create "tunnel1to2-route-1" --network "vpn-network-1" --next-hop-vpn-tunnel "tunnel1to2" --next-hop-vpn-tunnel-region "us-central1" --destination-range "10.1.3.0/24"
     
     ```

     

3. Create the vpn-2 gateway and tunnel2to1

* Create Vpn-2 gateway with the following properties:

  | Property   | Value           |
  | ---------- | --------------- |
  | Name       | vpn-2           |
  | Network    | vpn-network-2   |
  | Region     | europe-west1    |
  | IP address | vpn-2-static-ip |

  ```
  gcloud compute --project "Project_ID" target-vpn-gateways create "vpn-2" --region "europe-west1" --network "vpn-network-2"
  
  gcloud compute --project "Project_ID" forwarding-rules create "vpn-2-rule-esp" --region "europe-west1" --address "[VPN-2-STATIC-IP]" --ip-protocol "ESP" --target-vpn-gateway "vpn-2"
  
  gcloud compute --project "Project_ID" forwarding-rules create "vpn-2-rule-udp500" --region "europe-west1" --address "[VPN-2-STATIC-IP]" --ip-protocol "UDP" --ports "500" --target-vpn-gateway "vpn-2"
  
  gcloud compute --project "Project_ID" forwarding-rules create "vpn-2-rule-udp4500" --region "europe-west1" --address "[VPN-2-STATIC-IP]" --ip-protocol "UDP" --ports "4500" --target-vpn-gateway "vpn-2"
  
  ```

  

* Create tunnel2to1 with the following properties:

  | Property                 | Value             |
  | ------------------------ | ----------------- |
  | Name                     | tunnel2to1        |
  | Remote peer IP address   | [VPN-1-STATIC-IP] |
  | IKE pre-shared key       | gcprocks          |
  | Routing options          | Route-based       |
  | Remote network IP ranges | 10.5.4.0/24       |

  ```
  gcloud compute --project "Project_ID" vpn-tunnels create "tunnel2to1" --region "europe-west1" --peer-address "[VPN-1-STATIC-IP]" --shared-secret "gcprocks" --ike-version "2" --local-traffic-selector "0.0.0.0/0" --target-vpn-gateway "vpn-2"
  
  gcloud compute --project "Project_ID" routes create "tunnel2to1-route-1" --network "vpn-network-2" --next-hop-vpn-tunnel "tunnel2to1" --next-hop-vpn-tunnel-region "europe-west1" --destination-range "10.5.4.0/24"
  ```

## Task 3: Verify VPN connectivity

### **Verify server-1 to server-2 connectivity**

1.  SSH into server-1 and connect:

   ```
   gcloud compute ssh server-1 --zone=us-central1-a
   ```

2.  Test connectivity to server-2's internal IP address:

   ```
   ping -c 3 <insert server-2's internal IP address here>
   ```

3. Exit server-1 SSH terminal:

   ```
   exit
   ```

4. SSH into server-2  and connect:

   ```
   gcloud compute ssh server-2 --zone=europe-west1-c
   ```

5. Test connectivity into server-1 internal IP address:

   ```
   ping -c 3 <insert server-1's internal IP address here>
   ```

6. Exit server-2 SSH terminal:

   ```
   exit
   ```

### **Remove the external IP addresses**

Now that you verified VPN connectivity, you can remove the instances' external IP addresses. For demonstration purposes, just do this for the **server-1** instance.

1. Stop server-1 using the following code:

   ```
   gcloud compute instances stop server-1
   ```

To remove server-1's external IP address you have to delete  the existing access configs

2. Check for server-1's acces config name with the following code:

   ```
   gcloud compute instances describe server-1
   ```

   The result should be in the following format although the access config name can differ:

   ```
   networkInterfaces:
   - accessConfigs:
     - kind: compute#accessConfig
       name: external-nat
       natIP: 130.211.181.55
       type: ONE_TO_ONE_NAT
   ```

 3. Delete the access config file with the following command:

    ```
    gcloud compute instances add-access-config server-1 --access-config-name "ACCESS_CONFIG_NAME" --address Server-1_external_IP_address
    ```

Start server-1 again to confirm that the external IP address doesn't exist

1. Start server-1 with the following command

   ```
   gcloud compute instances start server-1
   ```

2. Check if server-1 has an external IP address with the following command:

   ```
   gcloud compute instances list
   ```

   

## Task 4: Review

In this lab, you configured a VPN connection between two networks with subnets in different regions. Then you verified the VPN connection by pinging VMs in different networks using their internal IP addresses.

You configured the VPN gateways and tunnels using the Cloud Console. However, this approach obfuscated the creation of forwarding rules, which you explored with the command line button in the Console. This can help in troubleshooting a configuration.

## End your lab