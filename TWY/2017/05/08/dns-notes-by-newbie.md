> http://www.30abysses.com/TWY/2017/05/08/dns-notes-by-newbie.md
> by TW Yang <twy@30abysses.com> 2017-05-08 CC-BY-4.0

# ＣＳ：ＤＮＳ入門筆記

除了在測試環境中架設管理過ＤＮＳ(Domain Name System)，在實戰環境中我只有
初級使用者的經驗，這篇文章記錄一些實驗、學習心得。（參考文件連結記載於這
篇文章未尾）。


##  基本原理

網路上有不少關於ＤＮＳ基本運作原理的的相關文章，以關鍵字去 Google 就能找
到許多資料，例如：

* "How does DNS work?"
* "authoritative vs. recursive DNS"
* "Start of Authority Record" (SOA)


##  進階觀念

視專案需求不同，對ＤＮＳ的要求也會改變，可以從以下幾個角度考量：

* management, customization: 滿足管理、客製化需求的能力
* deployment, maintenance:  部署、維護的花費
* speed, reliability: 速度、可靠性

就我觀察到的趨勢，各家網路服務商都是在往「提昇『滿足管理、客製化需求的能
力』，吸收『部署、維護的花費』」的方向在走，是故就個人、中小型專案而言，
能當作各家服務品質參考指標的就是「速度、可靠性」。

理論上來說， Anycast DNS  （相對於 Unicast DNS  ）在負載平衡、可靠性、速
度上佔優，但其部署、維護的花費也較高。除了從本地端使用 `ping`,
`traceroute`  等指令來測試ＤＮＳ伺服器的速度外，網路上有免費工具可以從世
界各地測試網路速度，例如：

* [https://asm.ca.com/en/ping.php][18]
* [https://asm.ca.com/en/traceroute.php][19]

[18]: https://asm.ca.com/en/ping.php
[19]: https://asm.ca.com/en/traceroute.php

這個網站整理了世界各地提供線上 `traceroute` 工具的網頁清單：

* [http://www.traceroute.org/][17] 

值得注意的是，如此測出來的數據只適合作為參考指標（而非絕對指標），最後還
是要實地測試專案的整體效能，才能針對瓶頸改善，滿足實務上的需求。


### Start of Authority (SOA)

SOA (Start of Authority)  ，如其名所述，是一網域名稱ＤＮＳ資訊的起點；當
你從域名註冊商(domain name registrar) 設定網域名稱的ＤＮＳ主機資訊時，就
是在設定該網域的 SOA  紀錄，而一個網域的 SOA  紀錄可以 `nslookup`, `dig`
這些指令查詢（或使用網路上各式的免費ＤＮＳ分析工具），例如：

```
$ dig example.com SOA

; <<>> DiG 9.10.3-P4-Ubuntu <<>> example.com SOA
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 60805
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;example.com.                   IN      SOA

;; ANSWER SECTION:
example.com.            3600    IN      SOA     sns.dns.icann.org. noc.dns.icann.org. 2017042703 7200 3600 1209600 3600

;; Query time: 12 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: Tue May 09 12:35:34 PDT 2017
;; MSG SIZE  rcvd: 97

$ nslookup -querytype=SOA stackoverflow.com
Server:         127.0.0.53
Address:        127.0.0.53#53

Non-authoritative answer:
stackoverflow.com
        origin = ns-cloud-e1.googledomains.com
        mail addr = cloud-dns-hostmaster.google.com
        serial = 1
        refresh = 21600
        retry = 3600
        expire = 259200
        minimum = 300

Authoritative answers can be found from:
```

可以看到 `example.com`  的 SOA  是 `sns.dns.icann.org` ；
`stackoverflow.com` 的 SOA  是 `ns-cloud-e1.googledomains.com`  。



# 結論

今日的個人、中小專案開發者大約只需要照著各網路服務商提供的文件，就能搞定
SOA 與各種ＤＮＳ設定、管理；而佈署、維護這些雜務則多是由網路服務商（收費
）代辦，讓開發者能專注於診斷分析效能瓶頸，滿足專案需求。



# 參考文件


## Overview

* [Function][1]
* [Structure][2]
* [Operation][3]

[1]: https://en.wikipedia.org/wiki/Domain_Name_System#Function
[2]: https://en.wikipedia.org/wiki/Domain_Name_System#Structure
[3]: https://en.wikipedia.org/wiki/Domain_Name_System#Operation


## Authoritative vs. Recursive

* [Authoritative Name Server][4]
  * [Start of Authority Record (SOA)][10]
    * [What is a SOA record?][12]
  * [DNS Zone Files][11]
* [Recursive Name Server][5]
* [Authoritative DNS Servers vs. Recursive DNS Servers][9]

[4]: https://en.wikipedia.org/wiki/Domain_Name_System#Authoritative_name_server
[5]: https://en.wikipedia.org/wiki/Domain_Name_System#Recursive_and_caching_name_server
[9]: http://www.noip.com/blog/2013/11/22/authoritative-dns-vs-recursive-dns-work/
[10]: http://www.inetdaemon.com/tutorials/internet/dns/configuration/zone_files/resource_records/soa.shtml
[11]: http://www.inetdaemon.com/tutorials/internet/dns/configuration/zone_files/index.shtml
[12]: https://support.dnsimple.com/articles/soa-record/


## Anycast vs. Unicast

* [Anycast DNS - Part 1, Overview][7]
* [What is “anycast” and how is it helpful?][8]

[7]: http://ddiguru.com/blog/118-introduction-to-anycast-dns
[8]: https://serverfault.com/questions/14985/what-is-anycast-and-how-is-it-helpful


## DNS Record Types

* [List of DNS record types][6]

> The most common types of records stored in the DNS database are for
> Start of Authority (SOA), IP addresses (A and AAAA), SMTP mail
> exchangers (MX), name servers (NS), pointers for reverse DNS lookups
> (PTR), and domain name aliases (CNAME). Although not intended to be a
> general purpose database, DNS can store records for other types of
> data for either automatic lookups, such as DNSSEC records, or for
> human queries such as responsible person (RP) records.
>
> https://en.wikipedia.org/wiki/Domain_Name_System

[6]: https://en.wikipedia.org/wiki/List_of_DNS_record_types


## Online Tools

* [http://www.dnssy.com/][14]
* [https://mxtoolbox.com/NetworkTools.aspx][13]
* [http://dns.squish.net/][15]
* [https://asm.ca.com/en/dnstool.php][16]
* [http://www.traceroute.org/][17]

[13]: https://mxtoolbox.com/NetworkTools.aspx
[14]: http://www.dnssy.com/
[15]: http://dns.squish.net/
[16]: https://asm.ca.com/en/dnstool.php
[17]: http://www.traceroute.org/


## Local Tools

```
nslookup -querytype=SOA example.com

dig example.com any
```
