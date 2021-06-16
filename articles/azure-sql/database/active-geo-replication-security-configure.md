---
title: Configuración de la seguridad para la recuperación ante desastres
description: Conozca las consideraciones sobre seguridad para configurar y administrar la seguridad después de una restauración de la base de datos o una conmutación por error a un servidor secundario.
services: sql-database
ms.service: sql-database
ms.subservice: high-availability
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: BustosMSFT
ms.author: robustos
ms.reviewer: mathoma
ms.date: 12/18/2018
ms.openlocfilehash: 6dde8aaa676e8045326f7078e1c3fd2a0ada6dc2
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110699846"
---
# <a name="configure-and-manage-azure-sql-database-security-for-geo-restore-or-failover"></a>Configuración y administración de la seguridad de Azure SQL Database para la restauración geográfica o la conmutación por error
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

En este artículo se describen los requisitos de autenticación para configurar y controlar la [replicación geográfica activa](active-geo-replication-overview.md) y los [grupos de conmutación por error automática](auto-failover-group-overview.md). También se proporcionan los pasos necesarios para configurar el acceso de usuario a la base de datos secundaria. Asimismo, también se describe cómo habilitar el acceso a la base de datos recuperada después de usar la [georestauración](recovery-using-backups.md#geo-restore). Para más información sobre opciones de recuperación, consulte [Información general sobre la continuidad empresarial](business-continuity-high-availability-disaster-recover-hadr-overview.md).

## <a name="disaster-recovery-with-contained-users"></a>Recuperación ante desastres con usuarios contenidos

A diferencia de los usuarios tradicionales, que deben asignarse a inicios de sesión en la base de datos maestra, un usuario independiente se administra completamente en la base de datos, lo que ofrece dos ventajas. En el escenario de replicación geográfica, los usuarios pueden proceder a conectarse a la nueva base de datos principal o a la base de datos recuperada mediante georrestauración, sin ninguna configuración adicional, ya que la base de datos administra los usuarios. También existen ventajas potenciales de escalabilidad y rendimiento con esta configuración desde la perspectiva del inicio de sesión. Para obtener más información, vea [Usuarios de base de datos independiente: hacer que la base de datos sea portátil](/sql/relational-databases/security/contained-database-users-making-your-database-portable).

El principal inconveniente es que la administración del proceso de recuperación ante desastres a escala es más compleja. Si tiene varias bases de datos que usan el mismo inicio de sesión, el mantenimiento de las credenciales que usan los usuarios independientes en varias bases de datos puede invalidar las ventajas de los usuarios independientes. Por ejemplo, la directiva de rotación de contraseñas requiere que se realicen cambios constantemente en varias bases de datos en lugar de cambiar la contraseña para el inicio de sesión una vez en la base de datos maestra. Por este motivo, si tiene varias bases de datos que utilizan el mismo nombre de usuario y la misma contraseña, no se recomienda usar usuarios contenidos.

## <a name="how-to-configure-logins-and-users"></a>Configuración de inicios de sesión y de usuarios

Si usa inicios de sesión y usuarios (en lugar de usuarios contenidos), debe realizar pasos adicionales para asegurarse de que existan los mismos inicios de sesión en la base de datos maestra. En las secciones siguientes, se describen los pasos necesarios y otras consideraciones.

  >[!NOTE]
  > También es posible usar inicios de sesión de Azure Active Directory (AAD) para administrar las bases de datos. Para más información, consulte [Inicios de sesión y usuarios de Azure SQL](./logins-create-manage.md).

### <a name="set-up-user-access-to-a-secondary-or-recovered-database"></a>Configuración del acceso de usuario a una base de datos secundaria o recuperada

Para que la base de datos secundaria se pueda utilizar como base de datos secundaria de solo lectura y garantizar el acceso adecuado a la nueva base de datos principal o a la base de datos recuperada mediante la georrestauración, la base de datos maestra del servidor de destino debe tener la configuración de seguridad adecuada antes de la recuperación.

Los permisos específicos para cada paso se describen más adelante en este tema.

La preparación del acceso de usuario a una base de datos secundaria de replicación geográfica debe realizarse como parte de la configuración de replicación geográfica. La preparación de acceso de usuario a las bases de datos georrestauradas debe realizarse en cualquier momento en que el servidor original esté en línea (por ejemplo, como parte de la obtención de detalles de recuperación ante desastres).

> [!NOTE]
> Si realiza una conmutación por error o la georrestauración en un servidor que no tiene configurado correctamente el acceso de inicio de sesión al mismo, se limitará a la cuenta de administrador del servidor.

La configuración de los inicios de sesión en el servidor de destino implica los tres pasos descritos a continuación:

#### <a name="1-determine-logins-with-access-to-the-primary-database"></a>1. Determinar los inicios de sesión con acceso a la base de datos principal

El primer paso del proceso es determinar qué inicios de sesión se deben duplicar en el servidor de destino. Esto se logra con un par de instrucciones SELECT, una en la base de datos maestra lógica del servidor de origen y otra, en la base de datos principal en sí.

El administrador del servidor o un miembro del rol de servidor **LoginManager** son los únicos que pueden determinar los inicios de sesión en el servidor de origen con la siguiente instrucción SELECT.

```sql
SELECT [name], [sid]
FROM [sys].[sql_logins]
WHERE [type_desc] = 'SQL_Login'
```

Solo un miembro del rol de base de datos db_owner, el usuario dbo o el administrador del servidor pueden determinar todas las entidades de seguridad de usuario de base de datos en la base de datos principal.

```sql
SELECT [name], [sid]
FROM [sys].[database_principals]
WHERE [type_desc] = 'SQL_USER'
```

#### <a name="2-find-the-sid-for-the-logins-identified-in-step-1"></a>2. Buscar el SID de los inicios de sesión que identificó en el paso 1

Al comparar los resultados de las consultas de la sección anterior y cotejar los SID, puede asignar el inicio de sesión de servidor a un usuario de base de datos. Los inicios de sesión que cuenten con un usuario de base de datos con un SID coincidente tienen acceso de usuario a esa base de datos como entidad de seguridad de usuario de base de datos.

La consulta siguiente se puede usar para ver todas las entidades de seguridad de usuario y sus SID en una base de datos. Solo un miembro del rol de servidor db_owner o el administrador del servidor pueden ejecutar esta consulta.

```sql
SELECT [name], [sid]
FROM [sys].[database_principals]
WHERE [type_desc] = 'SQL_USER'
```

> [!NOTE]
> Los usuarios de **INFORMATION_SCHEMA** y **sys** tienen SID *NULL* SID, mientras que el SID de **guest** es **0x00**. El SID de **dbo** puede empezar por *0x01060000000001648000000000048454* si el creador de la base de datos era el administrador del servidor en lugar de un miembro de **DbManager**.

#### <a name="3-create-the-logins-on-the-target-server"></a>3. Crear los inicios de sesión en el servidor de destino

El último paso consiste en ir al servidor o los servidores de destino y generar los inicios de sesión con los SID correspondientes. Esta es la sintaxis básica.

```sql
CREATE LOGIN [<login name>]
WITH PASSWORD = <login password>,
SID = <desired login SID>
```

> [!NOTE]
> Si desea conceder acceso de usuario a la base de datos secundaria, pero no a la principal, puede hacerlo modificando el inicio de sesión de usuario en el servidor principal con la sintaxis siguiente.
>
> ```sql
> ALTER LOGIN <login name> DISABLE
> ```
>
> DISABLE no cambia la contraseña, por lo que siempre puede habilitarlo si es necesario.

## <a name="next-steps"></a>Pasos siguientes

* Para obtener más información sobre cómo administrar el acceso a la base de datos y los inicios de sesión, consulte [Seguridad de SQL Database: administrar la seguridad del inicio de sesión y el acceso a la base de datos](logins-create-manage.md).
* Para obtener más información sobre los usuarios de base de datos independiente, consulte [Usuarios de base de datos independiente: hacer que la base de datos sea portátil](/sql/relational-databases/security/contained-database-users-making-your-database-portable).
* Para obtener más información sobre la replicación geográfica activa, consulte [Replicación geográfica activa](active-geo-replication-overview.md).
* Para obtener información acerca de los grupos de conmutación por error automática, consulte [Grupos de conmutación por error automática](auto-failover-group-overview.md).
* Para obtener información sobre cómo utilizar la funcionalidad de restauración geográfica, consulte [Restauración geográfica](recovery-using-backups.md#geo-restore)