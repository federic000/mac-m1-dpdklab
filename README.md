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

# a diagram to rule them all 

# ping testing 

# packeth testing 

here is a [link example](https://pages.github.com/)
