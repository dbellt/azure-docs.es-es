---
title: 'API REST: sincronización entre varias bases de datos'
description: Use un script de ejemplo de la API REST para sincronizar entre varias bases de datos.
services: sql-database
ms.service: sql-database
ms.subservice: sql-data-sync
ms.custom: sqldbrb=1
ms.devlang: REST API
ms.topic: sample
author: MaraSteiu
ms.author: masteiu
ms.reviewer: mathoma
ms.date: 03/12/2019
ms.openlocfilehash: b0e8b12c7a58f3376c4e05819dbe60c9436ecef8
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110693362"
---
# <a name="use-rest-api-to-sync-data-between-multiple-databases"></a>Uso de la API REST para sincronizar datos entre varias bases de datos 

[!INCLUDE[appliesto-sqldb](../../includes/appliesto-sqldb.md)]

En este ejemplo de la API REST, se configura SQL Data Sync para sincronizar datos entre varias bases de datos.

Para obtener información general acerca de SQL Data Sync, consulte [Sincronización de datos entre varias bases de datos locales y de la nube con Azure SQL Data Sync](../sql-data-sync-data-sql-server-sql-database.md).

> [!IMPORTANT]
> SQL Data Sync no admite en este momento Instancia administrada de Azure SQL.

## <a name="create-sync-group"></a>Creación de un grupo de sincronización

Use la plantilla [crear o actualizar](/rest/api/sql/syncgroups/createorupdate) para crear un grupo de sincronización.
 
Al crear un grupo de sincronización, no pase el esquema de sincronización (table\column) ni masterSyncMemberName porque, en este momento, el grupo de sincronización aún no tiene información de table\column.

Solicitud de ejemplo para crear un grupo de sincronización: 

```http
PUT https://management.azure.com/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-65440/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328/syncGroups/syncgroupcrud-3187?api-version=2015-05-01-preview
```

```json
{
  "properties": {
    "interval": -1,
    "lastSyncTime": "0001-01-01T08:00:00Z",
    "conflictResolutionPolicy": "HubWin",
    "syncDatabaseId": "/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-3521/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328",
    "hubDatabaseUserName": "hubUser"
  }
}
```

Respuesta de ejemplo para crear un grupo de sincronización:

Código de estado: 200
```json
{
  "properties": {
    "interval": -1,
    "lastSyncTime": "0001-01-01T08:00:00Z",
    "conflictResolutionPolicy": "HubWin",
    "syncDatabaseId": "/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-3521/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328",
    "hubDatabaseUserName": "hubUser",
    "syncState": "NotReady"
  },
  "id": "/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-3521/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328/syncGroups/syncgroupcrud-3187",
  "name": "syncgroupcrud-3187",
  "type": "Microsoft.Sql/servers/databases/syncGroups"
}
```

Código de estado: 201
```json
{
  "properties": {
    "interval": -1,
    "lastSyncTime": "0001-01-01T08:00:00Z",
    "conflictResolutionPolicy": "HubWin",
    "syncDatabaseId": "/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-3521/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328",
    "hubDatabaseUserName": "hubUser",
    "syncState": "NotReady"
  },
  "id": "/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-3521/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328/syncGroups/syncgroupcrud-3187",
  "name": "syncgroupcrud-3187",
  "type": "Microsoft.Sql/servers/databases/syncGroups"
}
```

## <a name="create-sync-member"></a>Creación de un miembro de sincronización

Use la plantilla [crear o actualizar](/rest/api/sql/syncmembers/createorupdate) para crear un miembro de sincronización.

Solicitud de ejemplo para crear un miembro de sincronización:

```http
PUT https://management.azure.com/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-65440/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328/syncGroups/syncgroupcrud-3187/syncMembers/syncgroupcrud-4879?api-version=2015-05-01-preview
```

```json
{
  "properties": {
    "databaseType": "AzureSqlDatabase",
    "serverName": "syncgroupcrud-3379.database.windows.net",
    "databaseName": "syncgroupcrud-7421",
    "userName": "myUser",
    "syncDirection": "Bidirectional",
    "syncState": "UnProvisioned"
  }
}
```
Respuesta de ejemplo para crear un miembro de sincronización:

Código de estado: 200
```json
{
  "properties": {
    "databaseType": "AzureSqlDatabase",
    "serverName": "syncgroupcrud-3379.database.windows.net",
    "databaseName": "syncgroupcrud-7421",
    "userName": "myUser",
    "syncDirection": "Bidirectional",
    "syncState": "UnProvisioned"
  },
  "id": "/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-65440/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328/syncGroups/syncgroupcrud-3187/syncMembers/syncgroupcrud-4879",
  "name": "syncgroupcrud-4879",
  "type": "Microsoft.Sql/servers/databases/syncGroups/syncMembers"
}
```

Código de estado: 201
```json
{
  "properties": {
    "databaseType": "AzureSqlDatabase",
    "serverName": "syncgroupcrud-3379.database.windows.net",
    "databaseName": "syncgroupcrud-7421",
    "userName": "myUser",
    "syncDirection": "Bidirectional",
    "syncState": "UnProvisioned"
  },
  "id": "/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-65440/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328/syncGroups/syncgroupcrud-3187/syncMembers/syncgroupcrud-4879",
  "name": "syncgroupcrud-4879",
  "type": "Microsoft.Sql/servers/databases/syncGroups/syncMembers"
}
```

## <a name="refresh-schema"></a>Actualización del esquema

Una vez creado correctamente el grupo de sincronización, actualice el esquema mediante las siguientes plantillas.

Use la plantilla [actualizar esquema central](/rest/api/sql/syncgroups/refreshhubschema) para actualizar el esquema de la base de datos central. 

Solicitud de ejemplo para actualizar el esquema de la base de datos central: 

```http
POST https://management.azure.com/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-65440/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328/syncGroups/syncgroupcrud-3187/refreshHubSchema?api-version=2015-05-01-preview
```

Respuesta de ejemplo para actualizar el esquema de la base de datos central: 

Código de estado: 200

Código de estado: 202

Use la plantilla de [lista de esquemas centrales](/rest/api/sql/syncgroups/listhubschemas) para mostrar el esquema de la base de datos central. 

Use la plantilla [actualizar esquema de miembro](/rest/api/sql/syncmembers/refreshmemberschema) para actualizar el esquema de la base de datos miembro. 

Use la plantilla de [lista de esquemas de miembro](/rest/api/sql/syncmembers/listmemberschemas) para mostrar el esquema de la base de datos miembro. 

Continúe con el paso siguiente cuando el esquema se actualice correctamente. 

## <a name="update-sync-group"></a>Actualización de un grupo de sincronización 

Use la plantilla [crear o actualizar](/rest/api/sql/syncgroups/createorupdate) para actualizar el grupo de sincronización.

Especifique el esquema de sincronización para actualizar el grupo de sincronización. Incluya el esquema y masterSyncMemberName, que es el nombre que contiene el esquema que quiere usar. 

Solicitud de ejemplo para actualizar el grupo de sincronización: 

```http
PUT https://management.azure.com/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-65440/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328/syncGroups/syncgroupcrud-3187?api-version=2015-05-01-preview
```

```json
{
  "properties": {
    "interval": -1,
    "lastSyncTime": "0001-01-01T08:00:00Z",
    "conflictResolutionPolicy": "HubWin",
    "syncDatabaseId": "/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-3521/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328",
    "hubDatabaseUserName": "hubUser"
  }
}
```

Respuesta de ejemplo para actualizar el grupo de sincronización: 

```json
{
  "properties": {
    "interval": -1,
    "lastSyncTime": "0001-01-01T08:00:00Z",
    "conflictResolutionPolicy": "HubWin",
    "syncDatabaseId": "/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-3521/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328",
    "hubDatabaseUserName": "hubUser",
    "syncState": "NotReady"
  },
  "id": "/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-3521/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328/syncGroups/syncgroupcrud-3187",
  "name": "syncgroupcrud-3187",
  "type": "Microsoft.Sql/servers/databases/syncGroups"
}
```

```json
{
  "properties": {
    "interval": -1,
    "lastSyncTime": "0001-01-01T08:00:00Z",
    "conflictResolutionPolicy": "HubWin",
    "syncDatabaseId": "/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-3521/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328",
    "hubDatabaseUserName": "hubUser",
    "syncState": "NotReady"
  },
  "id": "/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-3521/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328/syncGroups/syncgroupcrud-3187",
  "name": "syncgroupcrud-3187",
  "type": "Microsoft.Sql/servers/databases/syncGroups"
}
```
## <a name="update-sync-member"></a>Actualización de un miembro de sincronización

Use la plantilla [crear o actualizar](/rest/api/sql/syncmembers/createorupdate) para actualizar el miembro de sincronización.

Solicitud de ejemplo para actualizar un miembro de sincronización: 

```http
PUT https://management.azure.com/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-65440/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328/syncGroups/syncgroupcrud-3187/syncMembers/syncgroupcrud-4879?api-version=2015-05-01-preview
```

```json
{
  "properties": {
    "databaseType": "AzureSqlDatabase",
    "serverName": "syncgroupcrud-3379.database.windows.net",
    "databaseName": "syncgroupcrud-7421",
    "userName": "myUser",
    "syncDirection": "Bidirectional",
    "syncState": "UnProvisioned"
  }
}
```

Respuesta de ejemplo para actualizar un miembro de sincronización: 

Código de estado: 200
```json
{
  "properties": {
    "databaseType": "AzureSqlDatabase",
    "serverName": "syncgroupcrud-3379.database.windows.net",
    "databaseName": "syncgroupcrud-7421",
    "userName": "myUser",
    "syncDirection": "Bidirectional",
    "syncState": "UnProvisioned"
  },
  "id": "/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-65440/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328/syncGroups/syncgroupcrud-3187/syncMembers/syncgroupcrud-4879",
  "name": "syncgroupcrud-4879",
  "type": "Microsoft.Sql/servers/databases/syncGroups/syncMembers"
}
```

Código de estado: 201
```json
{
  "properties": {
    "databaseType": "AzureSqlDatabase",
    "serverName": "syncgroupcrud-3379.database.windows.net",
    "databaseName": "syncgroupcrud-7421",
    "userName": "myUser",
    "syncDirection": "Bidirectional",
    "syncState": "UnProvisioned"
  },
  "id": "/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-65440/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328/syncGroups/syncgroupcrud-3187/syncMembers/syncgroupcrud-4879",
  "name": "syncgroupcrud-4879",
  "type": "Microsoft.Sql/servers/databases/syncGroups/syncMembers"
}
```

## <a name="trigger-sync"></a>Desencadenamiento de la sincronización

Use la plantilla [desencadenar sincronización](/rest/api/sql/syncgroups/triggersync) para desencadenar una operación de sincronización.

Solicitud de ejemplo para la operación de desencadenamiento de la sincronización: 

```http
POST https://management.azure.com/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-65440/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328/syncGroups/syncgroupcrud-3187/triggerSync?api-version=2015-05-01-preview
```

Respuesta de ejemplo para la operación de desencadenamiento de la sincronización: 

Código de estado: 200

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

Encontrará más ejemplos de scripts de PowerShell de SQL Database en los [scripts de PowerShell de Azure SQL Database](../powershell-script-content-guide.md).

Para más información acerca de SQL Data Sync, consulte:

- Introducción: [Sincronización de datos entre varias bases de datos locales y en la nube con Azure SQL Data Sync](../sql-data-sync-data-sql-server-sql-database.md)
- Configuración de Data Sync
    - Uso de Azure Portal: [Tutorial: Configuración de SQL Data Sync para sincronizar datos entre Azure SQL Database y SQL Server](../sql-data-sync-sql-server-configure.md)
    - Uso de PowerShell: [Uso de PowerShell para sincronizar datos entre una base de datos de Azure SQL Database y SQL Server](sql-data-sync-sync-data-between-azure-onprem.md)
- Data Sync Agent: [Data Sync Agent para Azure SQL Data Sync](../sql-data-sync-agent-overview.md)
- Procedimientos recomendados: [Procedimientos recomendados para Azure SQL Data Sync](../sql-data-sync-best-practices.md)
- Supervisión: [Monitor SQL Data Sync with Azure Monitor logs](../monitor-tune-overview.md) (Supervisión de SQL Data Sync con registros de Azure Monitor)
- Solución de problemas: [Solución de problemas con Azure SQL Data Sync](../sql-data-sync-troubleshoot.md)
- Actualización del esquema de sincronización
    - Uso de Transact-SQL: [Automatización de la replicación de los cambios de esquema en Azure SQL Data Sync](../sql-data-sync-update-sync-schema.md)
    - Uso de PowerShell: [Uso de PowerShell para actualizar el esquema de sincronización en un grupo de sincronización existente](update-sync-schema-in-sync-group.md)

Para obtener más información sobre SQL Database, vea:

- [Información general de SQL Database](../sql-database-paas-overview.md)
- [Administración del ciclo de vida de las aplicaciones](/previous-versions/sql/sql-server-guides/jj907294(v=sql.110))