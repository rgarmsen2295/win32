---
Description: Returns an OIDs collection that represents the certificate policies used to create the Chain object.
ms.assetid: 7fe7d3ea-28fc-4c0a-9b43-a97518ac65db
title: CertificateStatus.CertificatePolicies method
ms.technology: desktop
ms.prod: windows
ms.author: windowssdkdev
ms.topic: article
ms.date: 05/31/2018
---

# CertificateStatus.CertificatePolicies method

\[CAPICOM is a 32-bit only component that is available for use in the following operating systems: Windows Server 2008, Windows Vista, and Windows XP. Instead, use the [**X509ChainStatus Structure**](https://www.bing.com/search?q=**X509ChainStatus+Structure**) in the [**System.Security.Cryptography.X509Certificates**](https://www.bing.com/search?q=**System.Security.Cryptography.X509Certificates**) namespace.\]

The **CertificatePolicies** method returns an [**OIDs**](oids.md) collection that represents the certificate policies used to create the [**Chain**](chain.md) object.

## Syntax


```VB
CertificateStatus.CertificatePolicies()
```



## Parameters

This method has no parameters.

## Return value

An [**OIDs**](oids.md) collection. Each [**OID**](oid.md) object in the collection represents a certificate policy OID.

## Remarks

Add certificate policy OIDs to the collection to specify the certificate policies that should be used to build the certificate trust chain.

## Requirements



|                                  |                                                                                        |
|----------------------------------|----------------------------------------------------------------------------------------|
| End of client support<br/> | Windows Vista<br/>                                                               |
| End of server support<br/> | Windows Server 2008<br/>                                                         |
| Redistributable<br/>       | CAPICOM 2.0 or later on Windows Server 2003 and Windows XP<br/>                  |
| DLL<br/>                   | <dl> <dt>Capicom.dll</dt> </dl> |



 

 



