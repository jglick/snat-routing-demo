# -*- mode: ruby; coding: utf-8 -*-

# tested with Vagrant 1.3.2, VirtualBox 4.2.10

@neta = '192.168.20' # first real subnet, with VPN server and undistinguished client
@netb = '192.168.21' # second real subnet, with router machine (VPN client offering an iroute) and a service
@netc = '192.168.21' # the subnet exposed as a route via VPN
@route = "#{@netc}.0 255.255.255.0"
@netvpn = '192.168.30' # the interfaces created for tun0
@host = '10.0.2.2' # VirtualBox host, which we use to port-forward just the VPN
@host_vpn_port = 11940 # a port on VBox host to use for the VPN
@keys = '/usr/share/doc/openvpn/examples/sample-keys'

Vagrant.configure("2") do |config|

  config.vm.box     = "precise32"
  config.vm.box_url = "http://files.vagrantup.com/precise32.box"

  # Persist the apt cache to avoid hitting the servers too often.
  # Cannot share all of /var/cache/apt since pkgcache.bin is mmap'd and VBox shared folders do not like this. (NFS share is too onerous.)
  # apt also seems to require the partial/ subdir to be precreated.
  # TODO check cacher-ng recipe in apt cookbook
  require 'fileutils'
  FileUtils.mkdir_p 'apt-cache/archives/partial'
  config.vm.synced_folder "apt-cache/archives", "/var/cache/apt/archives"

  config.vm.provision :shell, inline: 'apt-get -y install jed'

  config.vm.define :vpnserver do |config|
    config.vm.host_name = 'vpnserver'
    config.vm.network :private_network, ip: "#{@neta}.10"
    config.vm.network :forwarded_port, guest: 1194, host: @host_vpn_port
    config.vm.provision :shell, inline: <<SCRIPT_VPNSERVER
# during reprovisioning temporarily undoes route del default, so that machines cannot cheat and talk to one another across networks via host:
route add -net default gw #{@host} dev eth0 2>&- || :
apt-get -y install openvpn curl
route del default
mkdir -p /etc/openvpn
rm -f /tmp/client-connected-flag
cat > /etc/openvpn/client-connect.sh <<'CLIENT_CONNECT'
#!/bin/sh
# TODO stateful hack—assumes that vpnserver, vpnclient, and router are all (re-)provisioned in that order
# better to replace client-connect with client-config-dir, and either create actual distinct certificates for vpnclient vs. router (such as using askpass FILE and pass.crt/pass.key), or use username/password authentication if possible
if [ -f /tmp/client-connected-flag ]
then
    echo iroute #{@route} >> $1
else
    touch /tmp/client-connected-flag
fi
CLIENT_CONNECT
chmod a+x /etc/openvpn/client-connect.sh
cat > /etc/openvpn/bridge.conf <<CONF_VPNSERVER
server #{@netvpn}.0 255.255.255.0
proto tcp
dev tun
client-to-client
route #{@route}
push "route #{@route}"
script-security 2
client-connect /etc/openvpn/client-connect.sh
ca #{@keys}/ca.crt
cert #{@keys}/server.crt
key #{@keys}/server.key
dh #{@keys}/dh1024.pem
duplicate-cn
verb 3
CONF_VPNSERVER
service openvpn restart
SCRIPT_VPNSERVER
# logging goes to /var/log/syslog
  end

  config.vm.define :vpnclient do |config|
    config.vm.host_name = 'vpnclient'
    config.vm.network :private_network, ip: "#{@neta}.11"
    config.vm.provision :shell, inline: <<SCRIPT_VPNCLIENT
route add -net default gw #{@host} dev eth0 2>&- || :
apt-get -y install openvpn curl
route del default
mkdir -p /etc/openvpn
cat > /etc/openvpn/bridge.conf <<CONF_VPNCLIENT
client
dev tun
remote #{@host} #{@host_vpn_port} tcp
ns-cert-type server
ca #{@keys}/ca.crt
cert #{@keys}/client.crt
key #{@keys}/client.key
verb 3
CONF_VPNCLIENT
service openvpn restart
SCRIPT_VPNCLIENT
  end

  config.vm.define :router do |config|
    config.vm.host_name = 'router'
    config.vm.network :private_network, ip: "#{@netb}.10"
    config.vm.provision :shell, inline: <<SCRIPT_ROUTER
route add -net default gw #{@host} dev eth0 2>&- || :
apt-get -y install openvpn
route del default
mkdir -p /etc/openvpn
cat > /etc/openvpn/bridge.conf <<CONF_ROUTER
client
dev tun
remote #{@host} #{@host_vpn_port} tcp
ns-cert-type server
ca #{@keys}/ca.crt
cert #{@keys}/client.crt
key #{@keys}/client.key
verb 3
CONF_ROUTER
service openvpn restart
sysctl -w net.ipv4.ip_forward=1
iptables -t nat -F
iptables -t nat -A POSTROUTING -s #{@netvpn}.0/24 -j SNAT --to-source #{@netb}.10
SCRIPT_ROUTER
  end

  config.vm.define :service do |config|
    config.vm.host_name = 'service'
    config.vm.network :private_network, ip: "#{@netb}.11"
    config.vm.provision :shell, inline: <<SCRIPT_SERVICE
route add -net default gw #{@host} dev eth0 2>&- || :
apt-get -y install lighttpd
route del default
SCRIPT_SERVICE
  end

end
