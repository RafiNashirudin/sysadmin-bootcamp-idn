# Install httpd & mod_ssl
## httpd

instal httpd  
```
sudo dnf install -y httpd mod_ssl
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

enable module userdir
```
sudo semanage boolean -m --on httpd_enable_homedirs
```

Edit file */etc/httpd/conf.d/userdir.conf*
```
#UserDir disabled
UserDir public_html
```
   
Restart httpd  
```
sudo systemctl restart httpd
```

# Install & konfigurasi Firewalld
## firewalld

install firewalld
```
sudo dnf install firewalld
```

firewall rules
```
sudo firewall-cmd --add-service=http --permanent
sudo firewall-cmd --add-service=https --permanent
sudo firewall-cmd --reload
```

# Create user & konfigurasi
# create 3 User

bash script membuat 3 user
```bash
for user in akane ruby arima; do
    sudo useradd -m $user
    echo "$user:animelovers" | sudo chpasswd
    sudo mkdir -p /home/$user/public_html
    sudo chown -R $user:$user /home/$user/public_html
    sudo chmod 755 /home/$user
done
```

# Instalasi & Konfigurasi php, mariadb, & phpMyAdmin
## PHP

enable repo crb
```
sudo dnf config-manager --set-enabled crb
```

install repo epel
```
sudo dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
sudo dnf install -y https://dl.fedoraproject.org/pub/epel/epel-next-release-latest-9.noarch.rpm
```

install repo remi
```
sudo dnf install -y https://rpms.remirepo.net/enterprise/remi-release-9.rpm
```

list module php
```
sudo dnf module list php
```

*if other versions are enabled, reset once and switch to the version*
```
sudo dnf module reset php
```

enable php version
```
sudo dnf module install -y php:remi-8.2
```

install php
```
sudo dnf install -y php php-cli php-fpm php-json php-common php-mysqlnd php-zip php-gd php-mbstring php-curl php-xml php-pear php-bcmath php-json
```

check php version
```
php -v
```

enable php-fpm
```
sudo systemctl enable --now php-fpm
```

## mariadb

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
```
#!/bin/bash

# Konfigurasi
DB_PASSWORD="B-Komachi"
USERS=("akane" "ruby" "arima")

# Eksekusi MySQL
for user in "${USERS[@]}"; do
    mysql -u root -p -e "
    CREATE DATABASE wp_${user};
    CREATE USER 'wp_${user}'@'localhost' IDENTIFIED BY '${DB_PASSWORD}';
    GRANT ALL PRIVILEGES ON wp_${user}.* TO 'wp_${user}'@'localhost';
    FLUSH PRIVILEGES;"
done

echo "Database dan user MySQL telah dibuat untuk: ${USERS[*]}"
```

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

# Instalasi & Konfigurasi WordPress untuk Setiap User
## Wordpress

Download dan Ekstrak WordPress  
```
for user in akane ruby arima; do
    sudo -u $user wget -O /home/$user/public_html/latest.tar.gz https://wordpress.org/latest.tar.gz
    sudo -u $user tar -xzf /home/$user/public_html/latest.tar.gz -C /home/$user/public_html
    sudo -u $user mv /home/$user/public_html/wordpress/* /home/$user/public_html/
    sudo -u $user rm -rf /home/$user/public_html/latest.tar.gz /home/$user/public_html/wordpress
done
```

Konfigurasi **wp-config.php**  
```
for user in akane ruby arima; do
    sudo -u $user cp /home/$user/public_html/wp-config-sample.php /home/$user/public_html/wp-config.php
    sudo -u $user sed -i "s/database_name_here/wp_$user/" /home/$user/public_html/wp-config.php
    sudo -u $user sed -i "s/username_here/wp_$user/" /home/$user/public_html/wp-config.php
    sudo -u $user sed -i "s/password_here/B-Komachi/" /home/$user/public_html/wp-config.php
done
```

# Konfigurasi Domain
## httpd

Buat konfigurasi virtualhost di `/etc/httpd/conf.d/akane.local.conf`.
Tambahkan konfigurasi VirtualHost untuk masing-masing user:
```
sudo nano /etc/httpd/conf.d/akane.local.conf
```

```
<VirtualHost *:80>
    ServerAdmin webmaster@akane.local
    DocumentRoot "/home/akane/public_html"
    ServerName akane.local
    ErrorLog "/var/log/httpd/akane.local-error_log"
    CustomLog "/var/log/httpd/akane.local-access_log" common
</VirtualHost>
```
*(Ulangi untuk user2 dan user3 dengan nama virtualhost untuk http berbeda)*

Buat konfigurasi virtualhost untuk https di `/etc/httpd/conf.d/ssl-akane.local.conf`.
```
<VirtualHost *:443>
    ServerAdmin webmaster@akane.local
    DocumentRoot "/home/akane/public_html"
    ServerName akane.local
    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/akane.crt
    SSLCertificateKeyFile /etc/pki/tls/private/akane.key
    ErrorLog "/var/log/httpd/ssl-akane.local-error_log"
    CustomLog "/var/log/httpd/ssl-akane.local-access_log" common
</VirtualHost>
```
*(Ulangi untuk user2 dan user3 dengan nama virtualhost untuk https berbeda)*

Restart Apache  
```
sudo systemctl restart httpd
```

**Jika Menggunakan Domain Lokal (Tanpa DNS Publik)**  
Tambahkan ke `/etc/hosts`:  
```
127.0.0.1 user1.local
127.0.0.1 user2.local
127.0.0.1 user3.local
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
```
for user in akane ruby arima; do
  openssl req -new -x509 -days 365 -nodes -out /etc/pki/tls/certs/${user}.crt -keyout /etc/pki/tls/private/${user}.key \
    -subj "/C=ID/ST=Jawa Timur/L=Surabaya/O=Local User/CN=${user}.local"
done
```
Tambahkan SSL ke VirtualHost.

# Keamanan dan Hardening
## SELinux

untuk melihat type selinux yang sedang digunakan

```
getenforce
```

untuk mengganti type selinux bisa edit dibawah ini pada bagian "SELINUX=".

```
nano /etc/selinux/config

# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
# See also:
# https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/using_selinux/changing-selinux-states-and-modes_using-selinux#changing-selinux-modes-at-boot-time_changing-selinux-states-and-modes
#
# NOTE: Up to RHEL 8 release included, SELINUX=disabled would also
# fully disable SELinux during boot. If you need a system with SELinux
# fully disabled instead of SELinux running with no policy loaded, you
# need to pass selinux=0 to the kernel command line. You can use grubby
# to persistently set the bootloader to boot with selinux=0:
#
#    grubby --update-kernel ALL --args selinux=0
#
# To revert back to SELinux enabled:
#
#    grubby --update-kernel ALL --remove-args selinux
#
SELINUX=permissive
# SELINUXTYPE= can take one of these three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

```
grubby --update-kernel ALL --remove-args selinux
```

```
reboot
```

lalu cek status selinux

```
getenforce
```

```
sestatus
```

jika sudah "permissive" maka ganti lagi ke dalam mode enforcing

```
nano /etc/selinux/config
```

```

# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
# See also:
# https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/using_selinux/changing-selinux-states-and-modes_using-selinux#changing-selinux-modes-at-boot-time_changing-selinux-states-and-modes
#
# NOTE: Up to RHEL 8 release included, SELINUX=disabled would also
# fully disable SELinux during boot. If you need a system with SELinux
# fully disabled instead of SELinux running with no policy loaded, you
# need to pass selinux=0 to the kernel command line. You can use grubby
# to persistently set the bootloader to boot with selinux=0:
#
#    grubby --update-kernel ALL --args selinux=0
#
# To revert back to SELinux enabled:
#
#    grubby --update-kernel ALL --remove-args selinux
#
SELINUX=enforcing
# SELINUXTYPE= can take one of these three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

```
reboot
```

lalu cek kembali untuk selinux sudah berganti ke "enforcing" atau belum

```
sestatus
```

```
getenforce
```

# Konfigurasi SSH
## SSH

Backup file sshd_config:

```
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
```

Open file sshd_config:

```
sudo nano /etc/ssh/sshd_config
```

Look for the line that contains:

```
Port 22
```

Change it to:

```
Port 2222
```

Tambahkan:
```
AllowUsers akane ruby arima
```

Restart SSH:  
```
sudo systemctl restart sshd
```

Nonaktif Login Root SSH  
```
sudo sed -i 's/#PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
sudo systemctl restart sshd
```

# Konfigurasi port SSH Firewalld

## Firewalld

Allow port 2222

```
sudo firewall-cmd --permanent --add-port=2222/tcp
```

Reload firewalld

```
sudo firewall-cmd --reload
```

Check firewalld

```
sudo firewall-cmd --list-all
```

# Konfigurasi port SSH SELinux

## SELinux

Allow port 2222 SELinux:

```
sudo semanage port -a -t ssh_port_t -p tcp 2222
```

List port SSH SELinux

```
sudo semanage port -l | grep ssh
```

# restart SSH service and test SSH login

## SSH

Restart SSH

```
sudo systemctl restart sshd
```

SSH login test

```
ssh -i <Your-Key>.pem username@<IP Your SSH> -p <Port your SSH>
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
