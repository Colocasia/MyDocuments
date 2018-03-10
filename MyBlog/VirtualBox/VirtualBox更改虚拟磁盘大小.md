
[TOC]

# **VirtualBox 更改虚拟磁盘大小**#

好记性不如烂笔头啊，还是记录一下!

----------

Virtualbox 本身只支持vdi硬盘文件格式的扩容，对vmdk 格式的却不支持。但是却提供vmdk到vdi格式的转化，正好可以利用这一功能进行扩容。<br>

## **<font color=#191970 size=5>1.停止虚拟机</font> ** ##

```
    vagrant halt
```

## **<font color=#191970 size=5>2.装换为vdi</font> ** ##

从Virtualbox页面查看硬盘文件地址（选中虚拟机->右键->设置->存储）。进到文件所在目录后执行
``` 
    VBoxManage clonehd "box-disk1.vmdk" "box-disk1.vdi" --format vdi
```

## **<font color=#191970 size=5>3.给vdi格式硬盘文件扩容</font> ** ##

扩展镜像，此处以扩展到100G为例
``` 
    VBoxManage modifyhd "box-disk1.vdi" --resize 102400
```

## **<font color=#191970 size=5>4.重新挂载磁盘到虚拟机</font> ** ##

``` 
    VBoxManage storageattach centos7_default_1497586433071_72943 --storagectl "SATA Controller" --port 0 --device 0 --type hdd --medium box-disk1.vdi
```

## **<font color=#191970 size=5>5.启动虚拟机</font> ** ##

``` 
    vagrant up
```

## **<font color=#191970 size=5>6.扩展到虚拟机</font> ** ##

``` 
    fdisk -l  
    fdisk /dev/sda
```

以下几步一定要按步骤来

> 1. 按p显示分区表，默认是 sda1 和 sda2。
> 2. 按n新建主分区。
> 3. 按p设置为主分区。
> 4. 输入3设置为第三分区。
> 5. 输入两次回车设置默认磁盘起始位置。
> 6. 输入t改变分区格式
> 7. 输入3选择第三分区
> 8. 输入8e格式成LVM格式
> 9. 输入w执行

重新启动虚拟机

``` 
    reboot
```

创建物理卷

``` 
    pvcreate /dev/sda3
```

查看卷组，扩展到相应卷组，这里以centos为例

``` 
vgdisplay  
vgextend centos /dev/sda3 
```

扩展到相应逻辑卷，这里以/dev/mapper/centos-root为例

``` 
lvextend -l +100%FREE  /dev/mapper/centos-root 
```

更新文件系统，centos7和centos6使用不同命令

CentOS7
``` 
xfs_growfs /dev/mapper/centos-root 
```
CentOS6
``` 
resize2fs /dev/mapper/centos-root  
```

查看目录扩展是否成功
``` 
df -h  
```

大功告成~~

参考文档
>[vagrant管理虚拟机之虚拟机扩展磁盘、根目录](http://blog.csdn.net/zxjxingkong/article/details/62419379)




