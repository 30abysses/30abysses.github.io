> http://www.30abysses.com/TWY/2017/01/26/resume-tone.md
> by TW Yang <twy@30abysses.com> 2017-01-26 CC-BY-4.0

# 職涯、英語，寫，美式履歷表，語氣

有作過功課的人大概都知道「[美式履歷表][1]」 與臺、日常見的履歷表（格）
不一樣。美式履歷表從一張白紙開始，雖然有由經驗歸納出的常見樣版，但並沒有
嚴格統一規格，多半由求職者自由發揮。

[1]: https://en.wikipedia.org/wiki/R%C3%A9sum%C3%A9

關於「美式履歷表寫作」，網路上有許多英語資料，但中文（公開）資料仍偏少，
且多半只限於講解美式履歷表的基本格式，少有對寫作風格多作著墨（尤其是寫作
語氣的案例解析），是故，在此整理我這幾年來發佈的文章，以作參考。這篇從最
重要的「語氣」談起。

**聲明：我的經驗來自於我對北美軟體科技業的認識，不見得適用於所有行業。**



# 所謂「語氣」為何？

履歷表是求職申請的敲門磚，最常見的錯誤語氣，就是把它當作
「[起居注][2] 、流水帳」來寫，也就是錯誤地把焦點放在：

> 在某時某地，我以某方法作了某事，句點。

[2]: https://zh.wikipedia.org/zh-tw/%E8%B5%B7%E5%B1%85%E6%B3%A8

這種寫作語氣的錯誤在於：這就像是把 iPhone 秤斤論兩，照著金屬原料、物料的
單位價格來賣，而無視 iPhone 的整體價值。

易言之，比較以下兩句話：

1.  我以Ｎ牛頓的力量施加於該質量Ｍ的小孩產生Ａ的加速度讓他在Ｔ時間作出
    Ｓ距離的位移。
2.  我奮不顧身將該小孩從失控的卡車前拉開，使他免成車下亡魂。

`(1)` 與 `(2)`  說的是同一件事；雖然 `(1)`  在技術上完全正確，但 `(2)`
才能突顯「救了一條人命的價值」。

這就是所謂的「語氣」；並不只是從網路上的 "action verb"  清單裡挑動詞
填填看，而是要從讀者的角度去想，如何堅守誠信但又清楚展示你的價值。



# 三段連續技：成果 → 實例 → 細節

所謂三段連續技，即是成果(result)、實例(example)、細節(specifics)。先從一
實際案例來看（以下所有案例皆有略為修改，以保護當事人隱私）：

> Created Bash scripts and custom Eclipse plugins in Java for automatic
> code generation
>
> 建立 Bash 腳本、以 Java 客製化 Eclipse  插件來自動產生源碼。

這給出了實例與細節，但後繼無力；是故，稍作修改訂正，變成：

> Automated _X%_ _component_ code generation with Bash scripts and
> custom Eclipse plugins (Java).
>
> 以 Bash 腳本、 Java 客製 Eclipse  插件，將 _某元件_ _X%_  的源碼生成
> 自動化。

這就是由「成果」起手（ "X%" ）、說明實例（ "automated code generation"
），點出價值（由自動化節省人工，而人工就是成本），最後補充細節（
"Bash scripts", "Eclipse plugins (Java)"  ）。

除了點出你創造的價值，也為接下來的對話佈局，不管是由讀者發問，或是自己
導入以下這類這些話題都可以，例如

* 這個 _X%_ 是怎麼算出來的？這種算法有多精確？
* 為什麼 _component_  需要 automated code generation  ？
* 若能重來，會如何改進這個作法？

易言之，先以連續技作出有效打擊，再掌握節奏，進可攻，退可守，會比原本的
被動語氣（只丟實例與細節，等待讀者來猜測、推敲你做的事的價值）來得有利（
因為你的讀者很少會願意花那時間來解讀你的「言外之意」、也多半沒那個心力來
為你探索你自己都說不明白的成果價值）。

---

另一個案例

> Implemented automation E2E tests with xUnit and Moq from scratch;
> integrated with CI to send test report.
>
> 從頭以 xUnit  及 Moq  實作自動 E2E  測試，並與 CI 整合、送出測試報告。

與該履歷表作者溝通之後，了解到該團隊之前完全沒有自動測試，是故這東西作
出來後，是從頂頭上司到 CTO  都點讚的大功，是故，重點在那句
"from scratch"  ；訂正之後拆成三句：

> Reduced average testing efforts per build from _X_ man minutes to _Y_
> machine minutes.
>
> 100% automated CI E2E tests plus test result reporting, analysis,
> visualization.
>
> Designed and implemented full CI E2E test suite from scratch with
> xUnit and Moq.
>
> 將平均測試成本從 _X_  分鐘（手動）降至 _Y_  分鐘（自動）。
>
> 100%  完全自動化 CI E2E 測試，且自動化匯報測試結果、分析、並以視覺化
> 圖表呈現。
>
> 從零開始設計整個 CI EIE 測試庫；以 xUnit, Moq 實作。

同樣一件事，換個說法就鏗鏘有力擲地有聲；可惜該履歷表作者手上沒有
code coverage, branch coverage  等資料，不然這一段的打擊火力可以更強大。



# 「成果」的重要性

「成果」是整個連續技的開始，以格鬥遊戲來比喻的話，這三段連續技就是：

> 浮空 → 連擊 → 追打

（概念影片支援： [https://youtu.be/qcnrq7G6nag?t=36s][3]  ）

[3]: https://youtu.be/qcnrq7G6nag?t=36s

對應到成果、實例、細節就是：

> 成果（浮空） → 實例（連擊） → 細節（追打）

「成果」代表你創造出來的價值，同時展現你作事的態度（你如何量化你的成果？
如何導入最基本的 [Build-Measure-Learn][4] ？）；「成果」不一定要是什麼
豐功偉業，只要把你的職務作到 *卓越* ，就一定有拿得上檯面的「成果」。有了
成果，就能抓住履歷表讀者／雇主的注意力，掌握節奏，主控（面試）場面。

[4]: https://en.wikipedia.org/wiki/Lean_startup#Build.E2.80.93Measure.E2.80.93Learn



# 結論

如果是沒有特定目的而寫的履歷表（例如，就放一份在自己網站上，給獵頭人參考
用的），那稍微偏向流水帳的語氣是無妨（工作經歷、學歷、證照、技能）。如果
是為了特定職位而寫的，就要以 sales pitch  的心態去寫，也就是這整篇文章
一直強調的「成果、價值」。

「實例、細節」與左手一樣，只是輔助；如果一味糾結於「實例、細節」而忘了
「成果、價值」，那就像是把 iPhone 如廢五金秤斤論兩賣，本末倒置。
