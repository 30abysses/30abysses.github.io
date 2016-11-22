> http://www.30abysses.com/TWY/2016/11/21/c_sharp-gethashcode.md
> by TW Yang <twy@30abysses.com> 2016-11-21 CC-BY-4.0

# C# GetHashCode()

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
* "[Guidelines and rules for GetHashCode][7]" by _le_ **Eric Lippert**

[5]: https://msdn.microsoft.com/zh-tw/library/system.object.gethashcode.aspx
[6]: https://github.com/dotnet/coreclr
[7]: https://blogs.msdn.microsoft.com/ericlippert/2011/02/28/guidelines-and-rules-for-gethashcode/

微軟(Microsoft)除了開源 `.NET Core` ，目前也有釋出其他版本的
`.NET Framework`  參考用原始碼如下。

* https://github.com/Microsoft/referencesource/
* https://referencesource.microsoft.com/

經過大略的比對後，就 `GetHashCode()`  看起來是幾乎完全相同的；在此以
`coreclr` 版本為主。



# `GetHashCode()` 簡單來說就是……

在試著理解 `GetHashCode()`  的功能前，得先理解它要解決的問題：

> 比較兩個物件(object)是否「相同」

更完整的說法是：

> 有一物件甲，有一物件乙，試問：「甲、乙是否『相同』？」

所謂「相同」在此是個很抽象的觀念，在不同的場合有不同的解釋。

例如，假設甲、乙是「書」，那麼通常來說，如果它們的
[國際標準書號][8]([ISBN][9])相同，那我們就可以說甲、乙是兩本書是
「相同的書」。相對的，如果甲、乙這兩本書的國際標準書號不同，我們就會說甲
、乙是「不同的書」。

[8]: https://zh.wikipedia.org/zh-tw/%E5%9C%8B%E9%9A%9B%E6%A8%99%E6%BA%96%E6%9B%B8%E8%99%9F
[9]: https://en.wikipedia.org/wiki/International_Standard_Book_Number

然而，視場合不同，對「相同」在定義上嚴謹度的要求也不同；例如，就算甲、乙
兩本書有一樣的國際標準書號，其內容仍有可能相異（書有可能缺頁、被塗改）。
如果是在鑑定書籍，那就可能要逐字逐頁考證；可以想像，那將是很費功夫的事。

易言之：

* 如果兩本書的國際標準書號不同，那幾乎可以 100% 安全地決定「完全不需要花
  功夫去逐字逐頁比對」。
* 如果兩本書的國際標準書號相同，那就必須進一步考量「是否需要花功夫去逐字
  逐頁比對」。

再進一步說：

* 即使兩本書的國際標準書號相同，最後比對的結果仍可能發現這兩本書並不
  100%  相同（缺頁、被塗改）。

反過來說：

* 兩本事實上並不 100% 相同的書（缺頁、被塗改），仍有可能有相同的國際標準
  書號。


##  所以說，那個醬汁^H^H `GetHashCode()`  就是……

在 C# 程式的領域中， `GetHashCode()`  的作用就像上述例子中的
「國際標準書號」，提供一個快速篩選物件的作法；從其簽章(signature)

```
    public virtual int GetHashCode()
```

可以看出，它的設計是傳回一個 `int`  來代表該物件；而 `int`  的好處之一，
就是可以快速簡單地比較出異同。

易言之，與上述的「書、國際標準書號」的例子對應起來，就會是這個樣子：

* 如果兩個物件由 `GetHashCode()`  傳回值不同，那就不需要進一步比對。
* 如果兩個物件由 `GetHashCode()`  傳回值相同，那就有需要進一步比對。
* 即使兩個物件由 `GetHashCode()`  傳回值相同，最後比對的結果仍可能發現這
  兩個物件並不 100% 相同。
* 兩個事實上不 100% 相同的物件，其 `GetHashCode()`  傳回值仍有可能相同。

對應到[官方文件][1] 的話，就是這段：

> Two objects that are equal return hash codes that are equal. However,
> the reverse is not true: equal hash codes do not imply object
> equality, because different (unequal) objects can have identical hash
> codes.

[官方機器翻譯][5] ：

> 等於傳回雜湊程式碼是相等的兩個物件。 不過，反向並不成立︰ 相等的雜湊程
> 式碼不一定代表物件是否相等，因為不同 （相等） 的物件可以有相同的雜湊碼
> 。

我的手動釋譯：

> 相同的物件傳回相同的雜湊(hash)值。然而，這不代表「傳回相同雜湊值的都是
> 相同的物件」；因為相異的物件仍可能傳回相同的雜湊值。

用邏輯來說就是無法從「Ｐ→Ｑ」得出「Ｑ→Ｐ」；也就是 **無法** 從

> Ｐ(相同／相等物件) → Ｑ(傳回相同雜湊值)

得到

> Ｑ(傳回相同雜湊值) → Ｐ(相同／相等物件)


## `GetHashCode()`  如何知道該怎麼判斷／計算其傳回值？

簡單地說：「施主，這個問題你應該要問你自己。」

`GetHashCode()` 只有定義其傳回值的規則，真正的實作是取決於使用
`GetHashCode()` 這個架構的人。反過來說，製定 `GetHashCode()`  架構的人無
法事先預知在各種不同場合中對各種不同物件種類判定異同的規則，是故，他只能
在抽象層面上定義傳回值代表的意義以及說明 .NET Framework 會如何使用
`GetHashCode()` 的傳回值。

通常來說，除非你遇到要處理相等性(equality)及其沿生出的東西，例如
`Dictionary`, `HashSet`, `Hashtable`, **`LINQ`**, 不然，通常來說並不用特
別煩惱 `GetHashCode()`  的問題；常用的基本資料型態，例如 `int`,
`string`, `double`, 等等，都已有內建的 `GetHashCode()`  ，應該能應付一般
的使用情形。


##  為什麼「相異的物件仍可能傳回相同的雜湊值」？

簡單地說，[鴿巢原理][10]([Pigeonhole principle][11])。

[10]: https://zh.wikipedia.org/zh-tw/%E9%B4%BF%E5%B7%A2%E5%8E%9F%E7%90%86
[11]: https://en.wikipedia.org/wiki/Pigeonhole_principle

思考一下：

> 在十進位系統裡，若只使用一位數，能表示出最多幾種獨特(unique)的編號？

答案是 10 種，也就是 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 。

進一步思考：

> 在十進位系統裡，若只使用一位數為 11 個物件編號，是否有可能讓每個物件都
> 有其獨一無二的編號？

答案是不可能。因為為前 10 個物件編號時，就會用完僅有的 10 個獨特編號，而
最後一個物件就 **必須** 挑一個數字重覆使用。

這就是[鴿巢原理][10]說的：

> 若有n個籠子和n+1隻鴿子，所有的鴿子都被關在鴿籠裡，那麼至少有一個籠子有
> 至少2隻鴿子。

回頭來看 `GetHashCode()`  ，其傳回的 `int`  是一個 32 位元的數字，最多只
能表示大約 42.9 億種獨特的狀態，也就是 -2147483648, -2147483647,
-2147483646, ... -1, 0, 1, ... 2147483645, 2147483646, 2147483647  這
4294967295  種狀態。

如果有一種物件具有超過 42.9 億種獨特的狀態，那麼在為前 42.9 億種獨特狀態
編號之後，就會把每個 `int`  能代表的數字都會用過一次，接下來，在數學邏輯
上無法避免的，必須重覆使用之前用過的數字來編號，也就是所謂的
[碰撞][12]([collision][13]) 。

[12]: https://zh.wikipedia.org/zh-tw/%E7%A2%B0%E6%92%9E_(%E8%A8%88%E7%AE%97%E6%A9%9F%E7%A7%91%E5%AD%B8)
[13]: https://en.wikipedia.org/wiki/Collision_(computer_science)


### 超過 42.9 億種獨特狀態的物件真的存在嗎？

`long`, 64  位元的數字，可以表示大約 18.4 艾種狀態；一艾是 10 的 18 次方
，也就是是一億（10  的 8  次方）的一百億倍（10  的 10 次方）。

參考資料與官方機器翻譯：

* `int`
  * https://msdn.microsoft.com/en-us/library/5kzh1b5w.aspx
  * https://msdn.microsoft.com/zh-tw/library/5kzh1b5w.aspx
* `long`
  * https://msdn.microsoft.com/en-us/library/ctetwysk.aspx
  * https://msdn.microsoft.com/zh-tw/library/ctetwysk.aspx



# [最古の四人][3]， `GetHashCode()`

_le_ **Eric Lippert** 在他的文章
"[Guidelines and rules for GetHashCode][7]" 中提出了很有趣的問題：

> Why do we have this method on Object in the first place?
>
> 究竟為什麼這個方法(`GetHashCode()`)是宣告在 `Object` 上？

Eric Lippert  檢視其他宣告在 [`System.Object`][4] 上的方法：

> It makes perfect sense that every object in the type system should
> provide a GetType method; data’s ability to describe itself is a key
> feature of the CLR type system. And it makes sense that every object
> should have a ToString, so that it is able to print out a
> representation of itself as a string, for debugging purposes. It seems
> plausible that objects should be able to compare themselves to other
> objects for equality. But why should it be the case that every object
> should be able to hash itself for insertion into a hash table? Seems
> like an odd thing to require every object to be able to do.

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

我對 Eric Lippert 這兩段文字的解讀是：類似 [`System.IComparable`][14] ，
**或許** `GetHashCode()` 應該屬於像這樣的一個介面

```
    interface IHashable
    {
        int GetHashCode();
    }
```
**或許**  那樣作會比較乾淨(clean) 與優雅(elegant) 。

[14]: https://msdn.microsoft.com/en-us/library/system.icomparable.aspx

然而，就我自己對 C# / .NET  的感覺是：

> 我覺得把 `GetHashCode()`  宣告在 `System.Object`  上，且提供了一個在通
> 常情形下堪用的預設實作，是在理論與實務上取得了 **適當的平衡** 。

所謂「理論與實務上的適當的平衡」的實例之一，就是「讀寫檔案」。

在我印像中的 Java,  要讀寫檔案就得要把一堆有的沒的五四三 stream, reader,
writer 組合起來；但在 .NET / C#, 讀寫(小)檔案時， [`System.IO.File`][15]
已經把最常見的動作通通包裝好，並且後來更進一步加入類似 streaming  的支援
；但若想更精確地控制讀寫檔案的緩衝(buffer)等行為，還是可以回頭去組裝
stream, reader, writer  以達成目的。易言之，就是在實務上作到

* 讓常用的功能「方便(convenient)」。
* 讓複雜的動作「可能(possible)」。

而不是過度偏重理論上的乾淨與優雅。

[15]: https://msdn.microsoft.com/en-us/library/system.io.file.aspx



# `GetHashCode()` 守則(guideline)與法則(rule)

基本上，[MSDN `GetHashCode()`  官方文件][1] 已提供了相當的資訊，但其機器
翻譯版本實在慘不忍睹，甚至有錯…… `orz`  是故，以下摘要釋譯之。同時，
[Eric Lippert 的文章][7]也值得細讀。

有些地方會同時作些小實驗，測試環境如下：

* 64-bit W10 Pro 1607
* VS2015 Update 3


## Ｐ(相同／相等物件) → Ｑ(傳回相同雜湊值)

如果相同／相等物件傳回不同雜湊值，那代表著 `GetHashCode()`  或
`Equals()`  裡有臭蟲。


## Ｑ(傳回相同雜湊值) →╳→ Ｐ(相同／相等物件)

```
    Console.WriteLine(1L.GetHashCode());
    Console.WriteLine(4294967296L.GetHashCode());
```

很明顯的， `1L` 與 `4294967296L`  是不同／相異的物件，但兩者都會傳回相同
的雜湊值: `1` 。


##  不要將雜湊值儲存於外部

> `GetHashCode()` 傳回值應只在同 process 下的同一 application domain  內
> 使用；不同 .NET Framework 間的 `GetHashCode()`  實作有可能改變

測試方法：

```
    Console.WriteLine("Hello, world!".GetHashCode());
```

測試結果：

* .NET 2.0, 3.0, 3.5, 4.0 傳回 307044849
* .NET 4.5.*, 4.6.* 傳回 904533705
* .NET Core 1.0.1 傳回 -1650644819


##  不要將 `GetHashCode()`  產生的雜湊值用在密碼學(cryptography)用途上

簡單地說，「閃開！讓專業的來！」

* https://msdn.microsoft.com/en-us/library/system.security.cryptography.hashalgorithm.aspx
* https://msdn.microsoft.com/en-us/library/system.security.cryptography.keyedhashalgorithm.aspx


##  （只要物件本身沒改變） `GetHashCode()`  應該要有一致性

只要物件本身沒有足以影響 `Equals()` 結果的變化，那 `GetHashCode()`  產生
的雜湊也就不應變化。（但這只限於同一 process  下同一 application domain
）。

尤其是當此物件被存放在雜湊表這類「重度依賴雜湊值以確保其運作正確性」的資
料結構中時，若 `GetHashCode()`  無法傳回一致的雜湊值，可以想見這會造成各
種混亂。


## `GetHashCode()`  的計算需求要少、要快

`GetHashCode()` 本來的目的就是協助、加快比較物件之間的相等性；如果
`GetHashCode()` 要花大量時間、資源運算，那反而本末倒置了。


## `GetHashCode()`  不應丟出例外(exception)

官方原文：

> The GetHashCode method should not throw exceptions.

官方機器翻譯：

> GetHashCode 方法應該擲回例外狀況。

`ლ(ಠ益ಠლ)`


## `GetHashCode()`  產生的雜湊值應要平均分佈

愈能平均分佈，通常愈能避免碰撞。

同時，要考量到輸入資料的性質； Eric Lippert 有提到個案例：
「[美國郵遞區號][16]」，通通都是五個數字字元；而他當時寫的雜湊演算法無法
有效地就這樣的資料產生平均分佈的雜湊值（也就是產生了大量的碰撞問題），最
後嚴重拖累系統效能。

另外一點是安全上的考量；如果雜湊值演算法中有來自使用者的輸入資料，那麼在
理論上就有可能讓惡意使用者有機可趁，想辦法產生大量的碰撞，拖累你的系統效
能。

[16]: https://simple.wikipedia.org/wiki/ZIP_code



# 探索 `GetHashCode()` 原始碼

這個章節摘錄一些內建的 `GetHashCode()`  實作，作為參考。篇幅較大的原始碼
（尤其是 C++  原始碼），只有附上連結，就不再轉錄。

以下這個連結可以列出 `System` 下所有覆寫(override) `GetHashCode()`  的類
別：

https://github.com/dotnet/coreclr/search?utf8=%E2%9C%93&q=%22public+override+int+GetHashCode%22+extension%3Acs+path%3A%2Fsrc%2Fmscorlib%2Fsrc%2FSystem


## `System.Object`

https://github.com/dotnet/coreclr/blob/b78b71f220ccb28eb6a5f9ea903536bdb6cd3f3d/src/mscorlib/src/System/Object.cs#L83-L95

```
    // GetHashCode is intended to serve as a hash function for this object.
    // Based on the contents of the object, the hash function will return a suitable
    // value with a relatively random distribution over the various inputs.
    //
    // The default implementation returns the sync block index for this instance.
    // Calling it on the same object multiple times will return the same value, so
    // it will technically meet the needs of a hash function, but it's less than ideal.
    // Objects (& especially value classes) should override this method.
    // 
    public virtual int GetHashCode()
    {
        return RuntimeHelpers.GetHashCode(this);
    }
```

https://github.com/dotnet/coreclr/blob/b78b71f220ccb28eb6a5f9ea903536bdb6cd3f3d/src/mscorlib/src/System/Runtime/CompilerServices/RuntimeHelpers.cs#L169-L171

```
        [System.Security.SecuritySafeCritical]  // auto-generated
        [MethodImplAttribute(MethodImplOptions.InternalCall)]
        public static extern int GetHashCode(Object o);
```

https://github.com/dotnet/coreclr/blob/b78b71f220ccb28eb6a5f9ea903536bdb6cd3f3d/src/vm/ecalllist.h#L2354

```
FCClassElement("RuntimeHelpers", "System.Runtime.CompilerServices", gCompilerFuncs)
```

https://github.com/dotnet/coreclr/blob/b78b71f220ccb28eb6a5f9ea903536bdb6cd3f3d/src/vm/ecalllist.h#L1855

```
    FCFuncElement("GetHashCode", ObjectNative::GetHashCode)
```

https://github.com/dotnet/coreclr/blob/b78b71f220ccb28eb6a5f9ea903536bdb6cd3f3d/src/classlibnative/bcltype/objectnative.cpp

https://github.com/dotnet/coreclr/blob/b78b71f220ccb28eb6a5f9ea903536bdb6cd3f3d/src/classlibnative/bcltype/objectnative.cpp#L101-L148

```
    static FCDECL1(INT32, GetHashCode, Object* vThisRef);
```

最後來到

https://github.com/dotnet/coreclr/blob/b78b71f220ccb28eb6a5f9ea903536bdb6cd3f3d/src/vm/object.h#L472

https://github.com/dotnet/coreclr/blob/b78b71f220ccb28eb6a5f9ea903536bdb6cd3f3d/src/vm/object.cpp#L75-L155

```
    INT32 GetHashCodeEx();
```


## `System.Int32`

https://github.com/dotnet/coreclr/blob/b78b71f220ccb28eb6a5f9ea903536bdb6cd3f3d/src/mscorlib/src/System/Int32.cs#L76-L78

```
        public override int GetHashCode() {
            return m_value;
        }
```


## `System.UInt32`

https://github.com/dotnet/coreclr/blob/b78b71f220ccb28eb6a5f9ea903536bdb6cd3f3d/src/mscorlib/src/System/UInt32.cs#L76-L79

```
        // The absolute value of the int contained.
        public override int GetHashCode() {
            return ((int) m_value);
        }
```


## `System.Int64`

https://github.com/dotnet/coreclr/blob/b78b71f220ccb28eb6a5f9ea903536bdb6cd3f3d/src/mscorlib/src/System/Int64.cs#L75-L77

```
        public override int GetHashCode() {
            return (unchecked((int)((long)m_value)) ^ (int)(m_value >> 32));
        }
```


## `System.UInt64`

https://github.com/dotnet/coreclr/blob/b78b71f220ccb28eb6a5f9ea903536bdb6cd3f3d/src/mscorlib/src/System/UInt64.cs#L73-L76

```
        // The value of the lower 32 bits XORed with the uppper 32 bits.
        public override int GetHashCode() {
            return ((int)m_value) ^ (int)(m_value >> 32);
        }
```


## `System.Double`

https://github.com/dotnet/coreclr/blob/b78b71f220ccb28eb6a5f9ea903536bdb6cd3f3d/src/mscorlib/src/System/Double.cs#L190-L199

```
        [System.Security.SecuritySafeCritical]
        public unsafe override int GetHashCode() {
            double d = m_value;
            if (d == 0) {
                // Ensure that 0 and -0 have the same hash code
                return 0;
            }
            long value = *(long*)(&d);
            return unchecked((int)value) ^ ((int)(value >> 32));
        }
```


## `System.Decimal`

https://github.com/dotnet/coreclr/blob/b78b71f220ccb28eb6a5f9ea903536bdb6cd3f3d/src/mscorlib/src/System/Decimal.cs#L441-L445

```
        // Returns the hash code for this Decimal.
        //
        [System.Security.SecuritySafeCritical]  // auto-generated
        [MethodImplAttribute(MethodImplOptions.InternalCall)]
        public extern override int GetHashCode();
```

https://github.com/dotnet/coreclr/blob/b78b71f220ccb28eb6a5f9ea903536bdb6cd3f3d/src/classlibnative/bcltype/decimal.h#L26

https://github.com/dotnet/coreclr/blob/b78b71f220ccb28eb6a5f9ea903536bdb6cd3f3d/src/classlibnative/bcltype/decimal.cpp#L102-L126

```
    static FCDECL1(INT32, GetHashCode, DECIMAL *d);
```


## `System.String`

https://github.com/dotnet/coreclr/blob/b78b71f220ccb28eb6a5f9ea903536bdb6cd3f3d/src/mscorlib/src/System/String.Comparison.cs#L999-L1013

```
        // Gets a hash code for this string.  If strings A and B are such that A.Equals(B), then
        // they will return the same hash code.
        [System.Security.SecuritySafeCritical]  // auto-generated
        [ReliabilityContract(Consistency.WillNotCorruptState, Cer.MayFail)]
        public override int GetHashCode()
        {
#if FEATURE_RANDOMIZED_STRING_HASHING
            if (HashHelpers.s_UseRandomizedStringHashing)
            {
                return InternalMarvin32HashString(this, this.Length, 0);
            }
#endif // FEATURE_RANDOMIZED_STRING_HASHING

            return GetLegacyNonRandomizedHashCode();
        }
```

https://github.com/dotnet/coreclr/blob/b78b71f220ccb28eb6a5f9ea903536bdb6cd3f3d/src/mscorlib/src/System/String.Comparison.cs#L1015-L1069

```
        // Use this if and only if you need the hashcode to not change across app domains (e.g. you have an app domain agile
        // hash table).
        [System.Security.SecuritySafeCritical]  // auto-generated
        [ReliabilityContract(Consistency.WillNotCorruptState, Cer.MayFail)]
        internal int GetLegacyNonRandomizedHashCode() {
            unsafe {
                fixed (char* src = &m_firstChar) {
                    Contract.Assert(src[this.Length] == '\0', "src[this.Length] == '\\0'");
                    Contract.Assert( ((int)src)%4 == 0, "Managed string should start at 4 bytes boundary");
#if BIT64
                    int hash1 = 5381;
#else // !BIT64 (32)
                    int hash1 = (5381<<16) + 5381;
#endif
                    int hash2 = hash1;

#if BIT64
                    int     c;
                    char *s = src;
                    while ((c = s[0]) != 0) {
                        hash1 = ((hash1 << 5) + hash1) ^ c;
                        c = s[1];
                        if (c == 0)
                            break;
                        hash2 = ((hash2 << 5) + hash2) ^ c;
                        s += 2;
                    }
#else // !BIT64 (32)
                    // 32 bit machines.
                    int* pint = (int *)src;
                    int len = this.Length;
                    while (len > 2)
                    {
                        hash1 = ((hash1 << 5) + hash1 + (hash1 >> 27)) ^ pint[0];
                        hash2 = ((hash2 << 5) + hash2 + (hash2 >> 27)) ^ pint[1];
                        pint += 2;
                        len  -= 4;
                    }

                    if (len > 0)
                    {
                        hash1 = ((hash1 << 5) + hash1 + (hash1 >> 27)) ^ pint[0];
                    }
#endif
#if DEBUG
                    // We want to ensure we can change our hash function daily.
                    // This is perfectly fine as long as you don't persist the
                    // value from GetHashCode to disk or count on String A 
                    // hashing before string B.  Those are bugs in your code.
                    hash1 ^= ThisAssembly.DailyBuildNumber;
#endif
                    return hash1 + (hash2 * 1566083941);
                }
            }
        }
```

https://github.com/dotnet/coreclr/blob/b78b71f220ccb28eb6a5f9ea903536bdb6cd3f3d/src/mscorlib/src/System/String.Comparison.cs#L982-L986

```
        // Do not remove!
        // This method is called by reflection in System.Xml
        [System.Security.SecurityCritical]
        [MethodImplAttribute(MethodImplOptions.InternalCall)]
        internal static extern int InternalMarvin32HashString(string s, int strLen, long additionalEntropy);
```

https://github.com/dotnet/coreclr/blob/b78b71f220ccb28eb6a5f9ea903536bdb6cd3f3d/src/vm/ecalllist.h#L227

```
    FCFuncElement("InternalMarvin32HashString", COMString::Marvin32HashString)
```

https://github.com/dotnet/coreclr/blob/b78b71f220ccb28eb6a5f9ea903536bdb6cd3f3d/src/classlibnative/bcltype/stringnative.h#L89

https://github.com/dotnet/coreclr/blob/b78b71f220ccb28eb6a5f9ea903536bdb6cd3f3d/src/classlibnative/bcltype/stringnative.cpp#L171-L187

```
    static FCDECL3(INT32, Marvin32HashString, StringObject* thisRefUNSAFE, INT32 strLen, INT64 additionalEntropy);
```


## `System.Single`

https://github.com/dotnet/coreclr/blob/b78b71f220ccb28eb6a5f9ea903536bdb6cd3f3d/src/mscorlib/src/System/Single.cs#L166-L175

```
        [System.Security.SecuritySafeCritical]  // auto-generated
        public unsafe override int GetHashCode() {
            float f = m_value;
            if (f == 0) {
                // Ensure that 0 and -0 have the same hash code
                return 0;
            }
            int v = *(int*)(&f);
            return v;
        }
```


## `System.ValueType`

https://github.com/dotnet/coreclr/blob/b78b71f220ccb28eb6a5f9ea903536bdb6cd3f3d/src/mscorlib/src/System/ValueType.cs#L71-L83

```
        /*=================================GetHashCode==================================
        **Action: Our algorithm for returning the hashcode is a little bit complex.  We look
        **        for the first non-static field and get it's hashcode.  If the type has no
        **        non-static fields, we return the hashcode of the type.  We can't take the
        **        hashcode of a static member because if that member is of the same type as
        **        the original type, we'll end up in an infinite loop.
        **Returns: The hashcode for the type.
        **Arguments: None.
        **Exceptions: None.
        ==============================================================================*/
        [System.Security.SecuritySafeCritical]  // auto-generated
        [MethodImplAttribute(MethodImplOptions.InternalCall)]
        public extern override int GetHashCode();
```

https://github.com/dotnet/coreclr/blob/b78b71f220ccb28eb6a5f9ea903536bdb6cd3f3d/src/vm/ecalllist.h#L240

```
    FCFuncElement("GetHashCode", ValueTypeHelper::GetHashCode)
```

https://github.com/dotnet/coreclr/blob/b78b71f220ccb28eb6a5f9ea903536bdb6cd3f3d/src/vm/comutilnative.h#L248

https://github.com/dotnet/coreclr/blob/b78b71f220ccb28eb6a5f9ea903536bdb6cd3f3d/src/vm/comutilnative.cpp#L2778-L2823

```
    static FCDECL1(INT32, GetHashCode, Object* objRef);
```
