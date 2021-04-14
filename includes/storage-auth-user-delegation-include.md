---
title: archivo de inclusión
description: archivo de inclusión
services: storage
author: tamram
ms.service: storage
ms.topic: include
ms.date: 01/15/2020
ms.author: tamram
ms.custom: include file
ms.openlocfilehash: a0a4a10797109cb3363027e2445259d06d9aa071
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "76118066"
---
## <a name="about-the-user-delegation-sas"></a>Acerca de la SAS de delegación de usuarios

Un token de SAS para el acceso a un contenedor o blob se puede proteger mediante credenciales de Azure AD credenciales o una clave de cuenta. Una SAS protegida con credenciales de Azure AD se denomina SAS de delegación de usuarios, porque el token de OAuth 2.0 usado para firmar la SAS se solicita en nombre del usuario.

Microsoft recomienda usar credenciales de Azure AD cuando sea posible como procedimiento recomendado de seguridad, en lugar de usar la clave de cuenta, que se puede poner en peligro más fácilmente. Cuando el diseño de la aplicación requiera firmas de acceso compartido, use credenciales de Azure AD para crear una SAS de delegación de usuarios de cara a una seguridad superior. Para más información sobre la SAS de delegación de usuarios, consulte [Creación de una SAS de delegación de usuario](/rest/api/storageservices/create-user-delegation-sas).

> [!CAUTION]
> Cualquier cliente que posea una SAS válida puede acceder a los datos de la cuenta de almacenamiento según lo permitido por esa SAS. Es importante proteger una SAS de uso malintencionado o no intencionado. Sea cauto al distribuir una SAS y tenga un plan para revocar una SAS en peligro.

Para obtener más información sobre las firmas de acceso compartido, consulte [Otorgar acceso limitado a recursos de Azure Storage con firmas de acceso compartido (SAS)](../articles/storage/common/storage-sas-overview.md).
