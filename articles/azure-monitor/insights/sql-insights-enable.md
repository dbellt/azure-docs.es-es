---
title: Habilitación de SQL Insights
description: Habilitación de SQL Insights en Azure Monitor
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 03/15/2021
ms.openlocfilehash: 012aa364fe9e379455b6b63f7c9e541d2d5b97ed
ms.sourcegitcommit: 6f1aa680588f5db41ed7fc78c934452d468ddb84
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107726905"
---
# <a name="enable-sql-insights-preview"></a>Habilitación de SQL Insights (versión preliminar)
En este artículo se describe cómo habilitar [SQL Insights](sql-insights-overview.md) para supervisar las implementaciones de SQL. La supervisión se realiza desde una máquina virtual de Azure que establece una conexión con las implementaciones de SQL y usa vistas de administración dinámica para recopilar datos de supervisión. Puede controlar qué conjuntos de datos se recopilan y la frecuencia de recopilación mediante un perfil de supervisión.

> [!NOTE]
> Para habilitar SQL Insights mediante la creación del perfil de supervisión y la máquina virtual mediante una plantilla de Resource Manager, vea [Ejemplos de plantilla de Resource Manager para SQL Insights](resource-manager-sql-insights.md).

## <a name="create-log-analytics-workspace"></a>Creación de un área de trabajo de Log Analytics
SQL Insights almacena sus datos en una o varias [áreas de trabajo de Log Analytics](../logs/data-platform-logs.md#log-analytics-workspaces).  Antes de poder habilitar SQL Insights, debe [crear un área de trabajo](../logs/quick-create-workspace.md) o seleccionar una existente. Se puede usar una sola área de trabajo con varios perfiles de supervisión, pero el área de trabajo y los perfiles deben estar ubicados en la misma región de Azure. Para habilitar las características de SQL Insights y acceder a estas, debe tener el rol de [Colaborador de Log Analytics](../logs/manage-access.md) en el área de trabajo. 

## <a name="create-monitoring-user"></a>Creación de un usuario de supervisión 
Necesita un usuario en las implementaciones de SQL que desea supervisar. Siga los procedimientos que se indican a continuación para los diferentes tipos de implementaciones de SQL.

Las instrucciones siguientes cubren el proceso por tipo de implementación de SQL que puede supervisar.  Para lograr esto con un script en varios recursos de SQL a la vez, vea el [archivo LÉAME](https://github.com/microsoft/Application-Insights-Workbooks/blob/master/Workbooks/Workloads/SQL/SQL%20Insights%20Onboarding%20Scripts/Permissions_LoginUser_Account_Creation-README.txt) y el [script de ejemplo](https://github.com/microsoft/Application-Insights-Workbooks/blob/master/Workbooks/Workloads/SQL/SQL%20Insights%20Onboarding%20Scripts/Permissions_LoginUser_Account_Creation.ps1) siguientes.


### <a name="azure-sql-database"></a>Azure SQL Database
Abra Azure SQL Database con [SQL Server Management Studio](../../azure-sql/database/connect-query-ssms.md) o el [Editor de Power Query (versión preliminar)](../../azure-sql/database/connect-query-portal.md) en Azure Portal.

Ejecute el siguiente script para crear un usuario con los permisos necesarios. Reemplace *user* por un nombre de usuario y *mystrongpassword* por una contraseña.

```sql
CREATE USER [user] WITH PASSWORD = N'mystrongpassword'; 
GO 
GRANT VIEW DATABASE STATE TO [user]; 
GO 
```

:::image type="content" source="media/sql-insights-enable/telegraf-user-database-script.png" alt-text="Cree el script de usuario de Telegraf." lightbox="media/sql-insights-enable/telegraf-user-database-script.png":::

Compruebe que se ha creado el usuario.

:::image type="content" source="media/sql-insights-enable/telegraf-user-database-verify.png" alt-text="Compruebe el script de usuario de Telegraf." lightbox="media/sql-insights-enable/telegraf-user-database-verify.png":::

```sql
select name as username,
       create_date,
       modify_date,
       type_desc as type,
       authentication_type_desc as authentication_type
from sys.database_principals
where type not in ('A', 'G', 'R', 'X')
       and sid is not null
order by username
```

### <a name="azure-sql-managed-instance"></a>Instancia administrada de Azure SQL
Inicie sesión en Azure SQL Managed Instance y use [SQL Server Management Studio](../../azure-sql/database/connect-query-ssms.md) o una herramienta similar para ejecutar el siguiente script con el fin de crear el usuario de supervisión con los permisos necesarios. Reemplace *user* por un nombre de usuario y *mystrongpassword* por una contraseña.

 
```sql
USE master; 
GO 
CREATE LOGIN [user] WITH PASSWORD = N'mystrongpassword'; 
GO 
GRANT VIEW SERVER STATE TO [user]; 
GO 
GRANT VIEW ANY DEFINITION TO [user]; 
GO 
```

### <a name="sql-server"></a>SQL Server
Inicie sesión en la máquina virtual de Azure que ejecuta SQL Server y use [SQL Server Management Studio](../../azure-sql/database/connect-query-ssms.md) o una herramienta similar para ejecutar el siguiente script con el fin de crear el usuario de supervisión con los permisos necesarios. Reemplace *user* por un nombre de usuario y *mystrongpassword* por una contraseña.

 
```sql
USE master; 
GO 
CREATE LOGIN [user] WITH PASSWORD = N'mystrongpassword'; 
GO 
GRANT VIEW SERVER STATE TO [user]; 
GO 
GRANT VIEW ANY DEFINITION TO [user]; 
GO
```

Compruebe que se ha creado el usuario.

```sql
select name as username,
       create_date,
       modify_date,
       type_desc as type
from sys.server_principals
where type not in ('A', 'G', 'R', 'X')
       and sid is not null
order by username
```

## <a name="create-azure-virtual-machine"></a>Creación de una máquina virtual de Azure 
Tendrá que crear una o varias máquinas virtuales de Azure que se usarán para recopilar datos para supervisar SQL.  

> [!NOTE]
>  Los [perfiles de supervisión](#create-sql-monitoring-profile) especifican qué datos se van a recopilar de los diferentes tipos de SQL que desea supervisar. Cada máquina virtual de supervisión solo puede tener un perfil de supervisión asociado. Si necesita varios perfiles de supervisión, debe crear una máquina virtual para cada uno.

### <a name="azure-virtual-machine-requirements"></a>Requisitos para las máquinas virtuales de Azure
Las máquinas virtuales de Azure tienen los siguientes requisitos.

- Sistema operativo: Ubuntu 18.04 
- Tamaños de máquina virtual de Azure recomendados: Standard_B2s (2 CPU, 4 GiB de memoria) 
- Regiones admitidas: cualquier [región que admita el agente de Azure Monitor](../agents/azure-monitor-agent-overview.md#supported-regions)

> [!NOTE]
> El tamaño de la máquina virtual Standard_B2s (2 CPU, 4 GiB) admitirá hasta 100 cadenas de conexión. No debe asignar más de 100 conexiones a una sola máquina virtual.

Según la configuración de red de los recursos de SQL, es posible que las máquinas virtuales deban colocarse en la misma red virtual que los recursos de SQL. De este modo, podrán establecer conexiones de red para recopilar datos de supervisión.  

## <a name="configure-network-settings"></a>Configuración de la red
Cada tipo de SQL ofrece métodos para que la máquina virtual de supervisión tenga acceso de forma segura a SQL.  En las secciones siguientes se tratan las opciones basadas en el tipo de SQL.

### <a name="azure-sql-databases"></a>Azure SQL Database  

SQL Insights admite el acceso a su instancia de Azure SQL Database a través de su punto de conexión público, así como de su red virtual.

Para el acceso a través del punto de conexión público, debe agregar una regla en la página **Configuración de firewall** y en la sección de [configuración del firewall de IP](https://docs.microsoft.com/azure/azure-sql/database/network-access-controls-overview#ip-firewall-rules).  Para especificar el acceso desde una red virtual, puede establecer [reglas de firewall de red virtual](https://docs.microsoft.com/azure/azure-sql/database/network-access-controls-overview#virtual-network-firewall-rules) y establecer las [etiquetas de servicio que necesita el agente de Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/agents/azure-monitor-agent-overview#networking).  [En este artículo](https://docs.microsoft.com/azure/azure-sql/database/network-access-controls-overview#ip-vs-virtual-network-firewall-rules) se describen las diferencias entre estos dos tipos de reglas de firewall.

:::image type="content" source="media/sql-insights-enable/set-server-firewall.png" alt-text="Establecer el firewall del servidor" lightbox="media/sql-insights-enable/set-server-firewall.png":::

:::image type="content" source="media/sql-insights-enable/firewall-settings.png" alt-text="Configuración del firewall." lightbox="media/sql-insights-enable/firewall-settings.png":::


### <a name="azure-sql-managed-instances"></a>Instancias administradas de Azure SQL 

Si la máquina virtual de supervisión va a estar en la misma red virtual que los recursos de Instancia administrada de SQL Database, consulte [Conexión dentro de la misma red virtual](https://docs.microsoft.com/azure/azure-sql/managed-instance/connect-application-instance#connect-inside-the-same-vnet). Si la máquina virtual de supervisión va a estar en una red virtual diferente a la de los recursos de Instancia administrada de SQL Database, consulte [Conexión dentro de una red virtual diferente](https://docs.microsoft.com/azure/azure-sql/managed-instance/connect-application-instance#connect-inside-a-different-vnet).


### <a name="azure-virtual-machine-and-azure-sql-virtual-machine"></a>Máquina virtual de Azure y máquina virtual de Azure SQL  
Si la máquina virtual de supervisión está en la misma red virtual que los recursos de máquina virtual de SQL, consulte [Conexión a SQL Server en una red virtual](https://docs.microsoft.com/azure/azure-sql/virtual-machines/windows/ways-to-connect-to-sql#connect-to-sql-server-within-a-virtual-network). Si la máquina virtual de supervisión va a estar en una red virtual diferente de la de los recursos de máquina virtual de SQL, consulte [Conexión a SQL Server a través de Internet](https://docs.microsoft.com/azure/azure-sql/virtual-machines/windows/ways-to-connect-to-sql#connect-to-sql-server-over-the-internet).

## <a name="store-monitoring-password-in-key-vault"></a>Almacenamiento de la contraseña de supervisión en Key Vault
Debe almacenar las contraseñas de conexión del usuario de SQL en un almacén de claves en lugar de escribirlas directamente en las cadenas de conexión del perfil de supervisión.

Cuando configure el perfil para la supervisión de SQL, necesitará uno de los siguientes permisos en el recurso de Key Vault que va a usar:

- Microsoft.Authorization/roleAssignments/write 
- Permisos de Microsoft.Authorization/roleAssignments/delete como Administrador de acceso de usuario o Propietario 

Se creará automáticamente una nueva directiva de acceso como parte de la creación del perfil de SQL Monitoring que usa el almacén de claves especificado. Use *Allow access from All networks* (Permitir el acceso desde todas las redes) en la configuración de redes de Key Vault.


## <a name="create-sql-monitoring-profile"></a>Creación de un perfil de SQL Monitoring
Para iniciar SQL Insights, seleccione **SQL (preview)** SQL (versión preliminar), en la sección **Insights** del menú de **Azure Monitor** en Azure Portal. Haga clic en **Crear nuevo perfil**. 

:::image type="content" source="media/sql-insights-enable/create-new-profile.png" alt-text="Crear nuevo perfil." lightbox="media/sql-insights-enable/create-new-profile.png":::

El perfil almacenará la información que desea recopilar de los sistemas SQL.  Tiene una configuración específica para: 

- Azure SQL Database 
- Instancias administradas de Azure SQL 
- SQL Server que se ejecuta en máquinas virtuales  

Por ejemplo, puede crear un perfil denominado *SQL Production* y otro llamado *SQL Staging* con diferentes configuraciones para la frecuencia de recopilación de datos, los datos que se van a recopilar y el área de trabajo a la que se van a enviar los datos. 

El perfil se almacena como un recurso de la [regla de recopilación de datos](../agents/data-collection-rule-overview.md) en la suscripción y el grupo de recursos que seleccione. Cada perfil necesita lo siguiente:

- Name (Nombre). No se puede editar una vez creado.
- Ubicación. Es una región de Azure.
- Área de trabajo de Log Analytics en la que se van a almacenar los datos de supervisión.
- Opciones de recopilación relacionadas con la frecuencia y el tipo de datos de SQL Monitoring que se van a recopilar.

> [!NOTE]
> La ubicación del perfil debe ser la misma ubicación que la del área de trabajo de Log Analytics a la que se van a enviar los datos de supervisión.


:::image type="content" source="media/sql-insights-enable/profile-details.png" alt-text="Detalles de perfil." lightbox="media/sql-insights-enable/profile-details.png":::

Haga clic en **Crear perfil de supervisión** una vez que haya especificado los detalles del perfil de supervisión. Puede que el perfil tarde hasta un minuto en implementarse.  Si no ve el nuevo perfil que aparece en el cuadro combinado **Perfil de supervisión**, haga clic en el botón actualizar y debería aparecer una vez completada la implementación.  Una vez que haya seleccionado el nuevo perfil, seleccione la pestaña **Administrar perfil** para agregar una máquina de supervisión que se asociará con el perfil.

### <a name="add-monitoring-machine"></a>Incorporación de máquina de supervisión
Seleccione **Add monitoring machine** (Agregar máquina de supervisión) para abrir un panel de contexto con el fin de elegir la máquina virtual que desea configurar para supervisar las instancias de SQL y proporcionar las cadenas de conexión.

Seleccione la suscripción y el nombre de la máquina virtual de supervisión. Si usa Key Vault para almacenar la contraseña del usuario de supervisión, seleccione los recursos del almacén de claves con estos secretos y escriba la dirección URL y el nombre de secreto que se usarán en las cadenas de conexión. Consulte la sección siguiente para más información sobre cómo identificar la cadena de conexión para las diferentes implementaciones de SQL.


:::image type="content" source="media/sql-insights-enable/add-monitoring-machine.png" alt-text="Incorporación de máquina de supervisión." lightbox="media/sql-insights-enable/add-monitoring-machine.png":::


### <a name="add-connection-strings"></a>Incorporación de cadenas de conexión 
La cadena de conexión especifica el nombre de usuario que SQL Insights debe usar al iniciar sesión en SQL para ejecutar las vistas de administración dinámica. Si utiliza un almacén de claves para almacenar la contraseña del usuario de supervisión, proporcione la dirección URL y el nombre del secreto que se va a usar. 

Las cadenas de conexión varían según el tipo de recurso de SQL:

#### <a name="azure-sql-databases"></a>Azure SQL Database 
Especifique la cadena de conexión de esta forma:

```
sqlAzureConnections": [ 
   "Server=mysqlserver.database.windows.net;Port=1433;Database=mydatabase;User Id=$username;Password=$password;" 
}
```

Obtenga los detalles del elemento de menú **Cadenas de conexión** para la base de datos.

:::image type="content" source="media/sql-insights-enable/connection-string-sql-database.png" alt-text="Cadena de conexión de SQL Database" lightbox="media/sql-insights-enable/connection-string-sql-database.png":::

Para supervisar una réplica secundaria legible, incluya el par clave-valor `ApplicationIntent=ReadOnly` en la cadena de conexión. SQL Insights admite la supervisión de una única réplica secundaria. Los datos recopilados se etiquetarán para reflejar la réplica principal o secundaria. 


#### <a name="azure-virtual-machines-running-sql-server"></a>Máquinas virtuales de Azure que ejecutan SQL Server 
Especifique la cadena de conexión de esta forma:

```
"sqlVmConnections": [ 
   "Server=MyServerIPAddress;Port=1433;User Id=$username;Password=$password;" 
] 
```

Si la máquina virtual de supervisión está en la misma red virtual, use la dirección IP privada del servidor.  De lo contrario, use la dirección IP pública. Si utiliza una máquina virtual de Azure SQL, puede ver qué puerto usar aquí en la página **Seguridad** del recurso.

:::image type="content" source="media/sql-insights-enable/sql-vm-security.png" alt-text="Seguridad de la máquina virtual de SQL" lightbox="media/sql-insights-enable/sql-vm-security.png":::


### <a name="azure-sql-managed-instances"></a>Instancias administradas de Azure SQL 
Especifique la cadena de conexión de esta forma:

```
"sqlManagedInstanceConnections": [ 
      "Server= mysqlserver.database.windows.net;Port=1433;User Id=$username;Password=$password;", 
    ] 
```
Obtenga los detalles del elemento de menú **Cadenas de conexión** para la instancia administrada.


:::image type="content" source="media/sql-insights-enable/connection-string-sql-managed-instance.png" alt-text="Cadena de conexión de SQL Managed Instance" lightbox="media/sql-insights-enable/connection-string-sql-managed-instance.png":::

Para supervisar una réplica secundaria legible, incluya el par clave-valor `ApplicationIntent=ReadOnly` en la cadena de conexión. SQL Insights admite la supervisión de una única réplica secundaria, y los datos recopilados se etiquetarán para reflejar la réplica principal o secundaria. 


## <a name="monitoring-profile-created"></a>Se creó un perfil de supervisión 

Seleccione **Add monitoring virtual machine** (Agregar máquina virtual de supervisión) para configurar la máquina virtual que recopilará datos de los recursos de SQL. No vuelva a la pestaña **Información general**. En unos instantes, la columna Estado cambiará al estado "En recopilación" y debería ver los datos de los recursos de SQL que ha elegido supervisar.

Si no ve los datos, consulte [Solución de problemas de SQL Insights](sql-insights-troubleshoot.md) para identificar el problema. 

:::image type="content" source="media/sql-insights-enable/profile-created.png" alt-text="Perfil creado" lightbox="media/sql-insights-enable/profile-created.png":::

> [!NOTE]
> Si necesita actualizar el perfil de supervisión o las cadenas de conexión en las máquinas virtuales de supervisión, puede hacerlo a través de la pestaña **Administrar perfil** de SQL Insights.  Una vez guardadas las actualizaciones, los cambios se aplicarán en unos cinco minutos.

## <a name="next-steps"></a>Pasos siguientes

- Consulte [Solución de problemas de SQL Insights](sql-insights-troubleshoot.md) si SQL Insights no funciona correctamente tras su habilitación.
