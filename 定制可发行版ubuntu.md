###### 1.在工作目录下新建diy_ubuntu文件夹，主要的操作都在这个目录下进行

mkdir diy_ubuntu

###### 2.进入diy_ubuntu， 新建ubuntu-livecd文件夹，将原始iso挂载在/mnt下，更改属性，将所有的文件拷贝到ubuntu-livecd

sudo mount -t isofs, -o loop /path/store/iso/ubuntu-20.04.1-live-server-amd64.iso /mnt
cp -a /mnt/. ubuntu-livecd
chmod -R u+w ubuntu-livecd

###### 3.卸载iso，至此原始iso的作用就完毕了

sudo umount /mnt

###### 4.新建old文件夹，将原始iso内的文件系统挂载到old目录下(挂载是一个好东西，能让你看见二进制文件内的具体结构)

mkdir old
sudo mount -t squashfs -o loop,ro  ubuntu-livecd/casper/filesystem.squashfs old/

###### 5.建立一个2GB大小的文件系统,然后把这个文件当作一个设备文件格式化,新建new文件夹，然后把这个空白文件系统挂载到new上

sudo dd if=/dev/zero of=ubuntu-fs.ext2 bs=1M count=2147
sudo mke2fs ubuntu-fs.ext2
mkdir new
sudo mount -o loop ubuntu-fs.ext2 new

###### 6.把old目录下所有内容拷贝到new上，至此old就完成自己的任务，可以卸载了

sudo cp -a old/. new
sudo umount old/

###### 7.将本地的系统配置拷贝到new下，启动定制的目标系统

sudo cp /etc/resolv.conf new/etc/
sudo mount -t proc -o bind /proc new/proc
sudo chroot new /bin/bash

###### 8.这样就进入到定制的目标系统了，可以进行各种操作，包括往new目录内添加文件等

...

###### 9.离开定制的系统，卸载本地

exit
sudo umount new/proc
sudo rm new/etc/resolv.conf

###### 10.回到初始状态，,把manifest重新整一遍

sudo chroot new/ dpkg-query -W --showformat='${Package} ${Version}\n' > ubuntu-livecd/casper/filesystem.manifest

###### 11.做一下 磁盘整理

sudo dd if=/dev/zero of=new/dummyfile
sudo rm new/dummyfile

###### 12.重新压缩系统

sudo rm ubuntu-livecd/casper/filesystem.squashfs
cd new
sudo mksquashfs . ubuntu-livecd/casper/filesystem.squashfs

###### 13.改动都保存了。现在把new卸载

cd ..
sudo umount new

###### 14.把文件的md5重新算一下

cd ubuntu-livecd
sudo find . -type f -print0 |xargs -0 md5sum |sudo tee md5sum.txt

###### 15.建立光盘镜像

cd ..
sudo mkisofs -o ubuntu-new.iso -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -r -V "My Cool Ubuntu Live CD" -cache-inodes -J -l ubuntu-livecd

