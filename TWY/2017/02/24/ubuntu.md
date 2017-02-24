> http://www.30abysses.com/TWY/2017/02/24/ubuntu.md
> by TW Yang <twy@30abysses.com> 2017-02-24 CC-BY-4.0

# ＣＳ： Ubuntu Desktop 16.10 on Windows 10 Hyper-V

這篇是在 Windows 10 上用 Hyper-V  跑 Ubuntu Desktop 16.10 的筆記。


## 為什麼選 Ubuntu Desktop  ？

因為不想動腦。



# Ubuntu Desktop 官方下載網頁

* [https://www.ubuntu.com/download/desktop][1]

[1]: https://www.ubuntu.com/download/desktop

我選擇目前(2017-02-24)最新版， Ubuntu 16.10 ；下載的 .iso 大約 1.5 GB 。


## Hyper-V  對 Ubuntu 的支援

* [https://technet.microsoft.com/en-us/windows-server-docs/compute/hyper-v/supported-ubuntu-virtual-machines-on-hyper-v][2]

[2]: https://technet.microsoft.com/en-us/windows-server-docs/compute/hyper-v/supported-ubuntu-virtual-machines-on-hyper-v

* 創造新 Hyper-V VM 時，可以使用 Generation 2 VM
* 取消 `Secure Boot`
* 因為此 Ubuntu VM  只用來測試，並無特殊需求，所以不需裝 `linux-virtual`
  等 package



# 安裝

將 VM 設定為 "Boot from DVD Drive"; 把 DVD Drive media  指向之前下載的
.iso  後啟動 VM 就會進入安裝介面。

安裝流程除了需要最基本的輸入外（選擇時區、語言、鍵盤語系；創造使用者），
並沒有什麼要動腦之處。


##  安裝完成後必須手動 reboot VM

安裝完成後，系統會提醒使用者：

> Please remove the installation medium, then press ENTER

但在此畫面按 `ENTER`  後，系統仍不會有後續動作。

解決方法：從 Hyper-V Manager  手動 reboot 此 VM 。


##  其他

* Docker: [https://docs.docker.com/engine/installation/linux/ubuntu/][3]
  * https://hub.docker.com/r/microsoft/dotnet/
  * https://hub.docker.com/_/postgres/
* .NET Core on Docker: [https://www.microsoft.com/net/core#dockercmd][4]
* PostSQL
  * https://www.postgresql.org/download/linux/ubuntu/
  * https://help.ubuntu.com/community/PostgreSQL
  * https://help.ubuntu.com/lts/serverguide/postgresql.html

[3]: https://docs.docker.com/engine/installation/linux/ubuntu/
[4]: https://www.microsoft.com/net/core#dockercmd



# 結論

不需要特別調整， Ubuntu Desktop 16.10 在 Hyper-V  上可順利安裝、運作。
