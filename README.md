# Implement-Private-Google-Access-and-Cloud-NAT

This process happenes with google cloud platform 


Create a VPC network and firewall rules
First, create a VPC network for the VM instance and a firewall rule to allow SSH access.

In the Cloud Console, on the Navigation menu (Navigation menu), click VPC network > VPC networks.

Click Create VPC network.

For Name, type privatenet.

For Subnet creation mode, click Custom.

Specify the following, and leave the remaining settings as their defaults:

Property	Value (type value or select option as specified)
Name	privatenet-us
Region	us-central1
IP address range	10.130.0.0/20
Don't enable Private Google access yet!
Click Done.

Click Create and wait for the network to be created.

In the left pane, click Firewall.

Click Create firewall rule.

Specify the following, and leave the remaining settings as their defaults:

Property	Value (type value or select option as specified)
Name	privatenet-allow-ssh
Network	privatenet
Targets	All instances in the network
Source filter	IP ranges
Source IP ranges	35.235.240.0/20
Protocols and ports	Specified protocols and ports
For tcp, click the checkbox and specify port 22.

Click Create.

In order to connect to your private instance using SSH, you need to open an appropriate port on the firewall. IAP connections come from a specific set of IP addresses (35.235.240.0/20). Therefore, you can limit the rule to this CIDR range.
Create the VM instance with no public IP address
In the Cloud Console, on the Navigation menu (Navigation menu), click Compute Engine > VM instances.

Click Create.

Specify the following, and leave the remaining settings as their defaults:

Property	Value (type value or select option as specified)
Name	vm-internal
Region	us-central1
Zone	us-central1-c
Series	N1
Machine type	n1-standard-1 (1vCPU, 3.75 GB memory)
Boot disk	Debian GNU/Linux 10 (buster)
Click Management, security, disks, networking, sole tenancy.

Click Networking.

For Network interfaces, click the pencil icon to edit.

Specify the following, and leave the remaining settings as their defaults:

Property	Value (type value or select option as specified)
Network	privatenet
Subnetwork	privatenet-us
External IP	None
The default setting for a VM instance is to have an ephemeral external IP address. This behavior can be changed with a policy constraint at the organization or project level. To learn more about controlling external IP addresses on VM instances, refer to the external IP address documentation.
Click Done.
Click Create, and wait for the VM instance to be created.
On the VM instances page, verify that the External IP of vm-internal is None.
Click Check my progress to verify the objective.
Create the VM instance

SSH to vm-internal to test the IAP tunnel
In the Cloud Console, click Activate Cloud Shell (Cloud Shell).

If prompted, click Continue.

To connect to vm-internal, run the following command:

gcloud compute ssh vm-internal --zone us-central1-c --tunnel-through-iap
If prompted about continuing, type Y.
When prompted for a passphrase, press ENTER.
When prompted for the same passphrase, press ENTER.
Did the command prompt change to @vm-internal?

True

False

To test the external connectivity of vm-internal, run the following command:

ping -c 2 www.google.com
This should not work because vm-internal has no external IP address!

Wait for the ping command to complete.

To return to your Cloud Shell instance, run the following command:

exit
When instances do not have external IP addresses, they can only be reached by other instances on the network via a managed VPN gateway or via a Cloud IAP tunnel. Cloud IAP enables context-aware access to VMs via SSH and RDP without bastion hosts. For more information about this, see this blog post.
Task 2. Enable Private Google Access
VM instances that have no external IP addresses can use Private Google Access to reach external IP addresses of Google APIs and services. By default, Private Google Access is disabled on a VPC network.

Create a Cloud Storage bucket
Create a Cloud Storage bucket to test access to Google APIs and services.

In the Cloud Console, on the Navigation menu (Navigation menu), click Storage > Browser.

Click Create bucket.

Specify the following, and leave the remaining settings as their defaults:

Property	Value (type value or select option as specified)
Name	Enter a globally unique name
Location type	Multi-region
Click Create.

Note the name of your storage bucket for the next subtask. It will be referred to as [my_bucket].

Copy an image file into your bucket
Copy an image from a public Cloud Storage bucket to your own bucket.

In Cloud Shell, run the following command, replacing [my_bucket] with your bucket's name:

gsutil cp gs://cloud-training/gcpnet/private/access.svg gs://[my_bucket]
In the Cloud Console, click Refresh Bucket to verify that the image was copied.
You can click on the name of the image in the Cloud Console to view an example of how Private Google Access is implemented.

Access the image from your VM instance
Currently, which of your VM instances can access the image from your bucket?

Cloud Shell

vm-internal

In Cloud Shell, to try to copy the image from your bucket, run the following command, replacing [my_bucket] with your bucket's name:

gsutil cp gs://[my_bucket]/*.svg .
This should work because Cloud Shell has an external IP address!

To connect to vm-internal, run the following command:

gcloud compute ssh vm-internal --zone us-central1-c --tunnel-through-iap
If prompted, type Y to continue.

To try to copy the image to vm-internal, run the following command, replacing [my_bucket] with your bucket's name:

gsutil cp gs://[my_bucket]/*.svg .
This should not work: vm-internal can only send traffic within the VPC network because Private Google Access is disabled (by default).

Press Ctrl+C to stop the request.

Enable Private Google Access
Private Google Access is enabled at the subnet level. When it is enabled, instances in the subnet that only have private IP addresses can send traffic to Google APIs and services through the default route (0.0.0.0/0) with a next hop to the default internet gateway.

In the Cloud Console, on the Navigation menu (Navigation menu), click VPC network > VPC networks.
Click privatenet-us to open the subnet.
Click Edit.
For Private Google access, select On.
Click Save.
Click Check my progress to verify the objective.
Create a Cloud Storage bucket and Enable Private Google Access

Yes, enabling Private Google Access is as simple as selecting On within the subnet!
In Cloud Shell for vm-internal, to try to copy the image to vm-internal, run the following command, replacing [my_bucket] with your bucket's name:

gsutil cp gs://[my_bucket]/*.svg .
This should work because vm-internal's subnet has Private Google Access enabled!

To return to your Cloud Shell instance, run the following command:

exit
To view the eligible APIs and services that you can use with Private Google Access, see supported services in the Private Google Access overview.
Task 3. Configure a Cloud NAT gateway
Although vm-internal can now access certain Google APIs and services without an external IP address, the instance cannot access the internet for updates and patches. Configure a Cloud NAT gateway, which allows vm-internal to reach the internet.

Try to update the VM instances
In Cloud Shell, to try to re-synchronize the package index, run the following:

sudo apt-get update
The output should finish like this (do not copy; this is example output):

...
Reading package lists... Done
This should work because Cloud Shell has an external IP address!

To connect to vm-internal, run the following command:

gcloud compute ssh vm-internal --zone us-central1-c --tunnel-through-iap
If prompted, type Y to continue.

To try to re-synchronize the package index of vm-internal, run the following command:

sudo apt-get update
This should only work for Google Cloud packages because vm-internal only has access to Google APIs and services!

Press Ctrl+C to stop the request.

Configure a Cloud NAT gateway
Cloud NAT is a regional resource. You can configure it to allow traffic from all ranges of all subnets in a region, from specific subnets in the region only, or from specific primary and secondary CIDR ranges only.

In the Cloud Console, on the Navigation menu (Navigation menu), click Network services > Cloud NAT.

Click Get started to configure a NAT gateway.

Specify the following:

Property	Value (type value or select option as specified)
Gateway name	nat-config
VPC network	privatenet
Region	us-central1
For Cloud Router, select Create new router.

For Name, type nat-router

Click Create.

The NAT mapping section allows you to choose the subnets to map to the NAT gateway. You can also manually assign static IP addresses that should be used when performing NAT. Do not change the NAT mapping configuration in this lab.
Click Create.
Wait for the gateway's status to change to Running.
Click Check my progress to verify the objective.
Configure a Cloud NAT gateway

Verify the Cloud NAT gateway
It may take up to 3 minutes for the NAT configuration to propagate to the VM, so wait at least a minute before trying to access the internet again.

In Cloud Shell for vm-internal, to try to re-synchronize the package index of vm-internal, run the following command:

sudo apt-get update
The output should finish like this (do not copy; this is example output):

...
Reading package lists... Done
This should work because vm-internal is using the NAT gateway!

To return to your Cloud Shell instance, run the following command:

exit
The Cloud NAT gateway implements outbound NAT, but not inbound NAT. In other words, hosts outside of your VPC network can only respond to connections initiated by your instances; they cannot initiate their own, new connections to your instances via NAT.
Task 4. Configure and view logs with Cloud NAT Logging
Cloud NAT logging allows you to log NAT connections and errors. When Cloud NAT logging is enabled, one log entry can be generated for each of the following scenarios:

When a network connection using NAT is created.
When a packet is dropped because no port was available for NAT.
You can opt to log both kinds of events, or just one or the other. Created logs are sent to Cloud Logging.

Enabling logging
If logging is enabled, all collected logs are sent to Cloud Logging by default. You can filter these so that only certain logs are sent.

You can also specify these values when you create a NAT gateway or by editing one after it has been created. The following directions show how to enable logging for an existing NAT gateway.

In the GCP Console, on the Navigation menu (Navigation menu), click Network services > Cloud NAT.

Click on the nat-config gateway and then click Edit.

Click the Advanced configurations dropdown to open that section.

Under Stackdriver logging, select Translation and errors and then click Save.

NAT logging in Cloud Operations
Now that you have set up Cloud NAT logging for the nat-config gateway, let's find out where we can view our logs.

Click on nat-config to expose it's details. Then click on the Logs tab. Then click the link to Stackdriver Logging.

This will open a new tab with Operations logging.

You will see that there aren't any logs yet—that's because we just enabled this feature for the gateway. Keep this tab open and return to your other GCP Console tab.

Generating logs
As a reminder, Cloud NAT logs are generated for the following sequences:

When a network connection using NAT is created.
When a packet is dropped because no port was available for NAT.
Let's connect the host to the internal VM again to see if any logs are generated.

In Cloud Shell for vm-internal, to try to re-synchronize the package index of vm-internal, run the following command:

gcloud compute ssh vm-internal --zone us-central1-c --tunnel-through-iap
If prompted, type Y to continue.

Try to re-synchronize the package index of vm-internal by running the following:

sudo apt-get update
The output should look like this (do not copy; this is example output):

...
Reading package lists... Done
To return to your Cloud Shell instance, run the following command:

exit
Let's see if opening up this connection revealed anything new in our logs.

Viewing Logs
Return to your NAT Logging tab and under Configure dropdown, select Show newest logs first.

You should 
