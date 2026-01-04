https://access.redhat.com/solutions/4388431
```
[root@test ~]# subscription-manager repos --enable=fast-datapath-for-rhel-9-x86_64-rpms
Repository 'fast-datapath-for-rhel-9-x86_64-rpms' is enabled for this system.

[root@test ~]# subscription-manager repos --enable=openstack-17-for-rhel-9-x86_64-rpms
Repository 'openstack-17-for-rhel-9-x86_64-rpms' is enabled for this system.

[root@test ~]# dnf install openvswitch
Updating Subscription Management repositories.
Red Hat OpenStack Platform 17 for RHEL 9 x86_64 (RPMs)                                                                                   833 kB/s | 1.3 MB     00:01
Dependencies resolved.
=========================================================================================================================================================================
 Package                                                 Architecture         Version                           Repository                                          Size
=========================================================================================================================================================================
Installing:
 rhosp-openvswitch                                       noarch               2.17-5.el9ost                     openstack-17-for-rhel-9-x86_64-rpms                8.3 k
Installing dependencies:
 chkconfig                                               x86_64               1.24-2.el9                        rhel-9-for-x86_64-baseos-rpms                      182 k
 initscripts                                             x86_64               10.11.8-4.el9                     rhel-9-for-x86_64-baseos-rpms                      232 k
 ipcalc                                                  x86_64               1.0.0-5.el9                       rhel-9-for-x86_64-baseos-rpms                       44 k
 openstack-network-scripts                               x86_64               10.11.1-3.el9ost                  openstack-17-for-rhel-9-x86_64-rpms                 52 k
 openstack-network-scripts-openvswitch2.17               x86_64               10.11.1-3.el9ost                  openstack-17-for-rhel-9-x86_64-rpms                 13 k
 openvswitch-selinux-extra-policy                        noarch               1.0-39.el9fdp                     fast-datapath-for-rhel-9-x86_64-rpms                12 k
 openvswitch2.17                                         x86_64               2.17.0-158.el9fdp                 fast-datapath-for-rhel-9-x86_64-rpms               6.6 M
 rhosp-network-scripts-openvswitch                       noarch               2.17-5.el9ost                     openstack-17-for-rhel-9-x86_64-rpms                8.3 k
 unbound-libs                                            x86_64               1.16.2-21.el9                     rhel-9-for-x86_64-appstream-rpms                   551 k
Installing weak dependencies:
 geolite2-city                                           noarch               20191217-6.el9                    rhel-9-for-x86_64-appstream-rpms                    23 M
 geolite2-country                                        noarch               20191217-6.el9                    rhel-9-for-x86_64-appstream-rpms                   1.6 M

Transaction Summary
=========================================================================================================================================================================
Install  12 Packages

Total download size: 32 M
Installed size: 88 M
Is this ok [y/N]:

[root@test ~]# systemctl status openvswitch
○ openvswitch.service - Open vSwitch
     Loaded: loaded (/usr/lib/systemd/system/openvswitch.service; disabled; preset: disabled)
     Active: inactive (dead)
```
```
[mfau@mfau-thinkpadx1carbon5th ~]$ systemctl status openvswitch
● openvswitch.service - Open vSwitch
     Loaded: loaded (/usr/lib/systemd/system/openvswitch.service; enabled; preset: disabled)
    Drop-In: /usr/lib/systemd/system/service.d
             └─10-timeout-abort.conf
     Active: active (exited) since Sun 2025-12-28 09:13:20 EST; 24s ago
 Invocation: 2c1c58d886164df6b0005d5cb280aa8e
   Main PID: 11621 (code=exited, status=0/SUCCESS)
   Mem peak: 1M
        CPU: 2ms

Dec 28 09:13:19 mfau-thinkpadx1carbon5th.remote.csb systemd[1]: Starting openvswitch.service - Open vSwitch...
Dec 28 09:13:20 mfau-thinkpadx1carbon5th.remote.csb systemd[1]: Finished openvswitch.service - Open vSwitch.
[mfau@mfau-thinkpadx1carbon5th ~]$ ovs-vsctl show
ovs-vsctl: unix:/var/run/openvswitch/db.sock: database connection failed (Permission denied)
[mfau@mfau-thinkpadx1carbon5th ~]$ sudo ovs-vsctl show
7bbc2ba1-dbcd-4924-af19-769f1216fa50
    ovs_version: "3.4.1-1.fc42"
```