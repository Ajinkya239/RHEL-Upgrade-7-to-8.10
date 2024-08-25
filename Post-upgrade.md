## Performing post-upgrade tasks

### After the upgrade completes, determine whether the system is in the required state, at least:
- #### Verify that the current OS version is Red Hat Enterprise Linux 8:

      cat /etc/redhat-release
    Red Hat Enterprise Linux release <target_os_version> (Ootpa). 
    Replace target_os_version with the target OS version, for example 8.10.

- #### Check the OS kernel version:

      uname -r
    4.18.0-305.el<target_os>.x86_64, 
    The target_os should be either 8 or the target OS version, for example 8_10. Note that .el8 is important and the version should not be earlier than 4.18.0-305.

- #### Verify that the correct product is installed:

      subscription-manager list --installed
    Replace target_os_version with the target OS version, for example 8.10.

- #### Verify that the release version is set to the target OS version immediately after the upgrade:

      subscription-manager release
    Release: <target_os_version> , 
    Replace target_os_version with the target OS version, for example 8.10.


### After confirmed the upgrade, complete the following tasks:
#### 1. Remove all packages from the exclude list in the /etc/dnf/dnf.conf
  Remove any remaining Leapp packages, including the snactor package. During the in-place upgrade, Leapp packages that were installed with the Leapp utility are automatically added to the exclude list to prevent critical files from being removed or updated. After the in-place upgrade, you must remove these Leapp packages from the exclude list before they can be removed from the system.
  - a. To manually remove packages from the exclude list, edit the /etc/dnf/dnf.conf configuration file and remove the desired Leapp packages from the exclude list.
    OR
  - b. To remove all packages from the exclude list:•

        yum config-manager --save --setopt exclude=''

#### 2. Determine old kernel versions to remove:

      cd /lib/modules && ls -d *.el7*

#### 3. Remove weak modules from the old kernel. If you have multiple old kernels, repeat the following step for each kernel:

    [ -x /usr/sbin/weak-modules ] && /usr/sbin/weak-modules --remove-kernel <version>
  Replace <version> with the kernel version determined in the step 2.

#### 4. Remove the old kernel from the boot loader entry. If you have multiple old kernels, repeat this step for each kernel:

    /bin/kernel-install remove <version> /lib/modules/<version>/vmlinuz
  Replace <version> with the kernel version determined in the step 2.

#### 5. Locate remaining RHEL 7 packages:

    rpm -qa | grep -e '\.el[67]' | grep -vE '^(gpg-pubkey|libmodulemd|katello-ca-consumer)' | sort

#### 6. Remove remaining RHEL 7 packages, including old kernel packages, and the kernel-workaround package from your RHEL 8 system. 
  
    yum remove kernel-workaround $(rpm -qa | grep \.el7 | grep -vE 'gpg-pubkey|libmodulemd|katello-ca-consumer')

#### 7. Remove remaining Leapp dependency packages:

    yum remove leapp-deps-el8 leapp-repository-deps-el8

#### 8. Remove any remaining empty directories:

    rm -r /lib/modules/*el7*

#### 9. Remove all remaining upgrade-related data from the system:

    rm -rf /var/log/leapp /root/tmp_leapp_py3 /var/lib/leapp

#### 10. Replace the old rescue kernel and initial RAM disk with the current kernel and disk:
  - a. Remove the existing rescue kernel and initial RAM disk:

        rm /boot/vmlinuz-*rescue* /boot/initramfs-*rescue*

  - b. Reinstall the rescue kernel and related initial RAM disk:

        /usr/lib/kernel/install.d/51-dracut-rescue.install add "$(uname -r)" /boot "/boot/vmlinuz-$(uname -r)"

### Verification steps

#### 1. Verify that the old kernels have been removed from the bootloader entry:

    grubby --info=ALL | grep "\.el7" || echo "Old kernels are not present in the bootloader."

#### 2. Verify that the previously removed rescue kernel and rescue initial RAM disk files have been created for the current kernel:

    ls /boot/vmlinuz-*rescue* /boot/initramfs-*rescue* 

    lsinitrd /boot/initramfs-*rescue*.img | grep -qm1 "$(uname -r)/kernel/" && echo "OK" || echo "FAIL"
#### 3. Verify the rescue boot entry refers to the existing rescue files. See the grubby output:

    grubby --info $(ls /boot/vmlinuz-*rescue*)

--------------------------------

Thank you, and best of luck with your successful upgrade!
