
<br>
1. 公司OS版本区别 Rocky / Linx ，分别使用场景是什么

<br>
2. linx-4.2-x86_64+pacemaker+drbd+bind双机热备安装使用手册.odt 
gitlab --- 双机安装部署.pdf
文档是一样的？

<br>
3. gitlab 文件布局：
/doc 文档
/lnxsn
/lxHA4.2 --- 猜测 专门for linx4.2系统
/lxHA6.0.80 猜测 专门为 rock 系统
（好像linx 系统也有4。2版本 6。0版本 ， rock也有4.0/6。0版本）

/dmserer --what's this

保存的crm配置
/var/lib/heartbeat/crm/

注意multicast 地址配置一定要不一样。。。。。！！！！！


问题， 重启其中一个机器， 当机器启动后， 状态会乱


3.3.3 WebSite资源分析
    1）这里WebSite主要采用了apache服务器，添加了它的配置文件和监控器时间。
primitive WebSite ocf:heartbeat:apache \
        params configfile="/etc/apache/httpd.conf" \
        op monitor interval="1min" \
        meta target-role="Started" is-managed="true" resource_failure_stickiness="-20" migration-threshold="2" failure-timeout="20s"

primitive WebSite ocf:heartbeat:apache \
        params configfile="/etc/apache/httpd.conf" \
        op monitor interval="1min" \
        meta target-role="Started" is-managed="true" resource_failure_stickiness="-20" migration-threshold="2" failure-timeout="20s"

    重点在于将clusterIP和WebSite两个资源绑定在一起，这样外部才能通过集群IP访问正在运行WebSite的集群节点，如下。
colocation website-with-ip inf: WebSite ClusterIP
  2）WebSite验证：
在游览器中输入ClusterIP，例如：
192.168.11.244
就可以看到网页了。
http://localhost/server-status


colocation website-with-ip inf: WebSite rsc-vip

primitive WebSite ocf:heartbeat:apache  \
      params configfile=/etc/apache/httpd.conf \
      statusurl="http://localhost/server-status" \
      op monitor interval=1min



<Location /server-status>
    SetHandler server-status
    #Require host *
    Require ip any

</Location>



Require ip ip.address


3.2  Apache资源注意事项
3.2.1 查看Apache是否运行
Apache进程是否启动可通过如下命令查看：
# 
找到其中：
├─httpd───6*[httpd]
这一项若有则说明apache正在运行，可通过如下命令测试Apache是否可用:
# /etc/init.d/apache start
# /etc/init.d/apache stop
3.2.2 在corosync进程起来前确保Apache不运行
1）crm完全配置完成后（3.hosts3节完成后），运行命令：
# crm_mon
如果出现：
Failed actions:
    WebSite_start_0 (node=LinxA, call=8, rc=-2, status=Timed Out): unknown exec error)
2）此时，退出并运行命令：
# pstree
如果出现：
├─httpd───5*[httpd]
则说明apache在corosync前起来)
3）此时只需要运行如下即可：
# killall -9 corosync
# /etc/init.d/apache stop
# /etc/init.d/corosync start
         这是因为apache默认会使用配置文件/etc/apache/httpd.conf中的ServerName作为ip，而corosync启动apache希望使用的是3.3节中配置的cluster IP，所以corosync启动apache时一定会传参，因此要注意apache一定要交给corosync来管理，而不能人为启动。




primitive WebSite ocf:heartbeat:apache \
        params configfile="/etc/apache/httpd.conf" \
        op monitor interval="1min" \
        meta target-role="Started" is-managed="true"  migration-threshold="2" failure-timeout="20s"


colocation website-with-ip inf: WebSite ClusterIP


node cdns-30-1
node cdns-30-2


primitive ClusterIP ocf:heartbeat:IPaddr \
        params ip="192.168.11.132" cidr_netmask="255.255.255.0" \
        op monitor interval="30s" migration-threshold="2" failure-timeout="20s"
primitive WebSite ocf:heartbeat:apache \
        params configfile="/etc/apache/httpd.conf" statusurl="http://localhost/server-status" \
        op monitor interval="1min" \
        meta target-role="Started" is-managed="true" migration-threshold="2" failure-timeout="20s"
primitive st-ssh stonith:external/ssh \
        params hostlist="cdns-30-1 cdns-30-2" \
        meta target-role="Started" is-managed="true"
#location cli-prefer-WebSite WebSite rule $id="cli-prefer-rule-WebSite" inf: #uname eq cdns-30-1
location prefer-LinxAAA WebSite 200: cdns-30-1
colocation website-with-ip inf: WebSite ClusterIP
property $id="cib-bootstrap-options" \
        dc-version="1.1.7-ee0730e13d124c3d58f00016c3376a1de5323cff" \
        cluster-infrastructure="openais" \
        expected-quorum-votes="2" \
        stonith-enabled="true" \
        stonith-action="reboot" \
        stonith-enabled="false" \
        no-quorum-policy="ignore" \
        last-lrm-refresh="1527077328"

stonith-enabled="false" \
        no-quorum-policy="ignore" \


总结：Websit的问题是》机器名称最好规范（没有空格等）
  1 node LinxAAA \
  2         attributes standby="off"
  3 node LinxBBB \
  4         attributes standby="off"
  5 primitive ClusterIP ocf:heartbeat:IPaddr \
  6         params ip="192.168.11.178" cidr_netmask="255.255.255.0" \
  7         op monitor interval="30s" migration-threshold="2" failure-timeout="20s"
  8 primitive WebSite ocf:heartbeat:apache \
  9         params configfile="/etc/apache/httpd.conf" statusurl="http://localhost/server-status" \
 10         op monitor interval="1min" \
 11         meta target-role="Started" is-managed="true" migration-threshold="2" failure-timeout="20s"
 12 primitive st-ssh stonith:external/ssh \
 13         params hostlist="LinxAAA LinxBBB" \
 14         meta target-role="Started" is-managed="true"
 15 #location cli-prefer-WebSite WebSite rule $id="cli-prefer-rule-WebSite" inf: #uname eq LinxAAA
 16 location prefer-LinxAAA WebSite 200: LinxAAA
 17 colocation website-with-ip inf: WebSite ClusterIP
 18 property $id="cib-bootstrap-options" \
 19         dc-version="1.1.7-ee0730e13d124c3d58f00016c3376a1de5323cff" \
 20         cluster-infrastructure="openais" \
 21         expected-quorum-votes="2" \
 22         stonith-enabled="true" \
 23         stonith-action="reboot" \
 24         stonith-enabled="false" \
 25         no-quorum-policy="ignore" \
 26         last-lrm-refresh="1527077328"
~                                             



1 公司OS版本区别 Rocky / Linx ，分别使用场景是什么


2. linx-4.2-x86_64+pacemaker+drbd+bind双机热备安装使用手册.odt
gitlab --- 双机安装部署.pdf

文档是一样的？


实验步骤：
1 安装OS rocky6.0.42.41-x86_64-security-20161201.iso
(/home/chaodongqu)
chaodongqu
R0ck9

rock1 192.168.1.130 host-name :cdns-30-1
rock1 192.168.1.131 host-name: cdns-30-2


2. 安装drbd
pkgadd drbd#8.3.16-x86_64-linx-Rocky4.3.pkg.tar.gz

3。 install pacemakter

lxha2-2014-09-29/install.sh

4. 建立 pacemaker 所需的组名和用户名
创建用户名 hacluster 和组名 haclient
groupadd haclient
useradd -g haclient hacluster
passwd hacluster
输入 hacluster 口令(如 hacluster)
5. 修改 pacemaker 的相关文件的属主
find / -type d -name “heartbeat” -exec \
chown -R hacluster.haclient {} \;
chown -R hacluster.haclient /var/lib/pengine

6 修改 /etc/hosts
127.0.0.1       localhost ------------一定要去掉
192.168.11.130  localhost.in.linx localhost
192.168.11.130  cdns-30-1
192.168.11.131  cdns-30-2

127.0.0.1	 cdns-30-2
192.168.11.130  cdns-30-1
192.168.11.131  cdns-30-2

~                          
7 .没有理解
/etc/sysconfig/modules,在其中增加 drbd 和 lsm_linx


8 配置corosync
为什么 corosync 的 interface 要配置多个？


文档问题：
ssh-keygen -t dsa -f ~/.ssh/id_dsa -N ""

scp -r ~/.ssh/  root@cdns-30-2:~/

local0.


#
auth,authpriv.*                 /var/log/auth.log
*.*;auth,authpriv.none          -/var/log/syslog
#cron.*                         /var/log/cron.log

chmod u+rw ~/.ssh/id_rsa 

注意： multicastaddress 不可修改

保存的crm配置
/var/lib/heartbeat/crm/

primitive rsc-fs ocf:heartbeat:Filesystem \
params
device="/dev/drbd/by-res/r0"
fstype="ext3" options="acl"
directory="/bind"
primitive rsc-ha-script lsb:cdns-ha \
op monitor interval="30s" timeout="15s" \
meta migration-threshold="10" target-role="Started"
primitive rsc-vip ocf:heartbeat:IPaddr \
-19-linx4.2-x86_64+pacemaker+drbd+bind 安装配置使用手册-V0.1
params
ip="10.30.1.1"
broadcast="10.30.1.255" nic="eth0" \
cidr_netmask="255.255.255.0"
op monitor interval="1m" timeout="30s"



primitive rsc-vip ocf:heartbeat:IPaddr \
params ip="192.168.11.132" broadcast="192.168.11.255" nic="eth0" \
cidr_netmask="255.255.255.0"



node LinxAAA \
        attributes standby="off"
node LinxBBB \
        attributes standby="off"
primitive ClusterIP ocf:heartbeat:IPaddr \
        params ip="192.168.11.132" cidr_netmask="255.255.255.0" \
        op monitor interval="30s" migration-threshold="2" failure-timeout="20s"
primitive WebSite ocf:heartbeat:apache \
        params configfile="/etc/apache/httpd.conf" statusurl="http://localhost/server-status" \
        op monitor interval="1min" \
        meta target-role="Started" is-managed="true" migration-threshold="2" failure-timeout="20s"
primitive st-lxha stonith:external/lxha \
        params hostlist="LinxAAA LinxBBB" \
        meta target-role="Started" is-managed="true"
clone fencing st-lxha
#location cli-prefer-WebSite WebSite rule $id="cli-prefer-rule-WebSite" inf: #uname eq LinxAAA
location prefer-LinxAAA WebSite 200: LinxAAA
location prefer-LinxBB WebSite 400: LinxBBB
colocation website-with-ip inf: WebSite ClusterIP
property $id="cib-bootstrap-options" \
        dc-version="1.1.7-ee0730e13d124c3d58f00016c3376a1de5323cff" \
        cluster-infrastructure="openais" \
        expected-quorum-votes="2" \
        stonith-enabled="true" \
        stonith-action="reboot" \
        no-quorum-policy="ignore" \
        last-lrm-refresh="1527077328"



stonish 验证
/sbin/lxhacli 192.168.11.131 11123 0 reset

####总结： stonith问题： 

版本需要最新的（开始使用到一个错误版本） 。 标准ssh stonith是有问题的。
配置缺乏
stonith-enabled="true" \
        stonith-action="reboot" \

一定要用 验证命令先测试



##### Appach 问题：
修改Hosts对应名称为简单名称（linxAAAA） 以前的cdns-30-1 有问题。












