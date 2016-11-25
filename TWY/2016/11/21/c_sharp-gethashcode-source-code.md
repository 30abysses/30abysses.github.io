> http://www.30abysses.com/TWY/2016/11/21/c_sharp-gethashcode-source-code.md
> by TW Yang <twy@30abysses.com> 2016-11-21 CC-BY-4.0

# C# `GetHashCode()`  原始碼

這裡自 .NET Core 1.1.0  摘錄一些 `GetHashCode()`  實作作為參考。


## `mscorlib` 中有哪些類別(class) 有覆寫(override) `GetHashCode()`  ？

[用以下條件對在 Github.com 上對 `dotnet/coreclr` 蒐尋][1] 的話，可以找到
約 100個例子；在此僅挑選幾個原始(primitive) 資料類別。

```
    "public override int GetHashCode"
    extension:cs
    path:/src/mscorlib/src/System
```

[1]: https://github.com/dotnet/coreclr/search?utf8=%E2%9C%93&q=%22public+override+int+GetHashCode%22+extension%3Acs+path%3A%2Fsrc%2Fmscorlib%2Fsrc%2FSystem


---
## `System.Object`

先從 [`/mscorlib/src/System/Object.cs#L83-L95`][2]  追起。

[2]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/mscorlib/src/System/Object.cs#L83-L95

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

再追 [`/System/Runtime/CompilerServices/RuntimeHelpers.cs#L153-L155`][3]

[3]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/mscorlib/src/System/Runtime/CompilerServices/RuntimeHelpers.cs#L153-L155

```
        [System.Security.SecuritySafeCritical]  // auto-generated
        [MethodImplAttribute(MethodImplOptions.InternalCall)]
        public static extern int GetHashCode(Object o);
```

踏入原生(native)碼的領域， [`/vm/ecalllist.h#L2346`][4]

[4]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/vm/ecalllist.h#L2346

```
FCClassElement("RuntimeHelpers", "System.Runtime.CompilerServices", gCompilerFuncs)
```

繼續追

[`/vm/ecalllist.h#L1828`][5]

[5]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/vm/ecalllist.h#L1828

```
FCFuncStart(gCompilerFuncs)
```

[`/vm/ecalllist.h#L1855`][6]

[6]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/vm/ecalllist.h#L1843

```
    FCFuncElement("GetHashCode", ObjectNative::GetHashCode)
```

再追， [`/classlibnative/bcltype/objectnative.h#L33`][7]

[7]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/classlibnative/bcltype/objectnative.h#L33

```
    static FCDECL1(INT32, GetHashCode, Object* vThisRef);
```

[`/classlibnative/bcltype/objectnative.cpp#L101-L149`][8]

[8]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/classlibnative/bcltype/objectnative.cpp#L101-L149

```
// Note that we obtain a sync block index without actually building a sync block.
// That's because a lot of objects are hashed, without requiring support for
FCIMPL1(INT32, ObjectNative::GetHashCode, Object* obj) {

    CONTRACTL
    {
        FCALL_CHECK;
        INJECT_FAULT(FCThrow(kOutOfMemoryException););
    }
    CONTRACTL_END;

    VALIDATEOBJECT(obj);

    if (obj == 0)
        return 0;

    OBJECTREF objRef(obj);

#ifdef MDA_SUPPORTED
    if (!MDA_GET_ASSISTANT(ModuloObjectHashcode))
#endif
    {
        DWORD bits = objRef->GetHeader()->GetBits();

        if (bits & BIT_SBLK_IS_HASH_OR_SYNCBLKINDEX)
        {
            if (bits & BIT_SBLK_IS_HASHCODE)
            {
                // Common case: the object already has a hash code
                return  bits & MASK_HASHCODE;
            }
            else
            {
                // We have a sync block index. This means if we already have a hash code,
                // it is in the sync block, otherwise we generate a new one and store it there
                SyncBlock *psb = objRef->PassiveGetSyncBlock();
                if (psb != NULL)
                {
                    DWORD hashCode = psb->GetHashCode();
                    if (hashCode != 0)
                        return  hashCode;
                }
            }
        }
    }

    FC_INNER_RETURN(INT32, GetHashCodeHelper(objRef));
}
FCIMPLEND
```

[`/classlibnative/bcltype/objectnative.cpp#L77-L99`][9]

[9]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/classlibnative/bcltype/objectnative.cpp#L77-L99

```
NOINLINE static INT32 GetHashCodeHelper(OBJECTREF objRef)
{
    DWORD idx = 0;

    FC_INNER_PROLOG(ObjectNative::GetHashCode);

    HELPER_METHOD_FRAME_BEGIN_RET_ATTRIB_1(Frame::FRAME_ATTR_EXACT_DEPTH|Frame::FRAME_ATTR_CAPTURE_DEPTH_2, objRef);

#ifdef MDA_SUPPORTED
    MdaModuloObjectHashcode* pProbe = MDA_GET_ASSISTANT(ModuloObjectHashcode);
#endif

    idx = objRef->GetHashCodeEx();

#ifdef MDA_SUPPORTED
    if (pProbe)
        idx = idx % pProbe->GetModulo();
#endif

    HELPER_METHOD_FRAME_END();
    FC_INNER_EPILOG();
    return idx;
}
```

最後來到

[`/vm/object.h#L472`][10]

[10]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/vm/object.h#L472

```
    INT32 GetHashCodeEx();
```

[`/vm/object.cpp#L75-L155`][11]

[11]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/vm/object.cpp#L75-L155

```
INT32 Object::GetHashCodeEx()
{
    CONTRACTL
    {
        MODE_COOPERATIVE;
        THROWS;
        GC_NOTRIGGER;
        SO_TOLERANT;
    }
    CONTRACTL_END

    // This loop exists because we're inspecting the header dword of the object
    // and it may change under us because of races with other threads.
    // On top of that, it may have the spin lock bit set, in which case we're
    // not supposed to change it.
    // In all of these case, we need to retry the operation.
    DWORD iter = 0;
    DWORD dwSwitchCount = 0;
    while (true)
    {
        DWORD bits = GetHeader()->GetBits();

        if (bits & BIT_SBLK_IS_HASH_OR_SYNCBLKINDEX)
        {
            if (bits & BIT_SBLK_IS_HASHCODE)
            {
                // Common case: the object already has a hash code
                return  bits & MASK_HASHCODE;
            }
            else
            {
                // We have a sync block index. This means if we already have a hash code,
                // it is in the sync block, otherwise we generate a new one and store it there
                SyncBlock *psb = GetSyncBlock();
                DWORD hashCode = psb->GetHashCode();
                if (hashCode != 0)
                    return  hashCode;

                hashCode = ComputeHashCode();

                return psb->SetHashCode(hashCode);
            }
        }
        else
        {
            // If a thread is holding the thin lock or an appdomain index is set, we need a syncblock
            if ((bits & (SBLK_MASK_LOCK_THREADID | (SBLK_MASK_APPDOMAININDEX << SBLK_APPDOMAIN_SHIFT))) != 0)
            {
                GetSyncBlock();
                // No need to replicate the above code dealing with sync blocks
                // here - in the next iteration of the loop, we'll realize
                // we have a syncblock, and we'll do the right thing.
            }
            else
            {
                // We want to change the header in this case, so we have to check the BIT_SBLK_SPIN_LOCK bit first
                if (bits & BIT_SBLK_SPIN_LOCK)
                {
                    iter++;
                    if ((iter % 1024) != 0 && g_SystemInfo.dwNumberOfProcessors > 1)
                    {
                        YieldProcessor();           // indicate to the processor that we are spining
                    }
                    else
                    {
                        __SwitchToThread(0, ++dwSwitchCount);
                    }
                    continue;
                }

                DWORD hashCode = ComputeHashCode();

                DWORD newBits = bits | BIT_SBLK_IS_HASH_OR_SYNCBLKINDEX | BIT_SBLK_IS_HASHCODE | hashCode;

                if (GetHeader()->SetBits(newBits, bits) == bits)
                    return hashCode;
                // Header changed under us - let's restart this whole thing.
            }
        }
    }
}
```

再往下我就追不動了 :D


---
## `System.Int32`

[`/mscorlib/src/System/Int32.cs#L75-L78`][12]

[12]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/mscorlib/src/System/Int32.cs#L75-L78

```
        // The absolute value of the int contained.
        public override int GetHashCode() {
            return m_value;
        }
```


---
## `System.UInt32`

[`/mscorlib/src/System/UInt32.cs#L76-L79`][13]

[13]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/mscorlib/src/System/UInt32.cs#L76-L79

```
        // The absolute value of the int contained.
        public override int GetHashCode() {
            return ((int) m_value);
        }
```


---
## `System.Int64`

[`/mscorlib/src/System/Int64.cs#L75-L77`][14]

[14]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/mscorlib/src/System/Int64.cs#L75-L77

```
        public override int GetHashCode() {
            return (unchecked((int)((long)m_value)) ^ (int)(m_value >> 32));
        }
```


---
## `System.UInt64`

[`/mscorlib/src/System/UInt64.cs#L73-L76`][15]

[15]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/mscorlib/src/System/UInt64.cs#L73-L76

```
        // The value of the lower 32 bits XORed with the uppper 32 bits.
        public override int GetHashCode() {
            return ((int)m_value) ^ (int)(m_value >> 32);
        }
```


---
## `System.Double`

[`/mscorlib/src/System/Double.cs#L187-L199`][16]

[16]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/mscorlib/src/System/Double.cs#L187-L199

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


---
## `System.Decimal`

[`/mscorlib/src/System/Decimal.cs#L441-L445`][17]

[17]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/mscorlib/src/System/Decimal.cs#L441-L445

```
        // Returns the hash code for this Decimal.
        //
        [System.Security.SecuritySafeCritical]  // auto-generated
        [MethodImplAttribute(MethodImplOptions.InternalCall)]
        public extern override int GetHashCode();
```

[`/classlibnative/bcltype/decimal.h#L26`][18]

[18]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/classlibnative/bcltype/decimal.h#L26

```
    static FCDECL1(INT32, GetHashCode, DECIMAL *d);
```

[`/classlibnative/bcltype/decimal.cpp#L102-L126`][19]

[19]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/classlibnative/bcltype/decimal.cpp#L102-L126

```
FCIMPL1(INT32, COMDecimal::GetHashCode, DECIMAL *d)
{
    FCALL_CONTRACT;

    ENSURE_OLEAUT32_LOADED();

    _ASSERTE(d != NULL);
    double dbl;
    VarR8FromDec(d, &dbl);
    if (dbl == 0.0) {
        // Ensure 0 and -0 have the same hash code
        return 0;
    }
    // conversion to double is lossy and produces rounding errors so we mask off the lowest 4 bits
    //
    // For example these two numerically equal decimals with different internal representations produce
    // slightly different results when converted to double:
    //
    // decimal a = new decimal(new int[] { 0x76969696, 0x2fdd49fa, 0x409783ff, 0x00160000 });
    //                     => (decimal)1999021.176470588235294117647000000000 => (double)1999021.176470588
    // decimal b = new decimal(new int[] { 0x3f0f0f0f, 0x1e62edcc, 0x06758d33, 0x00150000 });
    //                     => (decimal)1999021.176470588235294117647000000000 => (double)1999021.1764705882
    //
    return ((((int *)&dbl)[0]) & 0xFFFFFFF0) ^ ((int *)&dbl)[1];
}
FCIMPLEND
```


---
## `System.String`

[`/mscorlib/src/System/String.Comparison.cs#L999-L1013`][20]

[20]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/mscorlib/src/System/String.Comparison.cs#L999-L1013

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

[`/mscorlib/src/System/String.Comparison.cs#L982-L986`][21]

[21]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/mscorlib/src/System/String.Comparison.cs#L982-L986

```
        // Do not remove!
        // This method is called by reflection in System.Xml
        [System.Security.SecurityCritical]
        [MethodImplAttribute(MethodImplOptions.InternalCall)]
        internal static extern int InternalMarvin32HashString(string s, int strLen, long additionalEntropy);
```

[`/vm/ecalllist.h#L2394`][22]

[22]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/vm/ecalllist.h#L2394

```
FCClassElement("String", "System", gStringFuncs)
```

[`/vm/ecalllist.h#L212`][23]

[23]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/vm/ecalllist.h#L212

```
FCFuncStart(gStringFuncs)
```

[`/vm/ecalllist.h#L236`][24]

[24]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/vm/ecalllist.h#L236

```
    FCFuncElement("InternalMarvin32HashString", COMString::Marvin32HashString)
```

[`/classlibnative/bcltype/stringnative.h#L89`][25]

[25]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/classlibnative/bcltype/stringnative.h#L89

```
    static FCDECL3(INT32, Marvin32HashString, StringObject* thisRefUNSAFE, INT32 strLen, INT64 additionalEntropy);
```

[`/classlibnative/bcltype/stringnative.cpp#L171-L188`][26]

[26]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/classlibnative/bcltype/stringnative.cpp#L171-L188

```
FCIMPL3(INT32, COMString::Marvin32HashString, StringObject* thisRefUNSAFE, INT32 strLen, INT64 additionalEntropy) {
    FCALL_CONTRACT;

    int iReturnHash = 0;

    if (thisRefUNSAFE == NULL) {
        FCThrow(kNullReferenceException);
    }

    BEGIN_SO_INTOLERANT_CODE_NOTHROW(GetThread(), FCThrow(kStackOverflowException));
    iReturnHash = GetCurrentNlsHashProvider()->HashString(thisRefUNSAFE->GetBuffer(), thisRefUNSAFE->GetStringLength(), TRUE, additionalEntropy);
    END_SO_INTOLERANT_CODE;

    FC_GC_POLL_RET();

    return iReturnHash;
}
FCIMPLEND
```

[`/mscorlib/src/System/String.Comparison.cs#L1015-L1069`][27]

[27]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/mscorlib/src/System/String.Comparison.cs#L1015-L1069

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


---
## `System.Single`

[`/mscorlib/src/System/Single.cs#L158-L167`][28]

[28]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/mscorlib/src/System/Single.cs#L158-L167

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


---
## `System.ValueType`

[`/mscorlib/src/System/ValueType.cs#L71-L83`][29]

[29]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/mscorlib/src/System/ValueType.cs#L71-L83

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

[`/vm/ecalllist.h#L2430`][30]

[30]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/vm/ecalllist.h#L2430

```
FCClassElement("ValueType", "System", gValueTypeFuncs)
```

[`/vm/ecalllist.h#L249`][31]

[31]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/vm/ecalllist.h#L249

```
    FCFuncElement("GetHashCode", ValueTypeHelper::GetHashCode)
```

[`/vm/comutilnative.h#L248`][32]

[32]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/vm/comutilnative.h#L248

```
    static FCDECL1(INT32, GetHashCode, Object* objRef);
```

[`/vm/comutilnative.cpp#L2770-L2816`][33]

[33]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/vm/comutilnative.cpp#L2770-L2816

```
// The default implementation of GetHashCode() for all value types.
// Note that this implementation reveals the value of the fields.
// So if the value type contains any sensitive information it should
// implement its own GetHashCode().
FCIMPL1(INT32, ValueTypeHelper::GetHashCode, Object* objUNSAFE)
{
    FCALL_CONTRACT;

    if (objUNSAFE == NULL)
        FCThrow(kNullReferenceException);

    OBJECTREF obj = ObjectToOBJECTREF(objUNSAFE);
    VALIDATEOBJECTREF(obj);

    INT32 hashCode = 0;
    MethodTable *pMT = objUNSAFE->GetMethodTable();

    // We don't want to expose the method table pointer in the hash code
    // Let's use the typeID instead.
    UINT32 typeID = pMT->LookupTypeID();
    if (typeID == TypeIDProvider::INVALID_TYPE_ID)
    {
        // If the typeID has yet to be generated, fall back to GetTypeID
        // This only needs to be done once per MethodTable
        HELPER_METHOD_FRAME_BEGIN_RET_1(obj);
        typeID = pMT->GetTypeID();
        HELPER_METHOD_FRAME_END();
    }

    // To get less colliding and more evenly distributed hash codes,
    // we munge the class index with two big prime numbers
    hashCode = typeID * 711650207 + 2506965631U;

    if (CanUseFastGetHashCodeHelper(pMT))
    {
        hashCode ^= FastGetValueTypeHashCodeHelper(pMT, obj->UnBox());
    }
    else
    {
        HELPER_METHOD_FRAME_BEGIN_RET_1(obj);
        hashCode ^= RegularGetValueTypeHashCode(pMT, obj->UnBox());
        HELPER_METHOD_FRAME_END();
    }

    return hashCode;
}
FCIMPLEND
```


---
## System.Enum

[`/mscorlib/src/System/Enum.cs#L811-L854`][34]

[34]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/mscorlib/src/System/Enum.cs#L811-L854

```
        [System.Security.SecuritySafeCritical]
        public override unsafe int GetHashCode()
        {
            // Avoid boxing by inlining GetValue()
            // return GetValue().GetHashCode();

            fixed (void* pValue = &JitHelpers.GetPinningHelper(this).m_data)
            {
                switch (InternalGetCorElementType())
                {
                    case CorElementType.I1:
                        return (*(sbyte*)pValue).GetHashCode();
                    case CorElementType.U1:
                        return (*(byte*)pValue).GetHashCode();
                    case CorElementType.Boolean:
                        return (*(bool*)pValue).GetHashCode();
                    case CorElementType.I2:
                        return (*(short*)pValue).GetHashCode();
                    case CorElementType.U2:
                        return (*(ushort*)pValue).GetHashCode();
                    case CorElementType.Char:
                        return (*(char*)pValue).GetHashCode();
                    case CorElementType.I4:
                        return (*(int*)pValue).GetHashCode();
                    case CorElementType.U4:
                        return (*(uint*)pValue).GetHashCode();
                    case CorElementType.R4:
                        return (*(float*)pValue).GetHashCode();
                    case CorElementType.I8:
                        return (*(long*)pValue).GetHashCode();
                    case CorElementType.U8:
                        return (*(ulong*)pValue).GetHashCode();
                    case CorElementType.R8:
                        return (*(double*)pValue).GetHashCode();
                    case CorElementType.I:
                        return (*(IntPtr*)pValue).GetHashCode();
                    case CorElementType.U:
                        return (*(UIntPtr*)pValue).GetHashCode();
                    default:
                        Contract.Assert(false, "Invalid primitive type");
                        return 0;
                }
            }
        }
```


---
## System.IntPtr

[`/mscorlib/src/System/IntPtr.cs#L114-L126`][35]

[35]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/mscorlib/src/System/IntPtr.cs#L114-L126

```
        [System.Security.SecuritySafeCritical]  // auto-generated
        public unsafe override int GetHashCode() {
#if FEATURE_CORECLR
#if BIT64
            long l = (long)m_value;
            return (unchecked((int)l) ^ (int)(l >> 32));
#else // !BIT64 (32)
            return unchecked((int)m_value);
#endif
#else
            return unchecked((int)((long)m_value));
#endif
        }
```


---
## System.DateTime

[`/mscorlib/src/System/DateTime.cs#L822-L827`][36]

[36]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/mscorlib/src/System/DateTime.cs#L822-L827

```
        // Returns the hash code for this DateTime.
        //
        public override int GetHashCode() {
            Int64 ticks = InternalTicks;
            return unchecked((int)ticks) ^ (int)(ticks >> 32);
        }
```


---
## System.TimeSpan

[`/mscorlib/src/System/TimeSpan.cs#L215-L217`][37]

[37]: https://github.com/dotnet/coreclr/blob/release/1.1.0/src/mscorlib/src/System/TimeSpan.cs#L215-L217

```
        public override int GetHashCode() {
            return (int)_ticks ^ (int)(_ticks >> 32);
        }
```
