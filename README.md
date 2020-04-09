# HashiCorp `vagrant` mock of **`consul`**
This repo contains a `Vagrantfile` mocking an example of a [Consul](https://www.consul.io/) cluster / setup consisting of three (3) server VM's as well as another three (2) client VM instances.
Official documentation, guides and documentation may be found at: [learn.hashicorp.com](https://learn.hashicorp.com/consul/datacenter-deploy/deployment-guide) or [consul.io/docs](https://www.consul.io/docs/)


## Makeup & Concept
A depiction below shows network [connectivity and overall PRC, Gossip, UDP/TCP ports (8300-8302, 8500, 8600)](https://learn.hashicorp.com/consul/datacenter-deploy/reference-architecture#network-connectivity) expected to be produced. Consul servers are configured with recommended hardening setting of [agent-encryption](https://learn.hashicorp.com/consul/security-networking/agent-encryption) & [TLS certificates](https://learn.hashicorp.com/consul/security-networking/certificates).

```
CONSUL SERVER AGENTS: ._________________.252
      (HA Mode)       | consul-server1  |               CONSUL CLIENT AGENTS:
                      | leader at start |                    ._________________.
                      |_________________|                    | consul-client1  |
                      ▲                 ▲               <----|_________________|
                     /                   \                           
                    /                     \                      ._________________.
     ._____________▼___.251            .___▼_____________.250    | consul-client2  |
     | consul-server2  |               | consul-server3  |  <----|_________________|
     |    follower     |◄-------------►|    follower     |
     |_________________|               |_________________|  ... + other nodes ...
```


### Prerequisites
Ensure that you already have the following hardware & software requirements:
 
##### HARDWARE
 - **RAM** **5**+ Gb Free at least (ensure you're not hitting SWAP either or are < 100Mb)
 - **CPU** **5**+ Cores Free at least (2 or more per instance better)
 - **Network** interface allowing IP assignment and interconnection in VirtualBox bridged mode for all instances.
 - - adjust `sNET='en0: Wi-Fi (Wireless)'` in **`Vagrantfile`** to match your system.

##### SOFTWARE
 - [**Virtualbox**](https://www.virtualbox.org/)
 - [**Virtualbox Guest Additions (VBox GA)**](https://download.virtualbox.org/virtualbox/)
 - > **MacOS** (aka OSX) - VirtualBox 6.x+ is expected to be shipped with the related .iso present under (eg):
 `/Applications/VirtualBox.app/Contents/MacOS/VBoxGuestAdditions.iso`
You may however need to download the .iso specific to your version (mount it) and execute the VBoxDarwinAdditions.pkg
 - [**Vagrant**](https://www.vagrantup.com/)
 - **Few** (**2-5**) **`shell`** or **`screen`** sessions to allow for multiple SSH sessions.


## Usage & Workflow
Refer to the contents of **`Vagrantfile`** for the number of instances, resources, Network, IP and provisioning steps.

The provided **`.sh`** script are installer helpers that download the latest binaries (or specific versions) and can install server / client mode systemd services.

**Inline Environment Variables** can be set for specific versions, modes (server / client), license and other settings that are part of `2.install_consul.sh`.

```bash
vagrant up ; # or 'vagrant reload' when adjusting Vagrantfile.
 # // ... output of provisioning steps.

vagrant global-status --prune ; # should show running nodes
 # id       name           provider   state   directory
 # -------------------------------------------------------------------------------------
 # 1dab856  consul-server1 virtualbox running /home/auser/hashicorp.vagrant_consul
 # 4cf4a84  consul-server2 virtualbox running /home/auser/hashicorp.vagrant_consul
 # 44f02f0  consul-server3 virtualbox running /home/auser/hashicorp.vagrant_consul
 # a406fd4  consul-client1 virtualbox running /home/auser/hashicorp.vagrant_consul
 # 1d165d0  consul-client2 virtualbox running /home/auser/hashicorp.vagrant_consul

vagrant ssh consul-server1 ;

# vagrant@consul-server1:~$ \
consul members list ;
 # Node            Address              Status  Type    Build  Protocol  DC   Segment
 # consul-server1  192.168.77.1:8301    alive   server  1.5.1  2         dc1  <all>
 # consul-server2  192.168.77.2:8301    alive   server  1.5.1  2         dc1  <all>
 # consul-server3  192.168.77.3:8301    alive   server  1.5.1  2         dc1  <all>
 # consul-client1  192.168.77.253:8301  alive   client  1.5.1  2         dc1  <default>
 # consul-client2  192.168.77.252:8301  alive   client  1.5.1  2         dc1  <default>

# vagrant@consul-server1:~$ \
consul operator raft list-peers ;
 # Node            ID                                    Address            State     Voter  RaftProtocol
 # consul-server1  eff2053c-50d6-ec34-e83a-e3ca2c258b18  192.168.77.1:8300  leader    true   3
 # consul-server2  7fb6abf8-a529-5c88-ab8b-d790ff7eab3e  192.168.77.2:8300  follower  true   3
 # consul-server3  b11bf00e-c84f-fdce-5dab-b3bd00d94cc4  192.168.77.3:8300  follower  true   3
# vagrant@consul-server1:~$ \
exit ;  # // CTRL+d to exit.


# // attempt some function from client1
vagrant ssh consul-client1 ;
# vagrant@consul-client1:~$ \
consul kv put test/write_test "SOME_RANDOM_VALUE_${RANDOM}" ;
 # Success! Data written to: test/write_test
# vagrant@consul-client1:~$ \
consul kv get test/write_test ;
 # SOME_RANDOM_VALUE_14578
exit ;  # // CTRL+d to exit when done

# // can attempt other scenarios of stopping nodes and seeing change in leaders
# // or enable ACL ...

# // ---------------------------------------------------------------------------
# when completely done:
vagrant destroy -f consul-server1 consul-server2 consul-server3 consul-client1 consul-client2 consul-client3 ; # ... destroy al
# vagrant box remove -f debian/buster64 --provider virtualbox ; # ... delete box images
```


## Notes
This is intended as a mere practise / training exercise & may also be used toward setting up [Consul ACL](https://learn.hashicorp.com/consul/day-0/acl-guide)

Reference:
 - [aphorise/hashicorp.vagrant_vault_consul](https://github.com/aphorise/hashicorp.vagrant_vault_consul/)
 - [Consul Configuration](https://www.consul.io/docs/agent/options.html)

------
