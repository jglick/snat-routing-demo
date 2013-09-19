[Background question](http://serverfault.com/questions/539710/iptables-combine-snat-with-network-remapping-for-openvpn)

Demo (tested with Vagrant 1.3.2, VirtualBox 4.2.10):

    vagrant up --provision
    vagrant ssh vpnserver # or vpnclient
    curl -I http://192.168.21.11/

This should go through the VPN running on ‘vpnserver’, SNAT’d through ‘router’, to ‘service’ and back.

(VPN logging goes to `/var/log/syslog`, and connected client list to `/var/run/openvpn.bridge.status`.)

Then the question becomes how to make the same thing work in case `@netb == @neta` rather than being distinct IP ranges.
Probably Vagrant is incapable of simulating this, since it autocreates host-only networks based on the first three quads of a requested IP address.
But it may suffice to just set

    @netc = '192.168.22'

and then try to make ‘router’ convert those packets to `@netb`. If successful,

    curl -I http://192.168.22.11/

should work from ‘vpnserver’ or ‘vpnclient’.
