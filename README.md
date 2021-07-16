# 1. Install Ubuntu-20.4 on Host Machine

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

# 4. Edit /etc/default/grub at Host Machine
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
GRUB_CMDLINE_LINUX="quiet splash intel_iommu=on vfio_iommu_type1.allow_unsafe_interrupts=1 iommu=pt vfio-pci.ids=10de:1cb1,10de:0fb9,144d:a804"
```
# 5. Update grub and Reboot at Host Machine
```
sudo update-grub
sudo reboot
```
# 6. Check VFIO at Host Machine
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
You should add the GPU and NVMe. See also attached png files.
```

# 8. You might add kernel 5.4.0-42
See also https://kazuhira-r.hatenablog.com/entry/2020/02/28/000625
```
$ sudo apt-get install linux-image-5.4.0-42-generic linux-headers-5.4.0-42-generic linux-modules-extra-5.4.0-42-generic
$ sudo /etc/default/grub
  You should edit followings, please note the GRUB_TIMEOUT_STYLE and GRUB_TIMEOUT parameters are comment out:
```
```
#GRUB_TIMEOUT_STYLE=hidden
#GRUB_TIMEOUT=0
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash intel_iommu=off"
```
```
$ sudo /etc/update-grub
$ reboot

After this step, You will be able to select kernel 5.4.0-42.
```

# 9. Install MOFED5.3 at Guest Machine
```
 Download MLNX_OFED_LINUX-5.3-1.0.0.1-ubuntu20.04-x86_64.tgz.
   $ sudo apt-get install python3-distutils
   $ cd MLNX_OFED_LINUX-5.3-1.0.0.1-ubuntu20.04-x86_64/
   $ sudo ./mlnxofedinstall --with-nfsrdma --with-nvmf --enable-gds --add-kernel-support
   $ sudo update-initramfs -u -k `uname -r`
   $ sudo reboot
```

# 10. Install Ubuntu on Guest Machine and check the result lspci at Guest Machine
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

