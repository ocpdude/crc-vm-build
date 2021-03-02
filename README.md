## Building Red Hat OpenShift Code Ready Container Platform w/ CentOS on VMware

Demo video, see YouTube :

The first step is to build your VM, for my demo I will use CentOS 7.9 minimal. I have allocated 4 vCPU, 16GB of memory and 35GB of thin disk storage. Please remember, to access the CPU settings, under "Hardware virtualization" check the box for "Expose hardware assisted virtualization to the guest OS".

OPTIONAL: If you want the full OCP experiance, including monitoring please allocate 8vCPU and at least 16GB of memory. I recommend 8vCPU and 24GB of memory since we're dealing with a 'nested' virutal machine.

### after your virtual machinge is setup, follow these steps
1. `sudo yum update -y ; sudo yum upgrade -y`
2. `sudo yum install -y wget xz libvirt haproxy`

### setup the vm for crc
3. `sudo systemctl status firewalld`
4. `sudo firewall-cmd --add-port={80/tcp,443/tcp,6443/tcp} --permanent`
5. `sudo systemctl restart firewalld`

### pull the crc and oc tools packages and install them
6. `wget https://mirror.openshift.com/pub/openshift-v4/clients/crc/latest/crc-linux-amd64.tar.xz`
7. `wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-client-linux.tar.gz`
8. `sudo unxz crc-linux-amd64.tar.xz`
9. `sudo tar xf crc-linux-amd64.tar`
10. `sudo mv crc-linux-1.22.0-amd64/crc /usr/local/bin/`
11. `sudo tar zxf openshift-client-linux.tar.gz -C /usr/local/bin/`

### setup crc and configure for remote access
12. `crc setup`
13. `sudo shutdown -r now`

#### OPTIONAL: Adding monitoring support to CRC/OCP
    i. Allocate more vCPU `crc config set cpus 6` 
    ii. Allocate additional memory `crc config set memory 16384` 
    iii. Enable the Monitoring Operator `crc config set enable-cluster-monitoring true`

14. `crc start`

15. Gather IP Addresses \
    `hostname --ip-address` \
    `crc ip`

16. `sudo vi /etc/haproxy/haproxy.cfg`
```
global
debug

defaults
log global
mode http
timeout connect 10s
timeout client 1m
timeout server 1m

frontend apps
bind $SERVER:80
bind $SERVER:443
mode tcp
default_backend apps

backend apps
mode tcp
option ssl-hello-chk
balance roundrobin
server crc_vm $CRC:443 check

frontend api
bind $SERVER:6443
mode tcp
default_backend api

backend api
mode tcp
balance roundrobin
option ssl-hello-chk
server crc_vm $CRC:6443 check
```
17. selinux enabled hosts only > `sudo setsebool -P haproxy_connect_any=1` \
    `sudo systemctl enable haproxy`\
    `sudo systemctl start haproxy`

18. `crc console`

19. add console path to /etc/hosts on your local machine \
`echo $THE_VM_SERVER_IP console-openshift-console.apps-crc.testing oauth-openshift.apps-crc.testing api.crc.testing >> /etc/hosts`

*note: substitute "$THE_VM_SERVER_IP" with the CRC VM address in your local /etc/hosts or better yet, add it to DNS or some DNSMasq config.

Tips: 
Find the console : `crc console --url` \
Reprint the default credentials : `crc console --credentials`