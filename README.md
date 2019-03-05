# VMtools
Version 1.1.0

Tools for VMware ESXi to use in ESXi

The tools included in the VMtools suite are:
- **ovf-export** 1.1.0 - export/backup one or more virtual machines to OVF format
- **ovf-deploy** 1.1.0 - deploy/restore a virtual machine from OVF template
- **rsync-static** 3.1.3 - a pre-build static rsync version (see usage in https://rsync.samba.org/)

## Table of contents
- [Installation](#installation-on-esxi-server)
- [How to backup & restore](#how-to-backup--restore-vms)
- [License](#license)

## Installation on ESXi server

### Install vmtools

1. Get the files on a Linux OS (eg. Ubuntu):
  ```
  cd /tmp
  git clone https://github.com/noelmartinon/vmtools
  rm -rf /tmp/vmtools/.git
  ```
2. Copy the vmtools to ESXi (ssh must be enabled in ESXi): 
  ```
  scp -rp /tmp/vmtools root@ESXi_IP:/vmfs/volumes/datastore1/
  # OR if rsync is installed in ESXi (see howto below):
  # rsync -au /tmp/vmtools root@ESXi_IP:/vmfs/volumes/datastore1/
  ```

### Install VMware ovftool (required)
(ovftool-4.2.0 is included in vmtools)

VMware ovftool is free.

1. Download the Linux version to install in ESXi from [**VMware site**](https://my.vmware.com/). For ESXi 5.5 get the "VMware OVF Tool for Linux 64-bit" v4.2.0  [VMware OVF Tool for Linux 64-bit](https://my.vmware.com/fr/web/vmware/details?productId=353&downloadGroup=OVFTOOL420#product_downloads) otherwise you may have errors at runtime.
2. Install the file on a Linux OS (eg. Ubuntu):
  ```
  sudo ./VMware-ovftool-4.2.0-5965791-lin.x86_64.bundle
  ```
3. Copy VMware ovftool to ESXi (ssh must be enabled in ESXi): 
  ```
  rsync -au /usr/lib/vmware-ovftool root@ESXi_IP:/vmfs/volumes/datastore1/vmtools/
  ```
4. In ESXi, change ovftool shebang from 'bash' to 'sh':
  ```
  sed -i 's/bash/sh/' /vmfs/volumes/datastore1/vmtools/vmware-ovftool/ovftool
  ```

### Install 'rsync' (optional)

1. Get rsync

  This command lines must be execute from a linux console and NOT in ESXi!
- From vmtools site:
  ```
  cd /tmp
  wget https://github.com/noelmartinon/vmtools/master/bin/rsync-static
  mv rsync-static rsync
  chmod 775 rsync
  scp -p rsync root@ESXi_IP:/usr/bin/
  ```

- From source (Centos 3.9 required):
  ```
  # In CentOS 3.9 (http://vault.centos.org/3.9/isos/i386/CentOS-3.9-server-i386.iso):
  sed -i "s/centos\.org/hmc\.edu/g" /etc/yum.conf
  rpm --import http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-3
  yum groupinstall "Development Tools"
  yum install gcc zlib-devel popt-devel glibc-static popt-static -y
  wget http://rsync.samba.org/ftp/rsync/rsync-3.1.3.tar.gz
  gunzip rsync-3.1.3.tar.gz
  tar -xvf rsync-3.1.3.tar
  cd rsync-3.1.3/
  sed -i "s/<zlib.h>/<zlib.h>\n\n#ifndef Z_INSERT_ONLY\n#define Z_INSERT_ONLY Z_SYNC_FLUSH\n#endif/g" token.c
  ./configure
  make CFLAGS="-static" EXEEXT="-static"
  strip rsync-static
  scp -p rsync-static root@ESXi_IP:/usr/bin/rsync
     
  # /!\ Trying under Debian and other CentOS build OK but got 'segmentation fault' in esxi 
  # sudo apt install libpopt-dev zlibc
  # cd /tmp
  # wget http://rsync.samba.org/ftp/rsync/rsync-3.1.3.tar.gz
  # tar -xvf rsync-3.1.3.tar.gz
  # cd rsync-3.1.3/
  # sed -i "s/<zlib.h>/<zlib.h>\n\n#ifndef Z_INSERT_ONLY\n#define Z_INSERT_ONLY Z_SYNC_FLUSH\n#endif/g" token.c
  # ./configure
  # make CFLAGS="-static" EXEEXT="-static"
  # strip rsync-static
  ```

2. Set cron for the next boot

  The following commands must be run from ESXi shell.
  
  Insert command lines in local.sh:
  ```
  ~ # vi /etc/rc.local.d/local.sh 
  /bin/echo '0   23    *   *   6   [ $(date +\%d) -le 07 ] && /vmfs/volumes/datastore1/vmtools/jobs/job1 > /tmp/ovf-export.$$' >> /var/spool/cron/crontabs/root
  /bin/kill $(cat /var/run/crond.pid)
  /usr/lib/vmware/busybox/bin/busybox crond 
  ```

## How to backup & restore VMs
The tools use [**VMware OVF Tool**](https://www.vmware.com/support/developer/ovf/) to export and deploy virtual machines.

Features:
- Schedule an export for one are more VMs
- Restore a VM from backup OVF
- Email a report
- Test functionality with the --**test-mode**=true parameter

Advice: Set the path to a nfs datastore to have an separate storage

### 'ovf-export' usage
The following commands must be run from ESXi shell.

If --**user** parameter is undefined then it is set as _root_.

**Important** The process shutdown each VM before export and restart it if it was powered off

- Simple usage:
  ```
  /vmfs/volumes/datastore1/vmtools/ovf-export --password="pwdesxi" --backup-point="/path/to/backups_dir" --backup-vms=myvm_name
  ```
- Schedule a backup job:

  1 - Create a job file (see example in _jobs/job_export_example_)
  
  **jobs/job1**:
  ```
  #!/bin/sh

  "/vmfs/volumes/datastore1/vmtools/ovf-export" \
  --user="root" \
  --password="pwdesxi" \
  --smtp-srv="smtp.domain.tld" \
  --smtp-port=25 \
  --smtp-usr="user@domain.tld" \
  --smtp-pwd="smtppwd" \
  --mail-from="esxi-backup@domain.tld" \
  --mail-to="admin@domain.tld" \
  --subject="ESXi / Rapport de sauvegarde" \
  --backup-point="/vmfs/volumes/datastore_ext/BACKUPS" \
  --backup-vms="server1,server2
  ```
  
  2 - Add cron task:
  
  Example: for every first saturday of month at 11:00 PM
  
  _Standard output is redirected to a temp file in /tmp_
  ```
  /bin/echo '0   23    *   *   6   [ $(date +\%d) -le 07 ] && /vmfs/volumes/datastore1/vmtools/jobs/job1 > /tmp/ovf-export.$$' >> /var/spool/cron/crontabs/root
  /bin/kill $(cat /var/run/crond.pid)
  /usr/lib/vmware/busybox/bin/busybox crond
  ```
   
### 'ovf-deploy' usage
The following commands must be run from ESXi shell.

If --**user** parameter is undefined then it is set as _root_.

If --**datastore** parameter is undefined then it is set as _datastore1_.

**Important** The process do not powered on VM when it ended

- Create VM from OVF file (see example in _jobs/job_deploy_example_):

  Here the VM name is 'myvm' and the destination datastore is 'datastore1'
  ```
  "/vmfs/volumes/datastore1/vmtools/ovf-deploy" \
  --user="root" \
  --password="pwdesxi" \
  --datastore="datastore1" \
  --smtp-srv="smtp.domain.tld" \
  --smtp-port=25 \
  --smtp-usr="user@domain.tld" \
  --smtp-pwd="smtppwd" \
  --mail-from="esxi-backup@domain.tld" \
  --mail-to="admin@domain.tld" \
  --subject="ESXi / Rapport de sauvegarde" \
  --ovf-file="/vmfs/volumes/datastore_ext/BACKUPS/myvm/myvm.ovf" \
  --vm-name="myvm"
  ```

## License
- GPL-2.0
- Copyright (C) 2018 Noël MARTINON
