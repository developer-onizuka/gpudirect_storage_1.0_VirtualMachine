If you want to understand what GDS is, then you can learn URL below:

https://github.com/developer-onizuka/what_is_GPUDirect-Storage

# 0. Hardware
```
(1) Optiplex 5050SFF  ... JPY 29,150
    Intel(R) Core(TM) i3-7500 CPU @ 3.50GHz
    DIMM slot1: DDR4 DIMM 8GB (Micron)
    DIMM slot2: Empty
    DIMM slot3: Empty
    DIMM slot4: Empty
    HDD 500GB  ---> Windows10 pro
    DVD DRIVE  ---> replace to SATA SSD(Ubuntu 20.04)
(2) SATA SSD  ... JPY 2,111
    HYUNDAI SSD 120GB
    P/N: C2S3T/120G
(3) DDR4 DIMM 8GB x2 ... JPY 5,555
    Micron Memory DDR4 2666MHz PC4-2400T-UA1-11
(4) DDR4 DIMM 8GB ... JPY 2,555
    Hynix Memory DDR4 2400MHz PC4-19200
    HMA81GU6AFR8N-UH
(5) NVMe SSD ... JPY 2,999
    SM961 Series MZ-VLW1280 128GB M.2 Type2280 PCIe3x4 NVMe 
    P/N: MZVLW128HEGR-000L1
    Performance Spec: Read 2800MB/s, Write 600MB/s
(6) ETC
    -Zheino 2nd 9.5mm Note PC drive mounter ... JPY 899
    -GLOTRENDS M.2 Heatsink ... JPY 650
(7) NVIDIA Quadro P1000 ... JPY 15,800

----- additional NVMes, see the section of #14.1 and #14.2 -----
(8) NVMe SSD ... JPY 2,050
    SK hynix BC501 NVMe Solid State Drive 128GB
    P/N: HFM128GDJTNG-8310A
    Performance Spec: Read 1400MB/s, Write 395MB/s
(9) NVMe SSD ... JPY ?????
    Phison Electronics Corporation E12 NVMe Controller 512GB
    P/N: PS5012-E12S-512G
    Performance Spec: Read 3400MB/s、Write 2400MB/s
```

# 1. Install Ubuntu-20.04 on Host Machine
```
   Install Ubuntu 20.04 as "Minimal Install" and don't select "install third-party software for graphics and Wi-Fi hardware and additional media formats".
   Followings are optional, but it is very convenient.
   $ sudo vi /etc/apt/apt.conf.d/20auto-upgrades
     APT::Periodic::Update-Package-Lists "0";
     APT::Periodic::Unattended-Upgrade "0";
   $ sudo visudo
     username ALL=NOPASSWD: ALL

   See also followings:
   https://qiita.com/RyodoTanaka/items/e9b15d579d17651650b7
   https://thr3a.hatenablog.com/entry/20170805/1501943406
```

# 2. Check GPU's bus id at Host Machine
```
$ lspci -nn |grep -i nvidia
01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GP107GL [Quadro P1000] [10de:1cb1] (rev a1)
01:00.1 Audio device [0403]: NVIDIA Corporation GP107GL High Definition Audio Controller [10de:0fb9] (rev a1)
```
# 3. Check NVMe's bus id at Host Machine
```
$ lspci -nn |grep -i nvme
03:00.0 Non-Volatile memory controller [0108]: Samsung Electronics Co Ltd NVMe SSD Controller SM961/PM961 [144d:a804]
```

The case of Hynix NVMe (Updated 2021/08/03)
```
$ lspci -nn |grep -i nvme
03:00.0 Non-Volatile memory controller [0108]: SK hynix BC501 NVMe Solid State Drive 512GB [1c5c:1327]
```


# 4. Edit /etc/default/grub at Host Machine
You should use the id above as the parameters of vfio-pci.ids.
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
GRUB_CMDLINE_LINUX="quiet splash intel_iommu=on vfio_iommu_type1.allow_unsafe_interrupts=1 iommu=pt vfio-pci.ids=10de:1cb1,10de:0fb9,144d:a804"

```
The case of Hynix NVMe (Updated 2021/08/03)
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
GRUB_CMDLINE_LINUX="quiet splash intel_iommu=on vfio_iommu_type1.allow_unsafe_interrupts=1 iommu=pt vfio-pci.ids=10de:1cb1,10de:0fb9,1c5c:1327"
```

# 5. Update grub and Reboot at Host Machine
```
sudo update-grub
sudo reboot
```
# 6. Check VFIO at Host Machine
You can find the Kernel driver replaced by "vfio-pci". 
I could not use the NVMe Device using Silicon Motion's controller [126f:2263] as PassThrough. 
I use the Samsung and Hynix NVMe instead of KLEVV which used Silicon Motion controller.
```
[    0.000000] Command line: BOOT_IMAGE=/boot/vmlinuz-5.8.0-59-generic root=UUID=33af4bcd-d3f9-4e6f-9ddd-5d6b2d02c044 ro quiet splash intel_iommu=on vfio_iommu_type1.allow_unsafe_interrupts=1 iommu=pt vfio-pci.ids=10de:1cb1,10de:0fb9,144d:a804 quiet splash vt.handoff=7
[    0.046801] Kernel command line: BOOT_IMAGE=/boot/vmlinuz-5.8.0-59-generic root=UUID=33af4bcd-d3f9-4e6f-9ddd-5d6b2d02c044 ro quiet splash intel_iommu=on vfio_iommu_type1.allow_unsafe_interrupts=1 iommu=pt vfio-pci.ids=10de:1cb1,10de:0fb9,144d:a804 quiet splash vt.handoff=7
[    0.509191] VFIO - User Level meta-driver version: 0.3
[    0.509355] vfio-pci 0000:01:00.0: vgaarb: changed VGA decodes: olddecodes=io+mem,decodes=io+mem:owns=none
[    0.527121] vfio_pci: add [10de:1cb1[ffffffff:ffffffff]] class 0x000000/00000000
[    0.547118] vfio_pci: add [10de:0fb9[ffffffff:ffffffff]] class 0x000000/00000000
[    0.567150] vfio_pci: add [144d:a804[ffffffff:ffffffff]] class 0x000000/00000000
[    3.166385] vfio-pci 0000:01:00.0: vgaarb: changed VGA decodes: olddecodes=io+mem,decodes=io+mem:owns=none

$ lspci -nnk -d 10de:1cb1
01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GP107GL [Quadro P1000] [10de:1cb1] (rev a1)
	Subsystem: NVIDIA Corporation GP107GL [Quadro P1000] [10de:11bc]
	Kernel driver in use: vfio-pci
	Kernel modules: nvidiafb, nouveau

$ lspci -nnk -d 10de:0fb9
01:00.1 Audio device [0403]: NVIDIA Corporation GP107GL High Definition Audio Controller [10de:0fb9] (rev a1)
	Subsystem: NVIDIA Corporation GP107GL High Definition Audio Controller [10de:11bc]
	Kernel driver in use: vfio-pci
	Kernel modules: snd_hda_intel
  
$ lspci -nnk -d 144d:a804
03:00.0 Non-Volatile memory controller [0108]: Samsung Electronics Co Ltd NVMe SSD Controller SM961/PM961 [144d:a804]
	Subsystem: Samsung Electronics Co Ltd NVMe SSD Controller SM961/PM961 [144d:a801]
	Kernel driver in use: vfio-pci
	Kernel modules: nvme
```

The case of Hynix NVMe (Updated 2021/08/03)
```
$ lspci -nnk -d 1c5c:1327
03:00.0 Non-Volatile memory controller [0108]: SK hynix BC501 NVMe Solid State Drive 512GB [1c5c:1327]
	Subsystem: SK hynix BC501 NVMe Solid State Drive 512GB [1c5c:0000]
	Kernel driver in use: vfio-pci
	Kernel modules: nvme
```

The case of Phison NVMe (Updated 2021/08/29)
```
$ lspci -nnk -d 1987:5012
03:00.0 Non-Volatile memory controller [0108]: Phison Electronics Corporation E12 NVMe Controller [1987:5012] (rev 01)
	Subsystem: Phison Electronics Corporation E12 NVMe Controller [1987:5012]
	Kernel driver in use: vfio-pci
	Kernel modules: nvme
```

# 7. Install KVM on Host Machine
```
$ sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils
$ sudo apt install virt-manager
```
```
$ sudo vi /etc/security/limits.conf 
# qemu kvm, need high memlock to allocate mem for vga-passthrough
@kvm    hard    memlock 8388608
@kvm    soft    memlock 8388608
```
```
$ virt-manager
You should add the GPU and NVMe. See also attached png files. You can do this step only after kernel driver was replaced by "vfio-pci" in step#7. 
```

# 8. You might add kernel 5.4.0-42 at Virtual Machine
See also https://kazuhira-r.hatenablog.com/entry/2020/02/28/000625
```
   Install Ubuntu 20.04 as "Minimal Install" and don't select "install third-party software for graphics and Wi-Fi hardware and additional media formats".
   Followings are optional, but it is very convenient.
   $ sudo vi /etc/apt/apt.conf.d/20auto-upgrades
     APT::Periodic::Update-Package-Lists "0";
     APT::Periodic::Unattended-Upgrade "0";
   $ sudo visudo
     username ALL=NOPASSWD: ALL

   See also followings:
   https://qiita.com/RyodoTanaka/items/e9b15d579d17651650b7
   https://thr3a.hatenablog.com/entry/20170805/1501943406
```
```
$ sudo apt-get install linux-image-5.4.0-42-generic linux-headers-5.4.0-42-generic linux-modules-extra-5.4.0-42-generic
$ sudo vi /etc/default/grub
  You should edit as followings, please note the GRUB_TIMEOUT_STYLE and GRUB_TIMEOUT parameters are comment out:
```
```
#GRUB_TIMEOUT_STYLE=hidden
GRUB_TIMEOUT=-1
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash intel_iommu=off"
```
```
$ sudo update-grub
$ reboot

After this step, You will be able to select kernel 5.4.0-42.
```

# 9. Install MOFED5.3 at Virtual Machine
```
Download MLNX_OFED_LINUX-5.3-1.0.0.1-ubuntu20.04-x86_64.tgz.
$ sudo apt-get install python3-distutils
$ cd MLNX_OFED_LINUX-5.3-1.0.0.1-ubuntu20.04-x86_64/
$ sudo ./mlnxofedinstall --with-nfsrdma --with-nvmf --enable-gds --add-kernel-support
$ sudo update-initramfs -u -k `uname -r`
$ sudo reboot
```

# 10. Install nvidia driver at the Virtual Machine
```
$ lspci -nn |grep -i nvidia
04:00.0 VGA compatible controller [0300]: NVIDIA Corporation GP107GL [Quadro P1000] [10de:1cb1] (rev a1)
05:00.0 Audio device [0403]: NVIDIA Corporation GP107GL High Definition Audio Controller [10de:0fb9] (rev a1)
$ ubuntu-drivers devices
$ sudo apt install nvidia-driver-460
$ reboot
$ nvidia-smi 
Unable to determine the device handle for GPU 0000:04:00.0: Unknown Error
```
# 11. Measures for "Unable to determine the device handle for GPU 0000:0x:00.0: Unknown Error" at Host Machine
```
$ virsh edit ubuntu20.04-gpu

Select an editor.  To change later, run 'select-editor'.
  1. /bin/nano        <---- easiest
  2. /usr/bin/vim.tiny
  3. /bin/ed

Choose 1-3 [1]: 1
Domain ubuntu20.04 XML configuration edited.
```
Adding the followings:
```
・・・
  <features>
  ・・・
    <hyperv>
      <vendor_id state='on' value='whatever'/>
    </hyperv>
    <kvm>
      <hidden state='on'/>
    </kvm>
  ・・・
  </features>
・・・
```
Login to the Vritual Machine and do nvidia-smi.
```
$ nvidia-smi
Sun Jul 11 20:12:05 2021       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 460.80       Driver Version: 460.80       CUDA Version: 11.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  Quadro P1000        Off  | 00000000:04:00.0 Off |                  N/A |
| 34%   45C    P8    N/A /  N/A |     11MiB /  4040MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|    0   N/A  N/A       741      G   /usr/lib/xorg/Xorg                  4MiB |
|    0   N/A  N/A      1268      G   /usr/lib/xorg/Xorg                  4MiB |
+-----------------------------------------------------------------------------+
```
# 12. Install CUDA-11.4 at Virtual Machine
```
$ wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-ubuntu2004.pin
$ sudo mv cuda-ubuntu2004.pin /etc/apt/preferences.d/cuda-repository-pin-600
$ wget https://developer.download.nvidia.com/compute/cuda/11.4.0/local_installers/cuda-repo-ubuntu2004-11-4-local_11.4.0-470.42.01-1_amd64.deb
$ sudo dpkg -i cuda-repo-ubuntu2004-11-4-local_11.4.0-470.42.01-1_amd64.deb
$ sudo apt-key add /var/cuda-repo-ubuntu2004-11-4-local/7fa2af80.pub
$ sudo apt-get update
$ sudo apt-get -y install cuda
```
# 13. Install GDS at Virtual Machine
```
$ sudo apt-get update
$ sudo apt install nvidia-gds
$ sudo modprobe nvidia_fs
$ dpkg -s nvidia-gds
$ /usr/local/cuda/gds/tools/gdscheck -p
 GDS release version: 1.0.0.82
 nvidia_fs version:  2.7 libcufile version: 2.4
 ============
 ENVIRONMENT:
 ============
 =====================
 DRIVER CONFIGURATION:
 =====================
 NVMe               : Supported
 NVMeOF             : Unsupported
 SCSI               : Unsupported
 ScaleFlux CSD      : Unsupported
 NVMesh             : Unsupported
 DDN EXAScaler      : Unsupported
 IBM Spectrum Scale : Unsupported
 NFS                : Unsupported
 WekaFS             : Unsupported
 Userspace RDMA     : Unsupported
 --Mellanox PeerDirect : Enabled
 --rdma library        : Not Loaded (libcufile_rdma.so)
 --rdma devices        : Not configured
 --rdma_device_status  : Up: 0 Down: 0
 =====================
 CUFILE CONFIGURATION:
 =====================
 properties.use_compat_mode : true
 properties.gds_rdma_write_support : true
 properties.use_poll_mode : false
 properties.poll_mode_max_size_kb : 4
 properties.max_batch_io_timeout_msecs : 5
 properties.max_direct_io_size_kb : 16384
 properties.max_device_cache_size_kb : 131072
 properties.max_device_pinned_mem_size_kb : 33554432
 properties.posix_pool_slab_size_kb : 4 1024 16384 
 properties.posix_pool_slab_count : 128 64 32 
 properties.rdma_peer_affinity_policy : RoundRobin
 properties.rdma_dynamic_routing : 0
 fs.generic.posix_unaligned_writes : false
 fs.lustre.posix_gds_min_kb: 0
 fs.weka.rdma_write_support: false
 profile.nvtx : false
 profile.cufile_stats : 0
 miscellaneous.api_check_aggressive : false
 =========
 GPU INFO:
 =========
 GPU index 0 Quadro P1000 bar:1 bar size (MiB):256 supports GDS
 ==============
 PLATFORM INFO:
 ==============
 IOMMU: disabled
 Platform verification succeeded
```

# 14. Throughput test at Virtual Machine
```
1. preparation about NVMe SSD
(1) mount nvme as "ordered" mode.
# sudo mount -t ext4 -o data=ordered /dev/nvme0n1 /mnt

2. Seq Read Throughput
(1) Storage->CPU
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 1 -I 0 -T 10 -i 256K
IoType: READ XferType: CPUONLY Threads: 1 DataSetSize: 16629760/10485760(KiB) IOSize: 256(KiB) Throughput: 1.641909 GiB/sec, Avg_Latency: 148.591764 usecs ops: 64960 total_time 9.659107 secs

(2) Storage->CPU->GPU
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 2 -I 0 -T 10 -i 256K
IoType: READ XferType: CPU_GPU Threads: 1 DataSetSize: 15093760/10485760(KiB) IOSize: 256(KiB) Throughput: 1.568393 GiB/sec, Avg_Latency: 155.557090 usecs ops: 58960 total_time 9.177888 secs

(3) Storage -> GPU (GDS)
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 0 -I 0 -T 10 -i 256K
IoType: READ XferType: GPUD Threads: 1 DataSetSize: 17141760/10485760(KiB) IOSize: 256(KiB) Throughput: 1.650979 GiB/sec, Avg_Latency: 147.865965 usecs ops: 66960 total_time 9.901797 secs
```

Write the data to NVMe from GPU thru GDS was failed as following, when I used Samsung NVMe device. It seems to be bad when data size is above 4096B. 
According to my result, non-GDS mode (x=1 or x=2) was fine. 

```
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 0 -I 1 -T 10 -i 1024
IoType: WRITE XferType: GPUD Threads: 1 DataSetSize: 10000/10485760(KiB) IOSize: 1(KiB) Throughput: 0.000893 GiB/sec, Avg_Latency: 1067.162200 usecs ops: 10000 total_time 10.678174 secs
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 0 -I 1 -T 10 -i 2048
IoType: WRITE XferType: GPUD Threads: 1 DataSetSize: 20000/10485760(KiB) IOSize: 2(KiB) Throughput: 0.001799 GiB/sec, Avg_Latency: 1059.693700 usecs ops: 10000 total_time 10.603274 secs
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 0 -I 1 -T 10 -i 4096
Error: IO failed stopping traffic, fd :27 ret:-5 errno :1
io failed :ret :-5 errno :1, file offset :0, block size  :4096
```
A point in doubt
----------------
I am always wondering which DMA engine (GPU's DMA engine or NVMe's DMA engine) plays the role of DMA between GPU mem and NVMe mem.
--> Ans, NVMe's DMA engine (https://github.com/developer-onizuka/what_is_GPUDirect-Storage/blob/main/README.md)



# 14.1 The case of Hynix NVMe (Updated 2021/08/03)
```
(8) NVMe SSD ... JPY 2,050
    SK hynix BC501 NVMe Solid State Drive 128GB
    P/N: HFM128GDJTNG-8310A
    Performance Spec: Read 1400MB/s, Write 395MB/s
       
2. Seq Read Throughput
(1) Storage->CPU
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 1 -I 0 -T 10 -i 256K
IoType: READ XferType: CPUONLY Threads: 1 DataSetSize: 13045760/10485760(KiB) IOSize: 256(KiB) Throughput: 1.300863 GiB/sec, Avg_Latency: 187.664973 usecs ops: 50960 total_time 9.563966 secs

(2) Storage->CPU->GPU
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 2 -I 0 -T 10 -i 256K
IoType: READ XferType: CPU_GPU Threads: 1 DataSetSize: 11765760/10485760(KiB) IOSize: 256(KiB) Throughput: 1.210645 GiB/sec, Avg_Latency: 201.645975 usecs ops: 45960 total_time 9.268366 secs

(3) Storage -> GPU (GDS)
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 0 -I 0 -T 10 -i 256K
IoType: READ XferType: GPUD Threads: 1 DataSetSize: 12789760/10485760(KiB) IOSize: 256(KiB) Throughput: 1.312793 GiB/sec, Avg_Latency: 185.954123 usecs ops: 49960 total_time 9.291081 secs
```

Same as Samsung NVMe. Write the data to NVMe from GPU thru GDS was failed as following, when I used SK Hynix NVMe device. It seems to be bad when data size is above 4096B. According to my result, non-GDS mode (x=1 or x=2) was fine. 
```
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 1 -I 1 -T 10 -i 4096K
IoType: WRITE XferType: CPUONLY Threads: 1 DataSetSize: 4096000/10485760(KiB) IOSize: 4096(KiB) Throughput: 0.210798 GiB/sec, Avg_Latency: 18523.702000 usecs ops: 1000 total_time 18.530790 secs
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 2 -I 1 -T 10 -i 4096K
IoType: WRITE XferType: CPU_GPU Threads: 1 DataSetSize: 4096000/10485760(KiB) IOSize: 4096(KiB) Throughput: 0.208823 GiB/sec, Avg_Latency: 18696.396000 usecs ops: 1000 total_time 18.706036 secs

$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 0 -I 1 -T 10 -i 4096K
Error: IO failed stopping traffic, fd :27 ret:-5 errno :1
io failed :ret :-5 errno :1, file offset :0, block size  :4194304
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 0 -I 1 -T 10 -i 1024
IoType: WRITE XferType: GPUD Threads: 1 DataSetSize: 12000/10485760(KiB) IOSize: 1(KiB) Throughput: 0.001227 GiB/sec, Avg_Latency: 776.935000 usecs ops: 12000 total_time 9.325954 secs
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 0 -I 1 -T 10 -i 2048
IoType: WRITE XferType: GPUD Threads: 1 DataSetSize: 18000/10485760(KiB) IOSize: 2(KiB) Throughput: 0.001765 GiB/sec, Avg_Latency: 1080.127889 usecs ops: 9000 total_time 9.723622 secs
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 0 -I 1 -T 10 -i 4096
Error: IO failed stopping traffic, fd :27 ret:-5 errno :1
io failed :ret :-5 errno :1, file offset :0, block size  :4096
```

# 14.2 The case of Phison NVMe (Updated 2021/08/29)
Phison is also same as Samsung NVMe and Write the data to NVMe from GPU thru GDS was failed. It seems to be bad when data size is above 4096B. According to my result, non-GDS mode (x=1 or x=2) was fine.
```
(9) NVMe SSD ... JPY ?????
    Phison Electronics Corporation E12 NVMe Controller 512GB
    P/N: PS5012-E12S-512G
    Performance Spec: Read 3400MB/s、Write 2400MB/s
       
2. Seq Read Throughput (4096K is better than 256KB for Phison NVMe Controller)
(1) Storage->CPU
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 1 -I 0 -T 10 -i 4096K
IoType: READ XferType: CPUONLY Threads: 1 DataSetSize: 25067520/10485760(KiB) IOSize: 4096(KiB) Throughput: 2.201092 GiB/sec, Avg_Latency: 1773.886275 usecs ops: 6120 total_time 10.861086 secs

(2) Storage->CPU->GPU
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 2 -I 0 -T 10 -i 4096K
IoType: READ XferType: CPU_GPU Threads: 1 DataSetSize: 20971520/10485760(KiB) IOSize: 4096(KiB) Throughput: 1.927207 GiB/sec, Avg_Latency: 2025.890430 usecs ops: 5120 total_time 10.377712 secs

(3) Storage -> GPU (GDS)
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 0 -I 0 -T 10 -i 4096K
IoType: READ XferType: GPUD Threads: 1 DataSetSize: 25067520/10485760(KiB) IOSize: 4096(KiB) Throughput: 2.301043 GiB/sec, Avg_Latency: 1696.841993 usecs ops: 6120 total_time 10.389311 secs
```

# 15. Update CUDA from 11.4 to 11.4.1 which is latest version of CUDA (Update 2021/08/04)
```
$ wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-ubuntu2004.pin
$ sudo mv cuda-ubuntu2004.pin /etc/apt/preferences.d/cuda-repository-pin-600
$ wget https://developer.download.nvidia.com/compute/cuda/11.4.1/local_installers/cuda-repo-ubuntu2004-11-4-local_11.4.1-470.57.02-1_amd64.deb
$ sudo dpkg -i cuda-repo-ubuntu2004-11-4-local_11.4.1-470.57.02-1_amd64.deb
$ sudo apt-key add /var/cuda-repo-ubuntu2004-11-4-local/7fa2af80.pub
$ sudo apt-get update
$ sudo apt-get -y install cuda
$ reboot
$ nvidia-smi
Wed Aug  4 14:19:31 2021       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 470.57.02    Driver Version: 470.57.02    CUDA Version: 11.4     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  Quadro P1000        On   | 00000000:04:00.0 Off |                  N/A |
| 34%   39C    P8    N/A /  N/A |     11MiB /  4040MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                             
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|    0   N/A  N/A      1065      G   /usr/lib/xorg/Xorg                  4MiB |
|    0   N/A  N/A      1657      G   /usr/lib/xorg/Xorg                  4MiB |
+-----------------------------------------------------------------------------+

$ sudo apt install nvidia-gds
$ sudo modprobe nvidia_fs
$ dpkg -s nvidia-gds
$ /usr/local/cuda/gds/tools/gdscheck -p
 GDS release version: 1.0.1.3
 nvidia_fs version:  2.7 libcufile version: 2.4
 ============
 ENVIRONMENT:
 ============
 =====================
 DRIVER CONFIGURATION:
 =====================
 NVMe               : Supported
 NVMeOF             : Unsupported
 SCSI               : Unsupported
 ScaleFlux CSD      : Unsupported
 NVMesh             : Unsupported
 DDN EXAScaler      : Unsupported
 IBM Spectrum Scale : Unsupported
 NFS                : Unsupported
 WekaFS             : Unsupported
 Userspace RDMA     : Unsupported
 --Mellanox PeerDirect : Enabled
 --rdma library        : Not Loaded (libcufile_rdma.so)
 --rdma devices        : Not configured
 --rdma_device_status  : Up: 0 Down: 0
 =====================
 CUFILE CONFIGURATION:
 =====================
 properties.use_compat_mode : true
 properties.gds_rdma_write_support : true
 properties.use_poll_mode : false
 properties.poll_mode_max_size_kb : 4
 properties.max_batch_io_timeout_msecs : 5
 properties.max_direct_io_size_kb : 16384
 properties.max_device_cache_size_kb : 131072
 properties.max_device_pinned_mem_size_kb : 33554432
 properties.posix_pool_slab_size_kb : 4 1024 16384 
 properties.posix_pool_slab_count : 128 64 32 
 properties.rdma_peer_affinity_policy : RoundRobin
 properties.rdma_dynamic_routing : 0
 fs.generic.posix_unaligned_writes : false
 fs.lustre.posix_gds_min_kb: 0
 fs.weka.rdma_write_support: false
 profile.nvtx : false
 profile.cufile_stats : 0
 miscellaneous.api_check_aggressive : false
 =========
 GPU INFO:
 =========
 GPU index 0 Quadro P1000 bar:1 bar size (MiB):256 supports GDS
 ==============
 PLATFORM INFO:
 ==============
 IOMMU: disabled
 Platform verification succeeded
```
