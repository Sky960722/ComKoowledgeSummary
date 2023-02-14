# Linux的硬盘和文件存储系统

## 总结

1. Linux格式化硬盘通常有两种方式，分别是MBR分区和GPT分区。MBR分区最多支持2TB的容量，而GPT分区则超过2TB。在这个基础上，可以通过软件磁盘阵列或者LVM动态加载空间。

2. LInux如果想要使用这个硬盘，在对这个硬盘分区之后，还需要进行格式化，然后挂在格式化后的文件存储系统。文件存储系统主要分为三个部分：

   1. 超级区块：主要记录此文件系统的整体信息
   2. inode：记录文件的属性
   3. 数据区块：实际记录文件的内容

3. 常见的文件存储系统分为：ext2、ext3、ext4和xfs。

4. 查看磁盘与目录的容量：

   ~~~
   df：列出文件系统的整体磁盘使用量
   du：查看文件系统的磁盘使用量（常用在查看目录所占磁盘空间）
     -s：仅列出总量，而不列出每个各别的目录占用容量
     -S：不包括子目录下的总计
   ~~~

5. 新增磁盘的操作步骤

   1. 对磁盘分区（MBR，GPT）
   2. 对该磁盘分区格式化
   3. 挂载该分区

   ~~~shell
   查看磁盘状态：
   lsblk：列出系统上的所有磁盘列表
   blkid：列出设备的UUID等参数
   parted：列出磁盘的分区表类型与分区信息
   
   磁盘分区：
   gdisk：GPT分区命令
   fdisk：MBR分区命令
   partprobe：更新Linux分区表信息（用于对硬盘分区操作完成后）
   
   磁盘分区格式化：
   mkfs.xfs:创建文件系统
   
   文件系统挂载与卸载：
   mount：挂载设备文件
   umount：卸载设备文件
   ~~~

6. 逻辑卷管理器

   LVM：Logical Volume Manager，逻辑卷管理器。

   定义：将几个物理分区通过软件组合成一块独立的硬盘，屏蔽操作系统和不同硬盘之间的差异，再将这块硬盘划分成可使用的分区(LV)。

   用途：动态扩展硬盘容量，无需重启换盘。

7. 逻辑卷管理器分为4个部分

   1. 物理卷（Physcial Volume，PV）：将实际的物理分区调整为LVM最底层的物理卷
   2. 卷组（Volume Group VG）
   3. 物理扩展块（PhyScial Extent PE）：LVM最小的存储数据单位
   4. 逻辑卷（Logicall Volume LV）：将VG分区
   
8. 操作步骤：
   
   1. 对不同硬盘进行分区
   2. 创建PV
   3. 合并PV为VG
   4. 切分VG为LV
   5. mkfs
   
   ~~~
   PV：
   pvcreate：将物理分区建立成为PV
   pvscan：查找目前系统里面任意具有PV的磁盘
   pvdisplay：显示出目前系统上面的PV状态
   pvremove：将PV属性删除，让该分区不具有PV属性。
   
   VG：
   vgcreate：建立VG
   vgscan：查找系统上面是否有VG存在
   vgdisplay：显示目前系统上面的VG状态
   vgextend：在vg内增加额外的pv
   vgreduce：在vg内删除pv
   vgchange：设置vg是否启动
   vgremove：删除一个vg
   
   LV:
   lvcreate：建立lv
   lvscan：查询系统上面的lv
   lvdisplay：显示系统上面的lv状态
   lvextend：在lv里面增加容量
   lvreduce：在lv里面减少容量
   lvremove：删除一个lv
   lvresize：对LV进行容量大小的调整
   ~~~
   
   
   
   

​		
