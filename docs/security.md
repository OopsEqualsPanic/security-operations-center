# Securing the Hypervisor

Since Secureblue comes with a plethora of different options for securing the system further, we will be utilizing them. 

## 1) The Kernel

The following is a list of hardened kernel arguments we will be adding to the system:
| Argument | Description |
|----------|-------------|
| ``ia32_emulation=0`` | Disables support of 32-bit applications, which we will not be using anyway. |
| ``nosmt=force`` | Force-disables simultaneous multi-threading by halving the CPU. Discretion advised. |
| ``amd_iommu=force_isolation`` | Forbids the IOMMU (Input/Output Memory Management Unit) driver from lifting isolation requirements for devices, preventing Direct Memory Access attacks. May cause issues on some hardware. |
| ``bdev_allow_write_mounted=0`` | Disables writes to mounted block devices to prevent filesystem corruption/crashes. |
| ``debugfs=off`` | Disables debugfs to prevent exposing kernel information to users, which would aid attackers. |
| ``efi=disable_early_pci_dma`` | Prevents malicious PCI devices from performing Direct Memory Access attacks during the boot process, and ensures that the busmaster bit (control bit for Direct Memory Access operations) is cleared on all PCI bridges before the operating system takes control. |
| ``gather_data_sampling=force`` | Enables microcode mitigation against the Gather Data Sampling vulnerability, which can allow attackers to infer sensitive data from vector registers. |
| ``mem_encrypt=on`` | Activates Secure Memory Encryption, which encrypts individual pages of memory to protect against physical attacks. |
| ``oops=panic`` | Causes the kernel to panic instead of continuing to run when encountering certain errors, preventing attackers from exploiting a compromised kernel. |
