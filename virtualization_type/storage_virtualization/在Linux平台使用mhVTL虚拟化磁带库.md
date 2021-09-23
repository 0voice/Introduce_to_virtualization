# 在Linux平台使用mhVTL虚拟化磁带库

## 1. 查看磁带库设备相关信息。

```C
[root@centos1 opt]# lsscsi -g
[0:0:0:0]    disk    ATA      VBOX HARDDISK    1.0   /dev/sda  /dev/sg0
[1:0:0:0]    mediumx STK      L700             0104  -         /dev/sg9
[1:0:1:0]    tape    IBM      ULT3580-TD5      0104  /dev/st0  /dev/sg1
[1:0:2:0]    tape    IBM      ULT3580-TD5      0104  /dev/st1  /dev/sg2
[1:0:3:0]    tape    IBM      ULT3580-TD4      0104  /dev/st2  /dev/sg3
[1:0:4:0]    tape    IBM      ULT3580-TD4      0104  /dev/st3  /dev/sg4
[1:0:8:0]    mediumx STK      L80              0104  -         /dev/sg10
[1:0:9:0]    tape    STK      T10000B          0104  /dev/st4  /dev/sg5
[1:0:10:0]   tape    STK      T10000B          0104  /dev/st5  /dev/sg6
[1:0:11:0]   tape    STK      T10000B          0104  /dev/st6  /dev/sg7
[1:0:12:0]   tape    STK      T10000B          0104  /dev/st7  /dev/sg8
```

一般磁带库有两个机械手(sg),八个驱动器(st),多个插槽(slot)，插槽中的磁带被放入了驱动之后才能被正常的读写，每盘磁带读写完毕，或读写满了需要手动更换下一盘磁带。

## 2. 装载磁带操作。

装载磁带命令格式：mtx -f load

### 1). 查看sg9机械手状态：
```C
[root@centos1 ~]# mtx -f /dev/sg9 status
  Storage Changer /dev/sg9:4 Drives, 43 Slots ( 4 Import/Export )
Data Transfer Element 0:Empty
Data Transfer Element 1:Empty
Data Transfer Element 2:Empty
Data Transfer Element 3:Empty
      Storage Element 1:Full :VolumeTag=E01001L4                            
      Storage Element 2:Full :VolumeTag=E01002L4                            
      Storage Element 3:Full :VolumeTag=E01003L4                            
      Storage Element 4:Full :VolumeTag=E01004L4                            
      Storage Element 5:Full :VolumeTag=E01005L4                            
      Storage Element 6:Full :VolumeTag=E01006L4                            
      Storage Element 7:Full :VolumeTag=E01007L4                            
      Storage Element 8:Full :VolumeTag=E01008L4                            
      Storage Element 9:Full :VolumeTag=E01009L4                            
      Storage Element 10:Full :VolumeTag=E01010L4                            
      Storage Element 11:Full :VolumeTag=E01011L4                            
      Storage Element 12:Full :VolumeTag=E01012L4                            
      Storage Element 13:Full :VolumeTag=E01013L4                            
      Storage Element 14:Full :VolumeTag=E01014L4                            
      Storage Element 15:Full :VolumeTag=E01015L4                            
      Storage Element 16:Full :VolumeTag=E01016L4                            
      Storage Element 17:Full :VolumeTag=E01017L4                            
      Storage Element 18:Full :VolumeTag=E01018L4                            
      Storage Element 19:Full :VolumeTag=E01019L4                            
      Storage Element 20:Full :VolumeTag=E01020L4                            
      Storage Element 21:Empty
      Storage Element 22:Full :VolumeTag=CLN101L4                            
      Storage Element 23:Full :VolumeTag=CLN102L5                            
      Storage Element 24:Empty
      Storage Element 25:Empty
      Storage Element 26:Empty
      Storage Element 27:Empty
      Storage Element 28:Empty
      Storage Element 29:Empty
      Storage Element 30:Full :VolumeTag=F01030L5                            
      Storage Element 31:Full :VolumeTag=F01031L5                            
      Storage Element 32:Full :VolumeTag=F01032L5                            
      Storage Element 33:Full :VolumeTag=F01033L5                            
      Storage Element 34:Full :VolumeTag=F01034L5                            
      Storage Element 35:Full :VolumeTag=F01035L5                            
      Storage Element 36:Full :VolumeTag=F01036L5                            
      Storage Element 37:Full :VolumeTag=F01037L5                            
      Storage Element 38:Full :VolumeTag=F01038L5                            
      Storage Element 39:Full :VolumeTag=F01039L5                            
      Storage Element 40 IMPORT/EXPORT:Empty
      Storage Element 41 IMPORT/EXPORT:Empty
      Storage Element 42 IMPORT/EXPORT:Empty
      Storage Element 43 IMPORT/EXPORT:Empty
```
4个驱动器中没有磁带。

### 2). 将磁带从39号插槽装入0号驱动器：
```C
[root@centos1 ~]# mtx -f /dev/sg9 load 39 0
[root@centos1 ~]# mtx -f /dev/sg9 status
  Storage Changer /dev/sg9:4 Drives, 43 Slots ( 4 Import/Export )
Data Transfer Element 0:Full (Storage Element 39 Loaded):VolumeTag = F01039L5                            
Data Transfer Element 1:Empty
Data Transfer Element 2:Empty
Data Transfer Element 3:Empty
.....
```
成功将39号槽位的磁带装入0号磁带驱动器。

### 3). 检查0号驱动器里磁带的状态：
```C
[root@centos1 ~]# mt -f /dev/st0 status
SCSI 2 tape drive:
File number=0, block number=0, partition=0.
Tape block size 0 bytes. Density code 0x58 (no translation).
Soft error count since last status=0
General status bits on (41010000):
 BOT ONLINE IM_REP_EN
```

### 4). 检查0号驱动器中磁头的位置：
```C
[root@centos1 ~]# mt -f /dev/st0 tell
At block 0.
```

### 5). 向磁带中写入数据：
```C
[root@centos1 ~]# ls
anaconda-ks.cfg  Desktop  install.log  install.log.syslog
[root@centos1 ~]# tar -cvf /dev/st0 Desktop
Desktop/
[root@centos1 ~]# mt -f /dev/st0 tell
At block 0.
```
st打头的表示写入完成后会自动倒带，下次写入会覆盖之前的数据。
```C
[root@centos1 ~]# tar -cvf /dev/nst0 Desktop
Desktop/
[root@centos1 ~]# mt -f /dev/st0 tell
At block 2.
```
nst打头的表示写入完成后不会自动倒带。

## 3. 卸载磁带库操作。

将磁带从0号驱动器拔出放入39号插槽：
```C
[root@centos1 ~]# mtx -f /dev/sg9 unload 39 0
Unloading Data Transfer Element into Storage Element 39...done
```

以上是一些简单的基本操作。


原文作者： 尛样儿

































