# Cloud Migration Project — Legacy System to AWS
### WordPress LAMP Stack Deployment
**Author:** Naylor Robinson
**Date:** February 2026
**Technology:** AWS EC2 | Ubuntu | Apache | MySQL | PHP | WordPress | Terraform

---

## 📋 SCENARIO

A company was running its website on an outdated legacy system — a physical on-premise server that was difficult to maintain, expensive to operate, and unable to scale with growing traffic demands. The server was running an aging operating system with outdated software, creating serious security vulnerabilities and performance bottlenecks. The company needed to modernize its infrastructure by migrating to the cloud to improve reliability, security, and scalability.

The goal of this project was to migrate the company's legacy system to AWS by provisioning a cloud-based virtual server and deploying a fully functional WordPress website on a LAMP stack (Linux, Apache, MySQL, PHP). This would replace the old physical server entirely and give the company a modern, manageable, and scalable web presence in the cloud.

---

## 🚧 OBSTACLE

Several obstacles were encountered throughout the migration process:

**1. Infrastructure Provisioning**
Setting up the correct AWS security group configuration was critical. The server needed specific ports open — Port 22 for SSH access, Port 80 for HTTP web traffic, and Port 443 for HTTPS — while remaining secure against unauthorized access.

**2. LAMP Stack Dependencies**
Installing and configuring Apache, MySQL, and PHP required careful sequencing. Each service had to be installed, started, and enabled in the correct order. Services also needed to be verified as running before proceeding to the next phase.

**3. MySQL Security and Database Setup**
Securing MySQL using `mysql_secure_installation` required careful responses to each prompt. Creating a dedicated WordPress database user and granting the correct privileges had to be executed precisely — running multiple SQL commands as a single statement caused syntax errors that needed troubleshooting.

**4. WordPress Configuration**
Connecting WordPress to the MySQL database required editing the `wp-config.php` configuration file with the exact database name, username, and password. Any mismatch between the MySQL credentials and the WordPress config would prevent the site from loading.

**5. File Permissions**
Setting the correct file ownership and permissions on the WordPress directory was essential. Incorrect permissions would prevent Apache from reading and writing files, causing the site to fail silently.

---

## ⚡ ACTION

The following actions were taken to complete the migration:

**Phase 1 — Infrastructure**
Used Terraform to provision an EC2 instance running Ubuntu 22.04 LTS with a security group allowing SSH (22), HTTP (80), and HTTPS (443) traffic. Infrastructure as code ensured the deployment was repeatable and consistent.

```bash
terraform init
terraform plan
terraform apply
```

**Phase 2 — Server Setup**
SSH'd into the EC2 instance and updated the Ubuntu system to the latest packages. Installed the full LAMP stack in sequence:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install apache2 -y
sudo apt install php libapache2-mod-php php-mysql -y
sudo apt install mysql-server -y
```

Started and enabled all services to ensure they survive server reboots:

```bash
sudo systemctl start apache2 && sudo systemctl enable apache2
sudo systemctl start mysql && sudo systemctl enable mysql
```

**Phase 3 — Security**
Ran MySQL secure installation to remove anonymous users, disable remote root login, and set a strong root password:

```bash
sudo mysql_secure_installation
```

**Phase 4 — WordPress Installation**
Downloaded, extracted, and deployed WordPress files to the Apache web directory:

```bash
wget https://wordpress.org/latest.tar.gz
tar -xvzf latest.tar.gz
sudo mv wordpress/* /var/www/html/
sudo chown -R www-data:www-data /var/www/html/
sudo chmod -R 755 /var/www/html/
```

**Phase 5 — Database Setup**
Created a dedicated WordPress database and user inside MySQL:

```sql
CREATE DATABASE wordpress_db;
CREATE USER 'wordpress_user'@'localhost' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wordpress_user'@'localhost';
FLUSH PRIVILEGES;
```

**Phase 6 — WordPress Configuration**
Renamed the sample config file and updated it with the correct database credentials:

```bash
sudo mv /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
sudo nano /var/www/html/wp-config.php
```

Updated the following lines:
```php
define('DB_NAME', 'wordpress_db');
define('DB_USER', 'wordpress_user');
define('DB_PASSWORD', 'your_password');
```

Restarted Apache and removed the default index page to allow WordPress to load:

```bash
sudo systemctl restart apache2
sudo rm /var/www/html/index.html
```

**Phase 7 — Launch**
Accessed the WordPress setup page via the EC2 public IP in the browser:
```
http://YOUR_EC2_PUBLIC_IP
```
Completed the WordPress installation wizard and verified the dashboard was accessible.

---

## ✅ RESULT

The migration was completed successfully. The company's legacy on-premise system was fully replaced by a cloud-hosted WordPress website running on AWS. The final result included:

- A fully operational WordPress site accessible via the EC2 public IP
- A secure LAMP stack with Apache serving web traffic, MySQL storing data, and PHP powering WordPress
- Automated infrastructure provisioning using Terraform making the deployment repeatable
- Services configured to start automatically on server reboot ensuring high availability
- A dedicated MySQL user with scoped permissions following security best practices

The company now has a modern, scalable web infrastructure in AWS that can handle increased traffic, is easier to maintain, and eliminates the cost and risk of managing physical hardware.

---

## 🔧 TROUBLESHOOT

The following issues were encountered and resolved during the project:

---

**Issue 1 — MySQL GRANT Syntax Error**
```
ERROR 1064 (42000): You have an error in your SQL syntax near 'GRANT ALL PRIVILEGES'
```
**Cause:** The `CREATE USER` and `GRANT ALL PRIVILEGES` commands were pasted together on the same line. MySQL interpreted them as a single malformed command.

**Fix:** Each SQL command must be run separately with its own semicolon. Commands were split and executed one at a time:
```sql
CREATE USER 'wordpress_user'@'localhost' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wordpress_user'@'localhost';
FLUSH PRIVILEGES;
```

---

**Issue 2 — Browser Could Not Reach EC2 Public IP**
```
ERR_CONNECTION_REFUSED — This site can't be reached
```
**Cause:** One of three possible causes — Apache not running, port 80 not open in the security group, or WordPress files not in the correct directory.

**Fix:** Verified Apache status with `sudo systemctl status apache2`, confirmed port 80 was open in the AWS security group inbound rules, and verified WordPress files were present in `/var/www/html/` using `ls /var/www/html/`.

---

**Issue 3 — MySQL Arrow Prompt (->)**
**Cause:** A command was entered without a semicolon causing MySQL to wait for the statement to be completed.

**Fix:** Typed a semicolon `;` and pressed Enter to complete the hanging statement, then re-ran the correct command.

---

**Issue 4 — Permission Denied on File Operations**
**Cause:** Running file operations without `sudo` in protected directories like `/var/www/html/`.

**Fix:** Added `sudo` prefix to all file operations in protected directories.

---

*This project was completed as part of a Cloud Engineering and DevOps training program.*
