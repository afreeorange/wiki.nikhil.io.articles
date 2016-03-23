On CentOS 7.2. With proxy and user namespace support.

Installation
------------

To enable user namespace support in the kernel,

    grubby --args="user_namespace.enable=1" --update-kernel=/boot/vmlinuz-3.10.0-327.10.1.el7.x86_64

Then install and start the service

    tee /etc/yum.repos.d/docker.repo <<-'EOF'
    [dockerrepo]
    name=Docker Repository
    baseurl=https://yum.dockerproject.org/repo/main/centos/$releasever/
    enabled=1
    gpgcheck=1
    gpgkey=https://yum.dockerproject.org/gpg
    EOF

    yum -y install docker-engine
    systemctl enable docker.service
    systemctl start docker.service

Configuration
-------------

Create this if it doesn't exist

    mkdir /etc/systemd/system/docker.service.d

### Proxy

    touch /etc/systemd/system/docker.service.d/http-proxy.conf

then add this

    [Service]
    Environment="HTTP_PROXY=http://proxy.example.com:80/"

### User Namespace Support

    touch /etc/systemd/system/docker.service.d/user-namespace.conf

then add this

    [Service]
    ExecStart=
    ExecStart=/usr/bin/docker daemon --userns-remap=dockremap:dockremap -H fd://

Create the `dockremap` user and group

    groupadd dockremap
    useradd -g dockremap dockremap

Then create the subordinate user and group ranges

    echo 'dockremap:100000:65535' >> /etc/subuid
    echo 'dockremap:100000:65535' >> /etc/subgid

Working with Docker
-------------------

Here's [a fantastic cheatsheet](https://github.com/wsargent/docker-cheat-sheet). Here's [a shorter one](https://coderwall.com/p/2es5jw/docker-cheat-sheet-with-examples).

