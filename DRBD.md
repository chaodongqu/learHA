https://blog.csdn.net/tjiyu/article/details/52723125

 The version of drbd84-utils shipped with CentOS 7.1 has a bug in the Pacemaker integration script. Until a fix is packaged, download the affected script directly from the upstream, on both nodes:

# curl -o /usr/lib/ocf/resource.d/linbit/drbd 'http://git.linbit.com/gitweb.cgi?p=drbd-utils.git;a=blob_plain;f=scripts/drbd.ocf;h=cf6b966341377a993d1bf5f585a5b9fe72eaa5f2;hb=c11ba026bbbbc647b8112543df142f2185cb4b4b'

This is a temporary fix that will be overwritten if the package is upgraded. 

问题： 网络不可达 git.linbit.com


[root@pcmk-1 ~]# firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.11.124" port port="7789" protocol="tcp" accept'

[root@pcmk-1 ~]# firewall-cmd --reload

[root@pcmk-2 ~]# firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.11.125" port port="7789" protocol="tcp" accept'
[root@pcmk-2 ~]# firewall-cmd --reload
