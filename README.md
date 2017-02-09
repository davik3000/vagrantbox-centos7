# Vagrantbox - CentOS7

## Create the base VM in VirtualBox ##

### Install VirtualBox ###
First, download and install the latest version of VirtualBox. _(currently v.4.3.8)_

### Install Vagrant ###
Secondly, download and install the latest version of Vagrant. _(currently v.1.8.6)_

### Prepare the Virtual Machine ###
_The following steps were written for VirtualBox 4.3.8 and may differ for other versions._

- Download the CentOS ISO you prefer. *(suggested type: Minimal)*
- Open VirtualBox and click New.
- Give the virtual machine a Name: `CentOS-7`
- Check and choose `Linux` from the Type dropdown menu.
- Check and choose `Red Hat (64 bit)` from the Version dropdown menu.
- Under Memory size, leave RAM at `512 MB` (Vagrant can change this on-the-fly later).
- Under Hard drive, select Create a virtual hard drive now, and click Create.
- Under File location, leave the default name.
- Under File size, choose what size you prefer. *(suggested size: 100.00 GB)*
- Under Hard drive file type, select `VDI (VirtualBox Disk Image)`.
- Under Storage on physical hard drive, select Dynamically allocated, and click Create.
- The virtual machine definition has now been created. Click the virtual machine name and click Settings.
- Go to the Storage tab, click Empty just under Controller: IDE, then on the right hand side of the window click the CD icon, and select Choose a virtual CD/DVD disk file.
- Navigate to where the ISO was downloaded, select it, and click Open.
- Go to the Audio tab and uncheck Enable Audio.
- Go to the Ports tab, then go to the USB subtab, and uncheck Enable USB Controller.
- Click Ok to close the Settings menu.

Finally, start up the virtual machine to begin installation.

### Install CentOS ###

> Be aware that no where in the following steps we install the VirtualBox Guest Additions.
> They will be installed automatically later.

Install the operating system however you like. Most of the default options can be used.
If the setup ask for root password and initial user, apply this settings:

- root password: `vagrant`
- initial user name: `vagrant` password: `vagrant`

Once the operating system has finished installing and booted, perform the following post-install steps to make it work with Vagrant.

### First boot ###
Boot the guest VM and:

- Install additional repository packages:

      # yum install -y openssh-clients man git vim wget curl ntp sudo

- Fix the boot time delay and apply the changes:

      # sed -ri 's/^GRUB_TIMEOUT.*$/GRUB_TIMEOUT=1/g' /etc/default/grub
      # grub2-mkconfig -o /boot/grub2/grub.cfg

- Create the `.ssh` folder for the user `vagrant`

      # mkdir -m 0700 -p /home/vagrant/.ssh

- Allow user vagrant to use sudo without entering a password:

      # echo "vagrant ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
  - If it cannot be modified, use `visudo` and manually alter it.

- Check and, if present, comment out `requiretty` in `/etc/sudoers`.
This change is important because it allows ssh to send remote commands using sudo.
Without this change vagrant will be unable to apply changes (such as configuring additional NICs) at startup:

      # sed -i 's/^\(Defaults.*requiretty\)/#\1/' /etc/sudoers
  - If it cannot be modified, use `visudo` and manually alter it.

- Apply the insecure key into `~vagrant/.ssh/authorized_keys`
  - Take the insecure key from https://raw.githubusercontent.com/mitchellh/vagrant/master/keys/vagrant.pub

        # curl https://raw.githubusercontent.com/mitchellh/vagrant/master/keys/vagrant.pub >> /home/vagrant/.ssh/authorized_keys

  - If you want to use your own SSH public/private key then create an SSH public/private key on your workstation (you may already have), and copy the public key to `/home/vagrant/.ssh/authorized_keys` on the virtual machine.

- Change permissions on authorized_keys files to be more restrictive:

      # chmod 600 /home/vagrant/.ssh/authorized_keys

- Make sure vagrant user and group owns the .ssh folder and its contents:

      # chown -R vagrant:vagrant /home/vagrant/.ssh

- Then:

  - Clean up yum:

        # yum clean all

  - Clean up the tmp directory:

        # rm -rf /tmp/*

  - Clean up the last logged in users logs:

        # rm -f /var/log/wtmp /var/log/btmp

  - Remove any specific proxy settings:

        # rm /etc/profile.d/proxy.sh

  - Clean up history:

        # history -c

  - Shutdown the virtual machine:

        # shutdown -h now

> *Shortcut:*

>        yum clean all && rm -rf /tmp/* && rm -f /var/log/wtmp /var/log/btmp && rm /etc/profile.d/proxy.sh && history -c && shutdown -h now


### Extract the VM from VirtualBox as Vagrant box ###

- Create a Vagrant box by extracting the current VM. Make sure that the `base` value matches the VM name in VirtualBox. In our case:

      # vagrant package --output centos-7.box --base CentOS-7

### Add the box to your local Vagrant Boxes repository ###

- Add the newly created Vagrant box to your local Vagrant Boxes repository (this will copy the Vagrant box to another location on your machine). Replace `<box name>` with a custom name.

      # vagrant box add --name <box name> centos-7.box

### Install your Vagrant box ###

- Create a new folder, that it will contain our new Vagrant machine definition.
  - Tip: put all our Vagrant definitions in one place, usually under `/VagrantVMs` in Linux, under `C:\VagrantVMs` in Windows.

- Execute:

      # vagrant init

- Edit the `Vagrantfile` and change the value of `config.vm.box` with the previously chosen one.

- Execute:

      # vagrant up
