# About anyconnect-vagrant
This project allows Cisco Anyconnect and Traps/Cortex to run in a VM using Vagrant.

**Please note: you need to stay logged in into the VM as long as you want to use the VPN.
On logout the VPN connection is automatically disconnected!** 

Currently, all DNS traffic is sent through the VPN.

The usual steps you would take are the following:

1. launch the VM, log in **and stay logged in**
1. connect and disconnect the VPN as needed
1. halt the VM

## Configuration

Steps:

* Obtain the AnyConnect linux installer program. Place in the file 'packages/anyconnect.tar.gz'
* Obtain the "Traps/Cortex" debian linux installer. Place in the file 'packages/cortex.deb' directory. 
* The routes which forwarded through the VPN are currently hardcoded in `forward.sh`
* Create a file `vpnconfig` which contains three lines:
  1. VPN hostname
  1. number of VPN group
  1. username
  1. password
  
Example for `vpnconfig`:

    vpn.company.com
    1
    joe
    joes-secret-password
    

## usage

### create some alias to simplify usage 

    $ cat ~/.bashrc | grep -i vpn
    #VPN
    VPNDIR="~/git/anyconnect-vagrant"
    alias vpn-vm-up="$VPNDIR/vm-login.sh"
    alias vpn-up="$VPNDIR/vpn-up.sh && $VPNDIR/vpn-poll-state.sh"
    alias vpn-vm-suspend="$VPNDIR/vm-suspend.sh"

Then you can simply

    vpn-vm-up
   
and in another shell

    vpn-up

and when you're done

    alias vpn-vm-suspend

### The following describes what happens in detail or how to call the scripts one by one.

#### launch the VM, log in and stay logged in

`./vm-login.sh`

* creates and provisions the VM (only on the first run)
* starts the VM if not running
* log in

#### connect and disconnect as needed

**The following scripts are meant to be executed on the host, not on the guest VM.**
This is due to the fact that we have to change state locally (forwarding) and on the VM (VPN). 

##### connect to VPN

`./vpn-up.sh`

* establishes the VPN connection
* forwards traffic through the VPN (will ask for root password to do this)

If you get `bash: /opt/cisco/anyconnect/bin/vpn: No such file or directory` when you first try this, 
the VM was not properly provisioned. Either try `/vagrant/install.sh` from within the VM, or just delete the VM 
and start again.

##### continuously check VPN state

`./vpn-poll-state.sh`

* shows status of
  * VPN
  * forwarding

This polls the state every 60 seconds and terminates `sshuttle` (and thereby the DNS forwarding) if VPN connection is down.
  
If you get `sshuttle is not running starting sshuttle` every time then try modifying the `forward.sh` to replace the 
`--daemon` with an `&`. On MacOS the daemon tag simply made `sshuttle` exit silently on startup.

##### check VPN state

`./vpn-state.sh`

* shows status of
  * VPN
  * forwarding

##### disconnect from VPN

`./vpn-down.sh`

* stop forwarding of the traffic
* disconnects from the VPN

#### suspend the VM

`./vm-suspend.sh`

* stop forwarding of the traffic
* disconnects from the VPN
* suspend the VM

This is usually a bit faster than halting the VM, also during next start of the VM.

#### halt the VM

`./vm-halt.sh`

* stop forwarding of the traffic
* disconnects from the VPN
* halts the VM

## Troubleshooting

If things go wrong try this:

### `connect.sh`

This script has to be executed on the VM itself. Therefore:

1. `./vm-login.sh`: log in into the VM
1. `/vagrant/connect.sh up`

This establishes the VPN connection. But no traffic is forwarded yet to the VPN.

### `forward.sh`

This script has to be executed locally on the host.

`./forward.sh`

Forward traffic from the host to the VPN. 

## Dependencies

### Vagrant

The VM is created and configured via Vagrant. Download it here: [Downloads | Vagrant by HashiCorp](https://www.vagrantup.com/downloads.html)

### VirtualBox

The VM runs in VirtualBox. Download it here:

* [Downloads – Oracle VM VirtualBox](https://www.virtualbox.org/wiki/Downloads)
* [VirtualBox 6.1.10 Oracle VM VirtualBox Extension Pack](https://www.virtualbox.org/wiki/Downloads#VirtualBox6.1.10OracleVMVirtualBoxExtensionPack)

### sshuttle

* [sshuttle/sshuttle: Transparent proxy server that works as a poor man's VPN. Forwards over ssh. Doesn't require admin. Works with Linux and MacOS. Supports DNS tunneling.](https://github.com/sshuttle/sshuttle)

**Ubuntu**: you need to compile your own up-to-date version of sshuttle and specify the path to that binary 
in `forward.sh`

### Python

sshuttle requires Python >= 3.5 

# Disclaimer

This project is for informational use only. Do not use it to bypass your company procedures or security policies. 
Use at your own risk. I can offer no support for this project. 
