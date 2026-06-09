<h1 align="center" style="color: red;">Service Management</h1>

## Introduction
👋 In this section, we will learn how to configure and manage the **SSHD**, **HTTPD**, and **NTP** services.

## SSHD Service:

### Theory:

SSH allows a user to connect securely to a remote machine and run commands on it.
All communication is encrypted — credentials and commands can't be intercepted.
> Well-known ports (0–1023) are reserved for standard services: HTTP (80), HTTPS (443), SSH (22).
### Practice:

#### Connect to another machine via SSH using a password:

On the server machine:
* `yum install openssh-server` → installs the SSH server
* `systemctl enable sshd`
* `systemctl start sshd` → start and enable the SSH service
  
* `firewall-cmd --add-port=22/tcp --permanent`
* `firewall-cmd --reload`
* `firewall-cmd --list-ports` → open SSH port 22 for incoming connections
* Verify sshd is listening on the correct port:
  * `ss -tlnp | grep sshd`
    
On the client machine:
* `ssh username@server_ip_address` → connect using password

Optional — instead of using the IP, assign a hostname:
* `vim /etc/hosts` → add: `server_ip    hostname`
* `ssh username@hostname` → connect using the hostname instead of the IP
  
#### Connect without a password (key-based authentication):

To allow passwordless login, generate and configure SSH keys on machine B and copy them to the server. This allows the server to recognize and authorize connections from machine B.

* `ssh-keygen -t rsa` → generate SSH keys (on the client machine).
* `ssh-copy-id user@server_machine` → copy the public key to the server; adds it to the `authorized_keys` file in the user's `.ssh` directory.
* `ssh user@server_machine`

Required permissions on server:
* `chmod 700 ~/.ssh`
* `chmod 600 ~/.ssh/authorized_keys`

#### Connect via a port other than 22:

On the server side:

* `vim /etc/ssh/sshd_config`
  `Port 2222` → change the port to 2222.
* `semanage port -a -t ssh_port_t -p tcp 2222` → register the new port in SELinux.
* `firewall-cmd --permanent --add-port=2222/tcp`
* `firewall-cmd --reload` → update firewall rules.
* `systemctl restart sshd` → restart the SSH service.
  On the client side:
* `ssh user@server_address -p 2222` → connect using the new port.
  
#### Harden SSH access (sshd_config options):

* Disable root login:
  * `vim /etc/ssh/sshd_config` → set `PermitRootLogin no`
* Force key-based auth only (disable password login):
  * `vim /etc/ssh/sshd_config` → set `PasswordAuthentication no`
* Restrict SSH to specific users:
  * `vim /etc/ssh/sshd_config` → add `AllowUsers user1 user2`
* After any change:
  * `systemctl restart sshd`
---

## HTTPD Service:

### Theory:
A web server serves files (HTML, images, apps) to browsers via HTTP.
- Browser sends `GET /index.html` → server responds with the file content.
- 
> Default web root: `/var/www/html` — `index.html` is the default landing page.
### Practice:

* `dnf install httpd`
* `systemctl enable --now httpd` → install and enable the service.
* `systemctl status httpd` → check that the service is active.
* `echo "bnj" >> /var/www/html/index.html`
* `firewall-cmd --permanent --add-service=http`
* `firewall-cmd --permanent --add-service=https`
* `firewall-cmd --reload` → allow HTTP and HTTPS in the firewall.

Once the web server is running and `index.html` is in `/var/www/html`, accessing the server’s IP/domain in a browser will show its content.

* `dnf install curl`
* `curl http://server_ip/index.html`
  (If it doesn't work, try `chown apache:apache /var/www/html/index.html`)

#### Change default port (80) to a new port:

* `vim /etc/httpd/conf/httpd.conf`
  `Listen <new_port>`
* `semanage port -a -t http_port_t -p tcp <new_port>` → register the new port in SELinux.
* `firewall-cmd --permanent --add-port=<new_port>/tcp`
* `firewall-cmd --reload`
* `systemctl restart httpd`
* `curl localhost:<new_port>/index.html` → test the new port.

#### Change default directory `/var/www/html` to `/var1/web/html`:

* `vim /etc/httpd/conf/httpd.conf`
  `DocumentRoot "/var1/web/html"`

```apache
<Directory "/var1/web">
    AllowOverride None
    Require all granted
</Directory>
<Directory "/var1/web/html">
```

* `semanage fcontext -a -t httpd_sys_content_t "/var1/web/html(/.*)?"`
* `restorecon -R -v /var1/web/html/`
* `echo "hello2" >> /var1/web/html/test.html`
* `systemctl restart httpd`
* `curl localhost/test.html` or `curl http://host_ip/test.html`
  (If it doesn't work, try `chown -R apache:apache /var1`)
---

## NTP Service:

### Theory:
An NTP server provides accurate time to client computers on a network. It maintains precise time by regularly syncing with atomic or GPS time sources and responds to client requests with high accuracy.

### Practice:

#### On the server (to allow client access):

* `dnf install chrony`
* `systemctl enable chronyd`
* `systemctl start chronyd` → install and start the service.
* `vim /etc/chrony.conf`
  `allow client_ip`
* `systemctl restart chronyd`
* `firewall-cmd --add-service=ntp --permanent`
* `firewall-cmd --reload`

#### On the client (to access the time):

* `dnf install chrony`
* `systemctl enable chronyd`
* `systemctl start chronyd`
* `vim /etc/chrony.conf`
  `server server_ip iburst`
* `systemctl restart chronyd`
* `chronyc sources -c` → check synchronization

To enable NTP:

* `timedatectl set-ntp true`
* `timedatectl`

Set a time zone:

* `timedatectl list-timezones`
* `timedatectl set-timezone <zone>`
* `timedatectl`
