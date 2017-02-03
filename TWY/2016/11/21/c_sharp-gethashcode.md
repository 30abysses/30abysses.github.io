> http://www.30abysses.com/TWY/2016/11/21/c_sharp-gethashcode.md
> by TW Yang <twy@30abysses.com> 2016-11-21 CC-BY-4.0

# ＣＳ： C# `GetHashCode()`

[`GetHashCode()`][1]  這個方法(method)有著《魔偶馬戲團》中
「[四大元老][2]」 （[最古の四人][3]） 的地位，直屬 [`System.Object`][4]
類別(class) ，棲身於 `mscorlib` 此組件(assembly)中。

網路上是可以找到些關於 `GetHashCode()`  的零碎討論，但這篇文章將從最有
權威性(authoritative) 的資料來源重新出發，再逐步探討細節。

[1]: https://msdn.microsoft.com/en-us/library/system.object.gethashcode.aspx
[2]: https://zh.wikipedia.org/zh-tw/%E5%82%80%E5%84%A1%E9%A6%AC%E6%88%B2%E5%9C%98%E8%A7%92%E8%89%B2%E5%88%97%E8%A1%A8#.E5.9B.9B.E5.A4.A7.E5.85.83.E8.80.81
[3]: https://ja.wikipedia.org/wiki/%E3%81%8B%E3%82%89%E3%81%8F%E3%82%8A%E3%82%B5%E3%83%BC%E3%82%AB%E3%82%B9%E3%81%AE%E7%99%BB%E5%A0%B4%E4%BA%BA%E7%89%A9#.E6.9C.80.E5.8F.A4.E3.81.AE.E5.9B.9B.E4.BA.BA.EF.BC.88.E3.83.AC.E3.83.BB.E3.82.AD.E3.83.A3.E3.83.88.E3.83.AB.E3.83.BB.E3.83.94.E3.82.AA.E3.83.8D.E3.83.BC.E3.83.AB.EF.BC.89
[4]: https://msdn.microsoft.com/en-us/library/system.object.aspx


##  資料來源

* [MSDN `GetHashCode()`  官方文件][1] （[官方機器翻譯版本][5]）
* [`coreclr` 原始碼(source code)][6]
* ["Guidelines and rules for GetHashCode"][7] by _le_ **Eric Lippert**

[5]: https://msdn.microsoft.com/zh-tw/library/system.object.gethashcode.aspx
[6]: https://github.com/dotnet/coreclr
[7]: https://blogs.msdn.microsoft.com/ericlippert/2011/02/28/guidelines-and-rules-for-gethashcode/

微軟(Microsoft)除了開源 `.NET Core` ，目前也有釋出其他版本的
`.NET Framework`  參考用原始碼如下。

* https://github.com/Microsoft/referencesource/
* https://referencesource.microsoft.com/

經過大略的比對後，就 `GetHashCode()`  看起來是幾乎完全相同的；在此以
`coreclr` 版本為主。


##  文章清單

相對於這篇文章的原始版本把全部的資訊放在一起，這個題目被拆成更小的題目分
開來談；[文章清單在此][8] 。

[8]: http://www.30abysses.com/TWY/2016/11/21/index.html
