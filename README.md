# Vagrantbox - CentOS7

## Create the base VM in VirtualBox ##

### Install VirtualBox ###
First, download and install the latest version of VirtualBox.

### Install Vagrant ###
Second, download and install the latest version of Vagrant.

### Prepare the Virtual Machine ###
The following steps were written for VirtualBox 4.3.8 and may differ for other versions.

- Download the CentOS ISO you prefer. *(suggested type: Minimal)*
- Open VirtualBox and click New.
- Give the virtual machine a Name: `CentOS-7`
- Check and choose `Linux` from the Type dropdown menu.
- Check and choose `Red Hat (64 bit)` from the Version dropdown menu.
- Under Memory size, leave RAM at 512 MB (Vagrant can change this on-the-fly later).
- Under Hard drive, select Create a virtual hard drive now, and click Create.
- Under File location, leave the default name.
- Under File size, choose what size you prefer. *(suggested size: 100.00 GB)*
- Under Hard drive file type, select VDI (VirtualBox Disk Image).
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
Boot the VM and:

- Install additional repository packages:

      yum install -y openssh-clients man git vim wget curl ntp

- Fix the boot time delay and apply the changes:

      sed -ri 's/^GRUB_TIMEOUT.*$/GRUB_TIMEOUT=1/g' /etc/default/grub
      grub2-mkconfig -o /boot/grub2/grub.cfg

- Create the `.ssh` folder for the user `vagrant`

      mkdir -m 0700 -p /home/vagrant/.ssh

- Allow user vagrant to use sudo without entering a password:

      echo "vagrant ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

- Check and, if present, comment out `requiretty` in `/etc/sudoers`.
This change is important because it allows ssh to send remote commands using sudo.
Without this change vagrant will be unable to apply changes (such as configuring additional NICs) at startup:

      sed -i 's/^\(Defaults.*requiretty\)/#\1/' /etc/sudoers


- Apply the insecure key into `~vagrant/.ssh/authorized_keys`
  - Take the insecure key from https://raw.githubusercontent.com/mitchellh/vagrant/master/keys/vagrant.pub

        curl https://raw.githubusercontent.com/mitchellh/vagrant/master/keys/vagrant.pub >> /home/vagrant/.ssh/authorized_keys

  - If you want to use your own SSH public/private key then create an SSH public/private key on your workstation (you may already have), and copy the public key to `/home/vagrant/.ssh/authorized_keys` on the virtual machine.

- Change permissions on authorized_keys files to be more restrictive:

      chmod 600 /home/vagrant/.ssh/authorized_keys

- Make sure vagrant user and group owns the .ssh folder and its contents:

      chown -R vagrant:vagrant /home/vagrant/.ssh

- Then, inside the VM:

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

  - *inline command:*

        yum clean all && rm -rf /tmp/* && rm -f /var/log/wtmp /var/log/btmp && rm /etc/profile.d/proxy.sh && history -c && shutdown -h now


### Extract the VM box from VirtualBox as Vagrant box ###

- Make sure the value of the base command line switch matches the name of the virtual machine in VirtualBox:

      # vagrant package --output centos-7.box --base CentOS-7

- Add the newly created Vagrant Box to vagrant (this will copy the Vagrant Box to another location):

      # vagrant box add --name jrc/CentOS-7 centos-7.box

### Install your Vagrant box ###

- Execute:

      # vagrant up

- If you have already installed the "vagrant-vbguest" plugin, you will get an error during the installation of the guest addition.
This will happen due to missing proxy settings.
With the VM up and running, you need to force `provision`, then restart the VM with `reload`. Now it will successfully install the guest addition.
Finally, restart again to `reload` all your settings and the configured shared folders.

      # vagrant provision
      # vagrant reload
      # vagrant reload

