> http://www.30abysses.com/TWY/2016/11/28/information-security.md
> by TW Yang <twy@30abysses.com> 2016-11-28 CC-BY-4.0

# ＣＳ：資訊安全

一位網友對《[C# System.Valuetype.GetHashCode() 潛在效能、安全問題][1]》
提出了個問題：

> james732 @ptt.cc
>
> 不過有可能從標準實作算出的hash去倒推特定的資料嗎？

[1]: http://www.30abysses.com/TWY/2016/11/21/c_sharp-gethashcode-valuetype.html

也許、大概、應該不太可能，然而，資訊安全([information security][2]) 是個
很複雜、反直覺的題目，每個系統的威脅模型([threat model][3]) 不同，很難從
單一弱點([attack vector][4] 來評估；但是，通常透露給攻擊者的資訊是愈少愈
好。

[2]: https://en.wikipedia.org/wiki/Information_security
[3]: https://en.wikipedia.org/wiki/Threat_model
[4]: https://en.wikipedia.org/wiki/Vector_(malware)

以下將討論一個理論上的例子，與一個實際上的例子。


---
##  理論上的例子

假設我們要設計個內建權限控制的檔案系統：

* 使用者可以下指令讀(`read`)檔案。
* 使用者必須要有「讀」的權限才能讀到檔案。

然後，假設目前只有這樣一個檔案：

```
    /foo/bar.txt
```

假設一個 **沒有** 閱讀權限的使用者下如下的指令時，

```
    read /foo/bar/baz.txt
```

很直覺地，我們會覺得應該傳回類似這樣的錯誤訊息

```
    錯誤：找不到 /foo/bar/baz.txt
```

到目前為止，一切很好，沒有問題。

然而，如果該 **沒有** 閱讀權限的使用者下如下的第二個指令時，

```
    read /foo/bar.txt
```

我們應該傳回怎樣的錯誤訊息？有以下至少兩種可能性：

```
(1)
    錯誤：你沒有權限讀 /foo/bar.txt

(2)
    錯誤：找不到 /foo/bar.txt
```

直覺上，大概會覺得 `(1)`  是比較好的使用者體驗；Ｂｕｔ！如果這是一個聰明
的攻擊者，他就能想通

> 雖然我沒有閱讀權限，但現在我知道了 /foo/bar.txt 的存在。

易言之，雖然他沒有閱讀權限，但他仍可利用 `read` 這個指令來探測(probe) 整
個檔案系統。這樣子意外地透露給攻擊者的資訊，視場合不同，造成的風險也不同
；有可能沒什麼大不了的，也有可能透露出商業機密（例如，有時檔案名產本身就
是新產品的代號(code name) 。）


### 那如果傳回一樣的錯誤訊息，就安全了嗎？

不一定。一個聰明的攻擊者多半可以從別的線索裡找到你想隱藏的資訊；例如說，
我們修正了上段中描述的問題，現在不管是「此檔案真的不存在」或「使用者沒有
權限讀此檔案」，該 `read` 指令都會傳回「找不到 _/path/file_ 」，ｂｕｔ！
此檔案系統在處理以下兩個程序「花費的時間」不同的話：

```
(a)
    確認檔案；檔案不存在；傳回錯誤。

(b)
    確認檔案；檔案存在；讀取使用者身份；讀取使用者權限；比對權限；傳回錯誤。

```

那麼，一個聰明的攻擊者仍有可能想通「一樣是『找不到 _/path/file_ 』的錯誤
，但執行速度仍有（微妙的）差異，可以判斷哪些檔案是『真的不存在』，哪些檔
案是『存在，但我沒有權限閱讀』。」


---
##  實際上的例子

之前 Bandai Namco 公開 [Summer Lesson][5] 時，有 *無聊人士* 去簡單地探測
了一下其官方網站，發現它雖然把目錄列表(directory listing) 功能關掉了，但
從其首頁上 Hikari 人物圖可以很明顯地看出命名規則：

> http://summer-lesson.bn-ent.net/images/chara/hikari/img_hikari.png

![Hikari][6]

[5]: http://summer-lesson.bn-ent.net/
[6]: http://summer-lesson.bn-ent.net/images/chara/hikari/img_hikari.png

該 *無聊人士* 就寫了個簡單的腳本(script)用「2012  日本最常見 50 個女性名
字」去踹官方主機，看看能不能碰巧挖出當時未公開的資料。

例如：

* 已知
  `GET http://summer-lesson.bn-ent.net/images/chara/hikari/img_hikari.png`
  傳回 `HTTP 200`
* 已知
  `GET http://summer-lesson.bn-ent.net/images/chara/akane/img_akane.png`
  傳回 `HTTP 404`

**如果**
`GET http://summer-lesson.bn-ent.net/images/chara/kasumi/img_kasumi.png`
傳回 `HTTP 403` ，那麼，就算讀不到 `img_kasumi.png` 的內容，攻擊者也知道
了 `kasumi` 的存在。

雖然最後沒挖出任何東西，但世界上就是有這類 *無聊人士* 愛玩這種解謎遊戲。



# 結論

一個系統到底要防守得多嚴密？這沒有一定的答案，需要從威脅模型分析入手，評
估整體風險、商業影響、成本，等等因素，才能知道。
