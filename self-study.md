### **Disk Partitioning and LVM (Logical Volume Management)**

- **Disk Partitioning**:
  - Partitioning is the process of dividing a disk into sections to use each independently. Each partition can hold its own file system or data. This is important for separating different types of data, like system files and application data.
  - I used **gdisk** to create partitions on EBS volumes. This tool allows for more flexible partitioning, especially for larger disks.
  - Typical usage:
    ```bash
    sudo gdisk /dev/xvdd
    ```
  - **Example**:
    - When partitioning the disk `/dev/xvdd`, I created a new partition for the web application using the `8E00` type for **Linux LVM**.

- **LVM (Logical Volume Management)**:
  - LVM allows me to manage disk space more flexibly than traditional partitions by creating **physical volumes**, **volume groups**, and **logical volumes**. With LVM, I can resize volumes, create snapshots, or combine multiple physical disks into one logical unit.
  - **Example**:
    - After partitioning, I turned the partition into a physical volume (`pvcreate`), added it to a volume group (`vgcreate`), and created logical volumes (`lvcreate`) for the application and logs.

    Commands used:
    ```bash
    sudo pvcreate /dev/xvdd1
    sudo vgcreate webdata-vg /dev/xvdd1
    sudo lvcreate -n apps-lv -L 14G webdata-vg
    ```

---

### **Mounting and Formatting**

- **Formatting**:
  - Before mounting, I formatted the new logical volumes with the **ext4** file system. Formatting prepares the logical volumes to store data.
  - Command used:
    ```bash
    sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
    ```

- **Mounting**:
  - After formatting, I mounted the logical volumes to directories where they’ll store data (like `/var/www/html` for the WordPress application). Mounting makes the volume accessible to the system.
  - Command used:
    ```bash
    sudo mount /dev/webdata-vg/apps-lv /var/www/html/
    ```

- **Persistent Mounts with /etc/fstab**:
  - To ensure the volumes automatically mount after reboot, I edited the `/etc/fstab` file using the volume UUID.
  - I retrieved the UUID with:
    ```bash
    sudo blkid
    ```

---

### **Web Server Setup (Apache & WordPress)**

- **Apache**:
  - Apache is a popular web server software. I used **yum** to install it on the server, then started the service to host the WordPress site.
  - Command used:
    ```bash
    sudo yum -y install httpd
    sudo systemctl start httpd
    ```

- **WordPress**:
  - I downloaded and extracted WordPress, which is the content management system (CMS) used to create the site. WordPress runs on PHP and MySQL, so I ensured these were also installed and configured.
  - Command used:
    ```bash
    wget http://wordpress.org/latest.tar.gz
    tar -xzvf latest.tar.gz
    ```

- **SELinux Configuration**:
  - Since I’m using RedHat, SELinux is enabled by default for extra security. I configured it to allow Apache to connect to external networks and work properly with WordPress.
  - Command used:
    ```bash
    sudo setsebool -P httpd_can_network_connect=1
    ```

---

### **Database Setup (MySQL)**

- **MySQL Database**:
  - I set up a MySQL database on a separate server. WordPress uses this database to store site data. I created a new database and user for WordPress to connect to.
  - Commands used in MySQL:
    ```sql
    CREATE DATABASE wordpress;
    CREATE USER 'myuser'@'Web-Server-Private-IP' IDENTIFIED BY 'password';
    GRANT ALL ON wordpress.* TO 'myuser'@'Web-Server-Private-IP';
    ```

- **Remote Access**:
  - WordPress and MySQL are hosted on different servers, so I configured MySQL to accept remote connections. I also adjusted the security group settings in AWS to allow MySQL traffic from the web server only.
  - Ensuring secure communication between the web and database servers was crucial to avoid unauthorized access.

---

### **Security and AWS Configurations**

- **Security Groups**:
  - In AWS, security groups act as firewalls to control inbound and outbound traffic for EC2 instances. I configured security groups to allow:
    - HTTP traffic (port 80) to the web server.
    - MySQL traffic (port 3306) only between the web and database servers using private IP addresses for extra security.

  - **Example**: For MySQL, I allowed inbound traffic on port 3306 from the web server’s private IP using `/32` notation to ensure only the web server could connect.

---

### **Service Management and Common Commands**

- **Service Management**:
  - After installing services like Apache and MySQL, I used `systemctl` to start, stop, or enable services to automatically start on boot.
  - Commands used:
    ```bash
    sudo systemctl start httpd
    sudo systemctl enable httpd
    sudo systemctl start mysqld
    ```

- **Useful System Commands**:
  - **List block devices**: `lsblk` — Showed me all connected disks and their partitions.
  - **Check disk space**: `df -h` — Showed available and used disk space on mounted file systems.
  - **Check LVM status**: `lvdisplay` — Displayed detailed info about logical volumes.

---
