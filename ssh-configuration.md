# Konfigurasi port SSH

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
ssh username@<IP Your SSH> -p <Port your SSH>
```
