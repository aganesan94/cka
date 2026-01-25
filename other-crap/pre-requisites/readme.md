# Install Vagrant

- Vagrant is a simple and powerful VM software. 
- It allows you to create VMs
- Consider a scenario where you want to use a specific set of softwares for development purposes and be able to port it to different computers. Vagrant makes it easy to do so.
- Vagrant allows to created isolated environments.
- Vagrant needs a virtual provider. Options are to include Oracle Virtual Box or VMWare Workstation Pro

# Pre-requisites

* Install vagrant
* Install Oracle Virtual Box
* Restart the device.

# Post Installation

Check if vagrant is installed

```
vagrant -v
```

```
config.vm.box - Refers to the OS
config.vm.provider - vm provider (virtualbox, vmware)
config.vm.network - how the host is going to network with the box
config.vm.synced_folder - Mount host folder on the vm
config.vm.provision - what we want to set up
```

## Initializing a vagrant folder

Navigate to a folder where  we need to set up the box

References: https://www.vagrantup.com/docs/cli/reload

```
vagrant init <vagrant-box-name>
```

This generates a vagrant file

```vb
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "ubuntu/focal64"

  # Network provider
  
  # uncomment this line if you want to access an app via localhost
  # config.vm.network "forwarded_port", guest: 80, host: 8888
  
  
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"
  
  # uncomment this line if you want to access via private IP address
  # alternatively a hostname can be modified in the following folders
  # Windows 10 - "C:\Windows\System32\drivers\etc\hosts"
  # Linux - "/etc/hosts"
  # Mac OS X - "/private/etc/hosts"
  # <ip-addr> <url-name> <www.url-name>

  # config.vm.network "private_network", ip: "192.168.33.10"
  # config.vm.network "public_network"

  # Shared folder
  # Use multiple synced folders to ensure that this is working.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    #vb.gui = true
  
    # Customize the amount of memory on the VM:
    #vb.memory = "1024"

    #vb.cpu = 1
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Uncommenting this out makes sure that the following commands run  
  # immediately after the vagrant provisions the shell.
  # you may also include all the necessary apps in a shell script and 
  # then execute the shell script here.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
end

```

* To get the status of the VM

```
vagrant status
```


* To get the vagrant environment up and running.
```
vagrant up
```

* To suspend a vagrant environment
```
vagrant suspend
```

* To resume a vagrant environment which is suspended
```
vagrant resume
```

* Reload the VM configurations
```
vagrant reload
```

* Restart a vagrant environment
```
vagrant restart
```

* To destroy a vagrant environment
```
vagrant destroy
```

* SSH into the box

```
vagrant ssh
vagrant ssh <provider-name>
```