# Virtualbox 安装安卓虚拟机

话说微软的内置安卓到底什么时候能发布啊？

[Virtualbox下载](https://www.virtualbox.org/wiki/Downloads)

## 安卓虚拟机

- [安装步骤](https://www.howtogeek.com/164570/how-to-install-android-in-virtualbox/)
  
  建议内存设置8G以上，不然很卡

- [安卓iso下载](http://www.android-x86.org/)
  
  本文使用版本：android-x86-8.1-r6.iso

  （x86版本类似chromebook的安卓版本，之前对安卓确实知之甚少了）

- [解决无法显示界面的问题](https://askubuntu.com/questions/1088222/boot-process-of-android-installed-in-virtualbox-seems-to-get-stuck)

    system setting->graphics controller->改为VBoxVGA

- 解决无法联网的问题：
  - 系统设定-> network -> Attached to -> 选择Bridged network （这样宿主机作为网桥把网连给虚拟机）
  <!-- - 启动进入debug -->
  - 启动进系统，Alt+F1进入终端
  ```
  ifconfig
  ```
  我的系统内可以看到wlan0和wifi_eth，猜测wifi_eth适用于桥接的虚拟有线端口，只要确保有这两个端口就ok
  - 网上很多方案有提到`netcfg`和`dhcpd`启动eth0的方案，猜测应该是老的Android版本有对于有线连接的支持，但再android 6之后就取消了对这些命令的支持，因此看到这些讲解就可以直接忽略了
  - Alt+F7推出终端，点击wifi连接
  - 可用wifi列表中应有Virtwifi一项，选择连接

到这里我的vm就可以上网了，如果有遇到其他问题也可以补充给我

- 遇到的问题
  - 即便内存调到16G了，很多应用（如chrome）会闪退
  - 系统画面依然卡顿到影响使用，不知道是不是由于设定了动态调整硬盘空间的原因
  - 很多应用提示与系统不兼容，能安装的应用很多也无法打开
  - 虚拟机的浏览器找到apk，无法下载，无法在线安装
  - 全屏打开应用，没有关闭等控制按键（与chromebook有所不同），只能后台关闭
  - ~~内置google应用，搜索之后完全打不开任何界面~~：更新应用+更新google play框架（更新应用会自动提示）解决

体验很差，懒得研究了，准备使用android studio解决
