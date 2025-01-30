## Overview

Provisioning performant VMs with GPU Direct RoCE (GDR) requires changes in following layers of the stack: 1. Host BIOS, 2. Network Interface Controller, 3. Host Hypervisor, 4. Guest VM XML, 5. Guest OS, and 6. Communication Library Runtime. The following figure visualizes the topology of an NVIDIA HGX platform based server.

![figure](https://github.com/apoorvemohan/HCIR/blob/main/vela_asplos2025/vmsetup/compute-topology.png)

##### 1. Host BIOS: The following were testing on Intel IceLake and Cascade Lake systems:
```
Enable VMX, IOMMU, 4k PCIe MaxReadRequest in BIOS.
```

##### 2. Network Interface Controller: The following fields and resprected values are critcal to achieve desired GPU Direct RoCE performance from within the A100 VMs (tested with ConnectX-6 ethernet variant type of cards - requires 2 reboots).
```
ADVANCED_PCI_SETTINGS               True(1) # A reboot is required after enabling ADVANCED_PCI_SETTINGS
MAX_ACC_OUT_READ                    44
PCI_WR_ORDERING                     per_mkey(0)
P2P_ORDERING_MODE                   SECURE_NONE(1)
ATS_ENABLED                         True(1)
ADVANCED_PCI_SETTINGS               True(1)
RDMA_SELECTIVE_REPEAT_EN            True(1)	# Another reboot is required to make all changes take effect
```

##### 3. Host Hypervisor: The following were testing with Linux Kernel 5.15.
```
3a. Kernel Command Line Parameters:

Enable IOMMU: intel_iommu=on iommu=pt
Bind GPUs and NVBridges to VFIO: vfio-pci.ids=<gpu and bridge vendor pci id>
Disable Open-Source Graphics Driver: rdblaclist=nouveau
Enable Hugepages: default_hugepagesz=1G hugepagesz=1G hugepages=<num pages>

3b. PCI Configurations:

Enable Access Control Servies (ACS): setpci -s <pci address> ECAP_ACS+0x6.w=0x5D
Enable Address Translation Services (ATS): setpci -s <pci address> ECAP_ATS+6.w=8000
Configure MaxReadReq to 4K: setpci -s <pci address> 68.w=5930
```
	
##### 4. Guest VM XML: Attached sample files for details (sample-optimized-a100-vm-i440fx.xml and sample-optimized-a100-vm-q35.xml).
```
4a. Add "<driver ats='on'/>" to PCI controllers.

4b. Emulate NUMA node as available on the physical node.

<numatune>
	<memory mode='strict' nodeset='0-1'/>
	<memnode cellid='0' mode='strict' nodeset='0'/>
	<memnode cellid='1' mode='strict' nodeset='1'/>
</numatune>
...
...
<numa>
	<cell id='0' cpus='0-23' memory='393216000' unit='KiB'/>
	<cell id='1' cpus='24-47' memory='393216000' unit='KiB'/>
</numa>

4c. Attach PCIe controllers to correct NUMA nodes "<node>1</node>".

<controller type='pci' index='3' model='pcie-expander-bus'>
	<model name='pxb-pcie'/>
	<target busNr='140'>
		<node>1</node>
	</target>
	<driver ats='on'/>
	<address type='pci' domain='0x0000' bus='0x00' slot='0x08' function='0x0'/>
</controller>

4d. Attach devices to the correct NUMA node.
```

##### 5. Guest OS: The following network configurations inside the guest os are important.
```
5a. Network Interface Parameters:

ip link set <interface_name> mtu 9000
ip link set <interface_name> txqueuelen 10000
ethtool -G <interface_name> tx 8192
ethtool -G <interface_name> rx 8192

5b. IPv4 ARP Parameters:

sysctl -w net.ipv4.conf.all.arp_filter=1
sysctl -w net.ipv4.conf.default.arp_filter=1
sysctl -w net.ipv4.conf.all.arp_ignore=2
sysctl -w net.ipv4.conf.default.arp_ignore=2
sysctl -w net.ipv4.conf.all.arp_announce=1
sysctl -w net.ipv4.conf.default.arp_announce=1

5c. Network Parameters (important for using TCP for training):

sysctl -w net.ipv4.tcp_timestamps=0
sysctl -w net.ipv4.tcp_sack=1
sysctl -w net.core.rmem_max=4194304
sysctl -w net.core.wmem_max=4194304 
sysctl -w net.core.rmem_default=4194304
sysctl -w net.core.wmem_default=4194304
sysctl -w net.core.optmem_max=4194304
sysctl -w net.ipv4.tcp_rmem="4096 87380 4194304"
sysctl -w net.ipv4.tcp_wmem="4096 65536 4194304"
sysctl -w net.ipv4.tcp_low_latency=1

5d. GPU Parameters:

nvidia-smi -pm 1 
gpufreq=$(nvidia-smi -i 0 --query-supported-clocks=gr --format=csv | cut -f 1 -d ' ' | sort -n -r | head -n 1)
memfreq=$(nvidia-smi -i 0 --query-supported-clocks=mem --format=csv | cut -f 1 -d ' ' | sort -n -r | head -n 1)
for idx in $(nvidia-smi -L |cut -f 1 -d :|cut -f 2 -d ' '|tr '\n' ' ');
do
	nvidia-smi -i $idx -rac 
	nvidia-smi -i $idx -ac $memfreq,$gpufreq 
	nvidia-smi -i $idx -lgc $gpufreq,$gpufreq 
	nvidia-smi -i $idx -lmc $memfreq,$memfreq 
	nvidia-smi -i $idx -cc 1 
done
```

##### 6. Communication Library Runtime: When using Nvidia-based network infrastructure, it is critical to configure Nvidia's Collective Communication Library to achive optimal perforance for GDR.
```
6a. NCCL Environment Variables:

CUDA_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
NCCL_IB_DISABLE=0
NCCL_IGNORE_CPU_AFFINITY=1
NCCL_BUFFSIZE=67108864 
NCCL_IB_PCI_RELAXED_ORDERING=1

# The following are environment/infrastructure setup.
NCCL_IB_HCA=<Comman Separated IB HCA List>
NCCL_IB_QPS_PER_CONNECTION=<>
NCCL_IB_GID_INDEX=<RoCE GID>
NCCL_ALGO=<Collective Algorithm>
NCCL_SOCKET_IFNAME=<Comman Separated ETH NIC List>
NCCL_SOCKET_NTHREADS=<>
NCCL_CROSS_NIC=<>

6b. NCCL Topology File: See sample-nccl-topology.xml for 8xA100 GPUs and 2xvNIC (default location /var/run/nvidia-topologyd/virtualTopology.xml).
```
