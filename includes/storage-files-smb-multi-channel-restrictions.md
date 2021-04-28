---
title: archivo de inclusión
description: archivo de inclusión
services: storage
author: roygara
ms.service: storage
ms.topic: include
ms.date: 04/26/2021
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 04ded4340eef0bfe5bc91ee2e3f2552975df2e05
ms.sourcegitcommit: 5f785599310d77a4edcf653d7d3d22466f7e05e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108065639"
---
SMB multicanal para los recursos compartido de archivos de Azure tiene actualmente las siguientes restricciones:
- Solo se admite para clientes de Windows. 
- El número máximo de caracteres es cuatro.
- No se admite SMB directo.
- Los puntos de conexión privados para las cuentas de almacenamiento no se admiten.
- En el caso de las cuentas de Storage con [autenticación basada en identidad](../articles/storage/files/storage-files-active-directory-overview.md) Active Directory Domain Services (AD DS) local o Azure AD DS habilitada para Azure Files, los clientes SMB no podrían usar el Explorador de archivos de Windows para configurar permisos NTFS en directorios y archivos.
    - Use la herramienta [icacls](/windows-server/administration/windows-commands/icacls) de Windows o el comando [Set-ACL](/powershell/module/microsoft.powershell.security/set-acl) para configurar los permisos.

