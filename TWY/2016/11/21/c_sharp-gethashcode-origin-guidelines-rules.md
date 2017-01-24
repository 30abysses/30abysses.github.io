> http://www.30abysses.com/TWY/2016/11/21/c_sharp-gethashcode-origin-guidelines-rules.md
> by TW Yang <twy@30abysses.com> 2016-11-21 CC-BY-4.0

# C# `GetHashCode()` 起源、守則、法則

_le_ **Eric Lippert** 在他的文章
"[Guidelines and rules for GetHashCode][1]" 中提出了很有趣的問題：

> Why do we have this method on Object in the first place?
>
> 究竟為什麼這個方法(`GetHashCode()`)是宣告在 `Object` 上？

[1]: https://blogs.msdn.microsoft.com/ericlippert/2011/02/28/guidelines-and-rules-for-gethashcode/

Eric Lippert  檢視宣告在 [`System.Object`][2] 上的其他方法：

> It makes perfect sense that every object in the type system should
> provide a GetType method; data’s ability to describe itself is a key
> feature of the CLR type system. And it makes sense that every object
> should have a ToString, so that it is able to print out a
> representation of itself as a string, for debugging purposes. It seems
> plausible that objects should be able to compare themselves to other
> objects for equality. But why should it be the case that every object
> should be able to hash itself for insertion into a hash table? Seems
> like an odd thing to require every object to be able to do.

[2]: https://msdn.microsoft.com/en-us/library/system.object.aspx

摘要釋譯如下：

* `GetType()`:  「資料『自我描述』的能力」是 CLR  類別系統的關鍵特點之一
* `ToString()`: 為了除錯，提供以「字串」印出自己的能力
* `Equals()`: 感覺上物件應要能與其它物件比較「相等性」
* `GetHashCode()`: **但是** ，為什麼強制要求所有的物件都要能計算雜湊值？

Eric Lippert  的感想：

> I think if we were redesigning the type system from scratch today,
> hashing might be done differently, perhaps with an IHashable
> interface. But when the CLR type system was designed there were no
> generic types and therefore a general-purpose hash table needed to be
> able to store any object.

摘要釋譯如下：

> 我想，如果我們今天從頭重新設計類別系統，大概會用不同方法來實作雜湊；或
> 許會用個 `IHashable`  介面。但是，當初設計 CLR  類別系統時尚無泛型類別
> ，是故一個通用的雜湊表需要能儲存任何物件。

我對 Eric Lippert 這兩段文字的解讀是：類似 [`System.IComparable`][3] ，
**或許** `GetHashCode()` 應該屬於像這樣的一個介面

```
    interface IHashable
    {
        int GetHashCode();
    }
```

**或許**  那樣作會比較乾淨(clean) 與優雅(elegant) 。

[3]: https://msdn.microsoft.com/en-us/library/system.icomparable.aspx

然而，就我自己對 C# / .NET  的感覺是：

> 我覺得把 `GetHashCode()`  宣告在 `System.Object`  上，且提供了一個在通
> 常情形下堪用的預設實作，是在理論與實務上取得了 **適當的平衡** 。

所謂「理論與實務上的適當的平衡」的實例之一，就是「讀寫檔案」。

在我印像中的 Java,  要讀寫檔案就得要把一堆有的沒的五四三 stream, reader,
writer 組合起來；但在 .NET / C#, 讀寫(小)檔案時， `System.IO.File`  已經
把最常見的動作通通包裝好，並且後來更進一步加入類似串流(stream)的支援；但
若想更精確地控制讀寫檔案的緩衝(buffer)等行為，還是可以回頭去組裝
stream, reader, writer  以達成目的。易言之，就是在實務上作到

* 讓常用的功能「方便(convenient)」。
* 讓複雜的動作「可能(possible)」。

而不是過度偏重理論上的乾淨與優雅、犧牲實用性。是故，我傾向於認為
`GetHashCode()` 的設計是可以接受的。


---
## `GetHashCode()`  守則(guideline)與法則(rule)

基本上，[MSDN `GetHashCode()`  官方文件][4] 與 [Eric Lippert 的文章][1]
已提供了相當的資訊；此處將其內容摘要整理之，附上一些實驗結果，並指出
MSDN  官方機器翻譯的一個錯誤。

[4]: https://msdn.microsoft.com/en-us/library/system.object.gethashcode.aspx


### 實驗環境

* 64-bit W10 Pro 1607
* VS2015 Update 3


---
## Ｐ(相同／相等物件) → Ｑ(傳回相同雜湊值)

所有的 `GetHashCode()`  實作都必須遵守這個法則。如果相同／相等物件傳回不
同雜湊值，那代表著 `GetHashCode()`  或 `Equals()` 的實作裡有臭蟲。


---
## Ｑ(傳回相同雜湊值) →╳→ Ｐ(相同／相等物件)

實驗：

```
    Console.WriteLine(1L.GetHashCode());
    Console.WriteLine(4294967296L.GetHashCode());
```

很明顯的， `1L` 與 `4294967296L`  是不同／相異的物件，但兩者都會傳回相同
的雜湊值: `1` 。這就是「即使傳回相同的雜湊值，也不代表是相等／相同的物件
」的例子。


---
##  不要將雜湊值儲存於外部媒介以供未來再使用

> `GetHashCode()` 傳回值應只在同 process 下的同一 application domain  內
> 使用；不同 .NET Framework 間的 `GetHashCode()`  實作有可能改變

測試方法：

```
    Console.WriteLine("Hello, world!".GetHashCode());
```

測試結果：

* .NET `2.0`, `3.0`, `3.5`, `4.0` 傳回 307044849
* .NET `4.5.*`, `4.6.*` 傳回 904533705
* .NET Core `1.0.1` 傳回 -1650644819

這就是「同樣的物件，在不同的環境下可能／可以傳回不同的雜湊值」的例子。


---
##  不要將 `GetHashCode()`  產生的雜湊值用在密碼學(cryptography)用途上

簡單地說，「閃開！讓專業的來！」若要算出有密碼學強度的雜湊值，應該使用以
下相關工具

* [`System.Security.Cryptography.HashAlgorithm`][5]
* [`System.Security.Cryptography.KeyedHashAlgorithm`][6]

[5]: https://msdn.microsoft.com/en-us/library/system.security.cryptography.hashalgorithm.aspx
[6]: https://msdn.microsoft.com/en-us/library/system.security.cryptography.keyedhashalgorithm.aspx


---
##  （只要物件本身沒改變） `GetHashCode()`  應該要有一致性

只要物件本身沒有足以影響 `Equals()` 結果的變化，那 `GetHashCode()`  產生
的雜湊也就不應變化。（但這只限於同一 process  下同一 application domain
）。

尤其是當此物件被存放在雜湊表這類「重度依賴雜湊值以確保其運作正確性」的資
料結構中時，若 `GetHashCode()`  無法傳回一致的雜湊值，可以想見這會造成各
種混亂。


---
## `GetHashCode()`  的計算需求要少、要快

`GetHashCode()` 本來的目的就是協助、加快比較物件之間的相等性；如果
`GetHashCode()` 要花大量時間、資源運算，那反而本末倒置了。


---
## `GetHashCode()`  不應丟出例外(exception)

官方原文：

> The GetHashCode method should not throw exceptions.

目前(2016-11-25)的[官方機器翻譯][7]：

> GetHashCode 方法應該擲回例外狀況。

`ლ(ಠ益ಠლ)`

[7]: https://msdn.microsoft.com/zh-tw/library/system.object.gethashcode.aspx


---
## `GetHashCode()`  產生的雜湊值應要平均分佈

愈能平均分佈，通常愈能避免碰撞。

同時，要考量到輸入資料的性質； Eric Lippert 有提到個案例：
「[美國郵遞區號][16]」，通通都是五個數字字元；而他當時寫的雜湊演算法無法
有效地就這樣的資料產生平均分佈的雜湊值（也就是產生了大量的碰撞問題），最
後嚴重拖累系統效能。

另外一點是安全上的考量；如果雜湊值演算法中有來自使用者的輸入資料，那麼在
理論上就有可能讓惡意使用者有機可趁，想辦法產生大量的碰撞，拖累你的系統效
能。

[8]: https://simple.wikipedia.org/wiki/ZIP_code
