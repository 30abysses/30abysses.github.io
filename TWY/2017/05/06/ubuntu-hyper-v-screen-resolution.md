> http://www.30abysses.com/TWY/2017/05/06/ubuntu-hyper-v-screen-resolution.md
> by TW Yang <twy@30abysses.com> 2017-05-06 CC-BY-4.0

# ＣＳ： Ubuntu on Hyper-V, 螢幕解析度

這篇是在 Windows 10 上用 Hyper-V  跑 Ubuntu Desktop 17.04,  更改螢幕解析
度的筆記。

Ubuntu  在 Hyper-V  上預設的的螢幕解析度只有 1152x864 ，以
"ubuntu Hyper-V screen resolution"  等關鍵字 Google 後會發現這是個ＦＡＱ
，解決方法大致上是：

1.  安裝 VM driver  。
2.  更改 `/etc/default/grub`  設定。
3.  執行 `update-grub`  。
4.  Reboot Ubuntu VM  。


##  安裝 VM driver

以 `apt list --installed linux*`  指令可以列出已安裝的相關套件；在 VM 環
境裡需要加裝 `linux-image-extra-virtual`  ，指令如下：

```
sudo apt-get install linux-image-extra-virtual
```


## 更改 `/etc/default/grub`  設定

在 `GRUB_CMDLINE_LINUX_DEFAULT` 加入 `video=hyperv_fb:1920x1080`  ，例如
：

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash video=hyperv_fb:1920x1080"
```


### `1920x1080` 是 Ubuntu 在 Hyper-V  螢幕解析度的上限。

Ubuntu 17.04  的 [ReleaseNotes][1]  記載它使用 Linux kernel 4.10  ；
`1920x1080` 這個神奇數字來自於原始碼中的註解：

[https://github.com/torvalds/linux/blob/v4.10/drivers/video/fbdev/hyperv_fb.c#L18-L35][2]

```
/*
 * Hyper-V Synthetic Video Frame Buffer Driver
 *
 * This is the driver for the Hyper-V Synthetic Video, which supports
 * screen resolution up to Full HD 1920x1080 with 32 bit color on Windows
 * Server 2012, and 1600x1200 with 16 bit color on Windows Server 2008 R2
 * or earlier.
 *
 * It also solves the double mouse cursor issue of the emulated video mode.
 *
 * The default screen resolution is 1152x864, which may be changed by a
 * kernel parameter:
 *     video=hyperv_fb:<width>x<height>
 *     For example: video=hyperv_fb:1280x1024
 *
 * Portrait orientation is also supported:
 *     For example: video=hyperv_fb:864x1152
 */
```

[1]: https://wiki.ubuntu.com/ZestyZapus/ReleaseNotes
[2]: https://github.com/torvalds/linux/blob/v4.10/drivers/video/fbdev/hyperv_fb.c#L18-L35


## 執行 `update-grub`

```
sudo update-grub
```


## Reboot Ubuntu VM

```
shutdown -r
```



# 結論

重開機(reboot)之後，此 Ubunbu (on Hyper-V)  應該就會以新設定的螢幕解析度
（在這個例子中是 1920x1080  ）取代預設的 1152x864 。
