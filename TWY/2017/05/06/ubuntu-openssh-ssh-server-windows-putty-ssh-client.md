> http://www.30abysses.com/TWY/2017/05/06/ubuntu-openssh-ssh-server-windows-putty-ssh-client.md
> by TW Yang <twy@30abysses.com> 2017-05-06 CC-BY-4.0

# ＣＳ： Ubuntu, SSH server

這篇筆記是實驗在 Ubuntu Desktop 17.04 (on Windows 10 Hyper-V) 設定 SSH
server  與在 Windows  上使用 PuTTY  作為 SSH client 的組合。

在網路上已經有許多教學文件，這篇筆記的重點在於實驗、測試結果。


##  安裝 `openssh-server`

```
sudo apt-get install openssh-server
```


##  修改 `/etc/ssh/sshd_config` 設定

* `/etc/ssh/sshd_config`  是一個文字檔；許多教學文件都建議在更動
  `sshd_config` 前先複製備份一份。
* 更動 `sshd_config`  後，以 `sudo systemctl restart ssh` 重新啟動 SSH
  server  。

`sshd_config` 的內容並不難懂，許多教學文件原則上都會建議關掉
`PasswordAuthentication`  以及用不到的各種 forwarding 功能（例如
`AllowTcpForwarding`  ）。


##  從 Windows  以 PuTTY  測試 SSH server

在 Windows  啟動 PuTTY  以 SSH  模式連上在上述步驟裡安裝、設定好的
SSH server  。

* 從 Ubuntu 本地端（從 Hyper-V Manager  連上），執行 `ip address show`
  可以取得 Ubuntu VM  的 IP address 。
* 從 `/etc/ssh/sshd_config` 可以看到 SSH server 開啟的 port （預設為
  `22`  ），也可以看到預設的 `HostKey`  檔案路徑（預設為 `/etc/ssh/`  目
  錄下的各個 `ssh_host_*_key` 檔案）。

若一切運作正常的話， PuTTY  會回報 SSH server 的 host key （以及其編碼方
式），並詢問你是否能信任該 host key ；接下來就是要驗證該 host key 。


### 驗證 SSH server host key

根據 PuTTY  回報的 SSH server host key  以及其編碼方式，我們應該可以在
Ubuntu  主機上的 `/etc/ssh/`  目錄下找到對應的 `ssh_host_*_key.pub` 檔案
。

這些 `ssh_host_*_key.pub` 檔案是文字檔，但其 key  的部分的 hash 編碼不見
得與 PuTTY  回報的 hash 編碼相同。可以用以下指令印出其 key  的 SHA256 或
MD5 hash  值（在這個例子中，使用的是 `ssh_host_rsa_key.pub` ；實務上，
PuTTY 回報的有可能是 `ecdsa`  或 `ed25519`  的編碼，依 PuTTY  回報的訊息
調整指令即可）：

```
ssh-keygen -l -f /etc/ssh/ssh_host_rsa_key.pub

ssh-keygen -l -f /etc/ssh/ssh_host_rsa_key.pub -E md5
```

如果 PuTTY  回報的 SSH server host key  符合從 Ubuntu 本地端看到的
host key, 那就可以讓 PuTTY  信任該 host key 。

實務上， PuTTY  會把被信任的 host key 存在 Windows Registry 裡，可以以下
指令查詢：

```
reg query HKCU\Software\SimonTatham\PuTTY\SshHostKeys
```



# 結論

剩下的步驟就是在 Windows  上使用 `puttygen.exe` 產生 public+private keys
，然後設定 `~/.ssh/authorized_keys` 。詳細的教學可以看這兩份文件：

* [How To Use SSH Keys with PuTTY on DigitalOcean Droplets (Windows users)][1]
* [How To Create SSH Keys With PuTTY to Connect to a VPS][2]

[1]: https://www.digitalocean.com/community/tutorials/how-to-use-ssh-keys-with-putty-on-digitalocean-droplets-windows-users
[2]: https://www.digitalocean.com/community/tutorials/how-to-create-ssh-keys-with-putty-to-connect-to-a-vps

唯一要注意的，就是 `puttygen.exe` 在 GUI  上列出的 public key 與其
`Save public key` 存成檔案的 public key 格式不同。而適用於
`~/.ssh/authorized_keys`  的是「`puttygen.exe` 在 GUI  上列出的
public key  」。

如果誤用了「`Save public key` 存成檔案的 public key 」，那麼在 PuTTY  連
線時會傳回 "server refused our key" 錯誤；只要更正
`~/.ssh/authorized_keys`  裡的 public key 格式就可修復。

