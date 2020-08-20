+ 内核编译
+ 模块参数
+ rpm & yum
   + 解压rpm文件
+ 提高打印等级
+ 压缩/解压缩
+ chmod修改文件权限
+ 通过perf调试性能
+ ubuntu
   + 设置ubuntu启动DHCP获取IP
   + ubuntu源码仓库
+ centOS
   + ip addr使用
   + centOS wiki
   + centOS ISO下载
   + centOS 源码包下载
   + centOS rpm制作指南
   + CI上安装基础centOS系统
   + 读取编译选项
   + centOS 配置grub cfg
   + centOS 配置DHCP
   + centOS 关闭模块驱动自动加载
   + 删除开机多余kernel
   + centOS yum源修改为内源
   + 关闭防火墙
   + CentOS CI版本建立内源（脚本）
+ EulerOS
   + 修改启动参数
   + 修改kdump的启动参数
   + Euler SSH 无法连接问题的解决
+ OS命令
   + SCSI
   + PCIe
   + NUMA

# 内核编译
``` C
export ARCH=arm64
export CROSS_COMPILE=aarch64-linux-gnu-
make defconfig
make -j64

// 更加严格的编译告警
make C=2 V=2 -j64
```

# 模块参数
/sys/module/<模块名称>/parameters

# rpm & yum

## 解压rpm文件
rpm2cpio xx.rpm | cpio -div

## 查看rpm包版本
rpm -q rdma-core

## 安装rpm包
rpm -ivh xxx.rpm

## 通过yum降级软件
yum downgrade libibverbs

降级软件需要注意： 
+ 1.只能降级到源支持的版本，如果源中没有该版本的软件，则无法降级；
+ 2.软件降级时会自动把依赖它的软件全部删除，如果需要一并降级，应该把依赖它的软件也一并加入这个命令

## yum查看工具的可用版本
yum list docker-ce --showduplicates | sort -r

## yum 删除软件
yum remove libibverbs

如果在列表中没有找到你需要的版本，那么说明你的源里面没有该版本的安装包

## 本地iso yum源安装软件的方法
+ 使用iso文件挂载，先在设置里面DVD项选择本地路径，将iso文件挂载上去。
``` C
连接BMC，在CD/DVD选项中，通过 镜像文件 的方式加载
```
+ 使用mount命令挂载光盘到mnt路径下面，或者指定的文件夹路径下。
``` C
mount /dev/cdrom /mnt/
```
+ 更改配置文件，/etc/yum.repos.d/CentOS-Media.repo,内容如下（以centOS 7为例）： 
``` C
[c7-media]
name=centos-$releasever - Media
baseurl=file:///home/luo/iso_mnt/     //这一行是ISO挂载的路径
gpgceck=1
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```
+ 更新yum
``` C
  yum clean all
  yum makecache
```

# 提高打印等级
echo 15 > /proc/sys/kernel/printk

# 压缩/解压缩
## zip
压缩服务器上当前目录的内容为xxx.zip文件
``` C
zip -r xxx.zip ./*
```
解压zip文件到当前目录
``` C
unzip filename.zip
```
## gz
``` C
解压：tar zxvfp FileName.tar.gz 
压缩：tar zcvfp FileName.tar.gz DirName
```

# chmod修改文件权限
## 递归修改文件夹
``` C
chmod -R 777 xxx
```

# 通过perf调试性能

## 参考blog
https://www.cnblogs.com/arnoldlu/p/6241297.html

## 通过perf record记录，通过perf report查看记录
``` C
perf record -g -F 99 -a

perf report --call-graph none
```
# ubuntu

## 设置ubuntu启动DHCP获取IP
/etc/network/interfaces 网络配置文件

## ubuntu源码仓库
``` C
git clone https://git.launchpad.net/~ubuntu-kernel/ubuntu/+source/linux/+git/bionic
```

# centOS

## ip addr使用
``` C
ip addr add 192.168.21.22/24 dev eth2
```
## centOS wiki
https://wiki.centos.org/HowTos/Custom_Kernel

## centOS ISO下载
http://archive.kernel.org/centos-vault/altarch/

## centOS 源码包下载
http://vault.centos.org/centos/7/os/Source/

## centOS rpm制作指南（主要为打bugfix patch的用途，对于回退patch无帮助）
http://3ms.huawei.com/km/groups/3845729/blogs/details/6377943

## CI上安装基础centOS系统
``` C
ISO： auto-install-daily.iso
      auto-install-daily_nvme.iso

上面这两个ISO放在 02_Skill\20_手册\12_OS\01_CentOS 7.6\02_ISO

这两个ISO是自动安装包，auto-install-daily.iso会默认安装在sda盘，auto-install-daily_nvme.iso会默认安装在ES3000上面。
注意在BMC->配置->系统启动项->引导介质->光驱，将boot从光驱中启动
```

## 读取编译选项
cat /boot/config-$(uname -r)

## centOS 配置grub cfg
``` C
如果要修改CentOS的启动参数，先修改 /etc/default/grub

在下面这一行中加入你需要的启动参数
GRUB_CMDLINE_LINUX="crashkernel=auto net.ifnames=0 console=ttyAMA0,115200"

然后通过grub2-mkconfig命令写入系统的grub
grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg
```
## centOS 配置DHCP
``` C
vi /etc/sysconfig/network-scripts/ifcfg-eth0

修改为以下配置：
NM_CONTROLLED设置成no，ONBOOT设置成yes BOOTPROTO设置成dhcp，最后保留DEVICE HWADDR，NM_CONTROLLED ， ONBOOT，BOOTPROTO这五项，其余全注释掉#

重启网络服务/etc/init.d/network restart

这个重启网络服务器的操作失败了，也没关系
```

## centOS 关闭模块驱动自动加载
``` C
在启动参数里面加上 modprobe.blacklist=hns3 即可
或者

vim /usr/lib/modprobe.d/dist-blacklist.conf  //这个没有效果，启动参数有效果，用hane可以屏蔽1616的网口
在末尾加上以下命令
blacklist hns3

blacklist hns_dsaf
blacklist hns_mdio
blakclist hns_enet_drv
blacklist hnae


各种例子：
禁止USB： 在grub的命令参数加上 usbcore.nousb
禁止SAS：  在grub的命令参数加上 modprobe.blacklist=hisi_sas_v2_hw
```

## 删除开机多余kernel
``` C
rpm -qa | grep kernel
yum remove -y kernel-xxx
```

## centOS yum源修改为内源
``` C
1.配置DNS
1) vi /etc/resolv.conf
2) 加入如下内容：
nameserver 10.189.32.59
nameserver 172.30.163.193                                                                                             
nameserver 172.30.163.142                                                                                             
nameserver 10.140.8.225

2.配置hosts
1）vi /etc/hosts
172.30.163.193 mirrors.huaweicloud.com
172.30.163.142 repo.huaweicloud.com
10.140.8.225 code-sh.rnd.huawei.com

这时候 可以 ping mirrors.huaweicloud.com

2.设置yum源的repo
vi /etc/yum.repos.d/CentOS-Base.repo
具体值如下，但是D2实验室的centOS系统里面已经填充这些信息，如果有，就不需要了
[base]
name=CentOS-$releasever - Base - mirrors.huaweicloud.com
baseurl=https://mirrors.huaweicloud.com/centos-altarch/7/os/$basearch/
#mirrorlist=https://mirrorlist.centos.org/?release=$releasever&arch=$basearch&ree
po=os
gpgcheck=0
gpgkey=https://mirrors.huaweicloud.com/centos-altarch/7/os/$basearch/RPM-GPG-KEYY-CentOS-7-$basearch

#released updates
[updates]
name=CentOS-$releasever - Updates - mirrors.huaweicloud.com
baseurl=https://mirrors.huaweicloud.com/centos-altarch/7/updates/$basearch/
#mirrorlist=https://mirrorlist.centos.org/?release=$releasever&arch=$basearch&ree
po=updates
gpgcheck=0
gpgkey=https://mirrors.huaweicloud.com/centos-altarch/7/os/$basearch/RPM-GPG-KEYY-CentOS-7-$basearch

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras - mirrors.huaweicloud.com
baseurl=https://mirrors.huaweicloud.com/centos-altarch/7/extras/$basearch/
#mirrorlist=https://mirrorlist.centos.org/?release=$releasever&arch=$basearch&ree
po=extras
gpgcheck=0
gpgkey=https://mirrors.huaweicloud.com/centos-altarch/7/os/$basearch/RPM-GPG-KEYY-CentOS-7-$basearch

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus - mirrors.huaweicloud.com
baseurl=https://mirrors.huaweicloud.com/centos-altarch/7/centosplus/$basearch/
#mirrorlist=https://mirrorlist.centos.org/?release=$releasever&arch=$basearch&ree
po=centosplus
gpgcheck=0
enabled=0
gpgkey=https://mirrors.huaweicloud.com/centos-altarch/7/os/$basearch/RPM-GPG-KEYY-CentOS-7-$basearch

3.备份删除/etc/yum.repos.d/下其他源文件，只保留自己要用的那个repo，比如上面这个
4.更新yum源
  yum clean all
  yum makecache
```
## 关闭防火墙
systemctl stop firewalld

## CentOS CI版本建立内源（脚本）
``` C
 #!/bin/sh -ex
# **********************************************************
# * Author        : lwx459299
# * Email         : liucaili2@huawei.com
# * Create time   : 2018-12-24 09:48
# * Last modified : 2018-12-24 10:55
# * Filename      : config_centos_repo.sh
# * Description   : auto config centos to use 172.19.20.112 repo 
# * usage         : bash config_centos_repo.sh
# **********************************************************
repo_version=$1
echo ${repo_version}
# config repo
cd /etc/yum.repos.d
ls
rm -rf *
ls
 
#mkdir origin
#mv * origin
 
 
# update arm CentOS.repo
cat << eof > base.repo
[base]
name=base
baseurl=http://172.19.20.112/centos/altarch/7/os/aarch64/
enabled=1
gpgcheck=0
 
[extras]
name=extras
baseurl=http://172.19.20.112/centos/altarch/7/extras/aarch64/
gpgcheck=0
enabled=1
 
[updates]
name=updates
baseurl=http://172.19.20.112/centos/altarch/7/updates/aarch64/
gpgcheck=0
enabled=1
 
eof
 
cat << eof > release.repo
[Estuary]
name=Estuary
baseurl=http://172.19.20.112/estuary-releases/5.0/centos/
enabled=1
gpgcheck=0
 
eof
 
if [[ ${repo_version} =~ "7-5" ]];then
    cat << eof >> release.repo
[estuary-kernel]
name=estuary-kernel
baseurl=http://172.19.20.112/estuary-releases/5.2/centos/
enabled=1
gpgcheck=0
 
eof
else
    cat << eof >> release.repo
[estuary-kernel]
name=estuary-kernel
baseurl=http://172.19.20.112/estuary-releases/kernel-5.3/centos/
enabled=1
gpgcheck=0
 
eof
fi
 
# update arm CentOS.repo
cat << eof > epel.repo
[epel]
name=estuary-epel
baseurl=http://172.19.20.112/epel/epel/
failovermethod=priority
enabled=1
gpgcheck=0
 
eof
 
cd -
yum clean all
yum remove mariadb-libs.aarch64 -y
rm -rf /var/cache/yum/*
ls /etc/yum.repos.d
cd ~
mkdir .pip
cd .pip
cat << eof > pip.conf
[global]
timeout = 60
index-url = http://10.90.31.108:81//simple
trusted-host = 10.90.31.108
[install]
use-mirrors = true
mirrors = http://10.90.31.108:81
eof
 
cd -
ls /etc/yum.repos.d
 
yum install ntpdate -y
sleep 2
ntpdate 10.3.23.46
sleep 2
date
hwclock --systohc
sleep 2
 
 
yum install ca-certificates -y
update-ca-trust force-enable
yum install wget
wget http://10.90.31.79:8083/test_dependents/hwproxy.crt
mv hwproxy.crt /etc/pki/ca-trust/source/anchors/
update-ca-trust extract
sleep 2
 
sleep 8
pids=$(ps -ef | grep dhclient | grep -v grep |awk '{print $2}')
for pid in $pids
do
    if [ -n "${pid}" ];then  kill -9 $pid;fi
done
sleep 2
dhclient
sleep 5
```

## 修改启动参数
``` C
如果要修改EulerOS的启动参数，先修改 /etc/default/grub

在下面这一行中加入你需要的启动参数
GRUB_CMDLINE_LINUX="crashkernel=auto net.ifnames=0 console=ttyAMA0,115200"

然后通过grub2-mkconfig命令写入系统的grub
grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg
```
## 修改kdump的启动参数
/etc/sysconfig/kdump

## Euler SSH 无法连接问题的解决
``` C
通过 service sshd restart 重启 SSH服务报错如下：

Redirecting to /bin/systemctl restart  sshd.service
Job for sshd.service failed because the control process exited with error code. See "systemctl status sshd.service" and "journalctl -xe" for details.

通过 journalctl -xe 查看log如下：
Aug 02 06:15:47 Euler sshd[7267]: /var/empty/sshd must be owned by root and not
Aug 02 06:15:47 Euler systemd[1]: sshd.service: main process exited, code=exited
Aug 02 06:15:47 Euler systemd[1]: Failed to start OpenSSH server daemon.
-- Subject: Unit sshd.service has failed

发现 Failed to start OpenSSH server daemon 这个报错， 网上直接搜索这个报错，有人建议  sshd -t 查看，log如下：

/var/empty/sshd must be owned by root and not group or world-writable.

通过 ls -ld /var/empty/sshd 查看 sshd文件的归属，没有看到有 root 这样的关键字，有可能没有归属到root用户下
drwx--x--x 2 1015 1020 4096 Jan  5  2018 /var/empty/sshd

通过 chown root /var/empty/sshd/ 修改这个文件的root归属，然后 重启ssh服务，问题解决。
```

# OS命令

## SCSI

### 基本命令
+ lsscsi
+ disktool -s or disktool -f a /dev/sdm
+ blkid //可以获取PARTUUID以及一些磁盘信息
+ smp_discover /dev/bsg/expander-0\:0

### smartctl
``` C
查看磁盘速率
smartctl -a /dev/sdb | grep SATA

参考资料：
https://www.cnblogs.com/fiberhome/p/8275961.html

https://blog.csdn.net/pansaky/article/details/86650134
```

### DIF盘状态查看
lsscsi -p

### DIF盘格式化与恢复
``` C
检查是否支持DIF
sg_vpd --page=ei --long /dev/sda |grep SPT

格式化命令：
DIF TYPE1
sg_format --format --fmtpinfo=2 /dev/sda

DIF TYPE3
sg_format --format --fmtpinfo=3 --pfu=1 /dev/sda

恢复成正常盘
sg_format -F -s 512 /dev/sda
```
### PCIe
``` C
lspci -s 0000:74:02 -v

lspci -tv
```
### NUMA
numactl
