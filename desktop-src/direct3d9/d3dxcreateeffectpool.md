---
Description: Create an effect pool. A pool is used to share parameters between effects.
ms.assetid: 5b202f85-b32b-4041-8873-0de535c2f59f
title: D3DXCreateEffectPool function
ms.technology: desktop
ms.prod: windows
ms.author: windowssdkdev
ms.topic: article
ms.date: 05/31/2018
---

# D3DXCreateEffectPool function

Create an effect pool. A pool is used to share parameters between effects.

## Syntax


```C++
HRESULT D3DXCreateEffectPool(
  _Out_ LPD3DXEFFECTPOOL *ppPool
);
```



## Parameters

<dl> <dt>

*ppPool* \[out\]
</dt> <dd>

Type: **[**LPD3DXEFFECTPOOL**](id3dxeffectpool.md)\***

Returns a pointer to the created pool.

</dd> </dl>

## Return value

Type: **[**HRESULT**](https://msdn.microsoft.com/windows/desktop/455d07e9-52c3-4efb-a9dc-2955cbfd38cc)**

If the method succeeds, the return value is S\_OK.

If the arguments are invalid, the method will return D3DERR\_INVALIDCALL.

If the method fails, the return value will be E\_FAIL.

## Remarks

For effects within a pool, shared parameters with the same name share values.

## Requirements



|                    |                                                                                          |
|--------------------|------------------------------------------------------------------------------------------|
| Header<br/>  | <dl> <dt>D3DX9Effect.h</dt> </dl> |
| Library<br/> | <dl> <dt>D3dx9.lib</dt> </dl>     |



## See also

<dl> <dt>

[Effect Functions](dx9-graphics-reference-effects-functions.md)
</dt> </dl>

 

 



