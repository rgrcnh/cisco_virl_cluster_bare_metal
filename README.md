# Build a Virl Cluster on Bare Metal Intallation With only two physical ethernets. 

These are my experience in building a Cisco Virl Cluster, with two machines, using the bare metal VIRL release and two physical interfaces, without any external resource.

On this cenario, the eth0 of each host will be connect on your LAN and eth1 will be back-to-back. We will be creating some VLANs to connect the FLAT, FLAT1 and SNAT networks over the tied interface. The internalnet_port will be the binded to the physical eth1 (using untagged frames), so it will be ever up.

So, let's go...


1. Connect back-to-back the two hosts on the second interface eth1 (the main interface eth0 of each should be attached to your LAN). Beware of using a cross-cable, if yours hosts does not support MID-X.

2. Configure your VIRL controller, with the following interfaces (editing /etc/virl.ini), and change the respective lines indicated bellow.

```
# those lines are not in sequence on /etc/virl.ini
public_port: eth0
l2_port: eth4
l2_port2: eth2 
l3_port: eth3 
internalnet_port: eth1 
compute1_public_port: eth0
compute1_l2_port: eth4
compute1_l2_port2: eth2
compute1_l3_port: eth3
compute1_internalnet_port: eth1
```

2.1 these are my virl.ini computer1 specific configuration, on the controller.
Change XXXX to your LAN values.

```
## Compute1 specifics
compute1_active: True
compute1_hostname: compute1
compute1_public_port: eth0
compute1_using_dhcp_on_the_public_port: False
compute1_static_ip: *XXXX #Change Here*
compute1_public_netmask: *XXXX #Change Here* 
compute1_public_gateway: *XXXX #Change Here*
compute1_first_nameserver: *XXXX #Change Here*
compute1_second_nameserver: *XXXX #Change Here*
compute1_l2_port: eth4
compute1_l2_address: 172.16.1.241
compute1_l2_mask: 255.255.255.0
compute1_l2_port2: eth2
compute1_l2_address2: 172.16.2.241
compute1_l2_mask2: 255.255.255.0
compute1_l3_port: eth3
compute1_l3_address: 172.16.3.241
compute1_l3_mask: 255.255.255.0
compute1_internalnet_port: eth1
compute1_internalnet_ip: 172.16.10.241
```

3. put those lines on /etc/rc.local
Those commands will start the VLANs up, using interfaces named as eth2, eth3 and eth4.

```
sudo ip link add link eth1 name eth2 type vlan id 20
sudo ip link add link eth1 name eth3 type vlan id 30
sudo ip link add link eth1 name eth4 type vlan id 40
```

4. Rehost: 

``` 
virl rehost
```

5. reboot the controller and check if eth0, eth1, eth2, eth3 and eth4 are all up.


6. Follow the Cisco/Virl [Cluste configuration Manual](http://virl-dev-innovate.cisco.com/virl.cluster.php), and stop at step 12 of "Customizing the Compute Node" section.

7 . Use the following configuration on computer1 for /etc/network/interfaces. Change your IP external address, mask, gateway and DNSs, according your LAN.


```
auto eth1
iface eth1 inet static
    address 172.16.10.241
    netmask 255.255.255.0
    mtu 1500
    post-up ip link set eth1 promisc on
auto lo
iface lo inet loopback
auto lo:1
iface lo:1 inet loopback
    address 127.0.1.1
    netmask 255.0.0.0
auto eth4
iface eth4 inet static
    address 172.16.1.241
    netmask 255.255.255.0
    post-up ip link set eth4 promisc on
auto eth2
iface eth2 inet static
    address 172.16.2.241
    netmask 255.255.255.0
    post-up ip link set eth2 promisc on
auto eth3
iface eth3 inet static
    address 172.16.3.241
    netmask 255.255.255.0
    post-up ip link set eth3 promisc on
auto eth0
iface eth0 inet static
    address *XXX #Change Here*
    netmask *XXX #Change Here*
    gateway *XXX #Change Here*
    dns-nameservers *XXX YYYY #Change Here*
```

8. Insert those lines on /etc/rc.local of computer1
The

```
sudo ip link add link eth1 name eth2 type vlan id 20
sudo ip link add link eth1 name eth3 type vlan id 30
sudo ip link add link eth1 name eth4 type vlan id 40
```
9. Edit the file neutron.conf of computer1

sudo vi /etc/neutron/neutron.conf

and change the following section to be looking like:

```
[linux_bridge]
physical_interface_mappings=flat:eth4,flat1:eth2,ext-net:eth3
```

10. restart the computer1

11. follow the steps of Cisco/Virl [Manual] (http://virl-dev-innovate.cisco.com/virl.cluster.php),  at step 13 of "Customizing the Compute Node" until the end. 

11. It should be working now.
