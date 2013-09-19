[Background question](http://serverfault.com/questions/539710/iptables-combine-snat-with-network-remapping-for-openvpn)

Demo (tested with Vagrant 1.3.2, VirtualBox 4.2.10):

    vagrant up --provision
    vagrant ssh vpnserver # or vpnclient
    sudo route del default # cannot manage to make this automatic!
    curl -I http://192.168.22.11/
    tracepath 192.168.22.11

This should go through the VPN running on ‘vpnserver’, SNAT’d and NETMAP’d through ‘router’, to ‘service’ and back.

(VPN logging goes to `/var/log/syslog`, and connected client list to `/var/run/openvpn.bridge.status`.)

When `@netc == @netb` then the `NETMAP` is unnecessary and a simple `SNAT` routes traffic correctly (e.g. `curl -I http://192.168.21.11/`).
The question is whether the same thing can work in case `@netb == @neta` rather than being distinct IP ranges.
Probably Vagrant is incapable of simulating this, since it autocreates host-only networks based on the first three quads of a requested IP address.
But setting `@netc` to a distinct third subnet suffices to demonstrate `SNAT` and `NETMAP` working together.
