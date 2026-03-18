# PrestaShop Deployment & Security Documentation

![Platform](https://img.shields.io/badge/Platform-Ubuntu-orange?style=flat-square)
![Web Server](https://img.shields.io/badge/Web%20Server-Apache-red?style=flat-square)
![Database](https://img.shields.io/badge/Database-MariaDB-blue?style=flat-square)
![Language](https://img.shields.io/badge/PHP-8.3-purple?style=flat-square)
![Security](https://img.shields.io/badge/Focus-Web%20Security-green?style=flat-square)
![Status](https://img.shields.io/badge/Project-Completed-brightgreen?style=flat-square)

This repository documents the full deployment, security hardening, and attack simulation testing of PrestaShop on Ubuntu.

---

## 📌 Project Overview

| Category | Details |
|---------|---------|
| Project Name | PrestaShop Deployment & Security Testing |
| Environment | Ubuntu Server (Virtual Machine) |
| Web Application | PrestaShop 8.1.5 |
| Database | MariaDB |
| Web Server | Apache2 |
| Attacker Machine | Kali Linux |
| Security Tools | Gobuster, Hydra |
| Attack Types Tested | Directory Enumeration, XSS, Brute Force |
| Objective | Deploy, secure, and validate defenses through simulated attacks |

---

## 🎯 Threat Model

| Asset | Threat | Attack Method | Potential Impact | Mitigation Applied |
|------|-------|--------------|------------------|-------------------|
| Admin Panel | Unauthorized access | Brute-force attack (Hydra) | Account compromise | Strong password + randomized admin URL |
| User Input Fields | Script injection | Cross-Site Scripting (XSS) | Session hijacking | Input sanitization |
| Web Directories | Information disclosure | Directory enumeration (Gobuster) | Exposure of sensitive paths | Server configuration hardening |
| Installation Files | Unauthorized setup access | Direct URL access | Site takeover | Installation folder removed |

---

## 📑 Table of Contents

- [1. Deployment Steps](#1-deployment-steps)
  - [1.1 System Update](#11-system-update)
  - [1.2 Install Apache](#12-install-apache)
  - [1.3 Install & Configure MariaDB](#13-install--configure-mariadb)
  - [1.4 Install PHP 8.3 & Required Extensions](#14-install-php-83--required-extensions)
  - [1.5 Configure PHP](#15-configure-php)
  - [1.6 Download & Deploy PrestaShop](#16-download--deploy-prestashop)
  - [1.7 Configure Apache Virtual Host](#17-configure-apache-virtual-host)
  - [1.8 Finalize Installation](#18-finalize-installation)
- [2. PrestaShop Security Checklist](#2-prestashop-security-checklist)
- [3. PrestaShop Attack Simulation](#3-prestashop-attack-simulation)
- [4. Findings](#4-findings)
- [5. Mitigation / Defense](#5-mitigation--defense)
- [6. Conclusion](#7-conclusion)

---

## 1. Deployment Steps

### 1.1 System Update
Update your system packages:

`sudo apt update && sudo apt upgrade -y`

### 1.2 Install Apache
Install Apache and enable the service:

`sudo apt install apache2 -y`  
`sudo systemctl enable apache2`  
`sudo systemctl start apache2`

### 1.3 Install & Configure MariaDB
Install MariaDB and start the service:

`sudo apt install mariadb-server mariadb-client -y`  
`sudo systemctl enable mariadb`  
`sudo systemctl start mariadb`

**Create the database and user for PrestaShop:**

```sql
sudo mysql -u root -p
```
CREATE DATABASE prestashop CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci; 
CREATE USER 'psuser'@'localhost' IDENTIFIED BY 'StrongPasswordHere'; 
GRANT ALL PRIVILEGES ON prestashop.* TO 'psuser'@'localhost'; 
FLUSH PRIVILEGES; 
EXIT;

> **Important:** Replace `StrongPasswordHere` with a strong, unique password. You will need the `psuser` username and this password during web installation.

### 1.4 Install PHP 8.3 & Required Extensions
Install PHP and extensions:

`sudo apt install php8.3-zip`  
`sudo apt install php8.3-gd php8.3-curl php8.3-intl php8.3-mbstring php8.3-xml php8.3-mysql`  
`sudo apt install php8.3-cli php8.3-common php8.3-soap php8.3-opcache libapache2-mod-php8.3 -y`  
`php -v`

### 1.5 Configure PHP
Edit **/etc/php/8.3/apache2/php.ini**

`sudo nano /etc/php/8.3/apache2/php.ini`

Then inside file, set:
- memory_limit = 512M 
- upload_max_filesize = 64M 
- post_max_size = 64M 
- max_execution_time = 300 
- max_input_time = 300 
- date.timezone = Europe/Bucharest

Restart Apache:

`sudo systemctl restart apache2`

### 1.6 Download & Deploy PrestaShop
Download PrestaShop and deploy:

`cd /tmp`  
`wget https://github.com/PrestaShop/PrestaShop/releases/download/8.1.5/prestashop_8.1.5.zip`  
`sudo apt install unzip -y`  
`unzip prestashop_8.1.5.zip -d prestashop`  
`sudo mv prestashop /var/www/html/`  
`sudo chown -R www-data:www-data /var/www/html/prestashop`  
`sudo chmod -R 755 /var/www/html/prestashop`

### 1.7 Configure Apache Virtual Host
Create **/etc/apache2/sites-available/prestashop.conf:**

`sudo nano /etc/apache2/sites-available/prestashop.conf`

Inside the file, add:

<VirtualHost *:80>
ServerName yourdomain.com
DocumentRoot /var/www/html/prestashop

<Directory /var/www/html/prestashop>
    AllowOverride All
    Require all granted
</Directory>

ErrorLog ${APACHE_LOG_DIR}/prestashop_error.log
CustomLog ${APACHE_LOG_DIR}/prestashop_access.log combined
</VirtualHost>

Enable site and rewrite module:

`sudo a2ensite prestashop`  
`sudo a2enmod rewrite`  
`sudo systemctl reload apache2`

### 1.8 Finalize Installation
Open a browser and access:

`http://<YOUR_UBUNTU_IP>/prestashop`

Complete the web installation wizard (language, license, database, admin account).  

Find the randomized admin folder:

`ls /var/www/html/prestashop | grep admin`

To Access Admin Panel:

`http://<YOUR_UBUNTU_IP>/prestashop/<admin_folder_name>`

Remove the installation directory:

`sudo rm -rf /var/www/html/prestashop/install`

---

## 2. PrestaShop Security Checklist

| Task | Action Taken | Verification / Notes |
|------|--------------|--------------------|
| Update Core & Modules | Latest versions installed | Dashboard → Modules → Module Manager |
| Remove Demo Accounts | Only admin exists | Dashboard → Configure → Team |
| Strong Admin Password | Created strong password | Admin login tested |
| File & Folder Permissions | Ownership www-data:www-data & permissions 755 | `ls -l /var/www/html/prestashop` |
| Session Cookies | HttpOnly & SameSite=Lax verified | Firefox DevTools → Storage → Cookies |
| Disable Unnecessary Modules | Only essential modules enabled | Module Manager check |
| Admin Path Protection | /install removed, randomized /admin folder | /install 404 confirmed |
| Payment Methods Configuration | Only required methods enabled | Payment → Payment Methods |

---

## 3. PrestaShop Attack Simulation

### 3.1 Lab Setup

Ubuntu PrestaShop target: Initially accessed via 127.0.0.1.  
Updated PrestaShop URL: Changed inside website configuration to 192.168.56.101 so the Kali attacker VM could reach the site.  
Kali attacker VM: Used to perform simulated attacks.  
Network: Host‑Only Adapter for Ubuntu ↔ Kali communication. 

### 3.2 Attack 1 — Directory Enumeration (Gobuster)
Command:  

`gobuster dir -u http://192.168.56.101/prestashop -w /usr/share/wordlists/dirb/common.txt`

## 3. PrestaShop Attack Simulation

### 3.2 Attack 1 — Directory Enumeration (Gobuster)
Command:  

`gobuster dir -u http://192.168.56.101/prestashop -w /usr/share/wordlists/dirb/common.txt`

Evidence: gobuster_scan.png

3.3 Attack 2 — XSS Injection Test
Payload:

`<script>alert('XSS')</script>`

Evidence: xss_test_result.png

3.4 Attack 3 — Brute Force Admin Login (Hydra)
Command:

`hydra -l admin@example.com -P passwords.txt 192.168.56.101 http-post-form "/prestashop/admin7xk29df/index.php:email=^USER^&passwd=^PASS^:Invalid"`

Evidence: hydra_attack.png

Apache log evidence:

`sudo tail -n 50 /var/log/apache2/error.log`

Evidence: apache_log_evidence.png

---

4.## Findings
   
- Brute-force login blocked
- XSS attack prevented
- Directory enumeration revealed paths, mitigated via configuration
- Logs confirm all attacks reached the server but were blocked

---

5. ## Mitigation / Defense
   
- Strong admin password enforced
- Admin URL randomized
- Directory listing disabled
- Secure cookies configured (HttpOnly, SameSite)
- Installation folder removed
- Unused modules disabled

---

6. Conclusion
The PrestaShop instance successfully resisted simulated attacks. Applied defenses (folder randomization, installer removal, input sanitization, strong credentials) provide a secure baseline for lab testing.
