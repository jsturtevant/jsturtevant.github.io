---
layout: post
title: Aligning-csharp-stackalloc-arrays.markdown
date: "2024-01-18"
categories:
  - csharp
  - dotnet
---

I recently needed ran into some alignment issues while building things with c# for Wasm component workloads.  The tool was allocating some space on on the stack to recieve data from a different component. I was using [`stackalloc`](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/stackalloc) command to accompish this to avoid allocation and garbage collection while calling out to the native function.

To recieve data from a different component you pass a pointer to some allocated memory and the other component will fill in the memory.  There are a lot of details on how this works in the [component model spec](https://github.com/WebAssembly/component-model/blob/main/design/mvp/CanonicalABI.md) but in this case I needed enough space to hold a 32 bit pointer and a 32 bit integer.  

In csharp, a 32 bit pointer can be represented by [`uint`](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/integral-numeric-types#characteristics-of-the-integral-types) since `4 bytes` == `32 bits` == `uint`. The function I wrote looked like:

```csharp
public  static unsafe byte[] ListResult()
{
    var retArea = stackalloc uint[2];
    ListResultWasmInterop.wasmImportListResult(ptr);

    //do stuuff with the data that was returned in ptr
}
```

When I ran some of the functions returned properly and some did not:

```
Pointer not aligned
```

This looks strange initially, I have enough space to save things and 