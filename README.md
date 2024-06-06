# mac-m1-dpdklab
dpdk installation and testing on a MAC M1 with VMware Fusion 

# the "executive" summary 
the intent of this project is to help all the people compiling dpdk apps for testing purposes on a MAC M1 (arm64-based) laptop. Considering the huge limitation of M1's about not allowing nested VM's, the lab uses external VM's to push traffic into OVS-dpdk. 

# hardware and software setup 
1. Any MAC with arm64 cpu would fit, in this case i'm using a 2021 MacBook PRO.
2. macOS 14.4 Sonoma and VMware Fusion 13.5.2
3. the VM used in the lab: Ubuntu 24.4 ARM64 for both client and OVS hoss, ubuntu-24.04-live-server-arm64.iso is the one i've used. 
4. hardware setup for the Ubuntu VM is as follows:
   - 4 processors
   - 8GB RAM
   - 100GB disk (thin)
   - 4 network interfaces
     - 1st is shared with your MAC or attached to a bridge in case you need SSH access from local terminal 
     - 2nd, 3rd and 4th are connected to isolated network (vmnet2/vmnet3 in Fusion) 
5. once Ubuntu installation is done set you preferred access method (ssh requires ens160 IP address setup, VNC, or login straight from Fusion console)
6. check internet connectivity, apt -y update/upgrade, go to next section. 

# dpdk installation and packages 
1. Download the dpdk tar archive from [official dpdk.org link](https://core.dpdk.org/download/) preferring an LTS version.
2. Officlal installation instructions for Linux [here](https://doc.dpdk.org/guides/linux_gsg/index.html)
3. follow chapter 2.2. Compilation of the DPDK - Required Tools and Libraries
   - ``` apt install build-essential ``` 
   - 
   - 

support ARM chipsets BlueField, DPAA, DPAA2, OCTEON

# ovs-dpdk installation and packages 

# a diagram to rule them all 

# ping testing 

# packeth testing 

here is a [link example](https://pages.github.com/)
