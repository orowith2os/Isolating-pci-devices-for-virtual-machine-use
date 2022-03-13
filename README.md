This can differ from BIOS to BIOS, but make sure it has IOMMU enabled. Also enable virtualization in your BIOS, it may be named SVM or AMD-Vi.

Requirements for isolating the GPU/any PCIe device:
IOMMU support in BIOS
A second GPU to use for the host OS, if you are passing through a GPU. This guide attaches the GPU to the `vfio-pci` module, so it will not be available for as long as the vfio kernel is running.

Install the `linux-vfio` kernel, or a Linux kernel with the ACS override patch. This may vary from system to system, and is required for the PCIe passthrough to work if your device does not have its own IOMMU group.

This will bind your GPU to the vfio-pci driver on bootup:

   1. Execute `lspci -nn` to find your device IDs. They should be surrounded by square brackets near the end, before (rev X).
    Edit your GRUB to contain `iommu=pt pcie_acs_override=downstream,multifunction, and vfio-pci.ids=id`. If you pass through mmore than one device, separate them with a comma, so it should look like `vfio-pci.ids=id`. This will bind your PCI device(s) of choice to the linux-vfio kernel driver.
   2. Regenerate your GRUB using `sudo grub-mkconfig -o /boot/grub/grub.cfg`
   Your grub configuration file should look like something along these lines: `GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet amd_iommu=on iommu=pt pcie_acs_override=downstream,multifunction vfio-pci.ids=id(,id)"`
   3. Regenerate your GRUB configuration file using `sudo grub-mkconfig -o /boot/grub/grub.cfg`
   Edit your mkinitcpio file, /etc/mkinitcpio.conf: add 'vfio_pci vfio vfio_iommu_type1 vfio_virqfd' to the MODULES= like, so it looks somewhat like this: `MODULES=(... vfio_pci vfio vfio_iommu_type1 vfio_virqfd ...)`.
   4. Also edit the hooks line to contain 'modconf' like so: `HOOKS=(... modconf ...)`
   Regenerate your mkinitcpio file: `sudo mkinitcpio -P`.

Reboot, and your PCI devices should be isolated and everything in its own IOMMU group. Running `lspci -nnv` should show the beginning numbers each go in order from the top down, e.g. the first number set is 00, the next is 01, next down is 02, so on, so on. Under `Kernel driver in use:` in your output, it should show `vfio-pci`. If it 

This is optional, but I will edit my grub to let it use the last selected kernel as the default on next system restart. 

   1. Edit the options GRUB_DEFAULT= and GRUB_SAVEDEFAULT= to GRUB_DEFAULT=saved and GRUB_SAVEDEFAULT=true.
   2. Again, optional, but you can add GRUB_DISABLE_SUBMENU=y to disable submenus. If you don't want this option, disable or remove it.
   3. Regenerate your GRUB: `sudo grub-mkconfig -o /boot/grub/grub.cfg`
