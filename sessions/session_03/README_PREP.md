# Preparation Material for Session 03

Install VirtualBox and Vagrant on your computer.


### Vagrant & VirtualBox

**Note :** In case you are running MacOS or Windows you might find helpful [installation instructions for your host operating system in the linked guide](https://www.itu.dk/people/ropf/blog/vagrant_install.html). If you're running running a Mac with an ARM processor (M1, M2, M3 as of 2025) then you can't install VirtualBox, but you'll have to install UTM instead of VirtualBox. The instructions below are alternating between how to work with VirtualBox or UTM. Read them attentively and apply what makes sense to your own situation. 



**Note:** Vagrant cannot be run in a VirtualBox VM. This means that you should install Vagrant and related plugins in your host OS (Windows/MacOS) if you are running Linux in a virtual box already. This is because vagrant creates VMs, and creating VMs from a VM can cause problems.

#### Install the Hypervisor (VirtualBox, UTM)

To install VirtualBox you can use sudo: 
```bash
sudo apt install virtualbox virtualbox-ext-pack
```
**Note:** If you are on a Mac with ARM processor you install UTM either via their UI: https://mac.getutm.app/ or with brew: `brew install --cask utm`

#### Install Vagrant 

Since the packaged Vagrant in the linked repositories is in a bit buggy version we install it directly from the package provided by the tool vendor:

```bash
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install vagrant
```

After installing Vagrant, we need to install a VirtualBox plugin that we will need later:
```bash
vagrant plugin install vagrant-vbguest
```
For UTM you install the corresponding plugin 
```
vagrant plugin install vagrant_utm
```
We install a few other useful plugins: 

```bash
vagrant plugin install vagrant-digitalocean
vagrant plugin install vagrant-scp
vagrant plugin install vagrant-reload
```

If you're using VirtualBox save the following into a file called `Vagrantfile` in your current directory :

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = "bento/ubuntu-22.04"

  config.vm.provider "virtualbox" do |vb|
    # Customize the amount of memory on the VM:
    vb.memory = "1024"
  end
end
```
If you're using UTM replace the above `config.fm.box` and `config.fm.provider` lines with a corresponding ones from UTM, i.e.

```ruby
config.vm.box = "utm/bookworm"

#...
config.vm.provider "UTM"
```

Now, run from your current directory (the one in which you saved the `Vagrantfile`):

```bash
vagrant up
```


The command downloads the OS image specified in the configuration and brings up a virtual machine on your computer (with the specified hypervisor as backend).
It will take some time since the corresponding OS image has to be downloaded first. That image will be cached on your disk, i.e., following VM instantiations of the same image will be faster.
Observe that no error message is displayed. If so, and in case you cannot find a solution for it, we will look at it in class.


In case you did not receive any error message, for now, you can run:

```bash
vagrant destroy
```

### Intro to VMs with Vagrant

  > Vagrant is a tool for building and managing virtual machine environments in a single workflow. With an easy-to-use workflow and focus on automation, Vagrant lowers development environment setup time, increases production parity, and makes the "works on my machine" excuse a relic of the past.
  >
  > Vagrant provides easy to configure, reproducible, and portable work environments built on top of industry-standard technology and controlled by a single consistent workflow to help maximize the productivity and flexibility of you and your team.
  >
  > https://www.vagrantup.com

In case you are in doubt about anything in the following quick intro, check the [brief and really good documentation](https://www.vagrantup.com/docs/).

#### Initialize & Start a VM

You can start your setup by creating a `Vagrantfile`. The argument to `vagrant init` is the name of the *box*. That is,

```bash
$ mkdir vm
$ cd vm
$ vagrant init bento/ubuntu-22.04
$ vagrant up
```

for UTM the equivalent init lines that would start an Ubuntu bookworm could be: 

```
vagrant init utm/bookworm
vagrant up --provider=utm

```

  > After running the above commands, you will have a fully functional VM in VirtualBox running Ubuntu. You can SSH into this machine with `vagrant ssh`, and when you are done playing around, you can terminate the virtual machine with `vagrant destroy`.




#### Accessing a VM

You log onto a VM with SSH as in the following.

```bash
$ vagrant ssh
```

You can leave the guest VM with:

```bash
$ vagrant@vagrant: exit
```

#### Teardown

A running VM can be ...

  * put to sleep

  ```bash
  $ vagrant suspend
  ```

  * turned off

  ```bash
  $ vagrant halt
  ```

  * removed completely

  ```bash
  $ vagrant destroy
  ```

#### State of VMs

The following command will display a list of all VMs and their respective states, i.e., `saved`, `poweroff`, `active`, etc.

```bash
$ vagrant global-status
```

#### VMs with other Operating Systems

That is, you can quickly setup development environments for many operating systems. For example, in the following the initialization and use of a FreeBSD and Windows VM.

```bash
$ mkdir vm_freebsd
$ cd vm_freebsd
$ vagrant init freebsd/FreeBSD-12.3-STABLE
$ vagrant up
```

```bash
$ mkdir vm_win
$ cd vm
$ vagrant init senglin/win-10-enterprise-vs2015community
$ vagrant up
```

You can find a catalog of available boxes at https://app.vagrantup.com/boxes/search

<img src="images/vm_catalogue.png" width="80%">


### Vagrant for **local** development setup

At last, apply `vagrant` for a usual scenario: reproduction of development environments.

The branch [`VMify_local`](https://github.com/itu-devops/flask-minitwit-mongodb/tree/VMify_local) of repository https://github.com/itu-devops/flask-minitwit-mongodb contains an _ITU-MiniTwit_ application similar to the one you took over last week. It was refactored to use a MongoDB database instead of an SQLite3 database.

The application is a two-layered application with a frontend server (called `webserver`) and a database server (called `dbserver`).
With the VM setup from the [`VMify_local`](https://github.com/itu-devops/flask-minitwit-mongodb/tree/VMify_local) branch, you have a complete running setup of the entire application that you can recreate on you computer, e.g., for local development.

To do so, clone the repository, switch to the right branch, check and try to develop a feeling for the contents of the [`Vagrantfile`](https://github.com/itu-devops/flask-minitwit-mongodb/blob/VMify_local/Vagrantfile), and start-up the VMs.

```bash
$ git clone https://github.com/itu-devops/flask-minitwit-mongodb.github
$ cd flask-minitwit-mongodb
$ git checkout VMify_local
$ vagrant up
Bringing machine 'dbserver' up with 'virtualbox' provider...
Bringing machine 'webserver' up with 'virtualbox' provider...
...
```

If everything starts error-free, you will find the running _ITU-MiniTwit_ at http://192.168.20.3:5000. With `vagrant ssh webserver` you can access the machine with the frontend code.

