# Embedded Dev Box Maker (64-bit)

This are the steps to create a development environment suited for cross-platform embedded development.

## Box specs

- VirtualBox
- Ubuntu GNU/Linux Precise 12.04 Server 64-bit
- 512MB of RAM
- 1 CPU core
- 8MB ram
- 80GB disk space

## Vagrant file
```ruby
    # -*- mode: ruby -*-
    # vi: set ft=ruby :

    VAGRANTFILE_API_VERSION = "2"

    Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
      config.vm.box = "precise64"
      config.vm.network :private_network, :ip => "192.168.50.4"
      config.ssh.forward_agent = true
      config.vbguest.auto_update = true

      config.vm.provider "virtualbox" do |v|
        v.name = "Embedded Dev Box Maker (64-bit)"
        v.memory = 512
      end
    end
```
## Steps to generate box

Download Vagrant VirtualBox box file precise64 in your host. Ubuntu 12.04 64 bits Precise LTS:
    vagrant box add precise64 http://files.vagrantup.com/precise64.box virtualbox

Run Vagrant in this directory or any directory that has the Vagrant listed in this document and let the provisioning run:
    vagrant up

SSH to the VM:
    vagrant ssh

Add the following at the top of /etc/apt/sources.list, it makes apt select the closest mirror:
    deb mirror://mirrors.ubuntu.com/mirrors.txt precise main restricted universe multiverse
    deb mirror://mirrors.ubuntu.com/mirrors.txt precise-updates main restricted universe multiverse
    deb mirror://mirrors.ubuntu.com/mirrors.txt precise-backports main restricted universe multiverse
    deb mirror://mirrors.ubuntu.com/mirrors.txt precise-security main restricted universe multiverse

Configure the correct timezone:
    dpkg-reconfigure tzdata

Update/upgrade all packages:
    sudo apt-get update ; sudo apt-get upgrade

You may need to specifically install your kernel and headers in order to upgrade them:
    sudo apt-get install linux-headers-server linux-image-server linux-server

Disconnect from your ssh session and shutdown your machine in the host:
    vagrant halt

Open VirtualBox and go to the machine (Embedded Dev Box Maker (64-bit)) settings and go to Ports->USB and click on "Enable USB Controller"
Click on "Add a new USB filter" and add the USB filter for your COM/Serial cable. Typical vendors are Prolific or FTDI.

Turn on your VM again and connect to it:
    vagrant up; vagrant ssh

Install the compilers and build tools:
    sudo apt-get install build-essential

Install the nfs server and client tools.
    sudo apt-get install nfs-kernel-server nfs-common portmap

Install the python-software-properties package:
    sudo apt-get install python-software-properties

Add the PPA for the gcc-arm-embedded toolchain:
    sudo add-apt-repository ppa:terry.guo/gcc-arm-embedded ; sudo apt-get update

Install the tools for ARM embedded development:
    sudo apt-get install gcc-arm-none-eabi

Install the tools ARM embedded linux development:
    sudo apt-get install gcc-arm-linux-gnueabi

Install the tools for AVR embedded development:
    sudo apt-get install avrdude avrdude-doc binutils-avr avr-libc gcc-avr gdb-avr

Install multistrap and related cross-architecture package development tools:
    sudo apt-get install multistrap dpkg-dev dpkg-cross

Install qemu and tools for cross-architecture binaries:
    sudo apt-get install qemu qemu-user-static binfmt-support debootstrap

Install tftp support:
    sudo apt-get install xinetd tftpd tftp

Install miscellaneous tools:
    sudo apt-get install git subversion curl wget

Install tools for serial port communications:
    sudo apt-get install screen picocom

Clean apt:
    sudo apt-get clean ; sudo apt-get autoclean ; sudo apt-get autoremove

Download Arduino 1.5.5+ for Linux 64-bits into /usr/local:
    cd /usr/local ; sudo wget http://downloads.arduino.cc/arduino-1.5.5-linux64.tgz
    sudo tar xfz arduino-1.5.5-linux64.tgz
    sudo mv arduino-1.5.5 arduino
    sudo rm arduino-1.5.5-linux64.tgz

Add the following lines to the default user's profile (~/.profile):
    PATH="$PATH:/usr/local/arduino/hardware/tools"
    PS1="[\[\033[0;31m\]\u\[\033[0m\]@\h:\[\033[1;37m\]\w\[\033[0m\]]\$ "

Create tftpd configuration file on /etc/xinetd.d/tftp with the following:
    service tftp
    {
    protocol        = udp
    port            = 69
    socket_type     = dgram
    wait            = yes
    user            = nobody
    server          = /usr/sbin/in.tftpd
    server_args     = /tftpboot
    disable         = no
    }

Configure tftpd:
    sudo mkdir /tftpboot
    sudo chmod -R 777 /tftpboot
    sudo chown -R nobody /tftpboot

Configure the hostname with embedded-dev64:
    sudoedit /etc/hostname

Create a folder for nfs exports in the default user home directory:
    cd ; mkdir exports
    chmod go+w exports

Disconnect from your ssh session and shutdown your machine in the host:
    vagrant halt

Repackage your freshly created embedded development environment into a new box:
    vagrant package --output embedded-dev64.box

Add your new box to vagrant so it can be used in your future endeavors:
    vagrant box add embedded-dev64 embedded-dev64.box virtualbox

Now you can use your embedded development environment under the name 'embedded-dev64'!
