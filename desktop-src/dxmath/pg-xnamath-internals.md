---
Description: This topic describes the internal design of the DirectXMath library.
ms.assetid: 31512657-c413-9e6e-e343-1ea677a02b8c
title: Library Internals
ms.technology: desktop
ms.prod: windows
ms.author: windowssdkdev
ms.topic: article
ms.date: 05/31/2018
---

# Library Internals

This topic describes the internal design of the DirectXMath library.

-   [Calling Conventions](#calling-conventions)
-   [Graphics Library Type Equivalence](#graphics-library-type-equivalence)
-   [Global Constants in the DirectXMath Library](#global-constants-in-the-directxmath-library)
-   [Windows SSE versus SSE2](#windows-sse-versus-sse2)
-   [Routine Variants](#routine-variants)
-   [Platform Inconsistencies](#platform-inconsistencies)
-   [Platform-specific Extensions](#platform-specific-extensions)
-   [Related topics](#related-topics)

## Calling Conventions

To enhance portability and optimize data layout, you need to use the appropriate calling conventions for each platform supported by the DirectXMath Library. Specifically, when you pass [**XMVECTOR**](xmvector-data-type.md) objects as parameters, which are defined as aligned on a 16-byte boundary, there are different sets of calling requirements, depending on the target platform:

**For 32-bit Windows**

For 32-bit Windows, there are two calling conventions available for efficient passing of [\_\_m128](http://msdn.microsoft.com/en-us/library/ayeb3ayc.aspx) values (which implements [**XMVECTOR**](xmvector-data-type.md) on that platform). The standard is [\_\_fastcall](http://msdn.microsoft.com/en-us/library/6xa169sk(VS.71).aspx), which can pass the first three [\_\_m128](http://msdn.microsoft.com/en-us/library/ayeb3ayc.aspx) values (**XMVECTOR** instances) as arguments to a function in a *SSE/SSE2* register. [\_\_fastcall](http://msdn.microsoft.com/en-us/library/6xa169sk(VS.71).aspx) passes remaining arguments via the stack.

Newer Microsoft Visual Studio compilers support a new calling convention, \_\_vectorcall, which can pass up to six [\_\_m128](http://msdn.microsoft.com/en-us/library/ayeb3ayc.aspx) values ([**XMVECTOR**](xmvector-data-type.md) instances) as arguments to a function in a *SSE/SSE2* register. It can also pass heterogeneous vector aggregates (also known as [**XMMATRIX**](/windows/desktop/api/DirectXMath/)) via *SSE/SSE2* registers if there is sufficient room.

**For 64-bit editions of Windows**

For 64-bit Windows, there are two calling conventions available for efficient passing of [\_\_m128](http://msdn.microsoft.com/en-us/library/ayeb3ayc.aspx) values. The standard is [\_\_fastcall](http://msdn.microsoft.com/en-us/library/6xa169sk(VS.71).aspx), which passes all [\_\_m128](http://msdn.microsoft.com/en-us/library/ayeb3ayc.aspx) values on the stack.

Newer Visual Studio compilers support the \_\_vectorcall calling convention, which can pass up to six [\_\_m128](http://msdn.microsoft.com/en-us/library/ayeb3ayc.aspx) values ([**XMVECTOR**](xmvector-data-type.md) instances) as arguments to a function in a *SSE/SSE2* register. It can also pass heterogeneous vector aggregates (also known as [**XMMATRIX**](/windows/desktop/api/DirectXMath/)) via *SSE/SSE2* registers if there is sufficient room.

**For Windows RT**

The Windows RT operating system supports passing the first four \_\_n128 values ([**XMVECTOR**](xmvector-data-type.md) instances) in-register.

**DirectXMath solution**

The **FXMVECTOR**, **GXMVECTOR**, **HXMVECTOR**, and **CXMVECTOR** aliases support these conventions:

-   Use the **FXMVECTOR** alias to pass up to the first three instances of [**XMVECTOR**](xmvector-data-type.md) used as arguments to a function.
-   Use the **GXMVECTOR** alias to pass the 4th instance of an [**XMVECTOR**](xmvector-data-type.md) used as an argument to a function.
-   Use the **HXMVECTOR** alias to pass the 5th and 6th instances of an [**XMVECTOR**](xmvector-data-type.md) used as an argument to a function. For info about additional considerations, see the \_\_vectorcall documentation.
-   Use the **CXMVECTOR** alias to pass any further instances of [**XMVECTOR**](xmvector-data-type.md) used as arguments.

> [!Note]  
> For output parameters, always use XMVECTOR\* or XMVECTOR& and ignore them with respect to the preceding rules for input parameters.

 

Because of limitations with \_\_vectorcall, we recommend that you not use **GXMVECTOR** or **HXMVECTOR** for C++ constructors. Just use **FXMVECTOR** for the first three [**XMVECTOR**](xmvector-data-type.md) values, then use **CXMVECTOR** for the rest.

The **FXMMATRIX** and **CXMMATRIX** aliases help support taking advantage of the HVA argument passing with \_\_vectorcall.

-   Use the **FXMMATRIX** alias to pass the first [**XMMATRIX**](/windows/desktop/api/DirectXMath/) as an argument to the function. This assumes you don't have more than two **FXMVECTOR** arguments or more than two float, double, or **FXMVECTOR** arguments to the ‘right’ of the matrix. For info about additional considerations, see the \_\_vectorcall documentation.
-   Use the **CXMMATRIX** alias otherwise.

Because of limitations with \_\_vectorcall, we recommend that you never use **FXMMATRIX** for C++ constructors. Just use **CXMMATRIX**.

In addition to the type aliases, you must also use the XM\_CALLCONV annotation to make sure the function uses the appropriate calling convention (\_\_fastcall versus \_\_vectorcall) based on your compiler and architecture. Because of limitations with \_\_vectorcall, we recommend that you not use XM\_CALLCONV for C++ constructors.

The following are example declarations that illustrate this convention:


```C++
XMMATRIX XM_CALLCONV XMMatrixLookAtLH(FXMVECTOR EyePosition, FXMVECTOR FocusPosition, FXMVECTOR UpDirection);

XMMATRIX XM_CALLCONV XMMatrixTransformation2D(FXMVECTOR ScalingOrigin,  float ScalingOrientation, FXMVECTOR Scaling, FXMVECTOR RotationOrigin, float Rotation, GXMVECTOR Translation);

void XM_CALLCONV XMVectorSinCos(XMVECTOR* pSin, XMVECTOR* pCos, FXMVECTOR V);

XMVECTOR XM_CALLCONV XMVectorHermiteV(FXMVECTOR Position0, FXMVECTOR Tangent0, FXMVECTOR Position1, GXMVECTOR Tangent1, HXMVECTOR T);

XMMATRIX(FXMVECTOR R0, FXMVECTOR R1, FXMVECTOR R2, CXMVECTOR R3)

XMVECTOR XM_CALLCONV XMVector2Transform(FXMVECTOR V, FXMMATRIX M);

XMMATRIX XM_CALLCONV XMMatrixMultiplyTranspose(FXMMATRIX M1, CXMMATRIX M2);
```



To support these calling conventions, these type aliases are defined as follows (parameters must be passed by value for the compiler to consider them for in-register passing):

**For 32-bit Windows apps**

When you use \_\_fastcall:


```C++
typedef const XMVECTOR  FXMVECTOR;
typedef const XMVECTOR&amp; GXMVECTOR;
typedef const XMVECTOR&amp; HXMVECTOR;
typedef const XMVECTOR&amp; CXMVECTOR;
typedef const XMMATRIX&amp; FXMMATRIX;
typedef const XMMATRIX&amp; CXMMATRIX;
```



When you use \_\_vectorcall:


```C++
typedef const XMVECTOR  FXMVECTOR;
typedef const XMVECTOR  GXMVECTOR;
typedef const XMVECTOR  HXMVECTOR;
typedef const XMVECTOR&amp; CXMVECTOR;
typedef const XMMATRIX  FXMMATRIX;
typedef const XMMATRIX&amp; CXMMATRIX;
```



**For 64-bit native Windows apps**

When you use \_\_fastcall:


```C++
typedef const XMVECTOR&amp; FXMVECTOR;
typedef const XMVECTOR&amp; GXMVECTOR;
typedef const XMVECTOR&amp; HXMVECTOR;
typedef const XMVECTOR&amp; CXMVECTOR;
typedef const XMMATRIX&amp; FXMMATRIX;
typedef const XMMATRIX&amp; CXMMATRIX;
```



When you use \_\_vectorcall:


```C++
typedef const XMVECTOR  FXMVECTOR;
typedef const XMVECTOR  GXMVECTOR;
typedef const XMVECTOR  HXMVECTOR;
typedef const XMVECTOR&amp; CXMVECTOR;
typedef const XMMATRIX  FXMMATRIX;
typedef const XMMATRIX&amp; CXMMATRIX;
```



**Windows RT**


```C++
typedef const XMVECTOR  FXMVECTOR;
typedef const XMVECTOR  GXMVECTOR;
typedef const XMVECTOR&amp; CXMVECTOR;
typedef const XMMATRIX&amp; FXMMATRIX;
typedef const XMMATRIX&amp; CXMMATRIX;
```



> [!Note]  
> While all the functions are declared inline and in many cases the compiler won't need to use calling conventions for these functions, there are cases where the compiler may decide it's more efficient to not inline the function and in these cases we want the best calling convention possible for each platform.

 

## Graphics Library Type Equivalence

To support the use of the DirectXMath Library, many DirectXMath Library types and structures are equivalent to the Windows implementations of the **D3DDECLTYPE** and **D3DFORMAT** types, as well as the [**DXGI\_FORMAT**](https://msdn.microsoft.com/VS|directx_sdk|~\dxgi_format.htm) types. 

| DirectXMath                      | D3DDECLTYPE                                                                           | D3DFORMAT                                                     | DXGI\_FORMAT                                                                                                                                                                                            |
|----------------------------------|---------------------------------------------------------------------------------------|---------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [**XMBYTE2**](/windows/desktop/api/DirectXPackedVector/ns-directxpackedvector-xmbyte2)       |                                                                                       |                                                               | DXGI\_FORMAT\_R8G8\_SINT                                                                                                                                                                                |
| [**XMBYTE4**](/windows/desktop/api/DirectXPackedVector/ns-directxpackedvector-xmbyte4)       | D3DDECLTYPE\_BYTE4 (Xbox Only)                                                        | D3DFMT\_x8x8x8x8                                              | DXGI\_FORMAT\_x8x8x8x8\_SINT                                                                                                                                                                            |
| [**XMBYTEN2**](/windows/desktop/api/DirectXPackedVector/ns-directxpackedvector-xmbyten2)     |                                                                                       | D3DFMT\_V8U8                                                  | DXGI\_FORMAT\_R8G8\_SNORM                                                                                                                                                                               |
| [**XMBYTEN4**](/windows/desktop/api/DirectXPackedVector/ns-directxpackedvector-xmbyten4)     | D3DDECLTYPE\_BYTE4N (Xbox Only)                                                       | D3DFMT\_x8x8x8x8                                              | DXGI\_FORMAT\_x8x8x8x8\_SNORM                                                                                                                                                                           |
| [**XMCOLOR**](/windows/desktop/api/DirectXPackedVector/ns-directxpackedvector-xmcolor)       | D3DDECLTYPE\_D3DCOLOR                                                                 | D3DFMT\_A8R8G8B8                                              | DXGI\_FORMAT\_B8G8R8A8\_UNORM (DXGI 1.1+)                                                                                                                                                               |
| [**XMDEC4**](/windows/desktop/api/DirectXPackedVector/ns-directxpackedvector-xmdec4)         | D3DDECLTYPE\_DEC4 (Xbox Only)                                                         | D3DDECLTYPE\_DEC3 (Xbox Only)                                 |                                                                                                                                                                                                         |
| [**XMDECN4**](/windows/desktop/api/DirectXPackedVector/ns-directxpackedvector-xmdecn4)       | D3DDECLTYPE\_DEC4N (Xbox Only)                                                        | D3DDECLTYPE\_DEC3N (Xbox Only)                                |                                                                                                                                                                                                         |
| [**XMFLOAT2**](/windows/desktop/api/DirectXMath/)     | D3DDECLTYPE\_FLOAT2                                                                   | D3DFMT\_G32R32F                                               | DXGI\_FORMAT\_R32G32\_FLOAT                                                                                                                                                                             |
| [**XMFLOAT2A**](/windows/desktop/api/DirectXMath/)   | D3DDECLTYPE\_FLOAT2                                                                   | D3DFMT\_G32R32F                                               | DXGI\_FORMAT\_R32G32\_FLOAT                                                                                                                                                                             |
| [**XMFLOAT3**](/windows/desktop/api/DirectXMath/)     | D3DDECLTYPE\_FLOAT3                                                                   |                                                               | DXGI\_FORMAT\_R32G32B32\_FLOAT                                                                                                                                                                          |
| [**XMFLOAT3A**](/windows/desktop/api/DirectXMath/)   | D3DDECLTYPE\_FLOAT3                                                                   |                                                               | DXGI\_FORMAT\_R32G32B32\_FLOAT                                                                                                                                                                          |
| [**XMFLOAT3PK**](/windows/desktop/api/DirectXPackedVector/ns-directxpackedvector-xmfloat3pk) |                                                                                       |                                                               | DXGI\_FORMAT\_R11G11B10\_FLOAT                                                                                                                                                                          |
| [**XMFLOAT3SE**](/windows/desktop/api/DirectXPackedVector/ns-directxpackedvector-xmfloat3se) |                                                                                       |                                                               | DXGI\_FORMAT\_R9G9B9E5\_SHAREDEXP                                                                                                                                                                       |
| [**XMFLOAT4**](/windows/desktop/api/DirectXMath/)     | D3DDECLTYPE\_FLOAT4                                                                   | D3DFMT\_A32B32G32R32F                                         | DXGI\_FORMAT\_R32G32B32A32\_FLOAT                                                                                                                                                                       |
| [**XMFLOAT4A**](/windows/desktop/api/DirectXMath/)   | D3DDECLTYPE\_FLOAT4                                                                   | D3DFMT\_A32B32G32R32F                                         | DXGI\_FORMAT\_R32G32B32A32\_FLOAT                                                                                                                                                                       |
| [**XMHALF2**](/windows/desktop/api/DirectXPackedVector/ns-directxpackedvector-xmhalf2)       | D3DDECLTYPE\_FLOAT16\_2                                                               | D3DFMT\_G16R16F                                               | DXGI\_FORMAT\_R16G16\_FLOAT                                                                                                                                                                             |
| [**XMHALF4**](/windows/desktop/api/DirectXPackedVector/ns-directxpackedvector-xmhalf4)       | D3DDECLTYPE\_FLOAT16\_4                                                               | D3DFMT\_A16B16G16R16F                                         | DXGI\_FORMAT\_R16G16B16A16\_FLOAT                                                                                                                                                                       |
| [**XMINT2**](/windows/desktop/api/DirectXMath/)         |                                                                                       |                                                               | DXGI\_FORMAT\_R32G32\_SINT                                                                                                                                                                              |
| [**XMINT3**](/windows/desktop/api/DirectXMath/)         |                                                                                       |                                                               | DXGI\_FORMAT\_R32G32B32\_SINT                                                                                                                                                                           |
| [**XMINT4**](/windows/desktop/api/DirectXMath/)         |                                                                                       |                                                               | DXGI\_FORMAT\_R32G32B32A32\_SINT                                                                                                                                                                        |
| [**XMSHORT2**](/windows/desktop/api/DirectXPackedVector/ns-directxpackedvector-xmshort2)     | D3DDECLTYPE\_SHORT2                                                                   | D3DFMT\_V16U16                                                | DXGI\_FORMAT\_R16G16\_SINT                                                                                                                                                                              |
| [**XMSHORTN2**](/windows/desktop/api/DirectXPackedVector/ns-directxpackedvector-xmshortn2)   | D3DDECLTYPE\_SHORT2N                                                                  | D3DFMT\_V16U16                                                | DXGI\_FORMAT\_R16G16\_SNORM                                                                                                                                                                             |
| [**XMSHORT4**](/windows/desktop/api/DirectXPackedVector/ns-directxpackedvector-xmshort4)     | D3DDECLTYPE\_SHORT4                                                                   | D3DFMT\_x16x16x16x16                                          | DXGI\_FORMAT\_R16G16B16A16\_SINT                                                                                                                                                                        |
| [**XMSHORTN4**](/windows/desktop/api/DirectXPackedVector/ns-directxpackedvector-xmshortn4)   | D3DDECLTYPE\_SHORT4N                                                                  | D3DFMT\_x16x16x16x16                                          | DXGI\_FORMAT\_R16G16B16A16\_SNORM                                                                                                                                                                       |
| [**XMUBYTE2**](/windows/desktop/api/DirectXPackedVector/ns-directxpackedvector-xmubyte2)     |                                                                                       |                                                               | DXGI\_FORMAT\_R8G8\_UINT                                                                                                                                                                                |
| [**XMUBYTEN2**](/windows/desktop/api/DirectXPackedVector/ns-directxpackedvector-xmubyten2)   |                                                                                       | D3DFMT\_A8P8, D3DFMT\_A8L8                                    | DXGI\_FORMAT\_R8G8\_UNORM                                                                                                                                                                               |
| [**XMUINT2**](/windows/desktop/api/DirectXMath/)       |                                                                                       |                                                               | DXGI\_FORMAT\_R32G32\_UINT                                                                                                                                                                              |
| [**XMUINT3**](/windows/desktop/api/DirectXMath/)       |                                                                                       |                                                               | DXGI\_FORMAT\_R32G32B32\_UINT                                                                                                                                                                           |
| [**XMUINT4**](/windows/desktop/api/DirectXMath/)       |                                                                                       |                                                               | DXGI\_FORMAT\_R32G32B32A32\_UINT                                                                                                                                                                        |
| [**XMU555**](/windows/desktop/api/DirectXPackedVector/ns-directxpackedvector-xmu555)         |                                                                                       | D3DFMT\_X1R5G5B5, D3DFMT\_A1R5G5B5                            | DXGI\_FORMAT\_B5G5R5A1\_UNORM                                                                                                                                                                           |
| [**XMU565**](/windows/desktop/api/DirectXPackedVector/ns-directxpackedvector-xmu565)         |                                                                                       | D3DFMT\_R5G6B5                                                | DXGI\_FORMAT\_B5G6R5\_UNORM                                                                                                                                                                             |
| [**XMUBYTE4**](/windows/desktop/api/DirectXPackedVector/ns-directxpackedvector-xmubyte4)     | D3DDECLTYPE\_UBYTE4                                                                   | D3DFMT\_x8x8x8x8                                              | DXGI\_FORMAT\_x8x8x8x8\_UINT                                                                                                                                                                            |
| [**XMUBYTEN4**](/windows/desktop/api/DirectXPackedVector/ns-directxpackedvector-xmubyten4)   | D3DDECLTYPE\_UBYTE4N                                                                  | D3DFMT\_x8x8x8x8                                              | DXGI\_FORMAT\_x8x8x8x8\_UNORM<br/> DXGI\_FORMAT\_R10G10B10\_XR\_BIAS\_A2\_UNORM (Use [**XMLoadUDecN4\_XR**](/windows/desktop/api/directxpackedvector.inl/nf-directxpackedvector-xmloadudecn4_xr) and [**XMStoreUDecN4\_XR**](/windows/desktop/api/directxpackedvector.inl/nf-directxpackedvector-xmstoreudecn4_xr).)<br/> |
| [**XMUDEC4**](/windows/desktop/api/DirectXPackedVector/ns-directxpackedvector-xmudec4)       | D3DDECLTYPE\_UDEC4 (Xbox Only)<br/> D3DDECLTYPE\_UDEC3 (Xbox Only)<br/>   | D3DFMT\_A2R10G10B10<br/> D3DFMT\_A2B10G10R10<br/> | DXGI\_FORMAT\_R10G10B10A2\_UINT                                                                                                                                                                         |
| [**XMUDECN4**](/windows/desktop/api/DirectXPackedVector/ns-directxpackedvector-xmudecn4)     | D3DDECLTYPE\_UDEC4N (Xbox Only)<br/> D3DDECLTYPE\_UDEC3N (Xbox Only)<br/> | D3DFMT\_A2R10G10B10<br/> D3DFMT\_A2B10G10R10<br/> | DXGI\_FORMAT\_R10G10B10A2\_UNORM                                                                                                                                                                        |
| [**XMUNIBBLE4**](/windows/desktop/api/DirectXPackedVector/ns-directxpackedvector-xmunibble4) |                                                                                       | D3DFMT\_A4R4G4B4, D3DFMT\_X4R4G4B4                            | DXGI\_FORMAT\_B4G4R4A4\_UNORM (DXGI 1.2+)                                                                                                                                                               |
| [**XMUSHORT2**](/windows/desktop/api/DirectXPackedVector/ns-directxpackedvector-xmushort2)   | D3DDECLTYPE\_USHORT2                                                                  | D3DFMT\_G16R16                                                | DXGI\_FORMAT\_R16G16\_UINT                                                                                                                                                                              |
| [**XMUSHORTN2**](/windows/desktop/api/DirectXPackedVector/ns-directxpackedvector-xmushortn2) | D3DDECLTYPE\_USHORT2N                                                                 | D3DFMT\_G16R16                                                | DXGI\_FORMAT\_R16G16\_UNORM                                                                                                                                                                             |
| [**XMUSHORT4**](/windows/desktop/api/DirectXPackedVector/ns-directxpackedvector-xmushort4)   | D3DDECLTYPE\_USHORT4 (Xbox Only)                                                      | D3DFMT\_x16x16x16x16                                          | DXGI\_FORMAT\_R16G16B16A16\_UINT                                                                                                                                                                        |
| [**XMUSHORTN4**](/windows/desktop/api/DirectXPackedVector/ns-directxpackedvector-xmushortn4) | D3DDECLTYPE\_USHORT4N                                                                 | D3DFMT\_x16x16x16x16                                          | DXGI\_FORMAT\_R16G16B16A16\_UNORM                                                                                                                                                                       |



 

## Global Constants in the DirectXMath Library

To reduce the size of the data segment, the DirectXMath Library uses the [XMGLOBALCONST](xmglobalconst.md) macro to make use of a number of global internal constants in its implementation. By convention, such internal global constants are prefixed by **g\_XM**. Typically, they are one of the following types: [**XMVECTORU32**](xmvectoru32-data-type.md), [**XMVECTORF32**](xmvectorf32-data-type.md), or **XMVECTORI32**.

These internal global constants are subject to change in future revisions of the DirectXMath Library. Use public functions that encapsulate the constants when possible rather than direct use of **g\_XM** global values. You can also declare your own global constants using [XMGLOBALCONST](xmglobalconst.md).

## Windows SSE versus SSE2

The SSE instruction set provides support only for single-precision floating-point vectors. DirectXMath must make use of the SSE2 instruction set to provide integer vector support. SSE2 is supported by all Intel processors since the introduction of the Pentium 4, all AMD K8 and later processors, and all x64-capable processors.

> [!Note]  
> Windows 8 for x86 requires support for SSE2. All versions of Windows x64 require support for SSE2. Windows RT (also known as Windows on ARM) requires ARM\_NEON.

 

## Routine Variants

There are several variants of DirectXMath functions that make it easier to do your work:

-   Comparison functions to create complicated conditional branching based on a smaller number of vector comparison operations. The name of these functions end in "R" such as XMVector3InBoundsR. The functions return a comparison record as a UINT return value, or as a UINT out parameter. You can use the **XMComparision\*** macros to test the value.
-   Batch functions for performing batch-style operations on larger vector arrays. The name of these functions end in "Stream" such as [**XMVector3TransformStream**](https://www.bing.com/search?q=**XMVector3TransformStream**). The functions operate on an array of inputs, and they generate an array of outputs. Typically, they take an input and output stride.
-   Estimation functions that implement a faster estimation instead of a slower, more accurate result. The name of these functions end in "Est" such as [**XMVector3NormalizeEst**](https://www.bing.com/search?q=**XMVector3NormalizeEst**). The quality and performance impact of using estimation varies from platform to platform, but we recommend that you use estimation variants for performance-sensitive code.

## Platform Inconsistencies

The DirectXMath library is intended for use in performance-sensitive graphics applications and games. Therefore, the implementation is designed for optimal speed doing normal processing on all supported platforms. Results at boundary-conditions, particularly those that generate floating-point specials, are likely to vary from target to target. This behavior will also depend on other run-time settings, such as the x87 control word for the Windows 32-bit no-intrinsics target or the SSE control word for both Windows 32-bit and 64-bit. Furthermore, there will be differences in boundary-conditions between various CPU vendors.

Don't use DirectXMath in scientific or other applications where numerical accuracy is paramount. Also, this limitation is reflected in the lack of support for double or other extended precision computations.

> [!Note]  
> The \_XM\_NO\_INTRINSICS\_ scalar code paths generally are written for compliance, not performance. Their boundary-condition results also will vary.

 

## Platform-specific Extensions

The DirectXMath library is intended to simplify C++ SIMD programming providing excellent support for x86, x64, and Windows RT platforms using broadly supported intrinsics instructions (SSE2 and ARM-NEON).

There are times, however, when platform-specific instructions may prove beneficial. Due to the way DirectXMath is implemented, in many cases it is trivial to use DirectXMath types directly in standard compiler-supported intrinsics statements, and to use DirectXMath as the fallback path for platforms that don't support the extended instruction.

For example, here is a simplified example of leveraging the SSE 4.1 dot-product instruction. Note that you must explicitly guard the code-path to avoid generating invalid instruction exceptions at run time. Ensure the code paths do significant enough work to justify the additional cost of branching, complexity of maintaining multiple code-paths, and so on.


```
#include <windows.h>
#include <stdio.h>

#include <DirectXMath.h>

#include <intrin.h>
#include <smmintrin.h>

using namespace DirectX;

bool g_bSSE41 = false;

void DetectCPUFeatures()
{
#ifndef _M_ARM
   // See __cpuid documentation on MSDN for more information

   int CPUInfo[4] = {-1};
   __cpuid( CPUInfo, 0 );

   if ( CPUInfo[0] >= 1 )
   {
       __cpuid(CPUInfo, 1 );
 
       if ( CPUInfo[2] & 0x80000 )
           g_bSSE41 = true;
   }
#endif
}

int main()
{
   if ( !XMVerifyCPUSupport() )
       return -1;

   DetectCPUFeatures();

   ...

   XMVECTORF32 v1 = { 1.f, 2.f, 3.f, 4.f };
   XMVECTORF32 v2 = { 5.f, 6.f, 7.f, 8.f };

   XMVECTOR r2, r3, r4;

   if ( g_bSSE41 )
   {
#ifndef _M_ARM
       r2 = _mm_dp_ps( v1, v2, 0x3f );
       r3 = _mm_dp_ps( v1, v2, 0x7f );
       r4 = _mm_dp_ps( v1, v2, 0xff );
#endif
   }
   else
   {
       r2 = XMVector2Dot( v1, v2 );
       r3 = XMVector3Dot( v1, v2 );
       r4 = XMVector4Dot( v1, v2 );
   }

   ... 

   return 0;
}
```



For more info about platform-specific extensions, see:

<dl>

[DirectXMath: SSE, SSE2, and ARM-NEON](http://blogs.msdn.com/b/chuckw/archive/2012/09/11/directxmath-sse-sse2-and-arm-neon.aspx)  
[DirectXMath: SSE3 and SSSE3](http://blogs.msdn.com/b/chuckw/archive/2012/09/11/directxmath-sse3-and-ssse3.aspx)  
[DirectXMath: SSE4.1 and SSE4.2](http://blogs.msdn.com/b/chuckw/archive/2012/09/11/directxmath-sse4-1-and-sse-4-2.aspx)  
[DirectXMath: AVX](http://blogs.msdn.com/b/chuckw/archive/2012/09/11/directxmath-avx.aspx)  
[DirectXMath: F16C and FMA](http://blogs.msdn.com/b/chuckw/archive/2012/09/11/directxmath-f16c-and-fma.aspx)  
</dl>

## Related topics

<dl> <dt>

[DirectXMath Programming Guide](ovw-xnamath-progguide.md)
</dt> </dl>

 

 



