<h2 align="center"> CREATING AND MOVING AN EBS VOLUME FROM ONE AVAILABILITY ZONE TO ANOTHER</h2>

The purpose of this project is to demonstrate how to move an EBS volume from one availability zone to another.

<h2 align="left"> Case Study</h2>
Suppose a company's web application (Online Stores) relies on a database hosted by an EC2 instance, backed by an EBS volume that stores all the application data, in one availability zone in a region. Suppose the database performance has degraded (server becomes very slow) due to increased load (many organizations owning resources) in that availability zone, the company can decide to move the EBS volume to a different availability zone within the same region with less congestion. A snapshot of the EBS volume would have to be taken, which would be used as a mirror to copy all the data to the EBS volume attached to the database server (EC2 Instance) in the new availability zone. A new server, (if there are no existing ones) would have to be created and the EBS volume attached to it. The old EBS volume, server and snapshot can then be deleted or re-assigned to other tasks, once the web application works perfectly. A major consideration that needs to be factored in, during this process is the downtime during the movement process. 

<p align="center">
<img src="network architecture" height="50%" width="70%"/>
</p>
<h5 align="center"> Network Architecture</h5>

<h2 align="left"> Creating the VPC</h2>

Based on the network architecture, I created a VPC with a CIDR of 192.168.0.0/16.

<p align="center">
<img src="vpc1" height="40%" width="60%"/>
</p>

<p align="center">
<img src="vpc2" height="40%" width="60%"/>
</p>

<h2 align="left"> Creating Internet Gateway</h2>

Next, I created the internet gateway which I attached to the vpc.

<p align="center">
<img src="igw" height="40%" width="60%"/>
</p>

<p align="center">
<img src="attaching-igw-to-vpc" height="40%" width="60%"/>
</p>
<h5 align="center"> Attaching Internet Gateway To VPC</h5>


<h2 align="left"> Creating Subnets</h2>

Next, I created the two public subnets in different AZs, Subnet A in us-east-1a and Subnet B in us-east-1c. After creation, I edited the subnet settings to auto-assign IPv4 address to both public subnets.

<p align="center">
<img src="subnet-creation" height="40%" width="60%"/>
</p>

<p align="center">
<img src="subnet-a" height="40%" width="60%"/>
</p>

<p align="center">
<img src="subnet-b" height="40%" width="60%"/>
</p>

Configuring Subnet Settings to Auto-Assign IPv4 Address to Subnets

<p align="center">
<img src="edit-subnet-settings" height="60%" width="80%"/>
</p>

<p align="center">
<img src="enable-assign-ipv4-address" height="40%" width="60%"/>
</p>

Click the save button after ticking the box to enable the auto-assign public IPv4 address

<h2 align="left"> Creating and Configuring Route Table</h2>

A default route is usually created after a vpc is created. I edited the name to "Main RT - Online Store" and edited the routing table to send all internet-bound traffic through the IGW. I also associated both subnets with the main route.

<p align="center">
<img src="editing-rt" height="60%" width="80%"/>
</p>

<p align="center">
<img src="attaching-both-subnets-to-rt" height="60%" width="80%"/>
</p>

<h2 align="left"> Configuring Security Group</h2>

I renamed the default security group that was created to 'Online Store SG' and configured it to allow SSH and HTTP traffic from my IP address.

<p align="center">
<img src="editing-sg-inbound-rules" height="60%" width="80%"/>
</p>

<h2 align="left"> Launching The EC2 Instance</h2>

The next step is to configure and launch an EC2 Instance in each availability zone, with the one in Subnet A tagged old and Subnet B tagged new. During the creation of the EC2 Instance in Subnet A, I added a new volume with a size of 10 GiB, which would serve as the attached EBS volume. 

The next thing to do after creating the EC2 instances is to log in to the instance with the attached volume (Online Store DB - Old) using SSH, format and mount the volume and create or download some files into the EBS. I then reboot the instance to ensure that everything is working, and at this point assume the database is very slow based on the initial scenario. 

Next, I went to the EBS on the EC2 Page, created a snapshot of the EBS volume, went to the snapshot to confirm the creation and copied the snapshot ID.

Next, I created a new EBS volume from the snapshot ensuring the EBS volume is created in the other availability zone as Subnet B (us-east-1c), went to the other EC2 Instance (Online Store DB - New), attached the newly created EBS to the instance, log in to the instance using SSH and confirm if the initial data are still intact.

Once I confirmed the data was intact (and assumed the online store application was working perfectly), I terminated the old database server (EC2 Instance- Online Store DB - Old), deleted the old EBS and snapshot.

Creating First EC2 Instance (Old) With Attached EBS Volume

<p align="center">
<img src="ec2-a-1" height="60%" width="45%"/>
</p>

<p align="center">
<img src="ec2-a-2" height="40%" width="55%"/>
</p>

<p align="center">
<img src="ec2-a-3" height="40%" width="55%"/>
</p>

<p align="center">
<img src="ec2-a-4" height="40%" width="40%"/>
</p>

<p align="center">
<img src="ec2-a-5" height="40%" width="40%"/>
</p>

Creating Second EC2 Instance (New)

<p align="center">
<img src="ec2-b-1" height="60%" width="45%"/>
</p>

<p align="center">
<img src="ec2-b-2" height="40%" width="55%"/>
</p>

<p align="center">
<img src="ec2-b-3" height="40%" width="55%"/>
</p>

<p align="center">
<img src="ec2-b-4" height="40%" width="40%"/>
</p>

Login To The First EC2 Instance

```commandline
ssh -i "practice.pem" ec2-user@54.221.188.182
```
<h5 align="left"> Listing all attached volumes</h5>
```commandline
lsblk
```
My root volume is labelled xvda while the attached volume is labelled xvdb

<h5 align="left"> Formatting the attached EBS Volume</h5>
```commandline
sudo mkfs -t ext4 /dev/xvdb
```

<h5 align="left"> Create a Mount Point</h5>
```commandline
sudo mkdir /mnt/myvolume
```

<h5 align="left"> Mount the Formatted EBS Volume</h5>
```commandline
sudo mount /dev/xvdb /mnt/myvolume
```

<h5 align="left"> Check if The Volume is Mounted Correctly</h5>
```commandline
df -h
```

<h5 align="left"> Navigate to Mount Point</h5>
```commandline
cd /mnt/myvolume
```

<h5 align="left"> Create Files into the Volume</h5>
```commandline
sudo touch test1.txt
sudo touch test2.txt
sudo wget -O Olympic.jpg https://img.olympics.com/images/image/private/t_s_16_9_g_auto/t_s_w960/f_auto/primary/gdipnj8m9xgxekrvnck8
```
<p align="center">
<img src="ebs-formating-and-mounting" height="60%" width="40%"/>
</p>

<p align="center">
<img src="ebs-formating-and-mounting2" height="60%" width="40%"/>
</p>


<h2 align="left"> Creating a Snapshot of the Volume</h2>

<p align="center">
<img src="creating-snapshot1" height="60%" width="40%"/>
</p>

<p align="center">
<img src="creating-snapshot2" height="60%" width="40%"/>
</p>

<p align="center">
<img src="creating-snapshot3" height="60%" width="40%"/>
</p>

<h2 align="left"> Creating a New EBS Volume From the Snapshot in US-EAST-1C</h2>

<p align="center">
<img src="creating-ebs-volume-from-snapshot1" height="60%" width="40%"/>
</p>

<p align="center">
<img src="creating-ebs-volume-from-snapshot2" height="60%" width="40%"/>
</p>

<p align="center">
<img src="creating-snapshot3" height="60%" width="40%"/>
</p>
I renamed the newly created volume, MyVolume2

<h2 align="left"> Attaching the New EBS Volume to EC2 Instance in US-EAST-1C</h2>

<p align="center">
<img src="attaching-ebs-volume-2nd-ec2 instance1" height="60%" width="40%"/>
</p>

<p align="center">
<img src="attaching-ebs-volume-2nd-ec2 instance2" height="60%" width="40%"/>
</p>

<p align="center">
<img src="attaching-ebs-volume-2nd-ec2 instance3" height="60%" width="40%"/>
</p>

<h2 align="left"> Testing and Validation</h2>

Login To The Second EC2 Instance (Online Store DB - NEW)

```commandline
ssh -i "practice.pem" ec2-user@3.235.154.2
```
<h5 align="left"> Listing all attached volumes</h5>
```commandline
lsblk
```
My root volume is labelled xvda while the attached volume is labelled as xvdbf

<h5 align="left"> Create a Mount Point</h5>
```commandline
sudo mkdir data
```

<h5 align="left"> Mount the EBS Volume</h5>
```commandline
sudo mount /dev/xvdbf data
```

<h5 align="left"> Check if The Volume is Mounted Correctly</h5>
```commandline
df -h
```

<h5 align="left"> Navigate to Mount Point</h5>
```commandline
cd data/
```

<h5 align="left"> List the Files in the Directory</h5>
```commandline
ls
```
All 3 files that I previously created are intact in the attached EBS and I can at this point assume the Online Store App is working perfectly, availability is excellent and the overall performance of the app is optimized.

<p align="center">
<img src="validating-data-is-present-in-ebs-volume" height="60%" width="40%"/>
</p>

Since everything is now working well, the old database can now be terminated, the previous EBS Volume (MyVolume) and snapshot deleted.
