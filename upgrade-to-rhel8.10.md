## Upgrade RHEL 7.9 to 8.10

### 1. Verify that the system is subscribed using subscription-manager

    subscription-manager status
  or
  
    subscription-manager --list
  
Ensure you have appropriate repositories enabled if not then following commands list repositories for the 64-bit Intel architecture,

##### Enable the Base repository:
    subscription-manager repos --enable rhel-7-server-rpms
##### Enable the Extras repository where Leapp and its dependencies are available:
    subscription-manager repos --enable rhel-7-server-extras-rpms


### 2. To upgrade to RHEL 8.10, we required the leapp utility, which is the recommended tool for a smooth transition.
- #### Install the Leapp utility:
      yum install leapp-upgrade



### 3. On your RHEL 7 system, perform the pre-upgrade phase:
  Running the pre-upgrade leapp command helps identify compatibility issues, deprecated features, and readiness, ensuring a smoother and more reliable upgrade to RHEL 8.10

        leapp preupgrade --target 8.10
  #### Verification of Pre-Upgrade Output
   - Please ensure that the error count in the pre-upgrade output is zero. Any errors detected must be addressed and resolved prior to proceeding with the upgrade, as unresolved issues may lead to complications during the upgrade process.
   - Review all inhibitors listed in the output and take appropriate actions to resolve them. Like unmount and comment the NFS mount point in /etc/fstab, /Var or /usr required space etc (Ignore 1 inhibitor is Leapp detected loaded kernel which have been removed in RHEL 8)

  #### Commands to Execute Before Starting the Upgrade for a Smooth Process
  Use below command to confirm the removal of the pam_pkcs11 module during the upgrade preparation with the Leapp tool. Confirming this removal helps avoid potential conflicts or issues related to this module, facilitating a smoother upgrade.

       leapp answer --section remove_pam_pkcs11_module_check.confirm=True
   
  Execute one of these commands to remove kernel modules that could interfere with the upgrade process:
  
  - rmmod floppy pata_acpi: Removes the floppy and pata_acpi modules from the kernel. These modules manage specific hardware components and might cause conflicts if left loaded during the upgrade.
  - modprobe -r floppy pata_acpi: Utilizes the modprobe utility to remove the same modules, handling any related dependencies.
     
        rmmod floppy pata_acpi
   or

        modprobe -r floppy pata_acpi
  Running these commands ensures that outdated or unnecessary modules are not loaded, thereby preventing compatibility issues and promoting a smoother upgrade.

#### Notes: 
  - Ensure that you have sufficient disk space available at least 5 GB free in /var and 4 GB free in /usr for a smooth upgrade.
  - Ensure you have console access to the system to monitor the upgrade in real-time. This access is crucial for observing progress and addressing any issues that may arise during the upgrade.

### 4. Performing the upgrade from RHEL 7.9 to RHEL 8.10
  After completing the necessary pre-upgrade preparations, you can proceed with upgrading the OS from RHEL 7.9 to RHEL 8.10.
  Run the following command to start the upgrade process:
      
      leapp upgrade --target 8.10

  After executing the leapp upgrade command, monitor the process continuously until it completes and prompts you to reboot the system.
  
### 5. Reboot the system and Monitor it on console page
  Once prompted, initiate a reboot of the system. Continuously observe the console page to monitor the completion of the upgrade process.
  
    reboot


Follow the next file for post-upgrade steps.
Thank you. :)
