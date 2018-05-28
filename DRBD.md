https://blog.csdn.net/tjiyu/article/details/52723125

 The version of drbd84-utils shipped with CentOS 7.1 has a bug in the Pacemaker integration script. Until a fix is packaged, download the affected script directly from the upstream, on both nodes:

# curl -o /usr/lib/ocf/resource.d/linbit/drbd 'http://git.linbit.com/gitweb.cgi?p=drbd-utils.git;a=blob_plain;f=scripts/drbd.ocf;h=cf6b966341377a993d1bf5f585a5b9fe72eaa5f2;hb=c11ba026bbbbc647b8112543df142f2185cb4b4b'

This is a temporary fix that will be overwritten if the package is upgraded. 

问题： 网络不可达 git.linbit.com


[root@pcmk-1 ~]# firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.11.124" port port="7789" protocol="tcp" accept'

[root@pcmk-1 ~]# firewall-cmd --reload

[root@pcmk-2 ~]# firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.11.125" port port="7789" protocol="tcp" accept'
[root@pcmk-2 ~]# firewall-cmd --reload

# fdisk  问题：
<br>
由于开始没有按照教程设置最小磁盘空间， 导致无法生成逻辑卷， 无法 lvcreate
最后解决办法是：使用以下步骤
<br> fdisk / pvcreate /  vgcreate <br>

2： 新建一个分区（partition）

新建一个主分区（primary partition）或逻辑分区（logical partition）都OK

[root@getlnx20 ~]# fdisk /dev/sdb
Device contains neither a valid DOS partition table, nor Sun, SGI or OSF disklabel
Building a new DOS disklabel with disk identifier 0xaa12f277.
Changes will remain in memory only, until you decide to write them.
After that, of course, the previous content won't be recoverable.
 
Warning: invalid flag 0x0000 of partition table 4 will be corrected by w(rite)
 
WARNING: DOS-compatible mode is deprecated. It's strongly recommended to
         switch off the mode (command 'c') and change display units to
         sectors (command 'u').
 
Command (m for help): n
Command action
   e   extended
   p   primary partition (1-4)
p
Partition number (1-4): 1
First cylinder (1-10443, default 1): 1
Last cylinder, +cylinders or +size{K,M,G} (1-10443, default 10443): 
Using default value 10443
 
Command (m for help): w
The partition table has been altered!
 
Calling ioctl() to re-read partition table.
Syncing disks.
clip_image002

3：创建PV（物理卷）

[root@getlnx20 ~]# pvcreate /dev/sdb1
  Physical volume "/dev/sdb1" successfully created
[root@getlnx20 ~]# pvscan
  PV /dev/sda2   VG VolGroup00   lvm2 [39.51 GiB / 0    free]
  PV /dev/sdb1                   lvm2 [80.00 GiB]
  Total: 2 [119.51 GiB] / in use: 1 [39.51 GiB] / in no VG: 1 [80.00 GiB]

4：创建VG（卷组）

[root@getlnx20 ~]# vgcreate -s 32M VolGroup01 /dev/sdb1
  Volume group "VolGroup01" successfully created
[root@getlnx20 ~]# vgscan
  Reading all physical volumes.  This may take a while...
  Found volume group "VolGroup01" using metadata type lvm2
  Found volume group "VolGroup00" using metadata type lvm2
clip_image003


### DRBD 步骤(官网)
1. prepare disk
2. configure DRDB resource  [ such as wwdataq]
3. create resourece [ drbdadm create-md wwwdata ]
3a. Now, repeat the above commands on the second node. This time, when we check the status, it shows:

note ::set primary....  drbdadm primary --force wwwdata

4. use this disk(resource )
mkfs.xfs /dev/drbd1
mount /dev/drbd1 /mnt


####公司DRBD
1. prepare disk
2. 配置 /etc/drbd.conf
3. create DataBlock 
/etc/sysconfig/modules --- > add DRBD
drbdadm create-md ....

3a. 然后在另一台机器上作同样的操作

4. start service : /etc/rc.d/init.d/drbd start
5. set primary 
drbdsetup /dev/drbd0 primary -o