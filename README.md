# mac-m1-dpdklab
dpdk installation and testing on a MAC M1 with VMware Fusion 

# the "executive" summary 
the intent of this project is to help all the people compiling dpdk apps for testing purposes on a MAC M1 (arm64-based) laptop. Considering the huge limitation of M1's about not allowing nested VM's, the lab uses external VM's to push traffic into OVS-dpdk. 

# hardware and software setup 
1. Any MAC with arm64 cpu would fit, in this case i'm using a 2021 MacBook PRO.
2. macOS 14.4 Sonoma and VMware Fusion 13.5.2
3. the VM used in the lab: Ubuntu 24.4-LTS ARM64 for both client and OVS hosts, `ubuntu-24.04-live-server-arm64.iso` is the one i've used. 
4. hardware setup for the Ubuntu VM is as follows:
   - 4 processors
   - 8GB RAM
   - 100GB disk (thin)
   - 4 network interfaces
     - 1st is shared with your MAC or attached to a bridge in case you need SSH access from local terminal 
     - 2nd, 3rd and 4th are connected to isolated network (vmnet2/vmnet3 in Fusion) 
5. once Ubuntu installation is done set you preferred access method (ssh requires ens160 IP address setup, VNC, or login straight from Fusion console)
6. on the VM let's check internet connectivity, issue ```apt -y update; apt -y upgrade```, go to next section. 

# dpdk installation and packages 
1. Download the dpdk tar archive from [official dpdk.org link](https://core.dpdk.org/download/) preferring an LTS version.
2. Official installation instructions for Linux [here](https://doc.dpdk.org/guides/linux_gsg/index.html)
3. as root, follow chapter 2.2. Compilation of the DPDK - Required Tools and Libraries is not exactly as in the docs, for Ubuntu 24.4 you can do it all via apt: 
   - ``` apt -y install build-essential ``` 
   - ``` apt -y install meson ``` (it will take care of ninja2 as well) 
   - ``` apt -y install python3-pyelftools ```
   - ``` apt -y install libnuma-dev ```
4. Reserving Hugepages for DPDK it's an important requirement:
   - ``` echo 2048 > /sys/devices/system/node/node0/hugepages/hugepages-2048kB/nr_hugepages ```
   - to keep it across reboots, edit `/etc/default/grub`
   - add `GRUB_CMDLINE_LINUX_DEFAULT="hugepages=2048"` and then ``` update-grub ```
5. untar the dpdk archive, cd into the extracted directory
6. ```meson setup <options> build ``` now for the options since dpdk does not support Apple's Silicon ARM64 (only BlueField, DPAA, DPAA2 and OCTEON are currently supported) we need to specify a generic Platform to make it finish the build process, as a consequence we do not get a configuration optimized build for the M1 SoC.
    -  ``` meson setup -Dplatform=generic build ```
    -  ``` cd build ```
    -  ``` ninja ```
    -  ``` ninja install ```
    -  ``` ldconfig ```
7. you can see now new dpdk libraries in your /usr/local/lib as in...
   


# ovs-dpdk installation and packages 


root@suppuione:~/dpdk-stable-23.11.1/usertools# ./dpdk-devbind.py -s

Network devices using kernel driver
===================================
0000:02:00.0 '82574L Gigabit Network Connection 10d3' if=ens160 drv=e1000e unused=uio_pci_generic *Active*
0000:0a:00.0 '82574L Gigabit Network Connection 10d3' if=ens192 drv=e1000e unused=uio_pci_generic 
0000:12:00.0 '82574L Gigabit Network Connection 10d3' if=ens224 drv=e1000e unused=uio_pci_generic 
0000:13:00.0 '82574L Gigabit Network Connection 10d3' if=ens225 drv=e1000e unused=uio_pci_generic 
0000:1a:00.0 '82574L Gigabit Network Connection 10d3' if=ens256 drv=e1000e unused=uio_pci_generic 



root@suppuione:~/dpdk-stable-23.11.1/usertools# ./dpdk-devbind.py -b uio_pci_generic ens192
root@suppuione:~/dpdk-stable-23.11.1/usertools# ./dpdk-devbind.py -b uio_pci_generic ens224
root@suppuione:~/dpdk-stable-23.11.1/usertools# ./dpdk-devbind.py -s

Network devices using DPDK-compatible driver
============================================
0000:0a:00.0 '82574L Gigabit Network Connection 10d3' drv=uio_pci_generic unused=e1000e
0000:12:00.0 '82574L Gigabit Network Connection 10d3' drv=uio_pci_generic unused=e1000e

Network devices using kernel driver
===================================
0000:02:00.0 '82574L Gigabit Network Connection 10d3' if=ens160 drv=e1000e unused=uio_pci_generic *Active*
0000:13:00.0 '82574L Gigabit Network Connection 10d3' if=ens225 drv=e1000e unused=uio_pci_generic 
0000:1a:00.0 '82574L Gigabit Network Connection 10d3' if=ens256 drv=e1000e unused=uio_pci_generic 





# a diagram to rule them all 

# ping testing 

root@suppuione2:~# ip netns exec ns1 ip add
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
3: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:78:3c:b1 brd ff:ff:ff:ff:ff:ff
    altname enp10s0
    inet 172.16.1.1/24 scope global ens192
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe78:3cb1/64 scope link 
       valid_lft forever preferred_lft forever
root@suppuione2:~# ip netns exec ns2 ip add
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
4: ens224: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:78:3c:bb brd ff:ff:ff:ff:ff:ff
    altname enp18s0
    inet 172.16.1.2/24 scope global ens224
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe78:3cbb/64 scope link 
       valid_lft forever preferred_lft forever


# packeth testing 

root@suppuione2:~# ip netns exec ns1 packeth -m 2 -i ens192 -f ./icmp.pcap -t 10 -d 0 
Sent 305524 packets on ens192; 98 bytes packet length; 305524 packets/s; 239.530 Mbit/s data rate; 298.191 Mbit/s link utilization
Sent 616242 packets on ens192; 98 bytes packet length; 310718 packets/s; 243.602 Mbit/s data rate; 303.260 Mbit/s link utilization
Sent 927035 packets on ens192; 98 bytes packet length; 310793 packets/s; 243.661 Mbit/s data rate; 303.333 Mbit/s link utilization
Sent 1238116 packets on ens192; 98 bytes packet length; 311081 packets/s; 243.887 Mbit/s data rate; 303.615 Mbit/s link utilization
Sent 1543973 packets on ens192; 98 bytes packet length; 305857 packets/s; 239.791 Mbit/s data rate; 298.516 Mbit/s link utilization
Sent 1857701 packets on ens192; 98 bytes packet length; 313728 packets/s; 245.962 Mbit/s data rate; 306.198 Mbit/s link utilization
Sent 2174599 packets on ens192; 98 bytes packet length; 316898 packets/s; 248.448 Mbit/s data rate; 309.292 Mbit/s link utilization
Sent 2477818 packets on ens192; 98 bytes packet length; 303219 packets/s; 237.723 Mbit/s data rate; 295.941 Mbit/s link utilization
Sent 2796134 packets on ens192; 98 bytes packet length; 318316 packets/s; 249.559 Mbit/s data rate; 310.676 Mbit/s link utilization



here is a [link example](https://pages.github.com/)
