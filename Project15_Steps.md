


#### Step 1

I start by getting a domain `hracompany.ga` from `www.freenom.com`

**Creating Organizational Unit** (OU) called **Dev**  
* AWS Organizations > AWS accounts
	* Select root account > Actions > Organizational Unit - Create New

Created a new account called **DevOps**  
* AWS Organizations > AWS accounts > Add an AWS Account

Moved **DevOps** account to Organizational Unit **Dev**  
* Select DevOps > Actions > Move  

Created Organizational Unit **Dev**  and put Account **DevOps**  
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/organizationalUnit.png)  

Login to newly created account **DevOps**  
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/accountDevOps.png)  

#### Create VPC 
* VPC > Your VPCs > Create VPC
* VPC Settings
  * Name tag: HRA-VPC
  * IPv4 CIDR: `10.0.0.0/16`
  * DNS hostnames: Enabled

Creating Internet Gateway **HRA-Igw** attach to **HRA-VPC**  
* VPC > Internet Gateways > Create internet gateway

* VCP > Subnets > Create subnet
  * Public Subnets
    * **Public Subnet 1** HRA-public-subnet-1  
		`10.0.1.0/24` | East 1a  
	* **Public Subnet 2** HRA-public-subnet-2  
		`10.0.3.0/24` | East 1b  
  * Private Subnets
	* **Private Subnet  1** HRA-private-subnet-1  
	`10.0.2.0/24` | East 1a
	* **Private Subnet  2** HRA-private-subnet-2  
	`10.0.4.0/24` | East 1b
	* **Private Subnet  3** HRA-private-subnet-3  
	`10.0.5.0/24` | East 1a
	* **Private Subnet  4** HRA-private-subnet-4  
	`10.0.6.0/24` | East 1b  

![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/subnet.png)  

**Creating Route Tables**
* VPC > Route Tables > Create route table
  * Created Route Tables  
	* **HRA-public-rtb**  
    * **HRA-private-rtb**  

![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/routeTable.png) 

**Associate Subnets to Route Tables**  
* **HRA-public-rtb** to Public Subnets 
  ![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/subnetAssociationPublicRTB.gif)	 
	
* **HRA-private-rtb** to Private Subnets  
  ![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/subnetAssociationPrivateRTB.gif)  

Edit Public Route Table **HRA-public-rtb** (to target **HRA-Igw** Internet Gateway)  
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/publicRTB_internetGateway.gif)  

**Create a NAT gateway**  
* VPC > Elastic IPs > Allocate Elastic IP Address  
	Allocate Elastic IP (Tag name **HRA-NAT**)  
	![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/NAT_ElasticIP.gif)  
	
* VPC > NAT Gateways > Create NAT gateway  
	Create the NAT in NAT Gateways (**HRA-NatGateway**) into **Public Subnet 1** and choose Elastic IP **HRA-NAT**  
	![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/NATcreation.gif)  

Edit Private Route Table **HRA-private-rtb** like so _dest:_ `0.0.0.0/0`  _target:_ **HRA-NatGateway**  
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/privateRTB_NAT.gif)  

**Creating Security Groups**
* VPC > SECURITY > Security Groups > Create security group  
  * **HRA-ext-ALB** | HTTP/S from anywhere  
  * **HRA-bastion** | SSH from anywhere (ideally from our current IP)  
  * **HRA-nginx-reverse-proxy** | HTTP/s from **HRA-ext-ALB** 
  * **HRA-int-ALB** | HTTP/S from **HRA-nginx-reverse-proxy** 
  * **HRA-webserver** | SSH from **HRA-bastion** , HTTP/S from **HRA-int-ALB**
  * **HRA-datalayer** | MYSQL/Aurora from **HRA-bastion**, NFS from **HRA-webserver**, MYSQL/Aurora from **HRA-webserver**  

![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/securiyGroups.png)  


**Creating a hosted zone**  
Tells **Route 53** how to respond to DNS queries for domain `hracompany.ga`  
Route 53 > Hosted zones > Create hosted zone  
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/hostedzone.gif)  

![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/records.png) 

The values under **Value/Route traffic to** are nameservers, need to add those in `www.freenom.com`  

**Request Certificate**  
* DNS Validation  
  Tag: Name **HRA-Cert**    

  ![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/requestCertificate2.gif)  

  Validation Step: button **Create record in Route 53**  
  ![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/createRecordsRoute53.png)  

Now we see in Route 53 > Hosted zones > `hracompany.ga`  
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/records2.png)  

**Creating Elastic File System**  
* EFS > Create file system  
  Name: **HRA-filesystem**  
  VPC: **HRA-VPC**  

Mount targets: **HRA-private-subnet-1** and **HRA-private-subnet-2**, where the webservers are, resources in these subnets will have the ability to mount the filesystem  
Apply Security Group **HRA-datalayer**  
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/EFS.gif)  

**Creating Access Point**  
[Working with Amazon EFS access points](https://docs.aws.amazon.com/efs/latest/ug/efs-access-points.html)  
Amazon EFS access points are application-specific entry points into an EFS file system that make it easier to manage application access to shared datasets.  

![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/createAccessPoint.gif)  

* Details:  
  * Name: wordpress  
	* Root directory path: `/wordpress  `
  * POSIX user:  
	* User ID: `0`  
	* Group ID: `0`  
  * Root directory creation permission:  
	* Owner user ID: `0`  
	* Owner group ID: `0`  
	* POSIX permission to apply to the root directory path : `0755`  
	* Tag name: wordpress-ap (ap access point)  
	
* Details:  
  * Name: tooling  
	* Root directory path: `/tooling` 
  * POSIX user:  
	* User ID: `0`   
	* Group ID: `0`  
  * Root directory creation permission:  
	* Owner user ID: `0`  
	* Owner group ID: `0`  
	* POSIX permission to apply to the root directory path : `0755`  
	* Tag name: tooling-ap (ap access point)  

![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/accessPoints.png)  

**Create RDS**  
* **Create KMS Key**  
  KMS > Customer managed keys > Create Key  
  * Step1:  (defaults)  
	 * Key type: Symmetric  
	 * Advanced options  
	    * Key material origin: KMS  
		* Regionality: Single-Region key  
  * Step2:  
	 * Alias: HRA-rds  
	 * Description: for the rds instance  
	 * Tags: Name HRA-rds-key  
  * Step3:   
     * Add yourself as administrator of key _(created AIM user Hector)_

![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/customerManagedKeys.png)  


**Create Subnet Group**  
Amazon RDS > Subnet groups > Create DB Subnet Group  
* Subnet group details  
  * Name: hra-rds-subnet  
    * Description: for rds subnets  
	* VPC: HRA-VPC  
  * Add subnets  
	* Availability Zones: us-east-1a, us-east-1b  
    * Subnets: Private 3 and 4  
  
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/subnetGroups.png)  


* Amazon RDS > Dashboard > Create database  
  Choose a database creation method: **Standard create**  
  Engine options:  **MySQL**  
  Templates: **Free tier** (wont able to select KMS key to encrypt DB), select production, you can see it by just scrolling down  
  Settings  
    * DB cluster identifier: **HRA-database**  
* Credentials Settings  
    * Master username: **HRAadmin**  
    * Master password: **admin12345**  
* Connectivity  
    * Virtual private cloud (VPC): **HRA-VPC**  
    * Subnet group: **hra-rds-subnet**  
* Existing VPC security groups: **HRA-datalayer**  
  Availability Zone: us-east-1a  
  Database options:  
    * Initial database name: test (no need to have it)  

Production gives option to select encryption key  		
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/productionRDSencryption.gif)  

Creating RDS with  **Free Tier**  
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/RDS_creation.gif)  

**Get the Endpoint:**  
`hra-database.cssi6ineszpw.us-east-1.rds.amazonaws.com`  


#### Creating and Preparing EC2 Instances
	We create 3 RedHat Instances bastion, nginx and webserver install various things to create an image from later  

**Bastion**  

``` bash
	#commands for ami installation  
	sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
	sudo yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
	sudo yum install wget vim python3 telnet htop git mysql net-tools chrony -y
	sudo systemctl start chronyd
	sudo systemctl enable chronyd
```

**Nginx**  
``` bash
#commands for ami installation  
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
yum install wget vim python3 telnet htop git mysql net-tools chrony -y
systemctl start chronyd
systemctl enable chronyd


#configure selinux policies
setsebool -P httpd_can_network_connect=1
setsebool -P httpd_can_network_connect_db=1
setsebool -P httpd_execmem=1
setsebool -P httpd_use_nfs 1


#install amazon efs utils for mounting the target on the Elastic file system
git clone https://github.com/aws/efs-utils
cd efs-utils
yum install -y make
yum install -y rpm-build
make rpm 
yum install -y  ./build/amazon-efs-utils*rpm

#setting up self-signed certificate
sudo mkdir /etc/ssl/private
sudo chmod 700 /etc/ssl/private
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/ACS.key -out /etc/ssl/certs/ACS.crt
sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048

```

* **Nginx Self Signed Certificate Output**  
	``` bash
	writing new private key to '/etc/ssl/private/ACS.key'
	-----
	You are about to be asked to enter information that will be incorporated
	into your certificate request.
	What you are about to enter is what is called a Distinguished Name or a DN.
	There are quite a few fields but you can leave some blank
	For some fields there will be a default value,
	If you enter '.', the field will be left blank.
	-----
	Country Name (2 letter code) [XX]:US
	State or Province Name (full name) []:FL
	Locality Name (eg, city) [Default City]:Miami
	Organization Name (eg, company) [Default Company Ltd]:HRAcompany
	Organizational Unit Name (eg, section) []:Dev
	Common Name (eg, your name or your server's hostname) []:172.31.17.2
	Email Address []:myemail@email.com
	```


**Webserver**  
``` bash
	#commands for ami installation  
	yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
	yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
	yum install wget vim python3 telnet htop git mysql net-tools chrony -y
	systemctl start chronyd
	systemctl enable chronyd
	
	#configure selinux policies
	setsebool -P httpd_can_network_connect=1
	setsebool -P httpd_can_network_connect_db=1
	setsebool -P httpd_execmem=1
	setsebool -P httpd_use_nfs 1
	
	#install amazon efs utils for mounting the target on the Elastic file system
	git clone https://github.com/aws/efs-utils
	cd efs-utils
	yum install -y make
	yum install -y rpm-build
	make rpm 
	yum install -y  ./build/amazon-efs-utils*rpm
	
	#setting up self signed certificate for the webserver (apache)
	yum install -y mod_ssl
	openssl req -newkey rsa:2048 -nodes -keyout /etc/pki/tls/private/ACS.key -x509 -days 365 -out /etc/pki/tls/certs/ACS.crt
	vi /etc/httpd/conf.d/ssl.conf
```
* **Apache Self Signed Certificate Output** 
  Same as Nginx except **hostname**   
	``` bash
	Common Name (eg, your name or your server's hostname) []:172.31.24.114`
	```
Need to change `vi /etc/httpd/conf.d/ssl.conf`

We need to change:  
`SSLCertificateFile /etc/pki/tls/certs/localhost.crt` **to** `SSLCertificateFile /etc/pki/tls/certs/ACS.crt`  

`SSLCertificateKeyFile /etc/pki/tls/private/localhost.key` **to** `SSLCertificateKeyFile /etc/pki/tls/private/ACS.key`

I used **sed**
``` bash
sed -i 's_SSLCertificateFile /etc/pki/tls/certs/localhost.crt_SSLCertificateFile /etc/pki/tls/certs/ACS.crt_g' /etc/httpd/conf.d/ssl.conf
```  

``` bash
sed -i 's_SSLCertificateKeyFile /etc/pki/tls/private/localhost.key_SSLCertificateKeyFile /etc/pki/tls/private/ACS.key_g' /etc/httpd/conf.d/ssl.conf
```  

#### Creating AMI from the instances  
_(delete instances after)_  

**HRA-webserver-ami** , description: for webserver
**HRA-bastion-ami** , description: for bastion
**HRA-nginx-ami** , description: for nginx  

![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/bastionAMI.gif)  

![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/amis.png)  

#### Creating Target Groups  
EC2 > Load Balancing >Target Groups > Create target group
	
	Target group name: HRA-nginx-target
	Protocol: HTTPS
	VPC: HRA-VPC
	Health check settings
		Protocol: HTTPS
		Path: /healthstatus
		
	Target group name: HRA-wordpress-target
	Protocol: HTTPS
	VPC: HRA-VPC
	Health check settings
		Protocol: HTTPS
		Path: /healthstatus

	Target group name: HRA-tooling-target
	Protocol: HTTPS
	VPC: HRA-VPC
	Health check settings
		Protocol: HTTPS
		Path: /healthstatus
















