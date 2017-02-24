> http://www.30abysses.com/TWY/2017/02/24/windows-10-development-environment.md
> by TW Yang <twy@30abysses.com> 2017-02-24 CC-BY-4.0

# ＣＳ： Windows 10 開發環境 VM

這篇是試用微軟官方釋出的 Windows 10 開發環境 VM 的筆記。



# 微軟官方下載網頁

* [https://developer.microsoft.com/en-us/windows/downloads/virtual-machines][1]

[1]: https://developer.microsoft.com/en-us/windows/downloads/virtual-machines


##  版本

目前(2017-02-24)官方提供兩個版本：

* Windows 10 Enterprise (Evaluation), 企業版（試用）
* Windows 10 Professional (Licensed), 專業版（需要完整授權啟動）

我下載的是 [Windows 10 Enterprise (Evaluation), Hyper-V][2] 版；此試用版
將於 2017-04-08 到期。

[2]: https://aka.ms/windev_VM_hyperv


##  試用版軟體授權

試用版軟體授權軟體授權([Microsoft Software License Terms][3]) 有三點值得
注意：

[3]: https://aka.ms/windowsdevelopervirtualmachineeula

> The Microsoft Software License Terms for the Windows 10 VMs supersede
> any conflicting Windows license terms included in the VMs.
>
> 若與 VM 中的 Windows  授權砥觸，以此份授權書為準。

> 1. INSTALLATION AND USE RIGHTS.
>
> c. You may use the software in the virtual hard disk image only to
> demonstrate and internally evaluate it. You may not use the software
> for commercial purposes. You may not use the software in a live
> operating environment.
>
> 這個試用版 VM 只能拿來展示、評估／試用；不得用於正式運作環境中。

> 4. NO ACTIVATION. To prevent its unlicensed use, the software contains
> activation enforcement technology. Because this is an evaluation-only
> license, you are not licensed to activate the software for any purpose
> even if it prompts you to do so. 
>
> 朕不給的，你不能要（即使系統提示你要啟動，你也不可以啟動這個試用版。）



# 使用

下載下來的 .zip 檔約 20 GB  ，解壓縮之後約 40 GB  。

解壓縮之後就是 Hyper-V VM 的格式，直接從 Hyper-V Manager
"Import Virtual Machine"  ，視硬體資源調整給 VM 的記憶體、 CPU  額度，有
需要的話接上網路，就可直接啟動 VM 。

VM  啟動之後，跑完 Windows Update ，再安裝
Visual Studio 2017 Community Edition RC 後， VM 大小約 55 GB  。

VM  內已有安裝 Visual Studio 2015 Community Edition ，但其試用期已過期，
需要登入微軟帳號才能啟動。



# 結論

基本上，只要 Hyper-V  主機(host)有足夠的硬體資源（記憶體、 CPU  、儲存空
間）的話，沒什麼意外；下載、解壓縮、 import Hyper-V VM  、調整 VM 設定，
就能使用。
