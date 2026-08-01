# 1) Installing Fedora IoT for Virtualization

First I needed to download the Fedora IoT .iso file from their sources:
https://fedoraproject.org/iot/download/

![Screenshot](images/fedora_iot.png)

I went with Fedora IoT (OSTree) under the AMD x86_64 section since that is my hardware.
After verifying the .iso, I proceeded to installation.

# 2) Rebasing to Secureblue

Secureblue is a set of security-focused patches and hardening that can either be applied to base Fedora images or installed as a standalone desktop operating system.
For my installation, the instructions advised to rebase from Fedora IoT to Secureblue's images.

![Screenshot](images/secureblue_images.png)

I went with ``iot-main-hardened`` since I did not have nvidia hardware, and ran this command to rebase to Secureblue:

```
$ sudo bootc switch ghcr.io/secureblue/iot-main-hardened:latest
```

After installing, its good to check the Supply-chain Levels for Software Artifacts (SLSA) provenance so you know your operating system image is not tampered with. 
I did that with ``ujust update-system``, a script bundled with Secureblue among other convenient setup scripts.

SLSA verification passed, so I continued.

# 3) Installing Cockpit
To enable cockpit, you can simply follow the instructions at https://cockpit-project.org/running.html#coreos

Since our system is using rpm-ostree instead of dnf, we will need to install cockpit's rpm packages.
```
$ rpm-ostree install cockpit-system cockpit-podman cockpit-ostree
```
After installing, the system requires a reboot.

Before enabling cockpit and starting it in podman, we need to enable ssh password authentication.

Since this is Secureblue and not regular Fedora IoT, sudo has been uninstalled in favor of run0, a more secure version.
```
$ echo 'PasswordAuthentication yes' | run0 tee /etc/ssh/sshd_config.d/02-enable-passwords.conf
$ run0 systemctl try-restart sshd
```

Before running the next command, podman needs to trust the podman image being offered, otherwise it will fail to install.
```
podman image trust set -t accept quay.io/cockpit/ws
```

Now running the command works (run as root):
```
run0 podman container runlabel --name cockpit-ws RUN quay.io/cockpit/ws
```

Then we can make it start on boot:
```
podman container runlabel INSTALL quay.io/cockpit/ws
systemctl enable cockpit.service
systemctl start cockpit.service
```

After it starts, you may not be able to access cockpit at first due to firewalld blocking the connection. Simply run:
```
firewall-cmd --add-service=cockpit --permanent
firewall-cmd --reload
firewall-cmd --list-services
```

If you want to connect using a hostname instead of the system's current internal IP address (192.168.x.x), using the avahi-daemon is not recommended.
This is because mDNS is insecure by design, allowing any and all devices on the current network to announce any name or service. 

Give the system a static IP address on its local area network, then modify the /etc/hosts file to include a hostname with the IP.


# 2) Configuring Virtual Machines

We're going to create 3 total virtual machines for this setup using the Proxmox web interface. 
| VM | Operating System | Identity |
|---------|--------------|------------|
| VM 1 | Secureblue Server (based on Fedora Atomic) | SIEM Host |
| VM 2 | Windows 11 | Endpoint |
| VM 3 | Windows 11 | Endpoint |
