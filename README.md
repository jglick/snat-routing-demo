[Background question](http://serverfault.com/questions/539710/iptables-combine-snat-with-network-remapping-for-openvpn)

Demo:

    vagrant up --provision
    vagrant ssh vpnserver # or vpnclient
    curl -I http://192.168.21.11/

This should go through the VPN running on ‘vpnserver’, SNAT’d through ‘router’, to ‘service’ and back.

Then the question becomes how to make the same thing work in case `@netb == @neta` rather than being distinct IP ranges.
Probably Vagrant is incapable of simulating this, since it autocreates host-only networks based on the first three quads of a requested IP address.
