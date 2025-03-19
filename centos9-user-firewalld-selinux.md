# User and Group Management

## User Management

Buat 10 User (user1 - user10)

```
# Loop dari angka 1 sampai 10
for i in {1..10}; do
    # Membuat user baru dengan nama user1, user2, ..., user10
    useradd user$i

    # Membuat password acak 12 karakter untuk setiap user
    PASSWORD=$(openssl rand -base64 12)

    # Mengatur password user menggunakan input dari stdin
    echo "$PASSWORD" | passwd --stdin user$i

    # Menampilkan informasi user dan password yang dibuat
    echo "User user$i dibuat dengan password: $PASSWORD"
done
```

## Group Management

Buat 2 Group (ganjil & genap) dan masukkan user sesuai kategorinya

```
groupadd ganjil
groupadd genap
```

Masukkan user ke grup sesuai dengan nomor:

```
# Loop dari angka 1 sampai 10
for i in {1..10}; do
    # Mengecek apakah angka i adalah genap
    if (( $i % 2 == 0 )); then
        # Jika genap, tambahkan user ke grup 'genap'
        usermod -aG genap user$i
    else
        # Jika ganjil, tambahkan user ke grup 'ganjil'
        usermod -aG ganjil user$i
    fi
done
```

Cek apakah user telah masuk ke grup yang sesuai:

```
for i in {1..10}; do
    groups user$i
done
```

# Firewalld

install firewalld

```
sudo dnf install firewalld
```

check firewall

```
systemctl status firewalld
```

enable firewall

```
systemctl enable --now firewalld
```

allow port 80

```
sudo firewall-cmd --zone=public --add-port=80/tcp --permanent
```

allow port 443

```
sudo firewall-cmd --zone=public --add-port=443/tcp --permanent
```

reload firewalld

```
firewall-cmd --reload
```

check firewalld

```
firewall-cmd --list-all
```

check default zone

```
firewall-cmd --get-default-zone
```

# Selinux

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
