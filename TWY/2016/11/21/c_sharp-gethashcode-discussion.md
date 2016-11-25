> http://www.30abysses.com/TWY/2016/11/21/c_sharp-gethashcode-discussion.md
> by TW Yang <twy@30abysses.com> 2016-11-21 CC-BY-4.0

# C# `GetHashCode()`  討論

這裡整理了與網友的討論；為了閱讀性，有些語句有重組過。

來源：

* [https://www.ptt.cc/bbs/Soft_Job/M.1479825273.A.2EF.html][1]

[1]: https://www.ptt.cc/bbs/Soft_Job/M.1479825273.A.2EF.html

---
> robler @ptt.cc
>
> 一般人要有需要比較物件時通常會試者實作IComparable介面
> 因為GetHashCode很難用來實際比較物件XD

是的， `GetHashCode()`  是協助比較物件「相同、相異」，主要應用在與 hash
有關的地方； `IComparable` 則是除了比較相同、相異之外，更進一步有
「順序(order)」 的意味，更適合應用在「排序」上。

易言之，兩者有微妙的差異。 :)


---
> erspicu @ptt.cc
>
> 比較好奇的是取得物件的HASH 能夠應用在哪些問題上?

在需要測試物件「相等性(equality)」的場合， `GetHashCode()`  就 *有可能*
派上用場。易言之，

> GetHashCode() ≠  測試物件相等性(equality)的終極解決方案

更完整的說法為

> GetHashCode() 這樣的設計雖然增加了整體複雜度，但它開拓了一塊空間，提供
> 改善「測試物件相等性(equality)」的效能的機會與可能性

例如，在
[`System.Collections.Generic.Dictionary<TKey,TValue>` 的原始碼][2]中就可
以看到許多使用 `GetHashCode()`  的場合。

[2]: https://github.com/dotnet/coreclr/blob/master/src/mscorlib/src/System/Collections/Generic/Dictionary.cs

若仔細去追 .NET core  的原始碼，會發現 hash, hash table （以及衍生出的
set, hash set, dictionary, ...) 通通都依賴適當的 caching, 適當的 hash 演
算法選擇，才能作到那些資料結構宣稱的各種 big-O  時間界限(boundary)；而這
些資料結構更是廣泛地被 .NET Framework 其他部分, 例如 `LINQ` 使用。

例如，雖然 C# `Dictionary`  此類別的官方文件宣稱：

> Getting or setting the value of this property approaches an O(1)
> operation
>
> 存、取值動作所花時間趨近於 O(1) 。

但從原始碼可以看出，事實上那非常依賴放進 `Dictionary` 裡的物件類別的
`GetHashCode()`  實作。

身為 `System.Object`  上最古老的四人之一， `GetHashCode()`  被所有類別繼
承著；若對軟體業產業上游、 framework design 這方向有興趣的話，這裡面是有
極多用血汗淚堆屍堆出來的知識可挖的 :D


---
> TSW @ptt.cc
>
> 雖然我覺得沒什麼使用上的必要，不過還是感謝分享心得。

是的，「使用上的必要」是看身處在整個產業鏈中哪個環節；這類知識在上游比較
有用，或著在資料數量 ｎ 夠大時才比較有用。

> TSW @ptt.cc
>
> 「資料數量 n 夠大時」 應該會有十分完整的 index 跟 compare 機制，感覺更
> 用不到的說@@

同意，「ｎ夠大」可以有很多解釋；例如，大到單一機器(的記憶體)「裝不下」，
也是一種大。大到一般人只能在抽象層面理解的數量級，也是一種大，等等。

然而，所謂「完整的 index 跟 compare 機制」，中的「 compare 機制」 ，即使
不是就真的去覆寫(override) GetHashCode()  這個方法(method), 我很難想像一
個「用不到類似 hashing 相關作法的、為了處理大量資料的 compare 機制」

當然，是有可能因為 indexing 作得很棒， key  選得很棒，以致於所有需要
compare 的場合通通不需要更進一步的優化，這個理論上的確是可能的；但在實務
上，我覺得很難實現，尤其是很難「一次到位」就作好。中間應該會有段過渡期，
是連系統、流程本身的規格都會繼續演化，ｎ本身的數量級會演化，對於「ｎ有多
大」的解讀也會演化，值不值得去作 indexing 的評估也會演化；通常是以三年、
五年、十年為單位在演化。

是故，實務上很難一次到位，除非作的東西夠小；然後無可避免的，一些 local
**temporary** optimization 最後就為成為這系統無法動搖的一部分 XD  就如下
面這句話說的：

> Nothing is more permanent than a temporary solution.
>
> 沒有比「暫時性方案」更恆遠的事物了。

易言之，我 100% 同意以下這句話

> 視情況，可能有比「從 GetHashCode()  著手」更好的解決方案。

但實務上，大概還是取決於天時地利人和 :D
