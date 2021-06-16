---
title: Introducción al desarrollo de aplicaciones
description: Aprenda sobre las bibliotecas de conectividad disponibles y los procedimientos recomendados para las aplicaciones que se conectan a SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.topic: conceptual
author: dzsquared
ms.author: drskwier
ms.reviewer: mathoma
ms.date: 11/14/2019
ms.custom: sqldbrb=2
ms.openlocfilehash: 68a1eca2800978803214384944349e227b620764
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110694506"
---
# <a name="application-development-overview---sql-database--sql-managed-instance"></a>Introducción al desarrollo de aplicaciones: SQL Database e Instancia administrada de SQL

[!INCLUDE[appliesto-sqldb-asa](../includes/appliesto-sqldb-asa.md)]

Este artículo le lleva por las consideraciones básicas que debe tener en cuenta un desarrollador al escribir código para conectarse a la base de datos en Azure. Este artículo se aplica a Azure SQL Database e Instancia administrada de Azure SQL.

## <a name="language-and-platform"></a>Plataforma y lenguaje

Puede usar distintas [plataformas y lenguajes de programación](connect-query-content-reference-guide.md) para conectarse y consultar a Azure SQL Database. Puede encontrar [aplicaciones de ejemplo](https://azure.microsoft.com/resources/samples/?service=sql-database&sort=0) que puede usar para conectarse a la base de datos.

Puede aprovechar herramientas de código abierto como [cheetah](https://github.com/wunderlist/cheetah), [sql-cli](https://www.npmjs.com/package/sql-cli) o [VS Code](https://code.visualstudio.com/). Además, Azure SQL Database funciona con herramientas de Microsoft como [Visual Studio](https://www.visualstudio.com/downloads/) y [SQL Server Management Studio](/sql/ssms/sql-server-management-studio-ssms). También puede usar Azure Portal, PowerShell y API REST, que le ayudan a conseguir una mayor productividad.

## <a name="authentication"></a>Authentication

El acceso a Azure SQL Database está protegido con inicios de sesión y firewalls. Azure SQL Database admite inicios de sesión y usuarios de SQL Server y de [autenticación de Azure Active Directory](authentication-aad-overview.md). Los inicios de sesión de Azure Active Directory solo están disponibles en Instancia administrada de SQL. 

Obtenga más información sobre cómo [administrar el acceso e inicio de sesión a bases de datos](logins-create-manage.md).

## <a name="connections"></a>Conexiones

En la lógica de conexión de cliente, reemplace el tiempo de espera predeterminado para que sea de 30 segundos. El valor predeterminado de 15 segundos es demasiado corto para conexiones que dependen de Internet.

Si usa un [grupo de conexiones](/dotnet/framework/data/adonet/sql-server-connection-pooling), asegúrese de cerrar la conexión en el momento en que el programa no la esté usando activamente y no esté preparándose para volver a usarla.

Evite las transacciones de larga ejecución, ya que cualquier error de conexión o de infraestructura puede revertir la transacción. Si es posible, divida la transacción en varias transacciones más pequeñas y use el [procesamiento por lotes para mejorar el rendimiento](../performance-improve-use-batching.md).

## <a name="resiliency"></a>Resistencia

Azure SQL Database es un servicio en la nube del que puede esperar errores transitorios que se producen en la infraestructura subyacente o en la comunicación entre las entidades de la nube. Aunque Azure SQL Database es resistente a los errores transitorios de infraestructura, estos pueden afectar a la conectividad. Cuando se produce un error transitorio al establecer una conexión con SQL Database, el código debe [reintentar la llamada](troubleshoot-common-connectivity-issues.md). Recomendamos hacer uso de la lógica de interrupción en la lógica de reintentos, ya que así el servicio no se sobrecargará con los reintentos de varios clientes a la vez. La lógica de reintento depende de los [códigos de error para las aplicaciones cliente de SQL Database](troubleshoot-common-errors-issues.md).

Para más información sobre cómo prepararse para los eventos de mantenimiento planeado en su Azure SQL Database, consulte [Planeación de los eventos de mantenimiento en Azure SQL Database](planned-maintenance.md).

## <a name="network-considerations"></a>Consideraciones sobre la red

- En el equipo que hospeda el programa cliente, asegúrese de que el firewall permita la comunicación TCP saliente en el puerto 1433.  Más información: [Configuración de un firewall de Azure SQL Database](firewall-configure.md).
- Si el programa cliente se conecta a SQL Database mientras el cliente se ejecuta en una máquina virtual (VM) de Azure, debe abrir determinados intervalos de puerto en la máquina virtual. Más información: [Puertos más allá del 1433 para ADO.NET 4.5 y SQL Database](adonet-v12-develop-direct-route-ports.md).
- En ocasiones, las conexiones de cliente a Azure SQL Database omiten el proxy e interactúan directamente con la base de datos. Los puertos que no sean 1433 se convierten en puertos importantes. Para más información, consulte [Arquitectura de conectividad de Azure SQL Database](connectivity-architecture.md) y [Puertos más allá de 1433 para ADO.NET 4.5 y SQL Database](adonet-v12-develop-direct-route-ports.md).
- Para más información sobre la configuración de red de una instancia de Instancia administrada de SQL, vea la [configuración de red de Instancia administrada de SQL](../managed-instance/how-to-content-reference-guide.md#network-configuration).

## <a name="next-steps"></a>Pasos siguientes

Descubra todas las funcionalidades de [SQL Database](sql-database-paas-overview.md) e [Instancia administrada de SQL](../managed-instance/sql-managed-instance-paas-overview.md).

Para empezar, consulte las guías de [Azure SQL Database](quickstart-content-reference-guide.md) e [Instancias administradas de Azure SQL](../managed-instance/quickstart-content-reference-guide.md).