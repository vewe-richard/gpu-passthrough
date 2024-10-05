
# Do experiment in my ubuntu laptop

### Check GPU driver in host (my laptop)
```azure
richard@richard-NH50-70RA:/usr/local/go/src/net$ lspci | grep -i nvidia     
01:00.0 VGA compatible controller: NVIDIA Corporation Device 1f91 (rev a1)  
01:00.1 Audio device: NVIDIA Corporation Device 10fa (rev a1)               
richard@richard-NH50-70RA:/usr/local/go/src/net$ lspci -s 01:00.0 -v        
01:00.0 VGA compatible controller: NVIDIA Corporation Device 1f91 (rev a1) (
])                                                                          
        Subsystem: CLEVO/KAPOK Computer Device 8560                         
        Flags: bus master, fast devsel, latency 0, IRQ 255                  
        Memory at a3000000 (32-bit, non-prefetchable) [size=16M]            
        Memory at 90000000 (64-bit, prefetchable) [size=256M]               
        Memory at a0000000 (64-bit, prefetchable) [size=32M]                
        I/O ports at 4000 [disabled] [size=128]                             
        Expansion ROM at a4080000 [disabled] [size=512K]                    
        Capabilities: <access denied>                                       
        Kernel modules: nvidiafb, nouveau
```

### Start a VM

##### Start a VM in qemu with kernel message to terminal
```azure
sudo qemu-system-x86_64 -enable-kvm -nographic -m 1024 -smp 2 -kernel /home/richard/work/knet/linux-stable/arch/x86/boot/bzImage -append "root=/dev/
sda5 rw console=ttyS0" -drive file=/home/richard/work/2022/france/docker/share/data/images/image-xdp.img,format=raw -netdev user,id=net0001,hostfwd=tcp::10001-:22 -device e1000,netdev=net0001,mac=52:54:98
:76:00:01 

```

We should add below  to `/etc/modprobe.d/blacklist.conf` 
```azure
blacklist nouveau
blacklist nvidia
```

Unbind from current driver
```azure
echo "0000:01:00.0" | sudo tee /sys/bus/pci/devices/0000:01:00.0/driver/unbind
echo "0000:01:00.1" | sudo tee /sys/bus/pci/devices/0000:01:00.1/driver/unbind
```

Bind to vfio
```azure
sudo modprobe vfio-pci
echo "vfio-pci" | sudo tee /sys/bus/pci/devices/0000:01:00.0/driver_override
echo "vfio-pci" | sudo tee /sys/bus/pci/devices/0000:01:00.1/driver_override
echo "0000:01:00.0" | sudo tee -a /sys/bus/pci/drivers/vfio-pci/bind
echo "0000:01:00.1" | sudo tee -a /sys/bus/pci/drivers/vfio-pci/bind
```

Check drivers are vfio-pci
```azure
richard@richard-NH50-70RA:/usr/local/go/src/net$ ls /sys/bus/pci/devices/0000:01:00.0/driver -l
lrwxrwxrwx 1 root root 0 10月  5 16:07 /sys/bus/pci/devices/0000:01:00.0/driver -> ../../../../bus/pci/drivers/vfio-pci
richard@richard-NH50-70RA:/usr/local/go/src/net$ ls /sys/bus/pci/devices/0000:01:00.1/driver -l
lrwxrwxrwx 1 root root 0 10月  5 16:08 /sys/bus/pci/devices/0000:01:00.1/driver -> ../../../../bus/pci/drivers/vfio-pci
```

Pass to qemu command line
```azure
-device vfio-pci,host=01:00.0 \
-device vfio-pci,host=01:00.1
```

Then we will see pci is passthrough from vm's lspci

Another important thing to be sure if still got a problem,
```azure
Verify IOMMU is Enabled:

Ensure that IOMMU is enabled in your BIOS/UEFI and on your kernel command line. 
You can add intel_iommu=on or amd_iommu=on to your GRUB configuration. 
Edit /etc/default/grub and update the GRUB_CMDLINE_LINUX_DEFAULT:
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash intel_iommu=on"
Then update GRUB:
sudo update-grub
```











