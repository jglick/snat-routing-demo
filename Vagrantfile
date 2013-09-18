# -*- mode: ruby; coding: utf-8 -*-

# tested with Vagrant 1.3.2, VirtualBox 4.2.10

@neta = '192.168.20'
@netb = '192.168.21'
@route = "#{@netb}.0 255.255.255.0"
@netvpn = '192.168.30'
@host = '10.0.2.2'
@keys = '/usr/share/doc/openvpn/examples/sample-keys'
@host_vpn_port = 11940

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
apt-get -y install openvpn
mkdir -p /etc/openvpn
cat > /etc/openvpn/client-connect.sh <<'CLIENT_CONNECT'
#!/bin/sh
echo iroute #{@route} >> $1
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
apt-get -y install openvpn
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
apt-get -y install openvpn
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
apt-get -y install lighttpd
SCRIPT_SERVICE
  end

end
