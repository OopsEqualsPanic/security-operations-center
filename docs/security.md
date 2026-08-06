# Securing the Hypervisor

Since Secureblue comes with a plethora of different options for securing the system further, we will be utilizing them. 

## 1) The Kernel

The following is a list of hardened kernel arguments we will be adding to the system:
| Argument | Description |
|----------|-------------|
| ``ia32_emulation=0`` | Disables support of 32-bit applications, which we will not be using anyway. |
| ``nosmt=force`` | Force-disables simultaneous multi-threading by halving the number of CPU cores, which could prevent some side-channel attacks and information leaks. Not recommended to add. |
| ``amd_iommu=force_isolation`` | Forbids the IOMMU (Input/Output Memory Management Unit) driver from lifting isolation requirements for devices, preventing Direct Memory Access attacks. May cause issues on some hardware. |
| ``bdev_allow_write_mounted=0`` | Disables writes to mounted block devices to prevent filesystem corruption/crashes. |
| ``debugfs=off`` | Disables debugfs to prevent exposing kernel information to users, which would aid attackers. |
| ``efi=disable_early_pci_dma`` | Prevents malicious PCI devices from performing Direct Memory Access attacks during the boot process, and ensures that the busmaster bit (control bit for Direct Memory Access operations) is cleared on all PCI bridges before the operating system takes control. |
| ``gather_data_sampling=force`` | Enables microcode mitigation against the Gather Data Sampling vulnerability, which can allow attackers to infer sensitive data from vector registers. |
| ``mem_encrypt=on`` | Activates Secure Memory Encryption, which encrypts individual pages of memory to protect against physical attacks. |
| ``oops=panic`` | Causes the kernel to panic instead of continuing to run when encountering certain errors, preventing attackers from exploiting a compromised kernel. |

To enable all of these, Secureblue provides a convenient setup script: ``ujust set-kargs-hardening``.

```
$ ujust set-kargs-hardening

Do you need support for 32-bit processes/syscalls? (This is mostly
used by legacy software, with some exceptions, such as Steam.) [y/n] n
Selected: disable 32-bit support.

Do you want to force disable Simultaneous Multithreading (SMT) /
Hyperthreading? (This can cause a reduction in the performance of
certain tasks in favor of security. Note that in most hardware SMT
will be disabled anyways to mitigate a known vulnerability; this turns
it off on all hardware regardless.) [y/n] n
Selected: do not force disable SMT/hyperthreading.

Would you like to set additional (unstable) hardening kernel
arguments? (Warning: Setting these kernel arguments may lead to boot
or stability issues on some hardware.) [y/n] y
```

2) Securing the Bash Environment

To give you a bit of context, LD_PRELOAD is an environment variable that allows one to load shared libraries before others, enabling you to override default functions in programs.

When this environment variable is hijacked to load, say, an attacker-controlled library, it modifies the default behavior of normal programs.

For example, an attacker could preload a custom library that modifies the output of the ls command to hide malware from appearing in its output. We don't want this.

We can lock down the bash environment to prevent LD_PRELOAD attacks by executing the following command: 
```
ujust toggle-bash-environment-lockdown
```
What this script does is fetch the current list of users on the system using UIDs from /etc/login.defs and stores them in uid_min and uid_max. Usernames are then taken via user_string and each user's home directory has its bash environment locked down.

It's also important to mention that an attacker can also gain root and modify the /etc/ld.so.preload file, and preload their own libraries that persist even when you lock your bash environment down.

It's imperative that you ensure the security of your system from the top going down. If disabling root is within reach, use it. Likewise, disabling or replacing privilege escalation vectors is also recommended, such as replacing set-UID root binaries (binaries that always run as root) with non-root alternatives.
