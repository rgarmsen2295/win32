---
Description: The CIM\_ProductFRU class represents an association between the product and a field replaceable unit (FRU), which provides information about product components that have been, or are being replaced.
audience: developer
author: REDMOND\\markl
manager: REDMOND\\markl
ms.assetid: 15d682e5-cd90-4fc4-8fff-e3fe1d2a0ac4
ms.prod: windows-server-dev
ms.technology:
- cimwin32
- windows-management-instrumentation
ms.tgt_platform: multiple
title: CIM\_ProductFRU class
ms.author: windowssdkdev
ms.topic: article
ms.date: 05/31/2018
---

# CIM\_ProductFRU class

The **CIM\_ProductFRU** class represents an association between the product and a field replaceable unit (FRU), which provides information about product components that have been, or are being replaced.

> \[!Important\]  
> The DMTF (Distributed Management Task Force) CIM (Common Information Model) classes are the parent classes upon which WMI classes are built. WMI currently supports only the [CIM 2.x version schemas](Http://Go.Microsoft.Com/FWLink/p/?LinkID=309367).

 

The following syntax is simplified from Managed Object Format (MOF) code and includes all of its inherited properties. Properties are listed in alphabetic order, not MOF order.

## Syntax

``` syntax
[Abstract, UUID("{EB98A1B2-DB36-11d2-85FC-0000F8102E5F}"), Association, AMENDMENT]
class CIM_ProductFRU
{
  CIM_FRU     REF FRU;
  CIM_Product REF Product;
};
```

## Members

The **CIM\_ProductFRU** class has these types of members:

-   [Properties](#properties)

### Properties

The **CIM\_ProductFRU** class has these properties.

<dl> <dt>

**FRU**
</dt> <dd> <dl> <dt>

Data type: **CIM\_FRU**
</dt> <dt>

Access type: Read-only
</dt> </dl>

Reference to the FRU.

</dd> <dt>

**Product**
</dt> <dd> <dl> <dt>

Data type: **CIM\_Product**
</dt> <dt>

Access type: Read-only
</dt> <dt>

Qualifiers: [**Max**](https://msdn.microsoft.com/library/aa393650) (1)
</dt> </dl>

Reference to the product to which the FRU is applied.

</dd> </dl>

## Remarks

WMI does not implement this class.

This documentation is derived from the CIM class descriptions published by the DMTF. Microsoft may have made changes to correct minor errors, conform to Microsoft SDK documentation standards, or provide more information.

## Requirements



|                                     |                                                                                         |
|-------------------------------------|-----------------------------------------------------------------------------------------|
| Minimum supported client<br/> | Windows Vista<br/>                                                                |
| Minimum supported server<br/> | Windows Server 2008<br/>                                                          |
| Namespace<br/>                | Root\\CIMV2<br/>                                                                  |
| MOF<br/>                      | <dl> <dt>CIMWin32.mof</dt> </dl> |
| DLL<br/>                      | <dl> <dt>CIMWin32.dll</dt> </dl> |



 

 



