---
title: Migración al control de acceso basado en rol de Azure | Microsoft Docs
description: Obtenga información sobre cómo migrar desde directivas de acceso de almacén a roles de Azure.
services: key-vault
author: msmbaldwin
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 8/30/2020
ms.author: mbaldwin
ms.openlocfilehash: a369ed26ca91dbf951b28b99250c6307608c5eb3
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107749085"
---
# <a name="migrate-from-vault-access-policy-to-an-azure-role-based-access-control-permission-model"></a>Migración desde la directiva de acceso de almacén a un modelo de permisos de control de acceso basado en rol de Azure

El modelo de directiva de acceso de almacén es un sistema de autorización existente integrado en Key Vault que sirve para proporcionar acceso a claves, secretos y certificados. Puede controlar el acceso mediante la asignación de permisos individuales a la entidad de seguridad (usuario, grupo, entidad de servicio e identidad administrada) en el ámbito de Key Vault. 

El control de acceso basado en rol de Azure es un sistema de autorización basado en [Azure Resource Manager](../../azure-resource-manager/management/overview.md) que proporciona administración de acceso específico a los recursos de Azure. Con Azure RBAC, puede controlar el acceso a los recursos mediante la creación de asignaciones de roles, que consta de tres elementos: entidad de seguridad, definición de roles (conjunto predefinido de permisos) y ámbito (grupo de recursos o recurso individual). Para más información, consulte [Control de acceso basado en rol de Azure (Azure RBAC)](../../role-based-access-control/overview.md).

Antes de migrar a Azure RBAC, es importante comprender sus ventajas y limitaciones.

Ventajas clave de Azure RBAC en comparación con las directivas de acceso de almacén:
- Proporciona un modelo de control de acceso unificado para los recursos de Azure: la misma API en los servicios de Azure.
- Administración de acceso centralizada para administradores: administrar todos los recursos de Azure en una sola vista.
- Integración con [Privileged Identity Management](../../active-directory/privileged-identity-management/pim-configure.md) para el control de acceso basado en el tiempo.
- Asignaciones de denegación: capacidad de excluir la entidad de seguridad en un ámbito concreto. Para información, vea [Descripción de las asignaciones de denegación de Azure](../../role-based-access-control/deny-assignments.md).

Desventajas de Azure RBAC:
- Latencia de las asignaciones de roles: la asignación de roles puede tardar varios minutos en aplicarse. Las directivas de acceso de almacén se asignan al instante.
- Número limitado de asignaciones de roles: 2000 asignaciones de roles por suscripción frente a 1024 directivas de acceso por instancia de Key Vault.

## <a name="access-policies-to-azure-roles-mapping"></a>Directivas de acceso para la asignación de roles de Azure

Azure RBAC tiene varios roles integrados de Azure que se pueden asignar a usuarios, grupos, entidades de servicio e identidades administradas. Si los roles integrados no satisfacen las necesidades específicas de la organización, puede crear [roles personalizados de Azure](../../role-based-access-control/custom-roles.md) propios.

Roles integrados de Key Vault para la administración de acceso a secretos, certificados y claves:
- Administrador de Key Vault
- Lector de Key Vault
- Responsable de certificados de Key Vault
- Agente criptográfico de Key Vault
- Usuario criptográfico de Key Vault
- Usuario de cifrado de servicio criptográfico de Key Vault
- Responsable de secretos de Key Vault
- Usuario de secretos de Key Vault

Para más información sobre los roles integrados existentes, vea [Roles integrados de Azure](../../role-based-access-control/built-in-roles.md).

Se pueden asignar directivas de acceso de almacén con permisos seleccionados individualmente o con plantillas de permisos predefinidas.

Plantillas de permisos predefinidas de directivas de acceso:
- Administración de claves, secretos y certificados
- Administración de claves y secretos
- Administración de secretos y certificados
- Administración de claves
- Administración de secretos
- Administración de certificados
- Conector de SQL Server
- Azure Data Lake Storage o Azure Storage
- Azure Backup
- Clave de cliente de Exchange Online
- Clave de cliente de SharePoint Online
- BYOK para información de Azure

### <a name="access-policies-templates-to-azure-roles-mapping"></a>Plantillas de directivas de acceso para la asignación de roles de Azure
| Plantilla de directiva de acceso | Operations | Rol de Azure |
| --- | --- | --- |
| Administración de claves, secretos y certificados | Claves: todas las operaciones <br>Certificados: todas las operaciones<br>Secretos: todas las operaciones | Administrador de Key Vault |
| Administración de claves y secretos | Claves: todas las operaciones <br>Secretos: todas las operaciones| Agente criptográfico de Key Vault <br> Responsable de secretos de Key Vault |
| Administración de secretos y certificados | Certificados: todas las operaciones <br>Secretos: todas las operaciones| Agente de certificados de Key Vault <br> Responsable de secretos de Key Vault|
| Administración de claves | Claves: todas las operaciones| Agente criptográfico de Key Vault|
| Administración de secretos | Secretos: todas las operaciones| Responsable de secretos de Key Vault|
| Administración de certificados | Certificados: todas las operaciones | Agente de certificados de Key Vault|
| Conector de SQL Server | Claves: obtener, enumerar, encapsular clave y desencapsular clave | Usuario de cifrado de servicio criptográfico de Key Vault|
| Azure Data Lake Storage o Azure Storage | Claves: obtener, enumerar y desencapsular clave | N/D<br> Rol personalizado obligatorio|
| Azure Backup | Claves: obtener, enumerar y copia de seguridad<br> Certificado: obtener, enumerar y copia de seguridad | N/D<br> Rol personalizado obligatorio|
| Clave de cliente de Exchange Online | Claves: obtener, enumerar, encapsular clave y desencapsular clave | Usuario de cifrado de servicio criptográfico de Key Vault|
| Clave de cliente de Exchange Online | Claves: obtener, enumerar, encapsular clave y desencapsular clave | Usuario de cifrado de servicio criptográfico de Key Vault|
| BYOK para información de Azure | Claves: obtener, descifrar y firmar | N/D<br>Rol personalizado obligatorio|


## <a name="assignment-scopes-mapping"></a>Asignación de ámbitos de asignación  

Azure RBAC para Key Vault permite la asignación de roles en los siguientes ámbitos:
- Grupo de administración
- Subscription
- Resource group
- Recurso de Key Vault
- Clave, secreto y certificado individuales

El modelo de permisos de la directiva de acceso de almacén está limitado a asignar la directiva solo al nivel de recurso de Key Vault, que: 

En general, se recomienda tener un almacén de claves por aplicación y administrar el acceso al nivel de almacén de claves. Hay escenarios en los que la administración del acceso en otros ámbitos puede simplificar la administración del acceso.

- **Infraestructura, operadores y administradores de seguridad: la administración de grupos de almacenes de claves al nivel de grupo de administración, suscripción o grupo de recursos con directivas de acceso de almacén requiere mantener directivas para cada almacén de claves. Azure RBAC permite crear una asignación de roles en el grupo de administración, la suscripción o el grupo de recursos. Esa asignación se aplicará a todos los almacenes de claves creados en el mismo ámbito. En este escenario, se recomienda usar Privileged Identity Management con acceso Just-in-Time a fin de proporcionar acceso permanente.
 
- **Aplicaciones: hay escenarios en los que la aplicación necesitaría compartir el secreto con otra aplicación. Mediante el uso de directivas de acceso de almacén, es necesario crear un almacén de claves independiente para evitar que se conceda acceso a todos los secretos. Azure RBAC permite asignar, en su lugar, un rol con el ámbito de un secreto individual mediante un almacén de claves único.

## <a name="vault-access-policy-to-azure-rbac-migration-steps"></a>Pasos para migrar directivas de acceso de almacén a Azure RBAC
Existen muchas diferencias entre Azure RBAC y el modelo de permisos de la directiva de acceso de almacén. Para evitar interrupciones durante la migración, se recomiendan los pasos siguientes.
 
1. **Identificar y asignar roles**: identifique roles integrados basados en la tabla de asignación anterior y cree roles personalizados cuando sea necesario. Asigne roles en ámbitos, en función de la guía de asignación de ámbitos. Para más información sobre cómo asignar roles al almacén de claves, consulte [Proporcionar acceso a Key Vault con un control de acceso basado en rol de Azure](rbac-guide.md).
1. **Validar la asignación de roles**: las asignaciones de roles en Azure RBAC pueden tardar varios minutos en propagarse. Para obtener instrucciones sobre cómo comprobar las asignaciones de roles, vea [Lista de las asignaciones de rol en un ámbito](../../role-based-access-control/role-assignments-list-portal.md#list-role-assignments-for-a-user-at-a-scope).
1. **Configurar la supervisión y las alertas en el almacén de claves**: es importante habilitar las alertas de registro y configuración de las excepciones de acceso denegado. Para más información, vea [Supervisión y alertas de Azure Key Vault](./alert.md).
1. **Establecer el modelo de permisos de control de acceso basado en rol de Azure en Key Vault**: al habilitar el modelo de permisos de Azure RBAC, se invalidarán todas las directivas de acceso existentes. Si se produce un error, se puede volver a cambiar el modelo de permisos con todas las directivas de acceso existentes que queden intactas.

> [!NOTE]
> El cambio del modelo de permiso requiere el permiso "Microsoft.Authorization/roleAssignments/write", que forma parte de los roles [Propietario](../../role-based-access-control/built-in-roles.md#owner) y [Administrador de acceso de usuario](../../role-based-access-control/built-in-roles.md#user-access-administrator). No se admiten roles de administrador de suscripciones clásicas como "Administrador de servicios" y "Coadministrador".

> [!NOTE]
> Cuando el modelo de permisos de Azure RBAC está habilitado, se producirá un error en todos los scripts que intenten actualizar las directivas de acceso. Es importante actualizar esos scripts para que usen Azure RBAC.

## <a name="troubleshooting"></a>Solución de problemas
-  La asignación de roles no funciona después de varios minutos: hay situaciones en las que las asignaciones de roles pueden tardar más tiempo. Es importante escribir lógica de reintento en el código para cubrir esos casos.
- Las asignaciones de roles desaparecieron cuando se eliminó Key Vault (eliminación temporal) y se recuperaron; actualmente, se trata de una limitación de la característica de eliminación temporal en todos los servicios de Azure. Es necesario volver a crear todas las asignaciones de roles después de la recuperación.    

## <a name="learn-more"></a>Más información

- [Información general de Azure RBAC](../../role-based-access-control/overview.md)
- [Tutorial de roles personalizados](../../role-based-access-control/tutorial-custom-role-cli.md)
- [Privileged Identity Management](../../active-directory/privileged-identity-management/pim-configure.md)