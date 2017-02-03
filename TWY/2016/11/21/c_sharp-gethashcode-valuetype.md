> http://www.30abysses.com/TWY/2016/11/21/c_sharp-gethashcode-valuetype.md
> by TW Yang <twy@30abysses.com> 2016-11-21 CC-BY-4.0

# ＣＳ： C# `System.Valuetype.GetHashCode()`  潛在效能、安全問題

`ValueType`  的 `GetHashCode()  預設實作是把所有欄位(field) 大鍋炒成
雜湊(hash)值；有看過些說法主張這可能造成潛在的效能問題外，然而，更嚴重的
是如其[原碼註解][1] 所指出的，這多少會在某種程度上把敏感資訊透露出去，也
就是有 **潛在的安全問題** 。

[1]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/vm/comutilnative.cpp#L2770-L2774

```
// The default implementation of GetHashCode() for all value types.
// Note that this implementation reveals the value of the fields.
// So if the value type contains any sensitive information it should
// implement its own GetHashCode().
FCIMPL1(INT32, ValueTypeHelper::GetHashCode, Object* objUNSAFE)
```

是故，原碼註解主張

> 若此 value type 含有敏感資訊，那它應該為 `GetHashCode()`  提供自己的實
> 作版本。

然而，很遺憾的， [`System.Valuetype.GetHashCode()` 的官方文件][2]
目前(2016-11-25)並沒有特別提醒這件事。

[2]: https://msdn.microsoft.com/en-us/library/system.valuetype.gethashcode.aspx
