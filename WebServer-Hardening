# Install httpd
## httpd

instal httpd  
```
sudo dnf install httpd -y
```

enable httpd
```
sudo systemctl enable --now httpd
```

check status httpd
```
sudo systemctl status httpd
```

# Enable Module Userdir
## module Userdir

Enable Modul Userdir  
```
sudo dnf install httpd-manual -y
sudo sed -i 's/#LoadModule userdir_module/LoadModule userdir_module/' /etc/httpd/conf/httpd.conf
sudo sed -i 's/#Include conf.d\/userdir.conf/Include conf.d\/userdir.conf/' /etc/httpd/conf/httpd.conf
```

Izinkan Akses Direktori Home User  
```
sudo sed -i 's/Deny from all/Require all granted/' /etc/httpd/conf.d/userdir.conf
```
   
Restart httpd  
```
sudo systemctl restart httpd
```

# Buat 3 User
bash script membuat 3 user
```bash
for user in user1 user2 user3; do
    sudo useradd -m $user
    echo "$user:password123" | sudo chpasswd
    sudo mkdir -p /home/$user/public_html
    sudo chown -R $user:$user /home/$user/public_html
    sudo chmod 755 /home/$user
done
```

# Instalasi & Konfigurasi mariadb
## mariadb

Install Dependensi  
```
sudo dnf install php php-mysqlnd php-fpm unzip -y
```

install mariadb
```
sudo dnf install mariadb-server
```

enable mariadb
```
sudo systemctl enable --now mariadb
```

check status mariadb
```
sudo systemctl status mariadb
```
   
Buat Database untuk Setiap User  
```bash
mysql -u root -p -e "
CREATE DATABASE wp_user1;
CREATE USER 'wp_user1'@'localhost' IDENTIFIED BY 'pass123';
GRANT ALL PRIVILEGES ON wp_user1.* TO 'wp_user1'@'localhost';
FLUSH PRIVILEGES;
"
```
*(Ulangi untuk user2 dan user3 dengan nama database & user berbeda)*  

# Instalasi & Konfigurasi WordPress untuk Setiap User
## Wordpress

Download dan Ekstrak WordPress  
```
for user in user1 user2 user3; do
    sudo -u $user wget -O /home/$user/public_html/latest.tar.gz https://wordpress.org/latest.tar.gz
    sudo -u $user tar -xzf /home/$user/public_html/latest.tar.gz -C /home/$user/public_html
    sudo -u $user mv /home/$user/public_html/wordpress/* /home/$user/public_html/
    sudo -u $user rm -rf /home/$user/public_html/latest.tar.gz /home/$user/public_html/wordpress
done
```

Konfigurasi **wp-config.php**  
```
for user in user1 user2 user3; do
    sudo -u $user cp /home/$user/public_html/wp-config-sample.php /home/$user/public_html/wp-config.php
    sudo -u $user sed -i "s/database_name_here/wp_$user/" /home/$user/public_html/wp-config.php
    sudo -u $user sed -i "s/username_here/wp_$user/" /home/$user/public_html/wp-config.php
    sudo -u $user sed -i "s/password_here/pass123/" /home/$user/public_html/wp-config.php
done
```

# Konfigurasi Domain
## httpd

Edit **/etc/httpd/conf.d/userdir.conf**
Tambahkan konfigurasi VirtualHost untuk masing-masing user:
```
sudo nano /etc/httpd/conf.d/userdir.conf
```
   
Tambahkan:  
```
<VirtualHost *:443>
    ServerAdmin webmaster@asrul.net
    DocumentRoot /home/user1/public_html
    ServerName your-ip
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/user1.crt
    SSLCertificateKeyFile /etc/ssl/private/user1.key
    ErrorLog "/var/log/httpd/user1.error_log"
    CustomLog "/var/log/httpd/user1.access_log" common
</VirtualHost>

<VirtualHost *:443>
    ServerAdmin webmaster@asrul.net
    DocumentRoot /home/user2/public_html
    ServerName your-ip
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/user2.crt
    SSLCertificateKeyFile /etc/ssl/private/user2.key
    ErrorLog "/var/log/httpd/user2.error_log"
    CustomLog "/var/log/httpd/user2.access_log" common
</VirtualHost>

<VirtualHost *:443>
    ServerAdmin webmaster@asrul.net
    DocumentRoot /home/user3/public_html
    ServerName your-ip
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/user3.crt
    SSLCertificateKeyFile /etc/ssl/private/user3.key
    ErrorLog "/var/log/httpd/user3.error_log"
    CustomLog "/var/log/httpd/user3.access_log" common
</VirtualHost>
```

Restart Apache  
```
sudo systemctl restart httpd
```

**Jika Menggunakan Domain Lokal (Tanpa DNS Publik)**  
Tambahkan ke `/etc/hosts`:  
```
127.0.0.1 user1.domain.com
127.0.0.1 user2.domain.com
127.0.0.1 user3.domain.com
```

# Instalasi SSL dengan Let's Encrypt
## Certbot

Install Certbot  
```
sudo dnf install certbot python3-certbot-apache -y
```
   
Dapatkan SSL untuk Domain  
```
sudo certbot --apache -d user1.domain.com -d user2.domain.com -d user3.domain.com
```

Perpanjangan Otomatis  
```
sudo crontab -e
```
   
Tambahkan:  
```
0 0 * * * certbot renew --quiet
```

## OpenSSL

*(Jika Menggunakan Domain Lokal, Gunakan OpenSSL untuk Buat Sertifikat Manual)*  
```bash
sudo openssl req -x509 -newkey rsa:4096 -keyout /etc/ssl/private/user1.key -out /etc/ssl/certs/user1.crt -days 365 -nodes
```
Tambahkan SSL ke VirtualHost.

# Tools Pengelolaan Database
## phpMyAdmin

Install phpMyAdmin  
```
sudo dnf install phpMyAdmin -y
```
   
Konfigurasi Apache untuk phpMyAdmin  
```
sudo nano /etc/httpd/conf.d/phpMyAdmin.conf
```

Tambahkan:
```
<Directory "/usr/share/phpMyAdmin">
    Require all granted
</Directory>
```
   
Restart Apache  
```
sudo systemctl restart httpd
```

Akses phpMyAdmin  
- `http://server-ip/phpmyadmin`

# Keamanan dan Hardening
## SELinux

Pastikan SELinux Aktif  
```
sudo setenforce 1
```

# Konfigurasi SSH
## SSH

atasi SSH ke IP Tertentu  
```
sudo nano /etc/ssh/sshd_config
```

Tambahkan:
```
AllowUsers user1 user2 user3
```

Restart SSH:  
```
sudo systemctl restart sshd
```

Nonaktifkan Login Root SSH  
```
sudo sed -i 's/#PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
sudo systemctl restart sshd
```

## Fail2Ban

Install Fail2Ban  
```
sudo dnf install fail2ban -y
```

enable Fail2Ban
```
sudo systemctl enable --now fail2ban
```
