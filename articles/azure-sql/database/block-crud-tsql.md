---
title: Bloqueo de comandos T-SQL para crear o modificar recursos de Azure SQL
titleSuffix: Block T-SQL commands to create or modify Azure SQL resources
description: En este artículo se detalla una característica que permite a los administradores de Azure bloquear comandos T-SQL para crear o modificar recursos de Azure SQL.
ms.service: sql-database
ms.subservice: security
ms.custom: ''
author: WilliamDAssafMSFT
ms.author: wiassaf
ms.topic: article
ms.date: 03/31/2021
ms.reviewer: ''
ROBOTS: NOINDEX
ms.openlocfilehash: 4ecaa8a3ee1d11ea13563ae5c74835b8d62fd960
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106555943"
---
# <a name="what-is-block-t-sql-crud-feature"></a>¿Qué es la característica de bloqueo de T-SQL CRUD?
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb-sqlmi.md)]


Esta característica permite a los administradores de Azure bloquear la creación o modificación de recursos de Azure SQL mediante T-SQL. Esto se aplica en el nivel de suscripción para bloquear los comandos T-SQL y que no afecten a los recursos de SQL en cualquier instancia administrada o de base de datos de Azure SQL.

## <a name="overview"></a>Información general

Para bloquear la creación o modificación de recursos con T-SQL y aplicar la administración de recursos mediante una plantilla de Azure Resource Manager (plantilla de ARM) para una suscripción determinada, se pueden utilizar las características en versión preliminar del nivel de suscripción de Azure Portal. Esto es especialmente útil cuando se usan [directivas de Azure](/azure/governance/policy/overview) para aplicar los estándares de la organización mediante plantillas de ARM. Como T-SQL no sigue las directivas de Azure, se puede aplicar un bloqueo en las operaciones de creación o modificación de T-SQL. La sintaxis bloqueada incluye instrucciones CRUD (creación, actualización, eliminación) para bases de datos en Azure SQL, específicamente `CREATE DATABASE`, `ALTER DATABASE` e instrucciones `DROP DATABASE`. 

Las operaciones de T-SQL CRUD se pueden bloquear mediante Azure Portal, [PowerShell](/powershell/module/az.resources/register-azproviderfeature) o la [CLI de Azure](/cli/azure/feature#az_feature_register).

## <a name="permissions"></a>Permisos

Para registrar o quitar esta característica, el usuario de Azure debe ser miembro del rol de propietario o colaborador de la suscripción.

## <a name="examples"></a>Ejemplos

En la sección siguiente se describe cómo se puede registrar o anular el registro de una característica en versión preliminar con el proveedor de recursos Microsoft.Sql en Azure Portal: 

### <a name="register-block-t-sql-crud"></a>Registro del bloqueo de T-SQL CRUD

1. Vaya a su suscripción en Azure Portal.
2. Seleccione la pestaña **Características en versión preliminar**. 
3. Seleccione **Block T-SQL CRUD** (Bloqueo de T-SQL CRUD).
4. Después de seleccionar **Block T-SQL CRUD** (Bloqueo de T-SQL CRUD), se abrirá una nueva ventana y se seleccionará **Registrar** para registrar este bloqueo con el proveedor de recursos Microsoft.Sql.

![Seleccione "Block T-SQL CRUD" (Bloqueo de T-SQL CRUD) en la lista de características en versión preliminar.](./media/block-tsql-crud/block-tsql-crud.png)

![Con la opción "Block T-SQL CRUD" (Bloqueo de T-SQL CRUD) seleccionada, seleccione Registrar.](./media/block-tsql-crud/block-tsql-crud-register.png)

  
### <a name="re-register-microsoftsql-resource-provider"></a>Nuevo registro de un proveedor de recursos de Microsoft.Sql 
Después de registrar el bloqueo de T-SQL CRUD con el proveedor de recursos Microsoft.Sql, debe volver a registrar el proveedor de recursos Microsoft.Sql para que los cambios surtan efecto. Para volver a registrar el proveedor de recursos Microsoft.Sql:

1. Vaya a su suscripción en Azure Portal.
2. Seleccione la pestaña **Proveedores de recursos**.
3. Busque y seleccione el proveedor de recursos **Microsoft.Sql**.
4. Seleccione **Volver a registrar**. 

> [!NOTE]
> El paso de volver a registrar es obligatorio para que el bloqueo de T-SQL se aplique a su suscripción. 

![Nuevo registro del proveedor de recursos Microsoft.Sql](./media/block-tsql-crud/block-tsql-crud-re-register.png)

### <a name="removing-block-t-sql-crud"></a>Eliminación del bloqueo de T-SQL CRUD
Para quitar el bloqueo en las operaciones de creación o modificación de T-SQL de su suscripción, anule primero el registro del bloqueo de T-SQL registrado anteriormente. A continuación, vuelva a registrar el proveedor de recursos Microsoft.Sql tal y como se ha mostrado antes para que la eliminación del bloqueo de T-SQL surta efecto. 


## <a name="next-steps"></a>Pasos siguientes

- [Información general sobre las funcionalidades de seguridad de Azure SQL Database](security-overview.md)
- [Procedimientos recomendados para la seguridad de Azure SQL Database](security-best-practice.md)
