﻿# Docker on Vagrant

## Why is that needed ?

Docker for Windows and Docker Toolbox are using VirtualBox/Hyper-V between the Windows host and the Linux VM where the Docker Daemon runs and creates local volumes inside the container. It raises several issues such as :

* Low performances;
* Bad and complicated permissions system;
* Lack of symbolic links that leads to issues for apps.

## Solution

* Docker VM (Ubuntu Xenial).
* Provisionning Docker, Docker-Compose and nginx-proxy using Vagrant.
* Winnfsd to share files between the Windows host and the Docker VM.
* Smartcd (Aliases auto (dis)Enabling when browsing the filesystem)

This solution is built from scratch in order to keep agile on the environment.

*Note:  nginx-proxy allows to connect to a webcontainer through `http://monappli.app` instead of `http://192.168.1.100:<port>`*.

## prerequisites

## Pré-requis
- [VirtualBox](https://www.virtualbox.org/) (**/!\\** Virtualisation must be enabled in you BIOS.)
- [Vagrant](https://www.vagrantup.com/)
- [Vagrant-vbguest](https://github.com/dotless-de/vagrant-vbguest) (`vagrant plugin install vagrant-vbguest`)
- [Vagrant-winnfsd](https://github.com/winnfsd/vagrant-winnfsd) (`vagrant plugin install vagrant-winnfsd`)
- [vagrant-disksize](https://github.com/sprotheroe/vagrant-disksize) (`vagrant plugin install vagrant-disksize`)
- [vagrant-proxyconf](https://github.com/tmatilai/vagrant-proxyconf) (`vagrant plugin install vagrant-proxyconf`)
- [Acrylic DNS Proxy](https://sourceforge.net/projects/acrylic) (Optionnal, [Intallation guide on StackOverflow](https://stackoverflow.com/questions/138162/wildcards-in-a-windows-hosts-file#answer-9695861), DNS local proxy to redirect `*.app` to the docker env. Same as for the `/etc/hosts` file but also allows wildcards `*`)

## Installation

- Clone the repository

```bash
git clone https://github.com/GFI-Informatique/docker-devbox
cd vagrant-docker
```

- Run Vagrant:

```bash
vagrant up
```

When running the `ubuntu/xenial` box for the first time, the image is downloaded and then provisionned as defined in the Vagrantfile.
The Vagrantfile is provisionning using the scripts located in the `provision` folder.

Once the VM is provisionned, you can access to it through:

```bash
vagrant ssh
```

The `docker` and `docker-compose` are available there.

## Setup

The VM can be configured by editing the `config.yaml` file. Just use the `config.example.yaml` modify it and save it as `config.yaml`.

## Git configuration (Host and VM)

* Use your first and last name and GFI mail.

```bash
git config --global user.name "Fisrt Last"
git config --global user.email "first.last@gfi.fr"
```

* To avoid flowding the commit history with merge commits.

```bash
git config --global pull.rebase true
```

## Configure carriage return

To avoid problems sharing file between Windows and Linux, it is usefull to do these things :

- Parameter git `core.autocrlf false` option.

```bash
git config --global core.autocrlf false
```

- Use the (LF) carriage return only with your editor. (Linux norm)

## Optionnal conf of Acrylic DNS Proxy

Acrylic DNS Proxy is used to route sets of DN to the VM without having to modify the `/etc/hosts` file.

- Define the DNS server IP to IPv4: `127.0.0.1`, and IPv6: `::1` in the network interface.

- Start Menu > Edit Acrylic Configuration File > Modify the following parameters:

```
PrimaryServerAddress=172.16.78.251
SecondaryServerAddress=10.45.6.3
TertiaryServerAddress=10.45.6.2
```

- Start Menu > Edit Acrylic Hosts File > Add this line at the end of the file:

```
192.168.1.100 *.app
```

## Vagrant cheatsheet

- Run VM

```bash
vagrant up
```

- Stop VM
```bash
vagrant halt
```

- Reboot VM
```bash
vagrant reload
```

- Create VM
```bash
vagrant provision
```

## Synchronisation des fichiers du projet via NFS

A NFS mounting-point can be used through the plugin `vagrant-winnfsd`.

You must edit the `synced_folder` section in the `config.yaml` file as described in the **Setup** section.

```yml
synced_folders:
  projects: # key
    source: "../projects" # absolute or relative path
    target: "/home/ubuntu/projects" # mapped folder
```

Once the `synced_folders` section is filled, Vagrant will automatically launch winfsd to mount specified files using NFS.

To use symbolic links, `winnsfd.exe` must be run with administrator priviledges.

- Open this folder: `%USERPROFILE%\.vagrant.d\gems\2.3.4\gems\vagrant-winnfsd-1.4.0\bin` (**/!\\** Change versions)
- Select `winnfsd.exe` > Right click > Properties
- Go to the "compatibility" tab, check the "Exectute as Administrator" box, then "Apply".

### Free the diskspace

Be aware that `dc down` destroy the containers! Please stop the containers (`dc stop`) to save the volumes before destroying them.

 ```
 docker system prune  --filter "until=24h"
 docker volume rm $(docker volume ls -qf dangling=true)
 ```

In certain cases, the folder `/var/lib/docker` is full of `*-removing` and `*-init` subfolders that can be deleted.

 ```
 # Use with root
 cd /var/lib/docker
 find . -name "*-init" -type d -exec rm -R {} +
 find . -name "*-removing" -type d -exec rm -R {} +
 ``` 

 ### VPN Issues

If the vpnc client can't reach connect, you must check the network interfaces MTU (1500).

See [https://www.virtualbox.org/ticket/13847](https://www.virtualbox.org/ticket/13847)