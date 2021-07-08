---
title: 'Roles integrados de RBAC local de Managed HSM: Azure Key Vault | Microsoft Docs'
description: Información general de los roles integrados de HSM administrado que se pueden asignar a usuarios, entidades de servicio, grupos e identidades administradas.
services: key-vault
author: amitbapat
ms.service: key-vault
ms.subservice: managed-hsm
ms.topic: tutorial
ms.date: 06/01/2021
ms.author: ambapat
ms.openlocfilehash: a9594157349754c7178f3c73cfe9529ad478de4a
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111411024"
---
# <a name="managed-hsm-local-rbac-built-in-roles"></a>Roles integrados de RBAC local de Managed HSM

El RBAC local de HSM administrado tiene varios roles integrados. Puede asignar estos roles a usuarios, entidades de servicio, grupos e identidades administradas. Para permitir que una entidad de seguridad realice una operación, debe asignarle un rol que le conceda permiso para llevar a cabo esa operación. Todos estos roles y operaciones solo le permiten administrar el permiso de operaciones del plano de datos. Para administrar los permisos de plano de control del recurso de HSM administrado, debe usar el [control de acceso basado en rol (Azure RBAC)](../../role-based-access-control/overview.md). Algunos ejemplos de operaciones en el plano de control son la creación de un nuevo HSM administrado o su actualización, traslado o eliminación.

## <a name="built-in-roles"></a>Roles integrados

|Nombre de rol|Descripción|ID|
|---|---|---|
|Managed HSM Administrator| Concede permisos para realizar todas las operaciones relacionadas con el dominio de seguridad, la copia de seguridad completa y la restauración, y la administración de roles. No se permite realizar ninguna operación de administración de claves.|a290e904-7015-4bba-90c8-60543313cdb4|
|Managed HSM Crypto Officer|Concede permisos para realizar todas las tareas de administración de roles, purga o recuperación de claves eliminadas, y exportación de claves. No se permite realizar otras operaciones de administración de claves.|515eb02d-2335-4d2d-92f2-b1cbdf9c3778|
|Managed HSM Crypto User|Concede permisos para realizar todas las operaciones de administración de claves, excepto la purga o recuperación de claves eliminadas, y la exportación de claves.|21dbd100-6940-42c2-9190-5d6cb909625b|
|Managed HSM Policy Administrator| Concede permiso para crear y eliminar asignaciones de roles.|4bd23610-cdcf-4971-bdee-bdc562cc28e4|
|Managed HSM Crypto Auditor|Concede permiso de lectura para leer (pero no usar) los atributos clave.|2c18b078-7c48-4d3a-af88-5a3a1b3f82b3|
|Usuario de cifrado del servicio de criptografía de HSM administrado| Concede permiso para usar una clave para el cifrado del servicio. |33413926-3206-4cdd-b39a-83574fe37a17|
|Managed HSM Backup| Concede permiso para realizar una copia de seguridad de HSM completa o de clave única.|7b127d3c-77bd-4e3e-bbe0-dbb8971fa7f8|

## <a name="permitted-operations"></a>Operaciones permitidas
> [!NOTE]  
> - Una "X" indica que un rol puede realizar la acción de datos. Una celda vacía indica que el rol no tiene permiso para realizar esa acción de datos.
> - Todos los nombres de la acción de datos tienen el prefijo "Microsoft.KeyVault/managedHsm", que se omite en las tablas siguientes para abreviar.
> - Todos los nombres de roles tienen el prefijo "Managed HSM", que se omite en la tabla siguiente para abreviar.

|Acción de datos | Administrador | Responsable de cifrado | Usuario de cifrado | Administrador de directivas | Cifrado del servicio criptográfico | Backup | Auditor de cifrado|
|---|---|---|---|---|---|---|---|
|**Administración de dominios de seguridad**|
/securitydomain/download/action|<center>X</center>||||||
/securitydomain/upload/action|<center>X</center>||||||
/securitydomain/upload/read|<center>X</center>||||||
/securitydomain/transferkey/read|<center>X</center>||||||
|**Administración de claves**|
|/keys/read/action|||<center>X</center>||<center>X</center>||<center>X</center>|
|/keys/write/action|||<center>X</center>||||
|/keys/create|||<center>X</center>||||
|/keys/delete|||<center>X</center>||||
|/keys/deletedKeys/read/action||<center>X</center>|||||
|/keys/deletedKeys/recover/action||<center>X</center>|||||
|/keys/deletedKeys/delete||<center>X</center>|||||<center>X</center>|
|/keys/backup/action|||<center>X</center>|||<center>X</center>|
|/keys/restore/action|||<center>X</center>||||
|/keys/export/action||<center>X</center>|||||
|/keys/release/action|||<center>X</center>||||
|/keys/import/action|||<center>X</center>||||
|**Operaciones de cifrado de claves**:|
|/keys/encrypt/action|||<center>X</center>||||
|/keys/decrypt/action|||<center>X</center>||||
|/keys/wrap/action|||<center>X</center>||<center>X</center>||
|/keys/unwrap/action|||<center>X</center>||<center>X</center>||
|/keys/sign/action|||<center>X</center>||||
|/keys/verify/action|||<center>X</center>||||
|**Administración de roles**|
|/roleAssignments/read/action|<center>X</center>|<center>X</center>|<center>X</center>|<center>X</center>|||<center>X</center>
|/roleAssignments/write/action|<center>X</center>|<center>X</center>||<center>X</center>|||
|/roleAssignments/delete/action|<center>X</center>|<center>X</center>||<center>X</center>|||
|/roleDefinitions/read/action|<center>X</center>|<center>X</center>|<center>X</center>|<center>X</center>|||<center>X</center>
|/roleDefinitions/write/action|<center>X</center>|<center>X</center>||<center>X</center>|||
|/roleDefinitions/delete/action|<center>X</center>|<center>X</center>||<center>X</center>|||
|**Administración de copia de seguridad y restauración**|
|/backup/start/action|<center>X</center>|||||<center>X</center>|
|/backup/status/action|<center>X</center>|||||<center>X</center>|
|/restore/start/action|<center>X</center>||||||
|/restore/status/action|<center>X</center>||||||
||||||||

## <a name="next-steps"></a>Pasos siguientes

- Consulte la información general sobre el [control de acceso basado en rol de Azure (Azure RBAC)](../../role-based-access-control/overview.md).
- Consulte un tutorial sobre la [Administración de roles de HSM administrado](role-management.md).
