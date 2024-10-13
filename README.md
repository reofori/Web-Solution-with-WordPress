# Web Solution with Wordpress

### Overview

In this project, I'm building on my experience with implementing web solutions across different technologies. As a DevOps engineer, it’s crucial to be familiar with PHP-based solutions, as PHP continues to be one of the most widely used programming languages for websites. This project tasks me with setting up the storage infrastructure on two Linux servers and deploying a basic web solution using **WordPress**, an open-source content management system (CMS) written in PHP, with **MySQL** or **MariaDB** as its backend relational database management system (RDBMS). 

I will implement WordPress using a **three-tier architecture** on two RedHat EC2 servers. My goal is to gain practical experience in Linux disk management and deploying a basic PHP-based web solution. This involves setting up the storage subsystems, installing WordPress, and connecting it to a remote MySQL database. Through this project, I aim to solidify my skills in storage management, web deployment, and database integration.


### Project Focus Areas

#### Storage Infrastructure (Part 1)

The first part of the project will focus on preparing and configuring the storage infrastructure for both the web and database servers. I will practice disk management, partitioning, and utilize volume management tools like `gdisk` for partitioning and `LVM` (Logical Volume Manager) for managing volumes. This hands-on experience is essential for understanding how data is stored and managed on the servers.

#### WordPress Installation and Connection to MySQL Database (Part 2)

In the second part, I will install WordPress, a content management system (CMS) written in PHP, and configure it to connect with a remote MySQL database. This step will reinforce my understanding of deploying web solutions by establishing a connection between the web server, where WordPress is installed, and the database server.
 
### The Importance of Three-tier Architecture

Web and mobile solutions are often structured using a **three-tier architecture**, which consists of three distinct layers:

- **Presentation Layer (PL)**: This is the front-end interface, such as the web browser, through which users interact with the application.
- **Business Layer (BL)**: This is where the application or web server resides, processing user requests and implementing the business logic. For this project, WordPress will serve as the business layer.
- **Data Access Layer (DAL)**: This is where data is stored and managed, which in this case is handled by the MySQL database server running on a separate EC2 instance.

By implementing this three-tier setup, I’ll gain valuable hands-on experience that mirrors how large-scale applications are structured, with a clear separation between user interaction, application processing, and data storage. The practical exposure to **disk partitioning** using `gdisk` and **volume management** with `LVM` further enhances my understanding of managing storage subsystems for both web and database servers.

### My Three-tier Setup

For this project, I’ll be working with:

1. **Client**: My laptop or PC will act as the client to interact with the WordPress web server.
2. **Web Server**: An EC2 Linux server running RedHat, where I will install WordPress.
3. **Database Server**: Another EC2 Linux server, also running RedHat, which will host the MySQL database.

### Learning Linux Disk Management

In addition to working with WordPress, I’ll delve deeper into **Linux disk management** by working with several tools to partition and manage disks. I’ll be using `gdisk` for partitioning and **LVM** to manage volumes. This project will allow me to practice these fundamental system administration skills, which are crucial when setting up servers for web applications.

### RedHat and AWS Cloud Services

While in previous projects I worked with **Ubuntu**, this project switches to **RedHat** to broaden my understanding of different Linux distributions. RedHat is a popular distribution used in many enterprises, and working with it is an important skill in my DevOps toolkit.

For this project, I will spin up EC2 instances on AWS, using **RedHat** as the operating system for both the web server and database server. This experience will further my familiarity with AWS services, and even though this project has a greater focus on disk management and web deployment, future projects will dive deeper into AWS-specific cloud concepts.

### Connecting via SSH

When connecting to the RedHat EC2 instances, I’ll need to use the `ec2-user` as the default user (as opposed to the `ubuntu` user used in previous projects). The SSH connection string will look like this:

```bash
ssh ec2-user@<Public-IP>
```

By the end of this project, I’ll have a deeper understanding of deploying a **WordPress-based solution** using a **three-tier architecture** on Linux servers, while gaining hands-on experience with **disk management** and **volume management**.

## STEPS INVOLVED
- [Step 1: Preparing the Web Server](#step-1-preparing-the-web-server)
- [Step 2: Prepare the Database Server](#step-2-prepare-the-database-server)
- [Step 3: Install WordPress on Your Web Server EC2](#step-3-install-wordpress-on-your-web-server-ec2)
- [Step 4: Install MySQL on the DB Server EC2](#step-4-install-mysql-on-the-db-server-ec2)
- [Step 5: Configure the Database for WordPress](#step-5-configure-the-database-for-wordpress)
- [Step 6: Configure WordPress to Connect to the Remote Database](#step-6-configure-wordpress-to-connect-to-the-remote-database)


## Step 1: Preparing the Web Server
1. Launch EC2 Instance:
- Start by launching an EC2 instance that will serve as the web server for WordPress. Make sure the instance uses RedHat and is in the same Availability Zone (AZ) as the volumes you will create.

![](img/new%20instance.png)

2. Create EBS Volumes:
- Create three (3) EBS volumes, each with a size of 10 GiB in the same AZ as your EC2 instance.

![](img/create%20volumes.png)

3. Attach EBS Volumes
- Attach the three volumes one by one to your EC2 web server instance. You can do this from the AWS Management Console by selecting the instance and attaching the newly created volumes.

![](img/attach%20volume.png)
![](img/attach%20volume%202.png)

4. SSH into your instance through your terminal
 ```bash
ssh ec2-user@<Public-IP>
```


5. Inspect Block Devices
- Use the following command to list all block devices connected to the server:
```bash
lsblk
```
![](img/inspect%20block%20devices.png)

- Check for the newly attached volumes. As seen above as /dev/xvdd, /dev/xvde, and /dev/xvdf.
- Alternatively, use:
```bash
ls /dev/
```
![](img/ls%20dev.png)

6. Check Disk Space
- Run the following command to view all mounted filesystems and their free space:
```bash
df -h
```
![](img/see%20free%20space.png)

7. Partition the Disks
- Use `gdisk` to create a partition on each of the disks:
    1. Press `n` to create a new partition:
    - `gdisk` will prompt you for the partition number. Since this is the first partition, you can press `Enter` to accept the default.

    2. First sector: The system will suggest a starting sector for the partition. Press `Enter` to accept the default.
    3. Last sector: By default, it will fill the partition with all available space (10 GiB in your case). Press Enter to accept the default.
    4. Partition type code: You’ll need to specify a partition type. Since you're using LVM, type `8E00 `for "Linux LVM". If not using LVM, you can leave it as default by pressing `Enter`.

    5. Writing the Partition Table to Disk, Once you’ve created the partition, you need to write the changes to the disk. To do this:
    - Press `w` to write the changes.
    - Confirm the action by typing yes when asked to proceed.

    At this point, the partition has been created, and gdisk will exit: 

    ![](img/using%20gdisk.png)
    6. Repeat the above process for the Other Disks:

    ```bash
    sudo gdisk /dev/xvde
    ```
    ```bash
    sudo gdisk /dev/xvdf
    ```
    7. During the process of  creating new partitions on one of the remaining disks, I accidentally  set the filesystem to default (8300, Linux filesystem) instead of 8E00(Linux LVM) which the other disks were set to. To avoid potential issues going forward:
    - Still in the gdisk, list partitions with:

    ```bash
    Command (? for help): p
    ```
    - Use the t command to change the type of the partition.
    ```bash
    Command (? for help): t
    ```
    - Enter the Correct Type Code
    - To confirm that the partition type has been changed, use the p command again to print the partition table:

    ![](img/fixing%20error.png)

8. View the Partitions
- After partitioning, view the newly created partitions using:
```bash
lsblk
```

![](img/view%20newly%20created%20partitions.png)

9. Install LVM
- Install the `lvm2` package:
```bash
sudo yum install lvm2
```
![](img/install%20lvm2.png)

- Check available partitions with `lvmdiskscan`
```bash
sudo lvmdiskscan
```

![](img/lvmdiskscan.png)

>For Redhat we use `yum` as the package manager, while Ubuntu uses `apt`.

10. Create Physical Volumes
- Use pvcreate to mark the partitions as physical volumes:
```bash
sudo pvcreate /dev/xvdd1 /dev/xvde1 /dev/xvdf1
```
- Confirm the successful creation of physical volumes by running:

```bash
sudo pvs
```

![](img/creating%20physical%20volumes.png)


11. Create a Volume Group
- Use `vgcreate` to combine the physical volumes into a single volume group named `webdata-vg`:
```bash
sudo vgcreate webdata-vg /dev/xvdd1 /dev/xvde1 /dev/xvdf1
```
- Verify the volume group creation with:
```bash
sudo vgs
```
![](img/volume%20group.png)

12. Create Logical Volumes
- Create two logical volumes:
    - apps-lv for storing website data, using 14 GiB of space
    ```bash
    sudo lvcreate -n apps-lv -L 14G webdata-vg
    ```
    - logs-lv for storing logs, using the remaining space:
    ```bash
    sudo lvcreate -n logs-lv -L 14G webdata-vg
    ```
- Confirm logical volumes creation by running:
```bash
sudo lvs
```

![](img/logical%20volumes.png)

13. Verify the entire setup
 - use:
 ```bash
 sudo vgdisplay -v #view complete setup - VG, PV, and LV
 sudo lsblk
 ```
 
 ![](img/verify%20the%20entire%20setup.png)

14. Format the Logical Volumes
- Format both logical volumes with the ext4 filesystem:
```bash
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```
![](img/formating%20lv%20to%20ext4%20fs.png)

15. Create Directories
- Create a directory for website files:
```bash
sudo mkdir -p /var/www/html
```
- Create a directory for log backups:
```bash
sudo mkdir -p /home/recovery/logs
```
16. Mount the Logical Volumes
- Mount the logical volume for the website:
```bash
sudo mount /dev/webdata-vg/apps-lv /var/www/html/
```
- Backup the current logs using `rsync` utility ( This is required before mounting the file system)

```bash
sudo rsync -av /var/log/ /home/recovery/logs/
```
![](img/directory%201.png)

- Mount the logical volume for logs ( Note, all existing data on /var/log will be deleted. That is why it is important to back it up.):
```bash
sudo mount /dev/webdata-vg/logs-lv /var/log/
```
- Restore the logs:
```bash 
sudo rsync -av /home/recovery/logs/ /var/log/
```

![](img/directory%202.png)


17. Configure Persistent Mounts
- Update the /etc/fstab file to ensure the mounts persist after a reboot. First, get the UUIDs of the devices:

```bash
sudo blkid
```

![](img/uuid.png)

> Locate the UUID of apps-lv and logs-lv


- Edit /etc/fstab:

```bash 
sudo vi /etc/fstab
```
- Add entries for both logical volumes using the UUIDs in this format:
```vim 
UUID=<UUID-of-apps-lv> /var/www/html ext4 defaults 0 0
UUID=<UUID-of-logs-lv> /var/log ext4 defaults 0 0
``` 



![](img/vi%20uuid%20modify.png)


18. Reload and Verify Setup
- Reload the daemon and mount all filesystems:
```bash
sudo mount -a
sudo systemctl daemon-reload
```
- Verify the setup with:
```bash
df -h
```
![](img/verify%20set%20up%20again.png)
> You should see both /var/www/html and /var/log mounted on your logical volumes.

## Step 2: Prepare the Database Server
1. Launch EC2 Instance
    - Start a new RedHat EC2 instance that will serve as your DB Server.

2. Set Up Storage Infrastructure
    - Repeat the storage steps from Step 1 of the Web Server setup, but this time
        - Create a logical volume called db-lv.
        - Mount the volume to the /db directory instead of /var/www/html.
    - Use the same steps to create and mount the logical volumes:

        ```bash
        sudo mkdir /db
        sudo mount /dev/webdata-vg/db-lv /db/
        ```

    - Ensure the configuration is persistent after a reboot by updating /etc/fstab with the db-lv UUID.
    - Verify the setup with:
        ```bash
        df -h
        ```

        ![](img/verify%20db%20server.png)

        > You should see both /db and /var/log mounted on your logical volumes.
## Step 3: Install WordPress on Your Web Server EC2

1. Update Package Repository
- Update the package repository:
```bash
sudo yum -y update
```
![](img/yum%20update.png)

2. Install wget, Apache, PHP, and Dependencies
- Install required packages:
```bash
 sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json
```

![](img/install%20wget%20apache%20and%20co.png)


3. Start Apache
- Enable and start the Apache web server:
```bash
sudo systemctl enable httpd
sudo systemctl start httpd
```
![](img/start%20httpd.png)

4. Install PHP and Dependencies
- Install additional PHP dependencies:
```bash
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
```
![](img/install%20php%20and%20dependencies.png)

5. Configure Apache for PHP
- Allow Apache to execute PHP code:
```bash
sudo setsebool -P httpd_execmem 1
sudo systemctl restart httpd
```


6. Download and Configure WordPress
- Download and install WordPress:
    1. Create a Directory for WordPress: First, create a directory that will hold the WordPress files.
        ```bash
        mkdir wordpress
        ```
    2. Change to the WordPress Directory: Move into the newly created wordpress directory to prepare for downloading WordPress.
        ```bash
        cd wordpress
        ```
    3. Download WordPress: Use the wget command to download the latest version of WordPress from its official website.
        ```bash
        sudo wget http://wordpress.org/latest.tar.gz
        ```
    4. Extract the Downloaded Archive: After downloading, extract the contents of the `latest.tar.gz `file using `tar`.
        ```bash
        sudo tar -xzvf latest.tar.gz
        ```
        ![](img/wordpress%20download.png)
    5. Remove the Compressed Archive: Clean up your workspace by removing the downloaded latest.tar.gz file since it’s no longer needed after extraction.
        ```bash
        sudo rm -rf latest.tar.gz
        ```
    6. Copy the Configuration File: WordPress comes with a sample configuration file that you need to copy and rename. This file will be used to configure your database settings.
        ```bash
        sudo cp wordpress/wp-config-sample.php wordpress/wp-config.php
        ```
    7. Move WordPress Files to the Web Root Directory: Finally, copy the entire WordPress directory into the web server’s root directory, which is typically located at /var/www/html/.
        ```bash
        sudo cp -R wordpress /var/www/html/
        ```

7. Setting Permissions for Apache
- Change Ownership of the WordPress Directory
    ```bash
    sudo chown -R apache:apache /var/www/html/wordpress
    ```
- Configure SELinux Context for WordPress Directory
    ```bash
    sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
    ```
- Allow Apache to Connect to the Network
    ```bash
    sudo setsebool -P httpd_can_network_connect=1
    ```

## Step 4: Install MySQL on the DB Server EC2
1. Install MySQL Server:
```bash
sudo yum update
sudo yum install mysql-server
```
![](img/install%20mysql-server.png)

2. Start MySQL Service
- Enable and start the MySQL service:
```bash
sudo systemctl enable mysqld
sudo systemctl start mysqld
```
3. Verify MySQL Status
- Check if MySQL is running:
```bash
sudo systemctl status mysqld
```
![](img/verify%20mysql-status.png)


## Step 5: Configure the Database for WordPress

1. Login to MySQL:
```bash
sudo mysql
```
2. Create the WordPress database and a user for the web server: 

```sql
CREATE DATABASE wordpress;
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass.1';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit;
```
![](img/create%20database%20for%20wp.png)

3. Open MySQL Port (3306)
- Modify the inbound rules for the DB server security group to allow MySQL connections (port 3306) only from the Web Server's private IP. For extra security, restrict access using /32 CIDR notation.

![](img/inbound%20rules.png)


## Step 6: Configure WordPress to Connect to the Remote Database
 1. Install the MySQL client on the Web Server:
 ```bash
 sudo yum install mysql
```
2. Test MySQL Connection
- Test if the Web Server can connect to the remote DB server:
```bash
sudo mysql -u myuser -p -h <DB-Server-Private-IP-Address>
```
- Once logged in, run the following SQL query to verify the connection:
```sql
SHOW DATABASES;
```
![](img/show%20databases.png)

3. Edit WordPress Configuration

- Edit the wp-config.php file located in /var/www/html/wordpress to include the database credentials using `sudo vi /var/www/html/wordpress/wp-config.php
` :

```php
define('DB_NAME', 'wordpress');
define('DB_USER', 'myuser');
define('DB_PASSWORD', 'mypass.1');
define('DB_HOST', '<DB-Server-Private-IP-Address>');
```

4. Access WordPress Setup from a Browser:
```
http://<Web-Server-Public-IP-Address>/wordpress/
```
Here, WordPress will walk you through the final setup steps, including:

- Site Title: Set the title of your WordPress site.
- Admin Username: Create a username for the admin account.
- Password: Create a secure password for the admin account.
- Email: Provide an email address for the administrator account.

Once you fill in these details, click Install WordPress.


![](img/browser%20wp1.png)


If it shows the above, the configuration on`/var/www/html/wordpress/wp-config.php` was successful.

![](img/browser%20wp2.png)

---

### Obstacles Overcome

During the process, there was only one notable obstacle:

1. **Partitioning the Disk with Incorrect Filesystem Type:**  
   When partitioning one of the disks, I accidentally set the partition type to the default Linux filesystem (8300) instead of Linux LVM (8E00). This could have caused issues later in volume management. To resolve this, I used the `t` command in `gdisk` to change the partition type back to the correct Linux LVM format. This was a crucial step in avoiding potential complications with logical volume management.

By catching this error early and correcting it with `gdisk`, I ensured that the logical volumes for the web and database servers were created and managed correctly without affecting the rest of the process.

### Lessons Learned

Throughout this project, I gained valuable insights that will serve me well in future implementations:

1. **Importance of Disk Management:**  
   Working with `gdisk` for partitioning and LVM for volume management on RedHat gave me hands-on experience in managing storage infrastructure. I learned how crucial it is to be cautious during partitioning and double-check the filesystem types before proceeding. Correctly configuring the storage subsystems ensured that the web server and database server could handle data storage effectively.

2. **Consistency in Configuration Across Servers:**  
   While the storage configuration for the web and database servers was largely similar, I learned how to adapt the process when dealing with different server roles. This reinforced the importance of consistency in disk management while allowing for flexibility depending on the server’s function, such as mounting logical volumes to different directories based on server roles (e.g., `/var/www/html` for web files vs. `/db` for database storage).

3. **Understanding Three-Tier Architecture in Practice:**  
   Implementing a three-tier architecture involving separate presentation, business, and data layers gave me practical experience in deploying scalable and secure web applications. The separation of WordPress as the business logic layer and MySQL as the data layer taught me how important it is to isolate different components of a web solution to ensure better performance and security.

4. **Remote Database Configuration and Security:**  
   Setting up a remote MySQL database and ensuring secure communication between the web and database servers highlighted the importance of configuring security groups and using private IP addresses. I also reinforced the use of CIDR notation for restricting MySQL access, which enhanced the security of my setup. It was a valuable learning point in managing secure connections between cloud-based services.

5. **RedHat vs. Ubuntu Differences:**  
   Switching to RedHat from Ubuntu made me more aware of the differences between Linux distributions, especially in terms of package management (`yum` vs. `apt`). This increased my adaptability to different enterprise environments where RedHat is more commonly used, providing me with broader exposure to different Linux server management tools.


### Conclusion

Completing this project was both a challenging and rewarding experience. I gained hands-on skills in setting up a WordPress-based web solution using a three-tier architecture, along with managing Linux disk partitioning, LVM, and secure web-database integration on RedHat EC2 instances. Each step reinforced my understanding of cloud-based infrastructure and server management, pushing me to navigate through complex technical configurations.

While there were some hectic moments—especially with managing storage infrastructure and ensuring secure communication between servers—these challenges taught me the importance of adaptability and attention to detail. This project has significantly boosted my confidence in tackling more advanced cloud deployments and left me better prepared for future technical challenges.



