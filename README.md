# Local-WP-TestEnvironment
ubuntu server in Virtual Machine to run Wordpress Test environment on a i3 windows(11) laptop

Needed;

-latest Ubuntu server .ISO 
https://ubuntu.com/tutorials/install-ubuntu-server

-VMware free
i found These

Download options:

    Workstation Windows
    VMware Player Windows
    Workstation Linux
    VMware Player Linux
    Workstation Windows 17.5.2
    Workstation Windows 17.6.1
    Workstation Linux 17.6.1


here;
https://www.techspot.com/downloads/189-vmware-workstation-for-windows.html

-nginx
-mariaDB
-wp, 
but you will do those over command line.
as you will not be able to copy-paste from ure HOST pc to your GUEST OS,
i suggest you SSH in

Guide;

1. Install Ubuntu Server on VMware

    Download the Ubuntu Server ISO.
    Create a new VM in VMware:
        Select Linux → Ubuntu (64-bit).>> it should detect it and autofill.
        Allocate at least 2GB RAM & 1 CPU.
        Set disk size to 20GB+.(basic site, go larger if you think its required.)
        Attach the ISO file and install Ubuntu Server. >>>>>>>>>>>>>   https://ubuntu.com/tutorials/install-ubuntu-server#1-overview
    During installation, set up:
        A keyboard layout
        A user and password (e.g., yvan).
        A static IP (optional) via Netplan or use DHCP.
   
    After installation, login and update system:

    sudo apt update && sudo apt upgrade -y

3. Configure Network & Check Connectivity

    Check current IP address:

ip a

Test internet connectivity:

    ping -c 4 google.com

3. Install & Configure SSH

    Install SSH server:

sudo apt install openssh-server -y

Enable & start SSH:

sudo systemctl enable ssh
sudo systemctl start ssh

Check SSH status:

sudo systemctl status ssh

Find VM IP (send "ip a" in terminal, should be around enp0s3) and connect via SSH (from another machine):
In a terminal, send:
    ssh username@your_vm_ip

    (Replace your_vm_ip with your actual VM IP.)
(ex. you have installed ubuntu with user papa and pw mama, and after "ip a" you have gathered 192.168.111.4 as IP.
Next you open up CMD on windows, or terminal on another pc behind your router, and "SSH in" by sending; "ssh papa@192.168.111.4"
the terminal will ask for your pw, you send mama, and you are in your local VM server's terminal, ready to copy paste like a pro.)

4. Install PHP 8.3

    Add PHP PPA & install PHP 8.3 + extensions:

sudo apt install -y software-properties-common
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update
sudo apt install -y php8.3 php8.3-fpm php8.3-mysql php8.3-xml php8.3-mbstring php8.3-curl php8.3-zip

Check PHP version:

    php -v

5. Install Nginx

    Install Nginx:

sudo apt install nginx -y

Enable & start Nginx:

    sudo systemctl enable nginx
    sudo systemctl start nginx

    Test Nginx: Open browser & go to http://your_vm_ip/.
    (You should see the "Welcome to Nginx" page.)

6. Install MariaDB (MySQL Server Alternative)

    Install MariaDB:

sudo apt install mariadb-server -y

Secure installation:

sudo mysql_secure_installation

(Follow the prompts to set a root password & secure the database.)
Create WordPress Database:

sudo mysql -u root -p

Inside MariaDB shell, run:(with the relevant wp_user/password inserted, and localhost, replaced by the VM's IP you found with "ip a")

    CREATE DATABASE wordpress;
    CREATE USER 'wp_user'@'localhost' IDENTIFIED BY 'password';
    GRANT ALL PRIVILEGES ON wordpress.* TO 'wp_user'@'localhost';
    FLUSH PRIVILEGES;
    EXIT;

7. Install WordPress step

    Install dependencies:

sudo apt install -y wget unzip

Download & extract WordPress:

wget https://wordpress.org/latest.tar.gz
tar -xzvf latest.tar.gz
sudo mv wordpress /var/www/html/

Set correct permissions:

sudo chown -R www-data:www-data /var/www/html/wordpress
sudo chmod -R 755 /var/www/html/wordpress

Configure WordPress settings:

cd /var/www/html/wordpress
cp wp-config-sample.php wp-config.php

Edit database credentials:

sudo nano wp-config.php
(this opens up the wp config file in ure terminal)
Modify these lines:

    define('DB_NAME', 'wordpress');
    define('DB_USER', 'wp_user');
    define('DB_PASSWORD', 'password');
    define('DB_HOST', 'localhost');

    (Save with CTRL+X, then Y, then Enter.)

8. Configure Nginx for WordPress

    Create WordPress site configuration:

sudo nano /etc/nginx/sites-available/wordpress
(opens up this file in ure terminal again)
Paste this configuration(RePLACE THE IP ADRESS):

server {
    listen 80;
    server_name your_domain_or_IP;

    root /var/www/html/wordpress;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }

    error_log /var/log/nginx/wordpress_error.log;
    access_log /var/log/nginx/wordpress_access.log;
}
(Save with CTRL+X, then Y, then Enter.)

Enable the site & restart Nginx:

    sudo ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled/
    sudo nginx -t
    sudo systemctl restart nginx

9. Complete WordPress Installation

    Open a browser & go to:

    http://your_vm_ip/
it should open up the WP installer language picker. If it doesnt, start over:p 
    Follow WordPress setup:
        Choose language.
        Enter Site Name, Admin Username, Password.
        Click Install WordPress.

10. Enable Firewall (UFW)

    Allow SSH, HTTP, HTTPS:

sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable

Check firewall rules:

    sudo ufw status

11. Configure a Domain Name (Optional)

    Ensure DNS points to your server’s IP.
    Edit Nginx config:

sudo nano /etc/nginx/sites-available/wordpress

Update server_name:

server_name yourdomain.com www.yourdomain.com;

Reload Nginx:

    sudo systemctl reload nginx

12. Final Cleanup

    Remove installation files:

rm latest.tar.gz

Reboot system:

sudo reboot



At this point, shut down the VM and find your VM files in documents/virtual machines. Archive them. this is your Base backup.

ENJOY your FREE , LOCAL WordPress Test Environment.
you can test and try all u want

i suggest starting with updraft plugin https://wordpress.org/plugins/updraftplus/
making a backup, and downloading the files.

