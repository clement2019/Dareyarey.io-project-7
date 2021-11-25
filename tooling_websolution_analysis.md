   ### PROJECT 7
 ### DEVOPS TOOLING WEBSITE SOLUTION
In previous project WordPress web solution i implemented a WordPress based solution that is ready to be filled with content and can be used as a full-fledged website or blog. 
Moving further added some more value to our solutions that the DevOps team could utilize. I now introduced a set of DevOps tools that will help the team in day-to-day activities
in managing, developing, testing, deploying, and monitoring different projects.
The tools we want the team to be able to use are well known and widely used by multiple DevOps teams.
NOW PREPARING	 NFS SERVER
I spinned up a new EC2 instance with RHEL Linux 8 Operating System.
Based on the experience gathered while working on LVM in project 6, I configured the LVM on the Server.
 Instead of formatting the disks as ext4 formatted them as xfs

I ensured there are 3 Logical Volumes. lv-opt , lv-apps, and lv-logs.
 Created mount points on /mnt directory for the logical volumes as follow:
Mounted lv-apps on /mnt/apps – To be used by webservers
Mounted lv-logs on /mnt/logs – To be used by webserver logs
Mounted lv-opt on /mnt/opt – To be used by Jenkins server in Project 8

I Installed NFS server, configure it to start on reboot and make sure it was up and ran the command below 
sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service

Exported the mounts for webservers’ subnet cidr to connect as clients. For simplicity, i installed all three Web Servers inside the same subnet, but in production set up would probably want to separate each tier inside its own subnet for higher level of security.
To check subnet cidr – opened EC2 details in AWS web console and located ‘Networking’ tab and open a Subnet link:

I created four instances of RedHat distribution of Linux and named them HFS, web1, web2 and web3. I created a volume of size 15GB and attached the volume to NFS (EC2 instance 
on Aws,i connected to the NFS server using SSH connection through my GITbash.
I ran Lsblk and got the screenshot shown below it shows my 15GB volume attached to this server

![image](https://user-images.githubusercontent.com/55473846/143414316-74f3f378-fce4-4605-ac52-a33d3aa43623.png)

Before i created a physical volume I first created a partition on that disk and I created a logical volume
I just ran 
sudo gdisk /dev/xvdf
 the above allows me to create a partition on the disk as shown below
 
 ![image](https://user-images.githubusercontent.com/55473846/143415339-47f8f398-b3e4-4bae-a66b-8c5b32749365.png)
 
 From the knowledge gathered from project 6
Now after running lsblk I go the below

![image](https://user-images.githubusercontent.com/55473846/143415845-39000dc6-ede2-47c1-ba8d-6f09a49f9039.png)

When I ran sudo pvcreate /dev/xvdf1
I found out lvm was not installed 
Then I now ran 
Sudo yum install lvm2


![image](https://user-images.githubusercontent.com/55473846/143417252-01032be8-b091-4402-8176-db508d5920b7.png)

i confirmed lvs

![image](https://user-images.githubusercontent.com/55473846/143417355-7c5f26da-8a51-4b25-ba24-110ed66cb75e.png)

When I now ran 
sudo pvcreate /dev/xvdf1

![image](https://user-images.githubusercontent.com/55473846/143417473-5ba7d62a-4363-4647-80fa-6b1700f65879.png)

To confirm what am doing 
I ran sudo pvs

![image](https://user-images.githubusercontent.com/55473846/143417591-703a084a-d6f2-4e76-9f76-3d17156f6c8b.png)

sudo vgcreate vg-webdata /dev/xvdf1

![image](https://user-images.githubusercontent.com/55473846/143417737-024a14ce-3b03-4e22-9c64-c3ceb609d454.png)

sudo lvcreate -n lv-apps -L 5G vg-webdata

![image](https://user-images.githubusercontent.com/55473846/143418003-65d37865-c6aa-4576-b8e1-9d07e73da4c9.png)


![image](https://user-images.githubusercontent.com/55473846/143418278-a872cce8-4730-4ac5-977f-654da9c42cdd.png)

After lv opt, lv-apps and lv-logs and creating space for them ,5gb,5gb and 5gb respectively as shown below


![image](https://user-images.githubusercontent.com/55473846/143419045-c94cc040-eeb1-4feb-b9cf-bb4986ff0ee4.png)

We now create a file system for them
By running the command
sudo mkfs.xfs /dev/vg-webdata/lv-apps


![image](https://user-images.githubusercontent.com/55473846/143419167-24cc1686-4ac9-4941-a85e-dcbf2a6f057b.png)

When I ran 
sudo mkfs.xfs /dev/vg-webdata/lv-logs


![image](https://user-images.githubusercontent.com/55473846/143419339-1fa009a1-286f-4168-9358-321fbaa67b0a.png)

When I ran 
sudo mkfs.xfs /dev/vg-webdata/lv-opt

![image](https://user-images.githubusercontent.com/55473846/143419479-43997195-b87c-450a-9b17-66790fba002f.png)

Now we need to make directories
sudo mkdir /mnt/apps

sudo mkdir /mnt/logs

sudo mkdir /mnt/opt


![image](https://user-images.githubusercontent.com/55473846/143419635-27e881d5-9173-4c0d-b7f3-92e5855bea1f.png)

![image](https://user-images.githubusercontent.com/55473846/143419753-7e59f8d9-40c8-4e47-9741-a9d603fcf794.png)

I ran the command 

sudo yum -y update
sudo yum install nfs-utils -y

![image](https://user-images.githubusercontent.com/55473846/143419931-e2c0a617-336c-4dc4-ba86-c6a3486a847e.png)

To start our NFS server and check the status
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service

![image](https://user-images.githubusercontent.com/55473846/143420170-7f3668cd-91b7-43f9-84b7-8a444bf7e2cf.png)

sudo chown -R nobody: /mnt/apps
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt

when I ran ls -l /mnt


Check which port is used by NFS and open it using Security Groups (add new Inbound Rule)
rpcinfo -p | grep nfs


![image](https://user-images.githubusercontent.com/55473846/143420382-6f0f00f2-2afc-4c1c-ac8c-815ae67882c6.png)

I have opened all traffic route no need to add 2049 again in my security group

I don’t need to do the below since I have opened all ports

 For NFS server to be accessible from your client, you must also open following ports: TCP 111, UDP 111, UDP 2049


CONFIGURE THE DATABASE SERVER

Now that I now know how to install and configure a MySQL DBMS to work with remote Web Server

1.	Install MySQL server


![image](https://user-images.githubusercontent.com/55473846/143420531-094f9f24-7cfd-4f9f-9b73-3245be5e698b.png)

To start mysql

sudo systemctl start mysql
Sudo systemctl start mysql
Sudo systemcltl enable mysql
Sudo systemctl status mysql


![image](https://user-images.githubusercontent.com/55473846/143420692-8f5abf41-937a-44e0-9971-7f248079b063.png)

Now to start installation of mysql
sudo mysql_secure_installation


![image](https://user-images.githubusercontent.com/55473846/143420811-eacc0cad-d8b7-4554-9e64-0639752dc16f.png)

1.	Create a database and name it tooling
And show databases;


![image](https://user-images.githubusercontent.com/55473846/143421033-dd12df24-3267-45ad-804a-d45cad385caa.png)

Create a database user and name it webaccess

CREATE USER 'webaccess'@'%' IDENTIFIED WITH  mysql_native_password BY 'password';

	Grant permission to webaccess user on tooling database to do anything only from the webservers subnet cidr
  
GRANT ALL PRIVILEGES ON tooling . * TO 'webaccess1'@'%' WITH GRANT OPTION;

Now confirm thatvthe users are created


![image](https://user-images.githubusercontent.com/55473846/143421226-1739ea70-8f75-4421-8110-51012f405413.png)


![image](https://user-images.githubusercontent.com/55473846/143421331-919867d3-e679-403b-8f13-d65727439d21.png)

Preparing the Web Servers

We need to make sure that our Web Servers can serve the same content from shared storage solutions, in our case – NFS Server and MySQL database.
You already know that one DB can be accessed for reads and writes by multiple clients. For storing shared files that our Web Servers will use – we will utilize NFS and mount previously created Logical Volume lv-apps to the folder where Apache stores files to be served to the users (/var/www).
This approach will make our Web Servers stateless, which means we will be able to add new ones or remove them whenever we need, and the integrity of the data (in the database and on NFS) will be preserved.
During the next steps we will do following:
•	I configured NFS client (this step must be done on all three servers)
•	I deploy a Tooling application to our Web Servers into a shared NFS folder
•	Configure the Web Servers to work with a single MySQL database
Launch a new EC2 instance with RHEL 8 Operating System

Install NFS client------ on one of the webservers named (web1)
sudo yum install nfs-utils nfs4-acl-tools -y

And after
sudo systemctl start nfs-server
sudo systemctl status nfs-server
sudo systemctl status nfs-server


![image](https://user-images.githubusercontent.com/55473846/143421499-fa8c7d09-9c19-4812-acf1-dbb7cc201f1e.png)


Mount /var/www/ and target the NFS server’s export for apps
But first create this direactory 
sudo mkdir /var/www

sudo mount -t nfs -o rw,nosuid 172.31.25.103:/mnt/apps /var/www


![image](https://user-images.githubusercontent.com/55473846/143421650-a07ebc55-3572-4354-b8a8-910c2961a2e2.png)

![image](https://user-images.githubusercontent.com/55473846/143421722-5ca91bba-231d-4fe5-b712-610c09e718b2.png)


Now I installed Apache on ethe webserver
sudo yum install httpd -y

![image](https://user-images.githubusercontent.com/55473846/143421816-fdbc7ede-517c-4176-9752-276151d77177.png)

Now lest check Apache installed condition 
sudo systemctl start httpd

sudo systemctl enable httpd

sudo systemctl status httpd

![image](https://user-images.githubusercontent.com/55473846/143421941-2555acc0-ce8c-43d1-8a3c-d4ab3b44ef3f.png)

Now check the content of 
Ls -l /var/log


 
Now we need to keep safe the content of 
Ls -l /var/log/httpd
I have backed up the content of 
Ls -l /var/log/httpd
By using the command 
sudo mv /var/log/httpd /var/log/httpd.bak

Now that I havre created back


![image](https://user-images.githubusercontent.com/55473846/143422187-921689c4-55fe-450d-a547-d53de5a73e17.png)


sudo mount -t nfs -o rw,nosuid 172.31.25.103:/mnt/logs /var/log/httpd

Now run the command to see that all is well

sudo mount -a

sudo systemctl daemon-reload

Install git on the web-server (web1)

![image](https://user-images.githubusercontent.com/55473846/143422418-94e45258-788b-433d-b657-043d4bb90032.png)

Now install git

asfter that clone the darey.io github repository

![image](https://user-images.githubusercontent.com/55473846/143422530-9f3eff7f-d715-42eb-8c3f-0a20684c3baf.png)

Now after cloning the darey.io github account and then ran
 ls -l

cd tooling 

 And now ran 

ls -l 

![image](https://user-images.githubusercontent.com/55473846/143422741-0ad2dc4a-ac81-4e38-852d-d1b238347af4.png)

When I move directory back to 
Cd tooling 
And find the conet of 
Ls -l /var/log


![image](https://user-images.githubusercontent.com/55473846/143422864-4b60ec38-1f6c-4d55-af27-cc1cc74d322b.png)

First move the content and ran

 sudo cp -R /var/log/httpd.bak/. /var/log/httpd

![image](https://user-images.githubusercontent.com/55473846/143423007-fcb7ced2-26eb-4809-9c31-aa60d6b3891e.png)


Now we must cop the content of html/. Into  /var/www/html/When I ran 
sudo cp -R html/. /var/www/html/


![image](https://user-images.githubusercontent.com/55473846/143423069-509bfbb1-5298-42d3-b37a-0f5e6f624adb.png)

Now when I changed the directory
Cd ..

And now ran  we can now see we have the conte of html copied already y into
 
Var/www/html when I ran 

ls -l /var/www/html


![image](https://user-images.githubusercontent.com/55473846/143423201-3a7af37a-5ee5-433f-bfdf-65b44fd1cae5.png)


Now we now need to disable Apache default page

By running the command below
sudo mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf_backup

So now I now ran 

sudo systemctl restart httpd

and I go this error

![image](https://user-images.githubusercontent.com/55473846/143423396-7e9427ae-b915-47df-83b1-bfd97e6e0e0c.png)


When I ran sudo systemctl status httpd



![image](https://user-images.githubusercontent.com/55473846/143423510-0532604d-0be6-40fd-bcc9-165df2de91fc.png)


![image](https://user-images.githubusercontent.com/55473846/143423653-a96976c5-e9fb-425a-9ac0-314f2b020330.png)

Now back at the webserver again
Update the website’s configuration to connect to the database (in functions.php file). Apply tooling-db.sql script.

I now install mysql on the webserver (web1)
As shown below
sudo yum install mysql-server


![image](https://user-images.githubusercontent.com/55473846/143423782-faff198a-99f0-4aa0-9f17-a80d453fda04.png)

Now since we have already coded the tooling already, we must import the tooling.dbl data 
We now go back to the tooling directory
cd../
cd ../..


![image](https://user-images.githubusercontent.com/55473846/143424584-884f6cf0-3d71-41c5-98d0-229bef519e34.png)

I can now connect to the database server from my webser(web1) now as shown below

![image](https://user-images.githubusercontent.com/55473846/143424712-c6a823c0-3203-4d34-b437-4bb1b45ce697.png)


Configure SELinux Policies
sudo chown -R apache:apache /var/www/html/
  sudo chcon -t httpd_sys_rw_content_t /var/www/html/ -R
  sudo setsebool -P httpd_can_network_connect=1
sudo setsebool -P httpd_can_network_connect_db 1



  sudo setsebool -P httpd_can_network_connect 1

sudo setsebool -P httpd_use_nfs 1

sudo setsebool -P httpd_can_network_connect_db 1


Then I now realse have not installed php on the webserver  and I want view a php solutions
I now ran this 

sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

After the successful installation of yum-utils and Remi-packages, search for the PHP modules which are available for download by running the command.
sudo dnf module list php
The output indicates that the currently installed version of PHP is PHP 7.2. To install the newer release, PHP 7.4, reset the PHP modules.


![image](https://user-images.githubusercontent.com/55473846/143424967-0c03f653-6cf1-4bc5-8677-fdea8d834354.png)

sudo dnf module reset php

Having reset the PHP modules, enable the PHP 7.4 module by running.

Finally, install PHP, PHP-FPM (FastCGI Process Manager) and associated PHP modules using the command.

![image](https://user-images.githubusercontent.com/55473846/143425142-3ae303c5-5eca-45e9-95b9-ad0d7edb05a3.png)


Perfect! We now have PHP 7.4 installed. Equally important, we need to start and enable PHP-FPM on boot-up.

Sudo yum -y update

\sudo yum install httpd -y

which httpd
sudo systemctl restart httpd
sudo systemctl enable httpd
sudo systemctl status httpd


![image](https://user-images.githubusercontent.com/55473846/143425382-bae29718-a0f5-4aac-bd97-03947dc1af3b.png)

![image](https://user-images.githubusercontent.com/55473846/143425382-bae29718-a0f5-4aac-bd97-03947dc1af3b.png)

![image](https://user-images.githubusercontent.com/55473846/143425497-dc0bcfc7-bc98-4a95-85cf-7b6bec943574.png)

Sudo git clone https://github.com/darey-io/tooling.git
Cd tooling
ls -l

![image](https://user-images.githubusercontent.com/55473846/143425670-5859fd47-ece8-4eaf-a8ca-87e7de5883ae.png)

![image](https://user-images.githubusercontent.com/55473846/143425694-ec7eb791-0233-4585-9f8f-bc97b17bb5ca.png)



This is the default page of the webserver with public ip address http://18.130.21.194/

![image](https://user-images.githubusercontent.com/55473846/143425737-8a301017-7bc4-4ac2-87b8-f74bf646e68d.png)

I installed the mysql on the webserver
sudo yum install mysql-server

Enter the tooling-db.sql script data
mysql -h 172.31.5.209 -u webaccess -p tooling < tooling-db.sql

curl localhost

![image](https://user-images.githubusercontent.com/55473846/143425964-7c511720-c08c-4fb2-8e7c-4a609db90562.png)

![image](https://user-images.githubusercontent.com/55473846/143426044-377ce735-589f-4d8b-8076-acfb7d8c575d.png)

sudo ls -l /var/log


![image](https://user-images.githubusercontent.com/55473846/143425988-d5581d28-bf80-48f4-a403-57ae656e435a.png)

Remove the default Red hat page
sudo mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf_backup

sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

sudo systemctl status httpd httpd



sudo setsebool -P httpd_can_network_connect 1

sudo setsebool -P httpd_use_nfs 1

sudo setsebool -P httpd_can_network_connect_db 1



I ran the following command again to be sure Appache is up and running

sudo systemctl restart httpd
sudo systemctl enable httpd
sudo systemctl status httpd


![image](https://user-images.githubusercontent.com/55473846/143426400-d6deeff2-e427-4d2d-8897-3d6b43f3ed48.png)

sudo vi functions.php
mysql -h 172.31.5.209 -u webaccess -p tooling < tooling-db.sql

Now since the database tooling-db.sql has been added it shoq the webserver and the database can taik to each other
 Ans since php is running
When I now check the browser using web server public ip addres first got the below screen shots
 

![image](https://user-images.githubusercontent.com/55473846/143426589-4b231110-c381-40c4-a3e8-f98552e38595.png)

But when I now again 
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
  sudo systemctl status php-fpm
  sudo systemctl restart httpd
  sudo setsebool -P httpd_execmem 1
  sudo systemctl restart httpd

and check the database server to be up and ruining I got the below screen shots and was able to login with username admin and the password admin



![image](https://user-images.githubusercontent.com/55473846/143426744-0b1699c5-0e9f-4552-bbb5-ab48f7fa0050.png)

![image](https://user-images.githubusercontent.com/55473846/143426846-62b75f1c-a2a3-4995-8c98-61433fc15b3f.png)

The result show that the two server can now talk to each other and data from the webserver can be render on the database server






















