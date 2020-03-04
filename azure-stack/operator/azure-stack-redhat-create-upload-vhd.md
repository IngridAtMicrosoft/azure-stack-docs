---
title: Prepare a Red Hat-based virtual machine for Azure Stack Hub 
titleSuffix: Azure Stack Hub
description: Learn to create and upload an Azure virtual hard disk (VHD) that contains a Red Hat Linux operating system.
author: sethmanheim
ms.topic: article
ms.date: 12/11/2019
ms.author: sethm
ms.reviewer: kivenkat
ms.lastreviewed: 12/11/2019

# Intent: As an Azure Stack operator, I want to prepare a red hat-based virtual machine for Azure Stack.
# Keyword: red hat operating system azure stack

---


# Prepare a Red Hat-based virtual machine for Azure Stack Hub

In this article, you'll learn how to prepare a Red Hat Enterprise Linux (RHEL) virtual machine (VM) for use in Azure Stack Hub. The versions of RHEL that are covered in this article are 7.1+. The hypervisors for preparation that are covered in this article are Hyper-V, kernel-based virtual machine (KVM), and VMware.

For Red Hat Enterprise Linux support information, see [Red Hat and Azure Stack Hub: Frequently Asked Questions](https://access.redhat.com/articles/3413531).

## Prepare a Red Hat-based VM from Hyper-V Manager

This section assumes that you already have an ISO file from the Red Hat website and have installed the RHEL image to a virtual hard disk (VHD). For more information about how to use Hyper-V Manager to install an operating system image, see [Install the Hyper-V Role and Configure a VM](https://technet.microsoft.com/library/hh846766.aspx).

### RHEL installation notes

* Azure Stack Hub doesn't support the VHDX format. Azure supports only fixed VHD. You can use Hyper-V Manager to convert the disk to VHD format, or you can use the convert-vhd cmdlet. If you use VirtualBox, select **Fixed size** as opposed to the default dynamically allocated option when you create the disk.
* Azure Stack Hub supports only generation 1 VMs. You can convert a generation 1 VM from VHDX to the VHD file format and from dynamically expanding to a fixed-size disk. You can't change a VM's generation. For more information, see [Should I create a generation 1 or 2 VM in Hyper-V?](https://technet.microsoft.com/windows-server-docs/compute/hyper-v/plan/should-i-create-a-generation-1-or-2-virtual-machine-in-hyper-v).
* The maximum size that's allowed for the VHD is 1,023 GB.
* When you install the Linux operating system, we recommend that you use standard partitions rather than Logical Volume Manager (LVM), which is often the default for many installations. This practice avoids LVM name conflicts with cloned VMs, particularly if you ever need to attach an operating system disk to another identical VM for troubleshooting.
* Kernel support for mounting Universal Disk Format (UDF) file systems is required. At first boot, the UDF-formatted media that's attached to the guest passes the provisioning configuration to the Linux VM. The Azure Linux Agent must mount the UDF file system to read its configuration and provision the VM.
* Don't configure a swap partition on the operating system disk. The Linux Agent can be configured to create a swap file on the temporary resource disk. More information about can be found in the following steps.
* All VHDs on Azure must have a virtual size aligned to 1 MB. When converting from a raw disk to VHD, you must ensure that the raw disk size is a multiple of 1 MB before conversion. More details can be found in the steps below.
* Azure Stack Hub supports cloud-init. [Cloud-init](https://docs.microsoft.com/azure/virtual-machines/linux/using-cloud-init) is a widely used approach to customize a Linux VM as it boots for the first time. You can use cloud-init to install packages and write files, or to configure users and security. Because cloud-init is called during the initial boot process, there are no additional steps or required agents to apply your configuration. For instructions on adding cloud-init to your image, see [Prepare an existing Linux Azure VM image for use with cloud-init](https://docs.microsoft.com/azure/virtual-machines/linux/cloudinit-prepare-custom-image).

### Prepare an RHEL 7 VM from Hyper-V Manager

1. In Hyper-V Manager, select the VM.

1. Select **Connect** to open a console window for the VM.

1. Create or edit the `/etc/sysconfig/network` file, and add the following text:

    ```sh
    NETWORKING=yes
    HOSTNAME=localhost.localdomain
    ```

1. Create or edit the `/etc/sysconfig/network-scripts/ifcfg-eth0` file, and add the following text as needed:

    ```sh
    DEVICE=eth0
    ONBOOT=yes
    BOOTPROTO=dhcp
    TYPE=Ethernet
    USERCTL=no
    PEERDNS=yes
    IPV6INIT=no
    NM_CONTROLLED=no
    ```

1. Ensure that the network service starts at boot time by running the following command:

    ```bash
    sudo systemctl enable network
    ```

1. Register your Red Hat subscription to enable the installation of packages from the RHEL repository by running the following command:

    ```bash
    sudo subscription-manager register --auto-attach --username=XXX --password=XXX
    ```

1. Modify the kernel boot line in your grub configuration to include additional kernel parameters for Azure. To make this modification, open `/etc/default/grub` in a text editor, and modify the `GRUB_CMDLINE_LINUX` parameter. For example:

    ```sh
    GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0"
    ```

   This modification ensures all console messages are sent to the first serial port, which can assist Azure support with debugging issues. This configuration also turns off the new RHEL 7 naming conventions for NICs.

   Graphical and quiet boot aren't useful in a cloud environment where we want all the logs to be sent to the serial port. You can leave the `crashkernel` option configured if desired. This parameter reduces the amount of available memory in the VM by 128 MB or more, which might be problematic on smaller VM sizes. We recommend that you remove the following parameters:

    ```sh
    rhgb quiet crashkernel=auto
    ```

1. After you're done editing `/etc/default/grub`, run the following command to rebuild the grub configuration:

    ```bash
    sudo grub2-mkconfig -o /boot/grub2/grub.cfg
    ```

1. [Optional after 1910 release] Stop and Uninstall cloud-init :

    ```bash
    systemctl stop cloud-init
    yum remove cloud-init
    ```

1. Ensure that the SSH server is installed and configured to start at boot time, which is usually the default. Modify `/etc/ssh/sshd_config` to include the following line:

    ```sh
    ClientAliveInterval 180
    ```

1. When creating a custom vhd for Azure Stack Hub, keep in mind that WALinuxAgent version between 2.2.20 and 2.2.35 (both exclusive) don't work on Azure Stack Hub environments before the 1910 release. You can use versions 2.2.20/2.2.35 versions to prepare your image. To use versions above 2.2.35 to prepare your custom image, update your Azure Stack Hub to 1903 release and above or apply the 1901/1902 hotfix.
    
    [Before 1910 release] Follow these instructions to download a compatible WALinuxAgent:
    
    1. Download setuptools.
        
        ```bash
        wget https://pypi.python.org/packages/source/s/setuptools/setuptools-7.0.tar.gz --no-check-certificate
        tar xzf setuptools-7.0.tar.gz
        cd setuptools-7.0
        ```
    
    1. Download and unzip the 2.2.20 version of the agent from our GitHub.

        ```bash
        wget https://github.com/Azure/WALinuxAgent/archive/v2.2.20.zip
        unzip v2.2.20.zip
        cd WALinuxAgent-2.2.20
        ```

    1. Install setup.py.

        ```bash
        sudo python setup.py install
        ```

    1. Restart waagent.
    
        ```bash
        sudo systemctl restart waagent
        ```

    1. Test if the agent version matches the one you downloaded. For this example, it should be 2.2.20.

        ```bash
        waagent -version
        ```

    [After 1910 release] Follow these instructions to download a compatible WALinuxAgent:
    
    1. The WALinuxAgent package, `WALinuxAgent-<version>`, has been pushed to the Red Hat extras repository. Enable the extras repository by running the following command:

        ```bash
        subscription-manager repos --enable=rhel-7-server-extras-rpms
        ```

    1. Install the Azure Linux Agent by running the following command:

        ```bash
        sudo yum install WALinuxAgent
        sudo systemctl enable waagent.service
        ```
    

1. Don't create swap space on the operating system disk.

    The Azure Linux Agent can automatically configure swap space by using the local resource disk that's attached to the VM after the VM is provisioned on Azure. The local resource disk is a temporary disk, and it might be emptied when the VM is deprovisioned. After you install the Azure Linux Agent in the previous step, modify the following parameters in `/etc/waagent.conf` appropriately:

    ```sh
    ResourceDisk.Format=y
    ResourceDisk.Filesystem=ext4
    ResourceDisk.MountPoint=/mnt/resource
    ResourceDisk.EnableSwap=y
    ResourceDisk.SwapSizeMB=2048    #NOTE: set this to whatever you need it to be.
    ```

1. If you want to unregister the subscription, run the following command:

    ```bash
    sudo subscription-manager unregister
    ```

1. If you're using a system that was deployed using an Enterprise Certificate Authority, the RHEL VM won't trust the Azure Stack Hub root certificate. You need to place that into the trusted root store. For more information, see [Adding trusted root certificates to the server](https://manuals.gfi.com/en/kerio/connect/content/server-configuration/ssl-certificates/adding-trusted-root-certificates-to-the-server-1605.html).

1. Run the following commands to deprovision the VM and prepare it for provisioning on Azure:

    ```bash
    sudo waagent -force -deprovision
    export HISTSIZE=0
    logout
    ```

1. Select **Action** > **Shut Down** in Hyper-V Manager.

1. Convert the VHD to a fixed size VHD using either the Hyper-V Manager "Edit disk" feature, or the Convert-VHD PowerShell command. Your Linux VHD is now ready to be uploaded to Azure.

## Prepare a Red Hat-based virtual machine from KVM

1. Download the KVM image of RHEL 7 from the Red Hat website. This procedure uses RHEL 7 as the example.

1. Set a root password.

    Generate an encrypted password, and copy the output of the command:

    ```bash
    openssl passwd -1 changeme
    ```

   Set a root password with guestfish:

    ```sh
    guestfish --rw -a <image-name>
    > <fs> run
    > <fs> list-filesystems
    > <fs> mount /dev/sda1 /
    > <fs> vi /etc/shadow
    > <fs> exit
    ```

   Change the second field of root user from "!!" to the encrypted password.

1. Create a VM in KVM from the qcow2 image. Set the disk type to **qcow2**, and set the virtual network interface device model to **virtio**. Then, start the VM, and sign in as root.

1. Create or edit the `/etc/sysconfig/network` file, and add the following text:

    ```sh
    NETWORKING=yes
    HOSTNAME=localhost.localdomain
    ```

1. Create or edit the `/etc/sysconfig/network-scripts/ifcfg-eth0` file, and add the following text:

    ```sh
    DEVICE=eth0
    ONBOOT=yes
    BOOTPROTO=dhcp
    TYPE=Ethernet
    USERCTL=no
    PEERDNS=yes
    IPV6INIT=no
    NM_CONTROLLED=no
    ```

1. Ensure that the network service starts at boot time by running the following command:

    ```bash
    sudo systemctl enable network
    ```

1. Register your Red Hat subscription to enable installation of packages from the RHEL repository by running the following command:

    ```bash
    subscription-manager register --auto-attach --username=XXX --password=XXX
    ```

1. Modify the kernel boot line in your grub configuration to include additional kernel parameters for Azure. To do this configuration, open `/etc/default/grub` in a text editor, and modify  the `GRUB_CMDLINE_LINUX` parameter. For example:

    ```sh
    GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0"
    ```

   This command also ensures that all console messages are sent to the first serial port, which can assist Azure support with debugging issues. The command also turns off the new RHEL 7 naming conventions for NICs.

   Graphical and quiet boot aren't useful in a cloud environment where all the logs are sent to the serial port. You can leave the `crashkernel` option configured if desired. This parameter reduces the amount of available memory in the VM by 128 MB or more, which might be problematic on smaller VM sizes. We recommend you remove the following parameters:

    ```sh
    rhgb quiet crashkernel=auto
    ```

1. After you're done editing `/etc/default/grub`, run the following command to rebuild the grub configuration:

    ```bash
    grub2-mkconfig -o /boot/grub2/grub.cfg
    ```

1. Add Hyper-V modules into initramfs.

    Edit `/etc/dracut.conf` and add content:

    ```sh
    add_drivers+="hv_vmbus hv_netvsc hv_storvsc"
    ```

    Rebuild initramfs:

    ```bash
    dracut -f -v
    ```

1. [Optional after 1910 release] Stop and Uninstall cloud-init:

    ```bash
    systemctl stop cloud-init
    yum remove cloud-init
    ```

1. Ensure that the SSH server is installed and configured to start at boot time:

    ```bash
    systemctl enable sshd
    ```

    Modify /etc/ssh/sshd_config to include the following lines:

    ```sh
    PasswordAuthentication yes
    ClientAliveInterval 180
    ```

1. When creating a custom vhd for Azure Stack Hub, keep in mind that WALinuxAgent version between 2.2.20 and 2.2.35 (both exclusive) don't work on Azure Stack Hub environments before the 1910 release. You can use versions 2.2.20/2.2.35 versions to prepare your image. To use versions above 2.2.35 to prepare your custom image, update your Azure Stack Hub to 1903 release and above or apply the 1901/1902 hotfix.

    [Before 1910 release] Follow these instructions to download a compatible WALinuxAgent:

    1. Download setuptools.
        
        ```bash
        wget https://pypi.python.org/packages/source/s/setuptools/setuptools-7.0.tar.gz --no-check-certificate
        tar xzf setuptools-7.0.tar.gz
        cd setuptools-7.0
        ```
        
    1. Download and unzip the 2.2.20 version of the agent from our GitHub.
        
        ```bash
        wget https://github.com/Azure/WALinuxAgent/archive/v2.2.20.zip
        unzip v2.2.20.zip
        cd WALinuxAgent-2.2.20
        ```
        
    1. Install setup.py.
        
        ```bash
        sudo python setup.py install
        ```
        
    1. Restart waagent.
        
        ```bash
        sudo systemctl restart waagent
        ```
        
    1. Test if the agent version matches the one you downloaded. For this example, it should be 2.2.20.
        
        ```bash
        waagent -version
        ```
        
    [After 1910 release] Follow these instructions to download a compatible WALinuxAgent:
    
    1. The WALinuxAgent package, `WALinuxAgent-<version>`, has been pushed to the Red Hat extras repository. Enable the extras repository by running the following command:

        ```bash
        subscription-manager repos --enable=rhel-7-server-extras-rpms
        ```

        1.Install the Azure Linux Agent by running the following command:

            ```bash
            sudo yum install WALinuxAgent
            sudo systemctl enable waagent.service
            ```

1. Don't create swap space on the operating system disk.

    The Azure Linux Agent can automatically configure swap space by using the local resource disk that's attached to the VM after the VM is provisioned on Azure. The local resource disk is a temporary disk, and it might be emptied when the VM is deprovisioned. After you install the Azure Linux Agent in the previous step, modify the following parameters in `/etc/waagent.conf` appropriately:

    ```sh
    ResourceDisk.Format=y
    ResourceDisk.Filesystem=ext4
    ResourceDisk.MountPoint=/mnt/resource
    ResourceDisk.EnableSwap=y
    ResourceDisk.SwapSizeMB=2048    #NOTE: set this to whatever you need it to be.
    ```

1. Unregister the subscription (if necessary) by running the following command:

    ```bash
    subscription-manager unregister
    ```

1. If you're using a system that was deployed using an Enterprise Certificate Authority, the RHEL VM won't trust the Azure Stack Hub root certificate. You need to place that into the trusted root store. For more information, see [Adding trusted root certificates to the server](https://manuals.gfi.com/en/kerio/connect/content/server-configuration/ssl-certificates/adding-trusted-root-certificates-to-the-server-1605.html).

1. Run the following commands to deprovision the VM and prepare it for provisioning on Azure:

    ```bash
    sudo waagent -force -deprovision
    export HISTSIZE=0
    logout
    ```

1. Shut down the VM in KVM.

1. Convert the qcow2 image to the VHD format.

    > [!NOTE]
    > There's a known bug in qemu-img versions >=2.2.1 that results in an improperly formatted VHD. The issue has been fixed in QEMU 2.6. It's recommended to use either qemu-img 2.2.0 or lower, or update to 2.6 or higher. Reference: https://bugs.launchpad.net/qemu/+bug/1490611.

    First convert the image to raw format:

    ```bash
    qemu-img convert -f qcow2 -O raw rhel-7.4.qcow2 rhel-7.4.raw
    ```

    Make sure that the size of the raw image is aligned with 1 MB. Otherwise, round up the size to align with 1 MB:

    ```bash
    MB=$((1024*1024))
    size=$(qemu-img info -f raw --output json "rhel-7.4.raw" | \
    gawk 'match($0, /"virtual-size": ([0-9]+),/, val) {print val[1]}')
    rounded_size=$((($size/$MB + 1)*$MB))
    qemu-img resize rhel-7.4.raw $rounded_size
    ```

    Convert the raw disk to a fixed-sized VHD:

    ```bash
    qemu-img convert -f raw -o subformat=fixed -O vpc rhel-7.4.raw rhel-7.4.vhd
    ```

    Or, with qemu version **2.6+**, include the `force_size` option:

    ```bash
    qemu-img convert -f raw -o subformat=fixed,force_size -O vpc rhel-7.4.raw rhel-7.4.vhd
    ```

## Prepare a Red Hat-based VM from VMware

This section assumes that you've already installed an RHEL VM in VMware. For details about how to install an operating system in VMware, see [VMware Guest Operating System Installation Guide](https://aka.ms/aa6z600).

* When you install the Linux operating system, we recommend that you use standard partitions rather than LVM, which is often the default for many installations. This method avoids LVM name conflicts with cloned VMs, particularly if an operating system disk ever needs to be attached to another VM for troubleshooting. LVM or RAID can be used on data disks if preferred.
* Don't configure a swap partition on the operating system disk. You can configure the Linux agent to create a swap file on the temporary resource disk. You can find more information about this configuration in the steps that follow.
* When you create the virtual hard disk, select **Store virtual disk as a single file**.

### Prepare an RHEL 7 VM from VMware

1. Create or edit the `/etc/sysconfig/network` file, and add the following text:

    ```sh
    NETWORKING=yes
    HOSTNAME=localhost.localdomain
    ```

1. Create or edit the `/etc/sysconfig/network-scripts/ifcfg-eth0` file, and add the following text:

    ```sh
    DEVICE=eth0
    ONBOOT=yes
    BOOTPROTO=dhcp
    TYPE=Ethernet
    USERCTL=no
    PEERDNS=yes
    IPV6INIT=no
    NM_CONTROLLED=no
    ```

1. Ensure that the network service will start at boot time by running the following command:

    ```bash
    sudo chkconfig network on
    ```

1. Register your Red Hat subscription to enable the installation of packages from the RHEL repository by running the following command:

    ```bash
    sudo subscription-manager register --auto-attach --username=XXX --password=XXX
    ```

1. Modify the kernel boot line in your grub configuration to include additional kernel parameters for Azure. To make this modification, open `/etc/default/grub` in a text editor, and modify the `GRUB_CMDLINE_LINUX` parameter. For example:

    ```sh
    GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0"
    ```

    This configuration also ensures that all console messages are sent to the first serial port, which can assist Azure support with debugging issues. It also turns off the new RHEL 7 naming conventions for NICs. We recommend that you remove the following parameters:

    ```sh
    rhgb quiet crashkernel=auto
    ```

    Graphical and quiet boot aren't useful in a cloud environment where we want all the logs to be sent to the serial port. You can leave the `crashkernel` option configured if desired. This parameter reduces the amount of available memory in the VM by 128 MB or more, which might be problematic on smaller VM sizes.

1. After you're done editing `/etc/default/grub`, run the following command to rebuild the grub configuration:

    ```bash
    sudo grub2-mkconfig -o /boot/grub2/grub.cfg
    ```

1. Add Hyper-V modules to initramfs.

    Edit `/etc/dracut.conf`, add content:

    ```sh
    add_drivers+="hv_vmbus hv_netvsc hv_storvsc"
    ```

    Rebuild initramfs:

    ```bash
    dracut -f -v
    ```

1. [Optional after 1910 release] Stop and uninstall cloud-init:

    ```bash
    systemctl stop cloud-init
    yum remove cloud-init
    ```

1. Ensure that the SSH server is installed and configured to start at boot time. This setting is usually the default. Modify `/etc/ssh/sshd_config` to include the following line:

    ```sh
    ClientAliveInterval 180
    ```

1. When creating a custom vhd for Azure Stack Hub, keep in mind that WALinuxAgent version between 2.2.20 and 2.2.35 (both exclusive) don't work on Azure Stack Hub environments before the 1910 release. You can use versions 2.2.20/2.2.35 versions to prepare your image. To use versions above 2.2.35 to prepare your custom image, update your Azure Stack Hub to 1903 release and above or apply the 1901/1902 hotfix.

    [Before 1910 release] Follow these instructions to download a compatible WALinuxAgent:

    1. Download setuptools.
    
        ```bash
        wget https://pypi.python.org/packages/source/s/setuptools/setuptools-7.0.tar.gz --no-check-certificate
        tar xzf setuptools-7.0.tar.gz
        cd setuptools-7.0
        ```
        
    1. Download and unzip the 2.2.20 version of the agent from our GitHub.
        
        ```bash
        wget https://github.com/Azure/WALinuxAgent/archive/v2.2.20.zip
        unzip v2.2.20.zip
        cd WALinuxAgent-2.2.20
        ```
        
    1. Install setup.py.
        
        ```bash
        sudo python setup.py install
        ```
        
    1. Restart waagent.
        
        ```bash
        sudo systemctl restart waagent
        ```
        
    1. Test if the agent version matches the one you downloaded. For this example, it should be 2.2.20.
        
        ```bash
        waagent -version
        ```
        
    [After 1910 release] Follow these instructions to download a compatible WALinuxAgent:
    
    1. The WALinuxAgent package, `WALinuxAgent-<version>`, has been pushed to the Red Hat extras repository. Enable the extras repository by running the following command:

    ```bash
    subscription-manager repos --enable=rhel-7-server-extras-rpms
    ```

    1.Install the Azure Linux Agent by running the following command:
        
        ```bash
        sudo yum install WALinuxAgent
        sudo systemctl enable waagent.service
        ```
        
1. Don't create swap space on the operating system disk.

    The Azure Linux Agent can automatically configure swap space by using the local resource disk that's attached to the VM after the VM is provisioned on Azure. Note that the local resource disk is a temporary disk, and it might be emptied when the VM is deprovisioned. After you install the Azure Linux Agent in the previous step, modify the following parameters in `/etc/waagent.conf` appropriately:

    ```sh
    ResourceDisk.Format=y
    ResourceDisk.Filesystem=ext4
    ResourceDisk.MountPoint=/mnt/resource
    ResourceDisk.EnableSwap=y
    ResourceDisk.SwapSizeMB=2048    NOTE: set this to whatever you need it to be.
    ```

1. If you want to unregister the subscription, run the following command:

    ```bash
    sudo subscription-manager unregister
    ```

1. If you're using a system that was deployed using an Enterprise Certificate Authority, the RHEL VM won't trust the Azure Stack Hub root certificate. You need to place that into the trusted root store. For more information, see [Adding trusted root certificates to the server](https://manuals.gfi.com/en/kerio/connect/content/server-configuration/ssl-certificates/adding-trusted-root-certificates-to-the-server-1605.html).

1. Run the following commands to deprovision the VM and prepare it for provisioning on Azure:

    ```bash
    sudo waagent -force -deprovision
    export HISTSIZE=0
    logout
    ```

1. Shut down the VM, and convert the VMDK file to the VHD format.

    > [!NOTE]
    > There's a known bug in qemu-img versions >=2.2.1 that results in an improperly formatted VHD. The issue has been fixed in QEMU 2.6. It's recommended to use either qemu-img 2.2.0 or lower, or update to 2.6 or higher. Reference: <https://bugs.launchpad.net/qemu/+bug/1490611>.

    First convert the image to raw format:

    ```bash
    qemu-img convert -f qcow2 -O raw rhel-7.4.qcow2 rhel-7.4.raw
    ```

    Make sure that the size of the raw image is aligned with 1 MB. Otherwise, round up the size to align with 1 MB:

    ```bash
    MB=$((1024*1024))
    size=$(qemu-img info -f raw --output json "rhel-7.4.raw" | \
    gawk 'match($0, /"virtual-size": ([0-9]+),/, val) {print val[1]}')
    rounded_size=$((($size/$MB + 1)*$MB))
    qemu-img resize rhel-7.4.raw $rounded_size
    ```

    Convert the raw disk to a fixed-sized VHD:

    ```bash
    qemu-img convert -f raw -o subformat=fixed -O vpc rhel-7.4.raw rhel-7.4.vhd
    ```

    Or, with qemu version **2.6+**, include the `force_size` option:

    ```bash
    qemu-img convert -f raw -o subformat=fixed,force_size -O vpc rhel-7.4.raw rhel-7.4.vhd
    ```

## Prepare a Red Hat-based VM from an ISO by using a kickstart file automatically

1. Create a kickstart file that includes the following content, and save the file. Stopping and uninstalling cloud-init is optional (cloud-init is supported on Azure Stack Hub post 1910 release). Install the agent from the redhat repo only after the 1910 release. Prior to 1910, use the Azure repo as done in the previous section. For details about kickstart installation, see the [Kickstart Installation Guide](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Installation_Guide/chap-kickstart-installations.html).

    ```sh
    Kickstart for provisioning a RHEL 7 Azure VM

    System authorization information
    auth --enableshadow --passalgo=sha512

    Use graphical install
    text

    Do not run the Setup Agent on first boot
    firstboot --disable

    Keyboard layouts
    keyboard --vckeymap=us --xlayouts='us'

    System language
    lang en_US.UTF-8

    Network information
    network  --bootproto=dhcp

    Root password
    rootpw --plaintext "to_be_disabled"

    System services
    services --enabled="sshd,waagent,NetworkManager"

    System timezone
    timezone Etc/UTC --isUtc --ntpservers 0.rhel.pool.ntp.org,1.rhel.pool.ntp.org,2.rhel.pool.ntp.org,3.rhel.pool.ntp.org

    Partition clearing information
    clearpart --all --initlabel

    Clear the MBR
    zerombr

    Disk partitioning information
    part /boot --fstype="xfs" --size=500
    part / --fstyp="xfs" --size=1 --grow --asprimary

    System bootloader configuration
    bootloader --location=mbr

    Firewall configuration
    firewall --disabled

    Enable SELinux
    selinux --enforcing

    Don't configure X
    skipx

    Power down the machine after install
    poweroff

    %packages
    @base
    @console-internet
    chrony
    sudo
    parted
    -dracut-config-rescue

    %end

    %post --log=/var/log/anaconda/post-install.log

    #!/bin/bash

    Register Red Hat Subscription
    subscription-manager register --username=XXX --password=XXX --auto-attach --force

    Install latest repo update
    yum update -y

    Stop and Uninstall cloud-init
    systemctl stop cloud-init
    yum remove cloud-init
    
    Enable extras repo
    subscription-manager repos --enable=rhel-7-server-extras-rpms

    Install WALinuxAgent
    yum install -y WALinuxAgent

    Unregister Red Hat subscription
    subscription-manager unregister

    Enable waaagent at boot-up
    systemctl enable waagent

    Disable the root account
    usermod root -p '!!'

    Configure swap in WALinuxAgent
    sed -i 's/^\(ResourceDisk\.EnableSwap\)=[Nn]$/\1=y/g' /etc/waagent.conf
    sed -i 's/^\(ResourceDisk\.SwapSizeMB\)=[0-9]*$/\1=2048/g' /etc/waagent.conf

    Set the cmdline
    sed -i 's/^\(GRUB_CMDLINE_LINUX\)=".*"$/\1="console=tty1 console=ttyS0 earlyprintk=ttyS0 rootdelay=300"/g' /etc/default/grub

    Enable SSH keepalive
    sed -i 's/^#\(ClientAliveInterval\).*$/\1 180/g' /etc/ssh/sshd_config

    Build the grub cfg
    grub2-mkconfig -o /boot/grub2/grub.cfg

    Configure network
    cat << EOF > /etc/sysconfig/network-scripts/ifcfg-eth0
    DEVICE=eth0
    ONBOOT=yes
    BOOTPROTO=dhcp
    TYPE=Ethernet
    USERCTL=no
    PEERDNS=yes
    IPV6INIT=no
    NM_CONTROLLED=no
    EOF

    Deprovision and prepare for Azure
    waagent -force -deprovision

    %end
    ```

1. Place the kickstart file where the installation system can access it.

1. In Hyper-V Manager, create a new VM. On the **Connect Virtual Hard Disk** page, select **Attach a virtual hard disk later**, and complete the New Virtual Machine Wizard.

1. Open the VM settings:

    a. Attach a new virtual hard disk to the VM. Make sure to select **VHD Format** and **Fixed Size**.

    b. Attach the installation ISO to the DVD drive.

    c. Set the BIOS to boot from CD.

1. Start the VM. When the installation guide appears, press **Tab** to configure the boot options.

1. Enter `inst.ks=<the location of the kickstart file>` at the end of the boot options, and press **Enter**.

1. Wait for the installation to finish. When it's finished, the VM is shut down automatically. Your Linux VHD is now ready to be uploaded to Azure.

## Known issues

### The Hyper-V driver couldn't be included in the initial RAM disk when using a non-Hyper-V hypervisor

In some cases, Linux installers might not include the drivers for Hyper-V in the initial RAM disk (initrd or initramfs) unless Linux detects that it's running in a Hyper-V environment.

When you're using a different virtualization system (like Oracle VM VirtualBox, Xen Project, and so on) to prepare your Linux image, you might need to rebuild initrd to ensure that at least the hv_vmbus and hv_storvsc kernel modules are available on the initial RAM disk. This is a known issue at least on systems that are based on the upstream Red Hat distribution.

To resolve this issue, add Hyper-V modules to initramfs and rebuild it:

Edit `/etc/dracut.conf`, and add the following content:

```sh
add_drivers+="hv_vmbus hv_netvsc hv_storvsc"
```

Rebuild initramfs:

```bash
dracut -f -v
```

For more information, see [rebuilding initramfs](https://access.redhat.com/solutions/1958).

## Next steps

You're now ready to use your Red Hat Enterprise Linux virtual hard disk to create new VMs in Azure Stack Hub. If this is the first time that you're uploading the VHD file to Azure Stack Hub, see [Create and publish a Marketplace item](azure-stack-create-and-publish-marketplace-item.md).

For more information about the hypervisors that are certified to run Red Hat Enterprise Linux, see [the Red Hat website](https://access.redhat.com/certified-hypervisors).
