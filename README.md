# SimpleWall
## A simple private home firewall based on nftables
SimpleWall is a set of two nftables scripts for setting up a basic firewall. \"Basic\"
means that it is intended for private home use with a manageable amount of interfaces and
firewall rules. The scripts were developed on an x86_64 machine with Debian 12.

### Prerequisites and Installation
Prerequisites are a linux machine and the packages _nftables_ and _systemctl_ installed
on it. Installation is performed manually by copying files to a destined location on the
target machine and setting up system services. The steps below are applicable for Debian.
Note that for most of the linux commands specified below you will need root privileges to
properly execute them.

#### Firewall scripts
Copy the two scripts `simplewall_init.conf` and `simplewall_rules.conf` to the directory
`/etc/`. The scripts contain nftables commands and they can be executed
manually with the nft command line tool
```
nft -f </path/to/script>
```
Additionally, command validity without actually executing the commands can be checked
using the -c option:
```
nft -c -f </path/to/script>
```
When executing the scripts manually, be aware of the following:
   - It is advisable _not_ to connect the WAN interface to an ISP modem yet, when setting up
     a machine as a firewall. This is to ensure that if anything goes wrong, the machine does
     not accidentally connect to the internet without firewall rules being in place yet.
   
   - Be aware that the init script `simplewall_init.conf` must be executed _prior to_
     subsequently executing the rules script `simplewall_rules.conf`. This is because the
     rules script references the basic tables and chains created by the init script and would
     run into an error, if these do not already exist.
     
   - When you execute the init script, netfilter chains with default policy `drop` will be
     created. Therefore, make shure you are connected to the firewall machine via its
     console rather than via any remote mechanism like ssh. In the latter case you will be
     blocked by the drop policies and will not be able to access your machine remotely anymore.

In the following subchapter, it is explained how to handle the execution of the two
firewall scripts with the help of system services. In this way, execution does not need to
be performed manually, but in a timely correct manner during system boot.

#### System services
In order to let the system start the firewall scripts automatically during boot, the
following steps have to be performed on the target machine:
1. Copy the following three files:
   ```
     simplewall_init.service
     simplewall_rules.service
     simplewall_rules.target
   ```
   to the directory `/etc/systemd/system/`.

   :point_right: Inspect the `ExecStart` declaration in the files `simplewall_init.service`
   and `simplewall_rules.service` to check if it matches the location of the
   [firewall scripts](#firewall-scripts). Edit and correct accordingly, if needed!
3. Create a new directory `simplewall_rules.target.wants` under `/etc/systemd/system/`:
   ```
     mkdir /etc/systemd/system/simplewall_rules.target.wants
   ```
4. Link the `simplewall_rules.service` into that newly created directory:
   ```
     ln -s /etc/systemd/system/simplewall_rules.service \
     /etc/systemd/system/simplewall_rules.target.wants/simplewall_rules.service
   ```
5. Load the newly installed services by executing `systemctl daemon-reload`
6. Set the system default target as `simplewall_rules.target`:
   ```
   systemctl set-default simplewall_rules.target
   ```
While you're at it with `systemctl` it would be wise to disable the `nftables`
service on your firewall machine with
```
systemctl disable nftables
```
You don't need that service if you intend to use SimpleWall as your firewall and
you would be better off disabling it, so that is does not interfere in any unwanted
way with foresaid services.

Your system is now ready to start the firewall scripts automatically at boot. Before
you reboot, you must edit the definitions regarding the network interface settings of
your machine in `simplewall_rules.conf`. This is described in the next chapter.


### Network interfaces
In a basic router/firewall setup the machine has at least two physical network interfaces:
the WAN connecting to the internet via an ISP and the LAN connecting to internal machines.
On a linux system, these are represented by logical interfaces (like `eth0` or `eth1`).
At the time the script `simplewall_rules.conf` is executed (either manually or by a service),
all interfaces defined within the script must exist. It is therefore important to check
the interfaces and edit the definitions accordingly, in the section under the comment
> \# Define your interfaces here

Furthermore, `simplewall_rules.conf` makes use of the flowtable concept of nftables to
offload the forward chain for better forwarding performance. As with the interfaces, all
devices added to the flowtable must exist. It is therefore essential to check for this
and edit the devices accordingly, in the section under the comment
> \# Add devices to the flowtable

### Firewall startup
When a firewall machine boots, some firewall rules should be in place ***before*** the
network interfaces are initialized and network traffic is enabled. This is the very
essence of a firewall. With SimpleWall, this is established by the service
`simplewall_init.service`. It initializes SimpleWall by executing the script
`simplewall_init.conf` before network setup. In this way it ensures that all network 
traffic is blocked by the `drop` defaults in the netfilter chains, as soon as the network
interfaces come up.

You do not want your firewall to block _all_ network traffic of course, but allow it
according to specific rules. This is established when the service
`simplewall_rules.service` comes into play. It starts at last during boot and sets your
firewall rules by executing the script `simplewall_rules.conf`, allowing
for network traffic as intended by the firewall.

If you followed the steps above, you are now ready to reboot your firewall machine.
After boot, you should be able to access it via ssh from machines in the internal
LAN (since you defined appropriate rules for this :wink: ).

### Operation
coming soon...
