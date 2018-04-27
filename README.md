# openvpn-server-vagrant

Spin up an OpenVPN Server


## Install Vagrant, VirtualBox and git

    http://www.vagrantup.com
    https://www.virtualbox.org (don't worry about setting up any VMs as the steps below will cover this)
    http://git-scm.com


## Set up

    Edit /etc/hosts locally and add `192.168.50.11 vpn.dev`
    $ git clone https://github.com/redgeoff/openvpn-server-vagrant.git
    $ cd openvpn-server-vagrant
    $ cp config-default.sh config.sh
    Edit config.sh and fill in your config
    $ vagrant up
    $ vagrant ssh


# Add a route to a subnet

Routes must be added to the server so that you clients know which traffic to route to the VPN Server. The following process should be repeated for each subnet in your network.

Edit `/etc/openvpn/server.conf` and add something like the following, where `172.31.26.0` is your network and 255.255.255.0 is the netmask.

    push "route 172.31.26.0 255.255.255.0"

Then restart the VPN Server:

    $ sudo systemctl restart openvpn@server


## Add a client

The following should be repeated for each new client/user for whom you wish to grant access to your VPN. Replace client-name with a unique name.

    $ sudo su -
    $ /vagrant/add-config.sh client-name

You will then find a file like the following that you should provide to the individual who will be connecting to your VPN. This ovpn file can then be used with Tunnelblick (OS X), OpenVPN (Linux, iOS, Android and Windows).

    ~/client-configs/files/client-name.ovpn


## Revoke client certificate

If you ever need to revoke access, simply execute:

    $ sudo su -
    $ /vagrant/revoke-full.sh client-name


## Extra Info

* See [Using a VPN Server to Connect to Your AWS VPC for Just the Cost of an EC2 Nano Instance](https://medium.com/@redgeoff/using-a-vpn-server-to-connect-to-your-aws-vpc-for-just-the-cost-of-an-ec2-nano-instance-3c81269c71c2)
* See [How To Set Up an OpenVPN Server on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-openvpn-server-on-ubuntu-16-04)

## Port Forwarding to Client

Enable port forwarding on the server:
    
    $ sysctl -w net.ipv4.ip_forward=1 

Setup port forwarding rules:
    
    $ sudo iptables -t nat -A PREROUTING -d SERVER_IP_ADDR -p tcp --dport 80 -j DNAT --to-dest CLIENT_IP_ADDR:SERVICE_PORT
    
Check NAT IPTABLES:

    $ sudo iptables -t nat -L
    
Flush NAT IPTables:

    $ sudo iptables -t nat -F
