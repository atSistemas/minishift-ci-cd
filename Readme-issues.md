
## Error in VirtualBox ips
**Error ./minishift start ...**

Caused By:
     Error: Get https://192.168.99.100:8443/healthz/ready: x509: certificate is valid for 10.0.2.15, 127.0.0.1, 172.17.0.1, 172.30.0.1, 192.168.99.101, not 192.168.99.100

**Documented:**
https://github.com/minishift/minishift/issues/1457


     Testing on VirtualBox with minishift/minishift-centos-iso#179 and the following configuration files:

     cat /var/lib/minishift/networking-eth0

     DEVICE=eth0
     USEDHCP=y
     cat /var/lib/minishift/networking-eth1

     DEVICE=eth1
     IPADDR=192.168.235.100
     NETMASK=24
     GATEWAY=192.168.235.1
     DNS1=8.8.8.8
     DNS2=8.8.4.4

**Solved** [minishift-start](bootstrap/minishift-start)

.. --iso-url=centos ..



**File Content:**

cat networking-eth0
DEVICE=eth0
USEDHCP=y

cat networking-eth1
DEVICE=eth1
IPADDR=192.168.99.***
NETMASK=24
GATEWAY=192.168.99.1
DNS1=8.8.8.8
DNS2=8.8.4.4



## Openshift Job: Timeout Error in Demo Installer
Solved updated restart policy in Yml
