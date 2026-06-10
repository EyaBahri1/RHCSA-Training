
# BaseOS & AppStream Repositories

## Introduction
👋 In this section, I will show you how to configure 'YUM repositories'.

Edit the repository files:

```bash
vim /etc/yum.repos.d/BaseOs.repo
[BaseOs]
name=BaseOs
baseurl=…/BaseOs
enabled=1
gpgcheck=1 (if a key URL is provided) otherwise gpgcheck=0
gpgkey=… (if a key URL is provided)
```

```bash
vim /etc/yum.repos.d/AppStream.repo
[AppStream]
name=AppStream
baseurl=…/AppStream
enabled=1
gpgcheck=1 (if a key URL is provided) otherwise gpgcheck=0
gpgkey=… (if a key URL is provided)
```

If you have created the files manually and are facing issues:
```bash
rm -rf /etc/yum.repos.d/*
reset the machine
subscription-manager refresh
```
## Lab — YUM / DNF Repository Configuration
---
### Exercise 1 — Configure a network repo

**Task:**
- Configure the following repositories on `serverA`:
  - BaseOS: `http://content.example.com/rhel9/BaseOS`
  - AppStream: `http://content.example.com/rhel9/AppStream`
- Install `httpd` to verify the repos work

**Solution:**
```bash
vim /etc/yum.repos.d/exam.repo
```
```ini
[BaseOS]
name=BaseOS
baseurl=http://content.example.com/rhel9/BaseOS
enabled=1
gpgcheck=0

[AppStream]
name=AppStream
baseurl=http://content.example.com/rhel9/AppStream
enabled=1
gpgcheck=0
```
```bash
dnf clean all
dnf repolist                  # verify both repos appear
dnf install -y httpd          # test
```
---
### Exercise 2 — Configure a local repo from ISO

**Task:**
- The RHEL9 ISO is available at `/root/rhel9.iso`
- Mount it persistently and configure it as a local repository
- Verify with `dnf repolist`

**Solution:**
```bash
mkdir /repo
mount -o loop /root/rhel9.iso /repo   # treats the ISO file as a virtual block device

# make it persistent after reboot
echo "/root/rhel9.iso /repo iso9660 loop,ro 0 0" >> /etc/fstab
mount -a                      # verify fstab is correct

vim /etc/yum.repos.d/local.repo
```
```ini
[local-BaseOS]
name=local BaseOS
baseurl=file:///repo/BaseOS
enabled=1
gpgcheck=0

[local-AppStream]
name=local AppStream
baseurl=file:///repo/AppStream
Enabled=1
gpgcheck=0
```
```bash
dnf clean all
dnf repolist                  # both repos must appear
dnf install -y httpd          # test
```
