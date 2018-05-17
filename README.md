#learHA
192.168.11.125 --- pcmk-1
192.168.11.124 --- pcmk-2

192.168.11.125 pcmk-1.clusterlabs.org pcmk-1
192.168.11.124 pcmk-2.clusterlabs.org pcmk-2

hacluster passwd qucd0701

启动cluster
开始是dead!!!

[root@pcmk-1 network-scripts]# systemctl status  pacemaker.service 
● pacemaker.service - Pacemaker High Availability Cluster Manager
   Loaded: loaded (/usr/lib/systemd/system/pacemaker.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
     Docs: man:pacemakerd
           http://clusterlabs.org/doc/en-US/Pacemaker/1.1-pcs/html/Pacemaker_Explained/index.html
[root@pcmk-1 network-scripts]# systemctl status  corosync.service
● corosync.service - Corosync Cluster Engine
   Loaded: loaded (/usr/lib/systemd/system/corosync.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
     Docs: man:corosync
           man:corosync.conf
           man:corosync_overview


[root@pcmk-1 network-scripts]# pcs cluster start --all
pcmk-1: Starting Cluster...
pcmk-2: Starting Cluster...
[root@pcmk-1 network-scripts]# systemctl status  corosync.service
● corosync.service - Corosync Cluster Engine
   Loaded: loaded (/usr/lib/systemd/system/corosync.service; disabled; vendor preset: disabled)
   Active: active (running) since 四 2018-05-17 03:26:18 EDT; 7s ago
     Docs: man:corosync
           man:corosync.conf
           man:corosync_overview
  Process: 13818 ExecStart=/usr/share/corosync/corosync start (code=exited, status=0/SUCCESS)
 Main PID: 13829 (corosync)
   CGroup: /system.slice/corosync.service
           └─13829 corosync

5月 17 03:26:18 pcmk-1 corosync[13829]:  [VOTEQ ] Waiting for all cluster members. Current votes: 1 expected_votes: 2
5月 17 03:26:18 pcmk-1 corosync[13829]:  [QUORUM] Members[1]: 1
5月 17 03:26:18 pcmk-1 corosync[13829]:  [MAIN  ] Completed service synchronization, ready to provide service.
5月 17 03:26:18 pcmk-1 corosync[13829]:  [TOTEM ] A new membership (192.168.11.124:8) was formed. Members joined: 2
5月 17 03:26:18 pcmk-1 corosync[13829]:  [QUORUM] This node is within the primary component and will provide service.
5月 17 03:26:18 pcmk-1 corosync[13829]:  [QUORUM] Members[1]: 1
5月 17 03:26:18 pcmk-1 corosync[13829]:  [QUORUM] Members[2]: 2 1
5月 17 03:26:18 pcmk-1 corosync[13829]:  [MAIN  ] Completed service synchronization, ready to provide service.
5月 17 03:26:18 pcmk-1 corosync[13818]: Starting Corosync Cluster Engine (corosync): [  OK  ]
5月 17 03:26:18 pcmk-1 systemd[1]: Started Corosync Cluster Engine.


5.2. Add a Resource
Our first resource will be a unique IP address that the cluster can bring up on either node. Regardless of where any cluster service(s) are running, end users need a consistent address to contact them on. Here, I will choose 192.168.122.120 as the floating address, give it the imaginative name ClusterIP and tell the cluster to check whether it is running every 30 seconds. 
Warning
The chosen address must not already be in use on the network. Do not reuse an IP address one of the nodes already has configured. 
- - 一定要使用一个网络不存在的ip地址作为cluster浮动IP地址
[root@pcmk-1 ~]# pcs resource create ClusterIP ocf:heartbeat:IPaddr2 \
    ip=192.168.122.120 cidr_netmask=32 op monitor interval=30s
（测试用192.168.11.171）

结果： 果然配置后， ping 192.168.11.171 可以啦
chaodongqu@chaodongqu:~$ ping 192.168.11.171 
PING 192.168.11.171 (192.168.11.171) 56(84) bytes of data. 
From 192.168.11.221 icmp_seq=1 Destination Host Unreachable 
From 192.168.11.221 icmp_seq=2 Destination Host Unreachable 
From 192.168.11.221 icmp_seq=3 Destination Host Unreachable 
^C 
--- 192.168.11.171 ping statistics --- 
4 packets transmitted, 0 received, +3 errors, 100% packet loss, time 3072ms 
pipe 4 
chaodongqu@chaodongqu:~$ ping 192.168.11.171 
PING 192.168.11.171 (192.168.11.171) 56(84) bytes of data. 
64 bytes from 192.168.11.171: icmp_seq=1 ttl=64 time=0.409 ms 
64 bytes from 192.168.11.171: icmp_seq=2 ttl=64 time=0.351 ms 
64 bytes from 192.168.11.171: icmp_seq=3 ttl=64 time=0.288 ms 
^C 
--- 192.168.11.171 ping statistics --- 
3 packets transmitted, 3 received, 0% packet loss, time 2047ms

[root@pcmk-2 ~]# pcs status 
Cluster name: mycluster 
Stack: corosync 
Current DC: pcmk-2 (version 1.1.18-11.el7-2b07d5c5a9) - partition with quorum 
Last updated: Thu May 17 03:51:59 2018 
Last change: Thu May 17 03:48:32 2018 by root via cibadmin on pcmk-1 

2 nodes configured 
1 resource configured 

Online: [ pcmk-2 ] 
OFFLINE: [ pcmk-1 ] 

Full list of resources: 

ClusterIP      (ocf::heartbeat:IPaddr2):       Started pcmk-2 

Daemon Status: 
 corosync: active/disabled 
 pacemaker: active/disabled 
 pcsd: active/enabled


以上说明 ClusterIp 工作在pcmk-2节点
问题： pacemaker是基于整机的故障处理（必然kill httpd能够实现故障切换不？）

