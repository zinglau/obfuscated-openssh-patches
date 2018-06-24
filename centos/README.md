How to use
======

1. Clone source repository from CentOS
    ```Bash
    git clone https://git.centos.org/r/rpms/openssh.git
    ```

2. Checkout the corresponding version tag. For full list of tags and branches, check [here](https://git.centos.org/summary/rpms!openssh.git).
    ```Bash
    cd openssh
    # git checkout tags/<tag> -b <branch>
    git checkout tags/imports/c7/openssh-7.4p1-16.el7 -b c7
    ```

3. Download the patch for corresponding version and save to `SOURCES/`

4. Modify `SPECS/openssh.spec` accordingly. This may include:
    * Changing `openssh_ver` or `openssh_rel` variable to indicate modification
        ```diff
         %define openssh_ver 7.4p1
        -%define openssh_rel 16
        +%define openssh_rel 16.obfs
         %define pam_ssh_agent_ver 0.10.3
         %define pam_ssh_agent_rel 2
        ```
    * Add the patch to patch list, and execute it **after all other patches**
        ```diff
         # Internal debug
         Patch0: openssh-5.9p1-wIm.patch
        +Patch1: openssh-7.4p1-obfuscate.patch
        
         #?
         Patch100: openssh-7.4p1-coverity.patch
        ```
        ```diff
         %patch700 -p1 -b .fips
        
         %patch100 -p1 -b .coverity
        +%patch1 -p1 -b .obfuscate
        
         %if 0
         # Nothing here yet
        ```

5. Build RPM
    ```Bash
    # You may need to install build dependencies first
    yum-builddep openssh
    # Test build (Executes the "%prep" stage. Normally this involves unpacking the sources and applying any patches.)
    rpmbuild -bp SPECS/openssh.spec
    # Build binary and source packages
    rpmbuild -ba SPECS/openssh.spec
    ```

6. If all finished without error, you would now have RPM packages in `RPMS/` that can be used. On a system with both `openssh-server` and `openssh-clients` installed, you have to update both packages plus `openssh`, or you would break dependencies. Otherwise, just install the package you need, e.g. `openssh-server` with `openssh` as dependency.
    ```Bash
    # cd RPMS/<arch>
    cd RPMS/x86_64
    yum localinstall openssh-7.4p1-16.obfs.el7.x86_64.rpm openssh-clients-7.4p1-16.obfs.el7.x86_64.rpm openssh-server-7.4p1-16.obfs.el7.x86_64.rpm
    ```

7. Modify your existing `sshd_config`. (You can use `sshd_config.rpmnew` as a reference)
    ```Bash
    vi /etc/ssh/sshd_config
    ```

8. If you specified a port other than 22 as either the default port (not obfuscated), or the obfuscated port, you would have to update your SELinux config (if SELinux config is enabled)
    ```Bash
    semanage port -l | grep ssh
    # ssh_port_t                     tcp      22
    
    # semanage port -a -t ssh_port_t -p tcp <your-port-number>
    semanage port -a -t ssh_port_t -p tcp 222
    ```

9. Reload your `sshd.service`. **Make sure to test the new configuration before disconnecting you old SSH connection**, in case anything goes wrong, you can at least revert changes to `sshd_config` and possibly `yum reinstall openssh-server`.
    ```Bash
    systemctl reload sshd
    ```

References
======

https://wiki.centos.org/PackageManagement/Rpm

https://blog.tinned-software.net/change-ssh-port-in-centos-with-selinux/
