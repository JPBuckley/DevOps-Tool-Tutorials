# What is Vagrant?
Vagrant is a tool that enables users to create and configure lightweight, reproducible, and portable development environments. It provisions machines on top of VirtualBox, VMware, AWS, or any other provider. Vagrant can leverage industry-standard provisioning tools such as Chef, Puppet, or standard shell scripts to automatically install and configure software on the virtual machine. For developers, Vagrant will isolate dependencies and their configuration within a single disposable, consistent environment, eliminating the "works on my machine" bugs.


## Basic Commands

When using Vagrant, you will need to be located in the directory of the that the ‘Vagrantfile’ is located in for that specific instance.

|Command/Syntax|Explanation|
|---|---|
|`vagrant -v`|Displays the installed version of Vagrant.|
|`vagrant init [OS_IMAGE] [URL]`|Creates the initial ‘Vagrantfile’ in the current working directory, and includes the provided OS_IMAGE and the URL as the defined OS of the VM and location to download the OS image.|
|`vagrant up`|Uses the information in the ‘Vagrantfile’ to create and power on the VM. If necessary, the OS image is downloaded at this time.|
|`vagrant ssh [BOX_NAME]`|Creates an ssh session from the local machine into the Vagrant VM (you must be in the working directory of that ‘Vagrantfile’). <br><br> If there are multiple servers deployed/managed from this project’s ‘Vagrantfile’ the name of the box you wish to ssh into must be declared where it says BOX_NAME. <br><br> If it is ever needed the default password for the vagrant user is “vagrant”.|
|`vagrant destroy`|Destroys the VM that Vagrant created as part of this project.|
|<code>vagrant box [list&#124;add&#124;remove]</code>|This will list, add, or remove Vagrant box images or server templates form your local machine.|
|`vagrant box repackage`|This will repackage or recreate your VM into a new Vagrant box image (server template) on your local machine, prior to any configuration tasks being performed.|
|`vagrant status`|Displays the status of your Vagrant VM, running, powered off, or suspended. Also, provides other options for managing the VM.|
|`vagrant reload`|FILL ME IN!!!|
|`vagrant suspend`|The suspend option essentially pauses or freezes a VM, allowing you to cleanup & resume quickly at that exact point later in time. Disk space is actively being consumed, but CPU & RAM are not. <br><br> Resume by using the `vagrant up` command.|
|`vagrant halt`|Cleanly shuts down the VM.|
|`vagrant halt --force`|Forces a shutdown of the VM, without a clean shutdown or exit of the VM.|
|`vagrant package [VM_NAME]`|FILL ME IN!!!|


# But How Does It Work?
Vagrant is configured on a per project basis. Each project has its own configuration file which is called its "Vagrantfile," which also happens to be what the file is named as well. The ‘Vagrantfile’ is a text file that lays out all the information about how the environment is to be setup. For example, the "Vagrantfile" specifies the Operating System, where to find a copy of the OS, how much RAM and CPU, what software to install, and so on.

It is important to remember that each "Vagrantfile" should be stored with its own project folder, and that each project can only have one "Vagrantfile".


# Vagrant Configuration Components
## Synced Folders
Folders can be synced or shared between host machines and the guest VM. Synced folders are enabled by default. The default configuration is to sync the contents of the folder that contains our ‘Vagrantfile’, typically this is also our root Project directory. These files are stored on the local machine, which means that they are not destroyed with the rest of the guest VM when we run the `vagrant destroy` command. With these files stored locally on the system, we are able to use our editor of choice. The definition or mapping for the synced folder are defined in the ‘Vagrantfile’. On the guest system, the contents of the synced folder is located inside the "/vagrant" directory. If this needs to be changed, edit the ‘Vagrantfile’ and update the line:

`config.vm.synced_folder [src/],[/new/location] end`

In this line, `src/` is the name of the synced folder on the local machine, and `/new/location` being the location for those files to be located on the guest OS.

Synced or shared folders are enabled by default. If necessary, they can be disabled, though it is probably easier to simply not use them. By default, the synced folders on the guest VM are owned by the "root" user and group.

## Networking
Vagrant automatically sets up the networking for the guest VM. One of these configurations is port forwarding between the guest VM and the local machine (this would not be done if the virtualization target is in a datacenter, on another remote computer, or a cloud provider). Just like with Synced folders, this is configured in the ‘Vagrantfile’. A common use case for this is for testing web server. The configuration line in the ‘Vagrantfile’ would look like:

`config.vm.network "forwarded_port", guest: 80, host: 8080`

Another example that can provide the port forwarding configuration would be:

`config.vm.network :forwarded_port, guest: 80, host: 8080, host_ip: "127.0.0.1"`

What this being defined is the network mappings between the guest VM’s port 80 and the local machine’s port 8080 which would read in a web browser as “http://localhost:8080”.

By default, port forwarding is enabled and setup for the ssh service between the guest VM’s port of 22 to the localhost’s port 2222.

## Environment Management
Manage the VM’s state within your Vagrant environment.

|Command/Syntax|Explanation|
|---|---|
|`vagrant suspend`|The suspend option essentially pauses or freezes a VM, allowing you to cleanup & resume quickly at that exact point later in time. Disk space is actively being consumed, but CPU & RAM are not. <br><br> Resume by using the `vagrant up` command.|
|`vagrant halt`|Cleanly shuts down the VM.|
|`vagrant halt --force`|Forces a shutdown of the VM, without a clean shutdown or exit of the VM.|
|`vagrant destroy`|Destroys the VM that Vagrant created as part of this project.|


## Automated Provisioning
### Provisioners
We have the option to install software on our VM’s manually, or during the provisioning process by using either shell scripts or configuration management systems, such as Chef or Puppet.

### Provisioning with Shell Scripts
When using a shell script to configure your VM, you will write a standard shell script that will contain all of the same commands as you would have entered to configure the VM manually. Be sure to also include the shell environment of the script at the beginning of the script. For example the first line being:

`#!/bin/bash`

OR

`#!/usr/bin/env bash`

You also want to add descriptions of what is being done in the script. For example:

`echo "Installing Apache and setting Apache up… please wait"`

Keep in mind that you will not need to run any of these commands with sudo because Vagrant is running the script as root when the `vagrant up` command is executed.

Vagrant will need to know about the provisioning script so that it can be leveraged when Vagrant creates the VM. To do this we need to edit the ‘Vagrantfile’ and add the line:

`config.vm.provision "shell", path: "provision.sh"`

Where it says "shell" we are telling Vagrant that we are using a shell script to configure the VM. The location of the script is defined by the path option. The path will begin in the root of the Synced Folder.

### Provisioning with Chef
We need to tell Vagrant to use Chef to configure our VM when we run the `vagrant up` command. We do this by editing the ‘Vagrantfile’ and add the line:

```
config.vm.provision "chef_solo" do |chef|
	chef.add_recipe "RECIPE_NAME" ```

Where it says `"chef_solo"` we are telling Vagrant that we are using Chef Solo to configure the VM, opposed to using Chef Client. Chef Solo is a local install of Chef Cookbooks, where Chef Client requires the downloading of Cookbooks from a central repository. The `"RECIPE_NAME"` item is the name of the recipe that we wish to use to configure our VM.

Chef and Vagrant require a specific folder structure for the Chef cookbooks to be located in for them to be found and used during provisioning. This is the default cookbook directory for the Chef recipes. This structure should begin at the root of the project directory tree, and will be:

`cookbooks/RECIPE_NAME/recipes/default.rb`

The `default.rb` file contains the configuration parameters for Chef to configure our VM as desired.

### Provisioning with Puppet
We need to tell Vagrant to use Puppet to configure our VM when we run the `vagrant up` command. We do this by editing the ‘Vagrantfile’ and add the line:

```
config.vm.provision "puppet" do |puppet|
	puppet.manifests_path = "manifests"
	puppet.manifest_file = "default.pp" ```

Where it says `"puppet"` we are telling Vagrant that we are using Puppet to configure the VM. The location of our Puppet Manifest files are defined by the line that reads, `puppet.manifests_path = `. Our default manifest file is defined with the line `puppet.manifests_path = `. The `RECIPE_NAME` item is the name of the recipe that we wish to use to configure our VM.

Vagrant is aware of the file and path of the default Puppet manifest files because they have been defined in the ‘Vagrantfile’. The default location for the Puppet manifest files are located at the root of the project’s directory tree, and will look something like this:
`manifests/default.pp`

The `default.pp` file contains the configuration parameters for Puppet to configure our VM as desired.


## Vagrant Networking
### Private Networking
Configuring a private network refers to a network and/or IP address range that is not publicly accessible form the Internet, in other words, this would be an Intranet network. To do this we need to edit the ‘Vagrantfile’ and either add or edit the `config.vm.network` line for either a static or DHCP address like below:

For DCHP Addressing:
`config.vm.network "private_network", type: "dhcp"`

For Static IP Addressing:
`config.vm.network "private_network", ip: "192.168.1.123"`

### Public Networking
Configuring a public network refers to a network and/or IP address range that is publicly accessible from the Internet. To do this we need to edit the 'Vagrantfile' and either add or edit the `config.vm.network` line for either a static or DHCP address like below:

For DCHP Addressing:
`config.vm.network "public_network", type: "dhcp"`

For Static IP Addressing:
`config.vm.network "public_network", ip: "8.8.8.8"`

For Static IP Addressing for a Specific Interface:
`config.vm.network "public_network", ip: "8.8.8.8", bridge: 'ifcfg-eth0'`


## Managing Multiple Machines with Vagrant
Sometimes we need to setup and test systems that are that require multiple servers, for example an Application server, a Web server, and a Database server. In multi-server configuration situations like this we need to tell Vagrant to setup these multiple systems and to configure each one appropriately.

In order to setup our multiple servers, we will need to define them in our ‘Vagrantfile’:

```
	Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
		config.vm.box = "precise64"
		config.vm.box_url = "http://vagrant.url.com/precise64.box"
			# Setup for Web Server
		 	config.vm.define "web" do |web|
				web.vm.hostname = "web"
				web.vm.box = "apache"
				web.vm.network "private_network", type: "dhcp"
				web.vm.network "forwarded_port", guest: 80, host: 8080
				web.vm.provision "puppet" do |puppet|
					puppet.manifests_path = "manifests"
					puppet.manifest_file = "default.pp"
					end
			end
			# Setup for App Server
			config.vm.define "app" do |app|
				app.vm.hostname = "app"
				app.vm.box = "apache"
				app.vm.network "private_network", type: "dhcp"
				end
			end
			# Setup for MYSQL DB Server
			config.vm.define "db" do |db|
				web.vm.hostname = "db"
				web.vm.box = "mysql"
				web.vm.network "private_network", type: "dhcp"
				end
			end```


## Vagrant Boxes
When it comes to managing the Vagrant boxes, Vagrant manages them on a global level opposed to a per-project level like the 'Vagrantfile' does. Remember that Vagrant uses the box images as a basic server template, and then uses the configuration specifications defined in the 'Vagrantfile' to provision and configure the VM. For example, if "Project1" were to download a CentOS 7.2 box image and configure it based on its 'Vagrantfile'; that box image would be available for use by any other project on that local machine to use to provision and configure a different VM based on the specifications inside of its own 'Vagrantfile'.  Once a box image has been deleted or removed from a local machine, it will need to be redownloaded the next time it is needed by a project.


# Other Vagrant References
For more in depth information about Vagrant and what it can do, be sure to check out the [Vagrant Documentation](https://www.vagrantup.com/intro/getting-started/index.html) on their website.
