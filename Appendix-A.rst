.. highlight:: none

Appendix - A Cheat Sheet
========================

**Setting up the Deployer Node**

-  Install Ununtu 14.04LTS or 16.04LTS to the deployer node. Insure SSH
   login is enabled.
-  Install the vim, vlan and bridge-utils packages::

    $ apt-get update
    $ apt-get install vim vlan bridge-utils
	

**Installing the OpenPOWER Cluster Genesis Software**::

    $ export GIT_SSL_NO_VERIFY=1
	
-  Change the root password to passw0rd::

    $ sudo passwd root    (set to passw0rd)
    
-  If the root account does not exist (is not unlocked)::

    $ sudo passwd root (then enter passw0rd twice)
    $ sudo -u root  (to unlock the root passwd) (to lock again after running genesis: sudo passwd -l root)

-  Install git::

    $ sudo apt-get install git
    $ git clone https://github.com/open-power-ref-design/cluster-genesis

**Running the OpenPOWER Cluster Genesis Software**::

    $ cd cluster-genesis
    $ ./scripts/install.sh   (this will take a few minutes to complete)
    $ source scripts/setup-env   (if you restart your shell session, you need to re-execute this and the next line.
    $ export ANSIBLE_HOST_KEY_CHECKING=False

-  copy your config.yml file to the /cluster-genesis directory
-  create the Genesis container::

    $ cd playbooks
    $ ansible-playbook -i hosts lxc-create.yml -K (create container. Verify container networks)

To begin cluster genesis::
	
    $ ansible-playbook -i hosts install.yml -K (begins cluster genesis)
    Allow several minutes to run.
   

After the command prompt returns, run the following to see the status/progress of
operating system load for each cluster node::

    sudo cobbler status (from within container at /home/deployer/cluster-genesis)

**Configuring networking on the cluster nodes**::

    $ ansible-playbook -i ../scripts/python/cluster-genesis/inventory.py gather_mac_addresses.yml -u root --private-key=~/.ssh/id\_rsa\_ansible-generated
    $ ansible-playbook -i ../scripts/python/cluster-genesis/inventory.py configure\_operating\_systems.yml -u root --private-key=~/.ssh/id\_rsa\_ansible-generated

**Accessing the deployment container**

-  To see a list of containers on the deployer::

    $ sudo lxc-ls

-  To access the container as root::

    $ sudo lxc-attach -n yourcontainername

alternately, you can ssh into the container;

To get the login information::

    $ grep "^deployer" ~/cluster-genesis/playbooks/hosts

    deployer ansible_user=deployer ansible_ssh_private_key_file=/home/ubuntu/.ssh/id_rsa_ansible-generated ansible_host=192.168.0.2

Logging into the container as user "deployer"::

    $ ssh -i ~/.ssh/id\_rsa\_ansible-generated deployer@192.168.0.2

Notes:

-  if you change the ip address of the container, (ie if you recreate
   the container) you may need to replace the cached ECDSA host key in
   the .ssh/known\_hosts file::

    $ ssh-keygen -R container-ip-address

-  if you reboot the deployer node you need to restart the deployment
   container::

    $ lxc-start -d -n <container name>

Checking the Genesis Log

Genesis writes status and error messages to;
/home/deployer/cluster-genesis/log.txt

You can display this file::

    $ cat /home/deployer/cluster-genesis/log.txt

**Checking the DHCP lease table**

From within the container::

    $ cat /var/lib/misc/dnsmasq.leases

**Logging into the cluster nodes**

from the deployer node (host namespace)::

    $ ssh -i ~/.ssh/id_rsa_ansible-generated userid-default@a.b.c.d

or as root::

    $ ssh -i ~/.ssh/id_rsa_ansible-generated root@a.b.c.d #(as root -i not needed from cluster nodes)

with password; from deployer or cluster node::

    $ ssh userid-default@a.b.c.d # password: password-default (from config.yml)

