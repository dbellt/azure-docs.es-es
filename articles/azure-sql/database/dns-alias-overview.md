---
title: Alias DNS
description: Las aplicaciones pueden conectarse a un alias y usarlo como nombre del servidor de Azure SQL Database. Mientras tanto, puede cambiar la instancia de SQL Database a la que apunta el alias en cualquier momento, a fin de facilitar las prueba, etc.
services: sql-database
ms.service: sql-database
ms.subservice: high-availability
ms.custom: seo-lt-2019 sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: BustosMSFT
ms.author: robustos
ms.reviewer: mathoma, jrasnick, vanto
ms.date: 06/26/2019
ms.openlocfilehash: 6ab22ec300fd9be945f7cd0d7f2b83e5f7a409b4
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110689120"
---
# <a name="dns-alias-for-azure-sql-database"></a>Alias DNS en Azure SQL Database
[!INCLUDE[appliesto-sqldb-asa](../includes/appliesto-sqldb-asa.md)]

Azure SQL Database tiene un servidor de Sistema de nombres de dominio (DNS). Las API de REST y PowerShell aceptan [llamadas para crear y administrar alias DNS](#anchor-powershell-code-62x) como nombre del [servidor SQL lógico](logical-servers.md).

Se puede usar un *alias DNS* en lugar del nombre de servidor. Los programas cliente pueden usar el alias en sus cadenas de conexión. El alias DNS proporciona una capa de traducción que puede redirigir los programas cliente a diferentes servidores. Esta capa evita las dificultades de tener que buscar y editar todos los clientes y sus cadenas de conexión.

Los usos habituales de un alias DNS son los siguientes:

- Cree un nombre sencillo de recordar para un servidor.
- Durante el desarrollo inicial, el alias puede hacer referencia a un servidor de prueba. Cuando la aplicación esté activa, puede modificar el alias para que haga referencia al servidor de producción. La transición de prueba a producción no requiere ninguna modificación en las configuraciones de varios clientes que se conectan al servidor.
- Suponga que la única base de datos de la aplicación se mueve a otro servidor. Puede modificar el alias sin necesidad de modificar las configuraciones de varios clientes.
- Durante una interrupción regional, se usa la restauración geográfica para recuperar la base de datos en un servidor y una región diferentes. Puede modificar el alias existente para que apunte al nuevo servidor de forma que la aplicación cliente existente pueda volver a conectarse a él.

## <a name="domain-name-system-dns-of-the-internet"></a>Sistema de nombres de dominio (DNS) de Internet

Internet se basa en el DNS. DNS traduce los nombres descriptivos en el nombre del servidor.

## <a name="scenarios-with-one-dns-alias"></a>Escenarios con un alias DNS

Suponga que necesita cambiar el sistema a un nuevo servidor. En el pasado, era necesario encontrar y actualizar cada una de las cadenas de conexión de cada programa cliente. Ahora, sin embargo, si las cadenas de conexión usan un alias DNS, solo se debe actualizar la propiedad del alias.

La característica de alias DNS de Azure SQL Database puede ayudar en los siguientes escenarios:

### <a name="test-to-production"></a>Transición de prueba a producción

Al iniciar el desarrollo de los programas cliente, haga que usen un alias DNS en sus cadenas de conexión. Las propiedades del alias deben apuntar a una versión de prueba del servidor.

Más adelante, cuando el nuevo sistema pase a producción, puede actualizar las propiedades del alias para que apunten al servidor de producción. No es necesario ningún cambio en los programas cliente.

### <a name="cross-region-support"></a>Compatibilidad con varias regiones

Una recuperación ante desastres podría desplazar el servidor a una región geográfica diferente. En sistemas que usan un alias DNS, se puede evitar la necesidad de encontrar y actualizar todas las cadenas de conexión de todos los clientes. En su lugar, puede actualizar un alias para que haga referencia al nuevo servidor que ahora hospeda la instancia de Azure SQL Database.

## <a name="properties-of-a-dns-alias"></a>Propiedades de un alias DNS

Las siguientes propiedades se aplican a todos los alias DNS del servidor:

- *Nombre único:* cada nombre de alias que se crea es único en todos los servidores, igual que lo son los nombres de servidor.
- *Se requiere un servidor:* no se puede crear un alias DNS a menos que haga referencia exacta a un servidor y el servidor ya exista. Un alias actualizado siempre debe hacer referencia exacta a un servidor existente.
  - Cuando se quita un servidor, el sistema de Azure también quita todos los alias DNS que hagan referencia a él.
- *No enlazado a ninguna región:* los alias DNS no están enlazados a ninguna región. Los alias DNS pueden actualizarse para que hagan referencia a un servidor que resida en cualquier región geográfica.
  - Sin embargo, cuando se actualiza un alias para que haga referencia a otro servidor, ambos servidores deben existir en la misma *suscripción* de Azure.
- *Permisos:* para administrar un alias DNS, el usuario debe tener permisos de *colaborador de servidor* u otros superiores. Para más información, vea la [introducción al control de acceso basado en rol de Azure en Azure Portal](../../role-based-access-control/overview.md).

## <a name="manage-your-dns-aliases"></a>Administración de los alias DNS

Tanto las API de REST como los cmdlets de PowerShell le permiten administrar los alias DNS mediante programación.

### <a name="rest-apis-for-managing-your-dns-aliases"></a>API de REST para administrar los alias DNS

La documentación de las API de REST está disponible cerca de la ubicación web siguiente:

- [API REST de Azure SQL Database](/rest/api/sql/)

Además, las API de REST puede verse en GitHub en:

- [API de REST del alias DNS de Azure SQL Database](https://github.com/Azure/azure-rest-api-specs/blob/master/specification/sql/resource-manager/Microsoft.Sql/preview/2017-03-01-preview/serverDnsAliases.json)

<a name="anchor-powershell-code-62x"></a>

### <a name="powershell-for-managing-your-dns-aliases"></a>PowerShell para administrar los alias DNS

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]
> [!IMPORTANT]
> El módulo de Azure Resource Manager para PowerShell todavía es compatible, pero todo el desarrollo futuro se realizará para el módulo Az.Sql. Para estos cmdlets, consulte [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Los argumentos para los comandos del módulo Az y los módulos AzureRm son esencialmente idénticos.

Existen cmdlets de PowerShell que llaman a las API de REST.

Un ejemplo de código de cmdlets de PowerShell que se usa para administrar alias DNS se documenta en:

- [PowerShell con alias DNS para Azure SQL Database](dns-alias-powershell-create.md)

Los cmdlets usados en el ejemplo de código son los siguientes:

- [New-AzSqlServerDnsAlias](/powershell/module/az.Sql/New-azSqlServerDnsAlias): crea un nuevo alias DNS en el sistema del servicio Azure SQL Database. El alias hace referencia al servidor 1.
- [Get-AzSqlServerDnsAlias](/powershell/module/az.Sql/Get-azSqlServerDnsAlias): obtiene y enumera todos los alias DNS asignados al servidor 1.
- [Set-AzSqlServerDnsAlias](/powershell/module/az.Sql/Set-azSqlServerDnsAlias): modifica el nombre del servidor con el que el alias está configurado para hacer referencia, del servidor 1 al servidor 2.
- [Remove-AzSqlServerDnsAlias](/powershell/module/az.Sql/Remove-azSqlServerDnsAlias): quita el alias DNS del servidor 2 mediante el nombre del alias.

## <a name="limitations"></a>Limitaciones

Actualmente, un alias DNS tiene las siguientes limitaciones:

- *Retraso de hasta 2 minutos:* un alias DNS tarda hasta 2 minutos en actualizarse o quitarse.
  - Independientemente de cualquier retraso breve, el alias deja inmediatamente de remitir las conexiones cliente al servidor heredado.
- *Búsqueda DNS:* por ahora, la única manera autoritativa de comprobar cuál es el servidor al que hace referencia un determinado alias es realizar una [búsqueda DNS](/windows-server/administration/windows-commands/nslookup).
- _No se admite la auditoría de tablas:_ no se puede usar un alias DNS en un servidor que tenga habilitada la *auditoría de tablas* en una base de datos.
  - La auditoría de tablas está en desuso.
  - Se recomienda pasar a la [auditoría de blobs](../../azure-sql/database/auditing-overview.md).

## <a name="related-resources"></a>Recursos relacionados

- [Introducción a la continuidad empresarial con Azure SQL Database](business-continuity-high-availability-disaster-recover-hadr-overview.md), que incluye la recuperación ante desastres.
- [Referencia de API REST en Azure](/rest/api/azure/)
- [API de alias DNS de servidor](/rest/api/sql/2020-11-01-preview/serverdnsaliases)

## <a name="next-steps"></a>Pasos siguientes

- [PowerShell con alias DNS para Azure SQL Database](dns-alias-powershell-create.md)
