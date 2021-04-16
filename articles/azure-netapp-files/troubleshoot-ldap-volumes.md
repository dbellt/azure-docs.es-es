---
title: Solución de problemas de volúmenes LDAP para Azure NetApp Files | Microsoft Docs
description: Describe las soluciones a las condiciones de error que puede encontrarse al configurar volúmenes LDAP para Azure NetApp Files.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 04/05/2021
ms.author: b-juche
ms.openlocfilehash: eea3f691bc6d91948dc73b4a02c89abfac12d384
ms.sourcegitcommit: bfa7d6ac93afe5f039d68c0ac389f06257223b42
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106498919"
---
# <a name="troubleshoot-ldap-volume-issues"></a>Solución de problemas de volúmenes LDAP

En este artículo se describen las soluciones a las condiciones de error que puede encontrarse al configurar volúmenes LDAP.

## <a name="errors-and-resolutions-for-ldap-volumes"></a>Errores y soluciones para volúmenes LDAP

|     Condiciones del error    |     Soluciones    |
|-|-|
| Error al crear un volumen SMB con el parámetro ldapEnabled como true: <br> `Error Message: ldapEnabled option is only supported with NFS protocol volume. ` | No se puede crear un volumen SMB con la opción LDAP habilitada. <br> Cree volúmenes SMB con la opción LDAP deshabilitada. |
| Error al actualizar el valor del parámetro ldapEnabled para un volumen existente: <br> `Error Message: ldapEnabled parameter is not allowed to update` |  No se puede modificar la configuración de la opción LDAP después de crear un volumen. <br> No actualice la configuración de la opción LDAP en un volumen ya creado. Consulte [Configuración de ADDS LDAP con grupos extendidos para el acceso a volúmenes NFS](configure-ldap-extended-groups.md) para más información. |
| Error al crear un volumen NFS habilitado para LDAP: <br> `Could not query DNS server` <br> `Sample error message:` <br> `"log": time="2020-10-21 05:04:04.300" level=info msg=Res method=GET url=/v2/Volumes/070d0d72-d82c-c893-8ce3-17894e56cea3  x-correlation-id=9bb9e9fe-abb6-4eb5-a1e4-9e5fbb838813 x-request-id=c8032cb4-2453-05a9-6d61-31ca4a922d85 xresp="200:  {\"created\":\"2020-10-21T05:02:55.000Z\",\"lifeCycleState\":\"error\",\"lifeCycleStateDetails\":\"Error when creating - Could not query DNS server. Verify that the network configuration is correct and that DNS servers are available.\",\"name\":\"smb1\",\"ownerId\ \":\"8c925a51-b913-11e9-b0de-9af5941b8ed0\",\"region\":\"westus2stage\",\"volumeId\":\"070d0d72-d82c-c893-8ce3-` |  Este error se produce porque no se puede acceder al DNS. <br> <ul><li> Compruebe si ha configurado el sitio correcto (ámbito de sitio) para Azure NetApp Files. </li><li> La razón por la que el DNS es inaccesible puede ser una dirección IP de DNS incorrecta o problemas de red. Compruebe la dirección IP de DNS especificada en la conexión de AD para asegurarse de que es correcta. </li><li> Asegúrese de que AD y el volumen estén en la misma región y en la misma red virtual. Si están en redes virtuales diferentes, asegúrese de establecer un emparejamiento entre las dos redes virtuales.</li></ul> |
| Error al crear un volumen a partir de una instantánea: <br> `Aggregate does not exist` | Azure NetApp Files no admite el aprovisionamiento de un nuevo volumen habilitado para LDAP a partir de una instantánea que pertenece a un volumen que no está habilitado para LDAP. <br> Intente crear un nuevo volumen que no está habilitado para LDAP a partir de la instantánea especificada. |

## <a name="next-steps"></a>Pasos siguientes  

* [Configuración de ADDS LDAP con grupos extendidos para el acceso a volúmenes NFS](configure-ldap-extended-groups.md)
* [Creación de un volumen de NFS para Azure NetApp Files](azure-netapp-files-create-volumes.md)
* [Creación de un volumen de protocolo dual (NFSv3 y SMB) para Azure NetApp Files](create-volumes-dual-protocol.md)