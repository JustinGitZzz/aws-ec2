# AWS EC2 Secure SSH and SFTP Server Configuration

## Objective

The AWS EC2 Secure SSH and SFTP Server Configuration project aims to use Amazon Web Service's (AWS) features, notably the EC2 and VPC, to create an externally facing Linux system. The primary goal of this project is to provide secure SSH and SFTP access, confine the user to the directory, and allow uploading and downloading files. This project is designed to emphasize server access security and better understand AWS configurations and management.

### Skills Learned
- Cloud Computing : Configuring and managing virtual servers on AWS EC2
- Networking and Security : Implementing SSH key-based authentication, firewall rules, and secure file transfer protocols
- User and Access Management : Configuring user permissions and access within the server
- Directory Confinement : Setting up chroot environment or directory access restrictions
- Routing and Traffic Management : Using AWS VPC to create route tables to control the traffic flow between subnets and configuring internet gateways and VPC endpoints for efficient and secure access

### Tools Used

- AWS EC2 : Launching and managing virtual server instances
- AWS VPC : Setting up subnets, route tables, and security groups
- AWS Security Groups : Configuring firewall rules to restrict access to specific IP addresses and ports
- SSH (Secure Shell) : Securing remote access to the EC2 instance
- SFTP (Secure File Transfer Protocol) : Securing file transfer
- IAM (Identifity and Access Management) : Managing user permissions and access within the server
- Shell Commands : Linux shell commands for user and file system management

## Steps
Configuring the Virtual Private Cloud (VPC)

Create a new VPC

![image](https://github.com/user-attachments/assets/7acd0ecc-3a38-4c6c-b4da-ce8d9beb0b33)

Create an Internet Gateway (IGW)

![image](https://github.com/user-attachments/assets/f1097873-396b-4f33-b8a3-936aa3fc5645)

Attach the IGW to the VPC made in the last step (I have already attached it hence why it is greyed out)

![image](https://github.com/user-attachments/assets/a30195dd-8694-42cf-a9af-8cf6899ca1d2)

Create a Public Subnet

![image](https://github.com/user-attachments/assets/c43c9bd6-d0cb-4580-919b-92575f217e09)

![image](https://github.com/user-attachments/assets/ead252d8-71ab-4fc0-a365-f77061219d6d)

Select the Subnet created and go to Actions > edit subnet settings > Modify auto-assign IP settings and enable "Auto-Assign IPv4" (Allowing instances in this subnet to receive a public IP automatically)

![image](https://github.com/user-attachments/assets/28dcc038-5339-4193-8040-343bd96ce824)

![image](https://github.com/user-attachments/assets/dc17c075-a7a3-46e8-abe0-432a3865e681)

Create a Route Table for Internet Access

![image](https://github.com/user-attachments/assets/0fb012e6-5da3-4b06-b4b2-da2073e1262f)

Select the Route Table created and go to Actions > edit routes > Add Route and set the destination for outbound internet traffic

![image](https://github.com/user-attachments/assets/238d556f-39a6-435d-a9ea-be5cdaaa0d0b)

Associate the Route Table with the Subnet in the Subnets tab

![image](https://github.com/user-attachments/assets/6b1c8331-73c6-4e8c-8dff-eeac5618a3a0)

Create a Security Group and set the Inbound Rules (0.0.0.0/0 allows for global access)

![image](https://github.com/user-attachments/assets/0e5cd479-5806-44ec-94ff-cacb9a84673c)


Open EC2 and click Launch Instances

![image](https://github.com/user-attachments/assets/d6136869-7188-4034-a76f-6795fe4f3311)

Enter a name and select the Amazon Linux 2 AMI

![image](https://github.com/user-attachments/assets/79b111ae-cdb1-48f5-a49f-5cf3bb7a63a7)

Select t2.micro for free-tier eligibility and choose/create a key pair to SSH into the instance

![image](https://github.com/user-attachments/assets/7a4ab4fb-060d-49f4-a163-9b88a53fd1e8)

Select the VPC, Subnet, and Security Group made earlier

![image](https://github.com/user-attachments/assets/636cdd1f-8f68-495a-846c-483047913a75)

Launch the instance and note the public IP address (this will be used to SSH)

SSH into the instance with ssh -i ~/user_key.pem ec2-user@<public-ip>

Congrats! The EC2 instance is now live and connected to the internet with SSH and SFTP access available.

### Now let's add a user to the instance!

SSH into the instance and create a user
<pre style="background-color: rgb(34, 34, 34); color : whitesmoke;">
    sudo adduser [user] 
</pre>

Create the .ssh directory for the user
<pre style="background-color: rgb(34, 34, 34); color : whitesmoke;">
sudo mkdir /home/[user]/.ssh
sudo chown [user]:[user] /home/[user]/.ssh
sudo chmod 700 /home/sftpjj/.ssh 
</pre>

Upload your public key to the EC2 instance
<pre style="background-color: rgb(34, 34, 34); color : whitesmoke;">
scp -i "~/user_key.pem" "~/key.pub" ec2-user@<public-ip>:/home/[user]/.ssh/
</pre>

SSH to the ec2 instance
<pre style="background-color: rgb(34, 34, 34); color : whitesmoke;">
ssh -i ~/user_key.pem ec2-user@<public-ip>
</pre>

Switch to the user
<pre style="background-color: rgb(34, 34, 34); color : whitesmoke;">
sudo su - [user]
</pre>

Move the uploaded key to ./authorized_keys
<pre style="background-color: rgb(34, 34, 34); color : whitesmoke;">
cat /home/[user]/.ssh/file.pub >> /home/user/.ssh/authorized_keys
</pre>

Remove the public key
<pre style="background-color: rgb(34, 34, 34); color : whitesmoke;">
rm /home/[user]/.ssh/file.pub
</pre>

Set Permissions on ./authorized_keys
<pre style="background-color: rgb(34, 34, 34); color : whitesmoke;">
chmod 600 /home/[user]/.ssh/authorized_keys
</pre>

Exit and connect to the instance with SFTP
<pre style="background-color: rgb(34, 34, 34); color : whitesmoke;">
sftp -i "~/user_key.pem" [user]@<public-ip>
</pre>


### Now let's configure the SSH server for the user

Start by using SSH to connect to the instance
<pre style="background-color: rgb(34, 34, 34); color : whitesmoke;">
ssh -i ~/user_key.pem ec2-user@<public-ip>
</pre>


Open the SSH configuration file with nano or vim to edit
<pre style="background-color: rgb(34, 34, 34); color : whitesmoke;">
sudo nano /etc/ssh/ssd_config
</pre>

Add the following configuration which apply the settings only to the specified user. This configuration restricts the user to the /home/[user] directory. The ForceCommand internal-sftp configuration forces the SSH session to run only the internal-sftp command which prevents the user from accessing a full SSH shell to ensure the user can only perform file transfer operations. AllowTcpForwaring no disables TCP forwarding for the user to prevent tunnels that can bypass network restrictions. X11Forwarding no removes graphical access.
<pre style="background-color: rgb(34, 34, 34); color : whitesmoke;">
Match User sftpjj
  ChrootDirectory /home/sftpjj
  ForceCommand internal-sftp
  AllowTcpForwarding no
  X11Forwarding no
</pre>

Save and exit the nano or vim and set the correct permissions for the root directory
<pre style="background-color: rgb(34, 34, 34); color : whitesmoke;">
sudo chown root:root /home/[user]
sudo chmod 755 /home/[user]
</pre>

Create a home directory for the user
<pre style="background-color: rgb(34, 34, 34); color : whitesmoke;">
sudo mkdir /home/[user]/uploads
sudo chown [user]:[user] /home/[user]/uploads
</pre>
    
Restart the SSH service to apply changes
<pre style="background-color: rgb(34, 34, 34); color : whitesmoke;">
sudo systemctl restart sshd
</pre>

Congrats! Now the SFTP-connected user is bound to a directory with set permissions and can only upload or download files.
























