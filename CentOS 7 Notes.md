    # Update system
    yum -y update
    yum -y groupinstall "Development Tools"
    reboot

    # Hostname and SSH keys
    hostname nikhil.io
    hostname > /etc/hostname
    SSH_KEY_PASSPHRASE=""
    ssh-keygen -q -N "$SSH_KEY_PASSPHRASE" -t rsa -f ~/.ssh/id_rsa
    ssh-keygen -q -N "$SSH_KEY_PASSPHRASE" -t dsa -f ~/.ssh/id_dsa

    # Disable SELinux
    sed -ie 's/SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config

    # Install NTP
    yum -y install ntp
    systemctl start ntpd
    systemctl enable ntpd

    # Basic packages; using IPTables for now
    yum -y install git zip rar nmap java-1.7.0-openjdk iptables python-virtualenv bind-utils mlocate

    # Nginx
    rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm

    # MySQL/MariaDB
    yum -y install mariadb mariadb-server
    systemctl start mariadb; systemctl enable mariadb
    mysql_secure_installation

    # PHP-FPM
    yum -y install php-fpm
    systemctl start php-fpm; systemctl enable php-fpm

    # EPEL 7 (beta)
    rpm -ivh http://mirror.ancl.hawaii.edu/linux/epel/beta/7/x86_64/epel-release-7-0.2.noarch.rpm
    sed -ie 's/enabled=1/enabled=0/g' /etc/yum.repos.d/epel.repo

    # EPEL packages
    yum install htop byobu bash-completion iotop ncdu sysstat --enablerepo=epel

    # Changed SSHD port
    # .bash_profile as in Github project
    updatedb

[Category: Nikhil's Notes](Category:_Nikhil's_Notes "wikilink")
[Category: Installation Logs](Category:_Installation_Logs "wikilink")
[Category: Linux](Category:_Linux "wikilink")
