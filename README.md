# gk41-pve-ovmf
gk41's OVMF,support physical display with GVT-D using PVE6.3!纯UEFI启动支持直通核显到物理显示器输出！
pure UEFI boot, not legacy or csm! gk41 mini pc,cpu:J4125 gpu:UHD600. 对于纯UEFI系统启动，直通核显到物理显示器，几乎是不可能的任务，尤其是VM装WIN10。 所有已知的核显直通教程，几乎都是教你用VNC装完系统后再改conf文件添加参数直通，等你改完参数启动机器基本都是黑屏无输出的，不管是HDMI亦或DP接口。网上教程的成功关键在于你必须用legacy传统模式启动，不然你连VBIOS都dump不出来，更谈不上成功直通到显示器。但是像我的小主机GK41压根没有CSM和legacy模式，BIOS只包含GOP驱动没有VBIOS。。。使用我提供的OVMF+args来启动VM虚拟机，你可以直通安装系统，而不是安装系统后直通，如果真心直通不了，也不会太浪费你的时间！如果你的小主机只支持UEFI启动，但是不是同一个核显，可以用UEFITool工具提取gop与vbt，替换我提供的OVMF文件中的GOP驱动与VBT表。WIN10 直通安装没有任何问题，可以正常直通到物理显示器，但是一旦装了UHD驱动就黑屏了，只能RDP远程进去，暂时没找到解决办法！ubuntu测试完美，直通安装与安装完显示器都能在4K 60HZ下工作。Good Luck!

PVE6.3直通核显前主机HOST操作，与网上通用教程一致：

A.首先编辑GRUB配置文件：
vi /etc/default/grub (覆盖同一行)

   GRUB_CMDLINE_LINUX_DEFAULT="quiet kvm.ignore_msrs=1 intel_iommu=on iommu=pt video=efifb:off vfio-pci.ids=8086:3185,8086:3198 modprobe.blacklist=i915,snd_hda_intel,snd_sof_pci,mei_me,snd_hda_codec_hdmi,snd_hda_codec_realtek"
   
更新GRUB：
  update-grub
  
B.添加所需的系统模块（驱动）：
vi /etc/modules

vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd 

更新：
update-initramfs -u

重启：
reboot

PVE6.3环境，minisforum-gk41-mini-pc直通显卡到物理显示器实现4K输出:

1.复制OVMF_CODE.fd+OVMF_VARS.fd覆盖PVE下的相同文件(/usr/share/pve-edk2-firmware/);
2.复制igd.rom到PVE下的kvm目录(/usr/share/kvm/);
3.对照100.conf编辑自己的虚拟机配置文件；
4.最好是SSH下，用qm start 100之类的启动虚拟机，这样启动的好处是万一有问题，能有错误提示！
经测试完美支持ubuntu20.04，deepin v20（虚拟机选windows类型）,4K 60HZ输出。

与现有直通最大不同：添加本机HOST提取的gop与vbt编译到ovmf，使BIOS启动就有图形输出能力，直接直通到物理显示器，安装系统，而不是等系统装完了才直通！
参考以下文档：
Add the VBT and GOP drivers to the OVMF
https://projectacrn.github.io/latest/tutorials/gpu-passthru.html
efirom IgdAssignmentDxe to igd.rom
https://bugzilla.tianocore.org/attachment.cgi?id=165&action=edit
