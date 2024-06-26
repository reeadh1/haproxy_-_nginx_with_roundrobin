<a name="br1"></a> 

In this practical , we will learn how to install and conﬁgure HAProxy on centos 7 &Nginx web servers

using roundrobin approach for load balancing.

Lab environment:

Operating system

: Centos 7

Ip address and hostname:

:192.168.0.160 haproxy-centos7

:192.168.0.161 nginx-node01

: 192.168.0.162 nginx-node02

= = = = = = = = =======================================================

**Step:1 >>** update /etc/hosts ﬁle of your HAproxy server and Login to your centos 7 systems and

change a static hostname for better understanding.

\# hostnamectl set-hostname haproxy-centos7 # on haproxy server

\# hostnamectl set-hostname nginx-node01 # on node 1

\# hostnamectl set-hostname nginx-node02 #on node 2

**Step 2:** For the three machine we have to we have to give the host name in *vim /etc/hosts*

**[root@haproxy-centos7 ~]# cat /etc/hosts**

**127.0.0.1 localhost localhost.localdomain localhost4 localhost4.localdomain4**

**::1 localhost localhost.localdomain localhost6 localhost6.localdomain6**

**192.168.0.160 haproxy-centos7**

**192.168.0.161 nginx-node01**

**192.168.0.161 nginx-node02**

**Same as other 2 servers**

*Now need to check , with ping from haproxy node to other two node*

**[root@haproxy-centos7 ~]# ping nginx-node02**

**PING nginx-node02 (192.168.0.161) 56(84) bytes of data.**

**64 bytes from nginx-node01 (192.168.0.161): icmp\_seq=1 ttl=64 time=0.902 ms**

**64 bytes from nginx-node01 (192.168.0.161): icmp\_seq=2 ttl=64 time=1.56 ms**



<a name="br2"></a> 

**PING nginx-node02 (192.168.0.161) 56(84) bytes of data.**

**64 bytes from nginx-node01 (192.168.0.161): icmp\_seq=1 ttl=64 time=0.967 ms**

**64 bytes from nginx-node01 (192.168.0.161): icmp\_seq=2 ttl=64 time=1.96 ms**

**64 bytes from nginx-node01 (192.168.0.161): icmp\_seq=3 ttl=64 time=1.82 ms**

*From haproxy now we can ping both the servers, that means connection is ok*

__Step 4*:*__*Now we will install haproxy , to our haproxy server*

(we must ensure servers are properly patched)

Run the comment **dnf update -y**

after patching need to reboot the machine

**Step 4: we we install haproxy to our haproxy server [root@haproxy-centos7 ~]# dnf install**

**haproxy -y**

[root@haproxy-centos7 ~]# dnf install haproxy -y

[root@haproxy-centos7 ~]# cd /etc/haproxy/

[root@haproxy-centos7 haproxy]# ls

haproxy.cfg

**Step 5: we need to make a backup if something goes wrong**

**[root@haproxy-centos7 haproxy]# cp haproxy.cfg haproxy.cfg-back**

**[root@haproxy-centos7 haproxy]# vi haproxy.cfg**

**( no need to change the other settings only front end and back-end we will change)**

#---------------------------------------------------------------------

**# main frontend which proxys to the backends**

**#---------------------------------------------------------------------**

**frontend load\_balancer**

**bind 192.168.0.160:80**

**option http-server-close**

**option forwardfor**

**stats uri /haproxy?stats**



<a name="br3"></a> 

**default\_backend nginx\_webservers**

**#---------------------------------------------------------------------**

**# static backend for serving up images, stylesheets and such**

**#---------------------------------------------------------------------**

**#---------------------------------------------------------------------**

**backend nginx\_webservers**

**mode http**

**balance roundrobin**

**option httpchk HEAD / HTTP/1.1\r\nHost:\ localhost**

**server web-server-01 192.168.0.161:80 check**

**server web-server-02 192.168.0.162:80 check**

**= = = = ======= =========================**

**Step 6:**To make all the things synchronize , all the logs if we want to monitoring centrally

Need to go Rsyslog ﬁle

**[root@haproxy-centos7 haproxy]# vi /etc/rsyslog.conf ( need to commend out 2 lines)**

**# Provides UDP syslog reception**

**$ModLoad imudp**

**$UDPServerRun 514**

**Step 7:** Now haproxy.conf deﬁne the logs , we will deﬁne where we put the logs

**[root@haproxy-centos7 haproxy]# vi /etc/rsyslog.d/haproxy.conf**

**local2.=info /var/log/haproxy-access.log**

**local2.notice /var/log/haproxy-info.log**

**we need to restart Rsyslog, as we do change**

**[root@haproxy-centos7 haproxy]# systemctl restart rsyslog**

**For enable booting enable**



<a name="br4"></a> 

**[root@haproxy-centos7 haproxy]# systemctl enable rsyslog**

**== ==**

**Step 8:** Now need to check selinux

**Need to unable a bullion for selinux**

**[root@haproxy-centos7 haproxy]# getsebool haproxy\_connect\_any**

**haproxy\_connect\_any --> oﬀ ( it can create problem , we need to enable it)**

**[root@haproxy-centos7 haproxy]# setsebool -P haproxy\_connect\_any 1**

We if we check –

**[root@haproxy-centos7 haproxy]# getsebool haproxy\_connect\_any**

**haproxy\_connect\_any --> on**

**====**

**Step 9:**Need to enable , start haproxy

**[root@haproxy-centos7 haproxy]# systemctl enable haproxy –now**

**[root@haproxy-centos7 haproxy]# systemctl restart haproxy –now**

**Step 10** : Now to add rule to ﬁrewall

**[root@haproxy-centos7 ~]# ﬁrewall-cmd --permanent --add-port=80/tcp**

**Success**

**Now need reload ﬁrewall**

**[root@haproxy-centos7 ~]# ﬁrewall-cmd --reload**

**Success**

**Now need to verify the ﬁrewall**

**Result>**

**[root@haproxy-centos7 ~]# ﬁrewall-cmd --list-all**

**public (active)**



<a name="br5"></a> 

**target: default**

**icmp-block-inversion: no**

**interfaces: ens33**

**sources:**

**services: dhcpv6-client ssh**

**ports: 80/tcp**

**protocols:**

**masquerade: no**

**forward-ports:**

**source-ports:**

**icmp-blocks:**

**rich rules:**

**Step 11:** ===== === nginx server installation on node 1 and node 2 ============

**[root@nginx-node01 ~]# sudo yum install epel-release -y**

**[root@nginx-node02 ~]# sudo yum install nginx -y (install node1 and node 2)**

**== = =enable nginx on both servers = = =**

**[root@nginx-node01 ~]# systemctl enable nginx --now**

**Created symlink from /etc/systemd/system/multi-user.target.wants/nginx.service to**

**/usr/lib/systemd/system/nginx.service.**

**Step 12:** Need to go to the following directory and creating a index.html ﬁle in both node

**[root@nginx-node01 ~]# cd /usr/share/nginx/html/**

**[root@nginx-node01 html]# echo "Nginx Node01 -Welcome to First Nginx Web Server" >**

**index.html**

**Step 13: ==== now ﬁrewall port add =====**

**[root@nginx-node02 html]# ﬁrewall-cmd --permanent --add-service=http**



<a name="br6"></a> 

**Now need to reload the ﬁrewall**

**[root@nginx-node02 html]# ﬁrewall-cmd –reload**

**= == == = = = for checking ======**

**[root@haproxy-centos7 ~]# curl 192.168.0.160**

**Nginx Node01 -Welcome to First Nginx Web Server**

**[root@haproxy-centos7 ~]# curl 192.168.0.160**

**Nginx Node02 -Welcome to Second Nginx Web Server**

\-

**- - - same work at wabbroser**

**To check haproxy**

[**http://192.168.0.160/haproxy?stats**](http://192.168.0.160/haproxy?stats)

**= = === Now checking the logs ============**

**[root@haproxy-centos7 ~]# tail -f /var/log/haproxy-access.log**



<a name="br7"></a> 


