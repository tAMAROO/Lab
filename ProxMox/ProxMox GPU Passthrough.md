# ProxMox Setup for PCIe Passthrough

## Guide followed Unchanged:
1. **Set the proper boot parameters for your setup:**
 - **For Legacy Systems - Add IOMMU Support:**

	- `nano /etc/default/grub`
   Add the following lines to the file:
	 For Intel CPU: GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on"
	 For AMD CPU: GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on"

	- **Save file and close**

	- **update-grub**


 - **For EFI Boot Systems - Add IOMMU Support:**

	- `nano /etc/kernel/cmdline`
    For Intel CPU: intel_iommu=on
    For AMD CPU: amd_iommu=on

	- **Save file and close**

 
 - **`proxmox-boot-tool refresh`**


2. **Load VFIO modules at boot:**

 - `nano /etc/modules`
  Add the following lines to the file:
	`vfio
	vfio_iommu_type1
	vfio_pci
	vfio_virqfd`

 - **Save file and close**


3. **Blacklist graphic drivers (optional):**

 - `echo "options vfio_iommu_type1 allow_unsafe_interrupts=1" > /etc/modprobe.d/iommu_unsafe_interrupts.conf`
 - `echo "options kvm ignore_msrs=1" > /etc/modprobe.d/kvm.conf`
 
 - `echo "blacklist nouveau" >> /etc/modprobe.d/blacklist.conf`
 - `echo "blacklist nvidia" >> /etc/modprobe.d/blacklist.conf`

4. **Configure GPU for PCIe Passthrough:**

	- **Find your GPU:**
	  `lspci`

	- **Enter the PCI identifier:**
	  `lspci -n -s 82:00 -v`

	- **Copy the HEX values from your GPU here:**
	  `echo "options vfio-pci ids=####.####,####.#### disable_vga=1"> /etc/modprobe.d/vfio.conf`


5. **Apply all changes:**

	- `update-initramfs -u -k all`

6. **Reboot the system**
---

## Guide updated to how I made it work:

1. **Set the proper boot parameters for your setup:**
 - **For Legacy Systems - Add IOMMU Support:**

	- `nano /etc/default/grub`
  **Added this parameter:**
  For Intel CPU: GRUB_CMDLINE_LINUX="vfio-pci.ids=####,####"
  
  >Example config:
  GRUB_DEFAULT=0
	GRUB_TIMEOUT=5
	GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
	GRUB_CMDLINE_LINUX_DEFAULT="quiet"
	GRUB_CMDLINE_LINUX="vfio-pci.ids=####,####"
  
>   Should probably look into the guide vs what I did since I might have put the right thing in the wrong place
{.is-danger}

  

2. **Load VFIO modules at boot:**

 - `nano /etc/modules`
  Add the following lines to the file:
	`vfio
	vfio_iommu_type1
	vfio_pci
	vfio_virqfd`
 
 - `nano /etc/initramfs-tools/modules`
  Add the following lines to the file:
	`vfio
	vfio_iommu_type1
	vfio_pci
	vfio_virqfd`
  
 - **Save file and close**
 
3. **Edit the /etc/modeprobe.d/vfio.conf config file:**
  Add the following lines to the file:
	`options vfio-pci ids=1028:6981`


3. **Blacklist graphic drivers (optional):**

 - `echo "options vfio_iommu_type1 allow_unsafe_interrupts=1" > /etc/modprobe.d/iommu_unsafe_interrupts.conf`
 - `echo "options kvm ignore_msrs=1" > /etc/modprobe.d/kvm.conf`
 
 - `echo "blacklist nouveau" >> /etc/modprobe.d/blacklist.conf`
 - `echo "blacklist nvidia" >> /etc/modprobe.d/blacklist.conf`
 - `echo "blacklist amdgpu" >> /etc/modprobe.d/blacklist.conf`
 
4. **Configure GPU for PCIe Passthrough:**

	- **Find your GPU:**
	  `lspci`

	- **Enter the PCI identifier:**
	  `lspci -n -s 82:00 -v`

	- **Copy the HEX values from your GPU here:**
	  `echo "options vfio-pci ids=####.####,####.#### disable_vga=1"> /etc/modprobe.d/vfio.conf`
>Was having issues with this so I took out `disable_vga=1` and it worked fine see step 3.
{.is-info}

5. **Apply all changes:**

	- `update-initramfs -u -k all`

6. **Reboot the system**


## Issues:


1. **Issue where when running `lspci -n -s #### -v` the kernel was still showing using the amdgpu module instead of the vfio module.**

 - **To switch a GPU's kernel module from amdgpu to vfio:**


 - 1. **Verify vfio module:**
    First, make sure the vfio module is built and available. You can do this by running sudo modprobe vfio. If it returns with no error, the module exists. 
 - 2. **Blacklist amdgpu (optional):**
		To prevent amdgpu from loading, you can blacklist it in /etc/modprobe.d/. Create a file (e.g., /etc/modprobe.d/blacklist-amdgpu.conf) and add blacklist amdgpu. 
 - 3. **Soft dependancies:**
		You can use soft dependencies in /etc/modprobe.d/ to ensure vfio loads before amdgpu. For example, in a file like /etc/modprobe.d/vfio-pci.conf, you might add options vfio-pci 					ids=1002:xxxx softdep amdgpu pre: vfio-pci. Replace xxxx with the specific PCI ID of your AMD GPU. 
 - 4. **Kernel command-line parameters:**
		You may need to add kernel parameters to force vfio to load and bind to the GPU. This often involves specifying the PCI ID of your GPU using options like vfio-pci.ids=1002:xxxx. How 		to 	add this depends on your bootloader (GRUB, systemd-boot, etc.). 
 - 5. **Rebuild initramfs:**
		If you've made changes to module configurations, you may need to rebuild the initramfs (initial RAM disk) to ensure the changes are picked up during boot. 
 - 6. **Reboot:**
		After making the necessary changes, reboot your system to apply the configuration. 

---

2. **Issue where when trying to run `update-initramfs -u -k all` would not update.  Found this to fix:**


 - 1. `proxmox-boot-tool status`

 - 2. **To check which partition is /boot with vfat format:**
		`lsblk -o +FSTYPE`

 - 3. **To initialize ESP sync first unmount boot partition:**
		`umount /boot/efi`

 - 4. **Then link the vfat partiton with proxmox-boot-tool:**
		`proxmox-boot-tool init /dev/XXXXXXXX where XXXXXXXX is the name of vfat partiton from lsblk +FSYSTEM`

 - 5. **Then mount:**
		`mount -a"

 - 6. **Then to update modules:**
		`update-initramfs -u -k all`
    
 - 7. Reboot
