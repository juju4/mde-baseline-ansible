# Microsoft Defender for Endpoint (MDE) Ansible baseline

Ensure MDE is installed, configured and active on Linux, MacOS or Windows system with ansible

define mde group in your inventory
```
% ansible-playbook mde-verify.yml
```

# Alternatives

* Use [MDE baseline Inspec profile](https://github.com/juju4/mde-baseline/)

# Known MDE setup issues

* Selinux enforced on RHEL/Centos7 issue. Working on RHEL/Centos8
```
Error : wdavdaemon[20263]: /opt/microsoft/mdatp/sbin/wdavdaemon: error while loading shared libraries: libwdavdaemon_core.so: cannot enable executable stack as shared object requires: Permission denied
```
To solve this
```
$ sepolicy generate -n wdavdaemon --init /opt/microsoft/mdatp/sbin/wdavdaemon
Loaded plugins: product-id, subscription-manager
Created the following files:
wdavdaemon.te # Type Enforcement file
wdavdaemon.if # Interface file
wdavdaemon.fc # File Contexts file
wdavdaemon_selinux.spec # Spec file
wdavdaemon.sh # Setup Script
$ sudo audit2allow -i /var/log/audit/audit.log -M wdavdaemon
[pp file should have: allow unconfined_service_t self:process execstack;]
$ sudo semodule -i wdavdaemon.pp
$ sudo systemctl restart mdatp
$ systemctl status mdatp
```

* /var/opt/microsoft/mdatp need to have exec mount flag
an alternate option is to move to a compatible location
```
$ sudo systemctl stop mdatp
$ sudo install -d -m 755 /usr/local/microsoft
$ sudo bash -c 'mv /var/opt/microsoft/mdatp /usr/local/microsoft/ && ln -s /usr/local/microsoft/mdatp /var/opt/microsoft/'
$ sudo systemctl restart mdatp
```
This will likely address health issue: engine v1 not available

* Analyzer tool requires python3.7 and perf on Linux. Support can provide an alternate tool without requiring python3

* Default analyzer zip does not support RHEL/Centos 6. reach support for appropriate package

# References

* https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/
* https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/linux-install-with-ansible, https://github.com/MicrosoftDocs/microsoft-365-docs/blob/public/microsoft-365/security/defender-endpoint/linux-install-with-ansible.md
* https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/linux-install-with-puppet
* https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/linux-deploy-defender-for-endpoint-with-chef
* https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/microsoft-defender-endpoint-linux
* https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/health-status
If definitions are not updated automatically, try manually
```
mdatp definitions update
```
* https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/linux-support-connectivity
* https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/linux-support-perf
part of performance impact may be related to auditd and usual path is to exclude matching binary from auditd coverage, either with support tools, either directly in auditd rules.
