---
title: Interoperabilidad de características con FCI y DNN de SQL Server
description: 'Información acerca de las atenciones adicionales que se requieren cuando se trabaja con ciertas características de SQL Server y un recurso de nombre de red distribuida (DNN) con una instancia de clúster de conmutación por error de SQL Server en máquinas virtuales de Azure. '
services: virtual-machines
documentationCenter: na
author: MashaMSFT
editor: monicar
tags: azure-service-management
ms.service: virtual-machines-sql
ms.subservice: hadr
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 06/02/2020
ms.author: mathoma
ms.openlocfilehash: 03dcd9f4750e9bf3e6ec5568255b06ccb8316a0b
ms.sourcegitcommit: ff1aa951f5d81381811246ac2380bcddc7e0c2b0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111569465"
---
# <a name="feature-interoperability-with-sql-server-fci--dnn"></a>Interoperabilidad de características con FCI y DNN de SQL Server
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Existen determinadas características de SQL Server que dependen de un nombre de red virtual (VNN) codificado de forma rígida. Por lo tanto, al utilizar el recurso de nombre de red distribuida (DNN) con la instancia de clúster de conmutación por error y SQL Server en máquinas virtuales de Azure, se requieren algunas atenciones adicionales. 

En este artículo aprenderá a configurar el alias de red cuando utilice el recurso DNN, así como qué características de SQL Server requieren que se le preste una atención adicional.

## <a name="create-network-alias-fci"></a>Creación de alias de red (FCI)

Algunos componentes del lado servidor dependen de un valor de VNN codificado de forma rígida y requieren un alias de red que asigne el VNN al nombre DNS de DNN para que funcione correctamente. Siga los pasos descritos para la [creación de un alias de servidor](/sql/database-engine/configure-windows/create-or-delete-a-server-alias-for-use-by-a-client) y cree un alias que asigne el VNN al nombre DNS de DNN. 

En el caso de una instancia predeterminada, puede asignar el VNN al nombre DNS de DNN directamente, de modo que VNN = nombre DNS de DNN.
Por ejemplo, si el nombre VNN es `FCI1`, el nombre de instancia es `MSSQLSERVER` y el DNN es `FCI1DNN` (clientes que se han conectado previamente a `FCI` y ahora se conectan a `FCI1DNN`), asigne el VNN `FCI1` al DNN `FCI1DNN`. 

En el caso de una instancia con nombre, se debe realizar la asignación de alias de red para la instancia completa, como `VNN\Instance` = `DNN\Instance`. Por ejemplo, si el nombre VNN es `FCI1`, el nombre de instancia es `instA` y el DNN es `FCI1DNN` (clientes que se han conectado previamente a `FCI1\instA` y ahora se conectan a `FCI1DNN\instaA`), asigne el VNN `FCI1\instaA` al DNN `FCI1DNN\instaA`. 



## <a name="client-drivers"></a>Controladores cliente

Para los controladores ODBC, OLEDB, ADO.NET, JDBC, PHP y node.js, los usuarios deben especificar explícitamente el nombre DNS de DNN como nombre de servidor en la cadena de conexión. Para garantizar una conectividad rápida tras la conmutación por error, agregue `MultiSubnetFailover=True` a la cadena de conexión si el cliente SQL lo admite. 

## <a name="tools"></a>Herramientas

Los usuarios de [SQL Server Management Studio](/sql/ssms/sql-server-management-studio-ssms), [sqlcmd](/sql/tools/sqlcmd-utility), [Azure Data Studio](/sql/azure-data-studio/what-is) y [SQL Server Data Tools](/sql/ssdt/sql-server-data-tools) deben especificar explícitamente el nombre DNS de DNN como nombre de servidor en la cadena de conexión. 

## <a name="availability-groups-and-fci"></a>Grupos de disponibilidad y FCI

Puede configurar un grupo de disponibilidad Always On mediante una instancia de clúster de conmutación por error como una de las réplicas. En esta configuración, la dirección URL del punto de conexión de creación de reflejo para la réplica de FCI debe utilizar el DNN de la FCI. Del mismo modo, si la FCI se usa como réplica de solo lectura, el enrutamiento de solo lectura a la réplica de FCI debe utilizar el DNN de la FCI. 

El formato del punto de conexión de creación de reflejo es: `ENDPOINT_URL = 'TCP://<DNN DNS name>:<mirroring endpoint port>'`. 

Por ejemplo, si el nombre DNS de DNN es `dnnlsnr` y `5022` es el puerto del punto de conexión de creación de reflejo de la FCI, el fragmento de código de Transact-SQL (T-SQL) para crear la dirección URL del punto de conexión tiene el siguiente aspecto: 

```sql
ENDPOINT_URL = 'TCP://dnnlsnr:5022'
```

Del mismo modo, el formato de la dirección URL de enrutamiento de solo lectura es: `TCP://<DNN DNS name>:<SQL Server instance port>`. 

Por ejemplo, si el nombre DNS del DNN es `dnnlsnr` y `1444` es el puerto que usa la FCI de SQL Server de destino de solo lectura, el fragmento de código de T-SQL para crear la dirección URL de enrutamiento de solo lectura tiene el siguiente aspecto: 

```sql
READ_ONLY_ROUTING_URL = 'TCP://dnnlsnr:1444'
```

Puede omitir el puerto en la dirección URL si es el puerto predeterminado 1433. Para una instancia con nombre, configure un puerto estático para la instancia con nombre y especifíquelo en la dirección URL de enrutamiento de solo lectura.  

## <a name="replication"></a>Replicación

La replicación tiene tres componentes: editor, distribuidor, suscriptor. Cualquiera de estos componentes puede ser una instancia de clúster de conmutación por error. Dado que el VNN de la FCI se utiliza mucho en la configuración de la replicación, tanto de forma explícita como implícita, es posible que sea necesario un alias de red que asigne el VNN al DNN para que la replicación funcione. 

Siga utilizando el nombre VNN como nombre de la FCI dentro de la replicación, pero cree un alias de red en las siguientes situaciones remotas *antes de configurar la replicación*:

| **Componente de replicación (FCI con DNN)** | **Componente remoto** | **Asignación de alias de red** | **Servidor con asignación de red**| 
|---------|---------|---------|-------- | 
|Publicador | Distribuidor. | VNN del editor para el DNN del editor| Distribuidor.| 
|Distribuidor.|Suscriptor |VNN del distribuidor para el DNN del distribuidor| Suscriptor | 
|Distribuidor.|Publicador | VNN del distribuidor para el DNN del distribuidor | Publicador| 
|Suscriptor| Distribuidor.| VNN del suscriptor para el DNN del suscriptor | Distribuidor.| 

Por ejemplo, suponga que tiene un editor que esté configurado como una FCI mediante el DNN en una topología de replicación y el distribuidor es remoto. En este caso, cree un alias de red en el servidor de distribuidor para asignar el VNN del editor al DNN del editor: 

:::image type="content" source="media/failover-cluster-instance-dnn-interoperability/alias-in-configuration-manager.png" alt-text="Configure el nombre DNS de DNN como alias de red mediante el Administrador de configuración de SQL Server." :::

Utilice el nombre de instancia completo para una instancia con nombre, como en el ejemplo de imagen siguiente: 

:::image type="content" source="media/failover-cluster-instance-dnn-interoperability/alias-named-instance-configuration-manager.png" alt-text="Utilice el nombre de instancia completo al configurar un alias de red para una instancia con nombre." :::

## <a name="database-mirroring"></a>Creación de reflejo de la base de datos

Puede configurar la creación de reflejo de la base de datos con una FCI como asociado de creación de reflejo de la base de datos. Configúrela mediante [Transact-SQL (T-SQL)](/sql/database-engine/database-mirroring/example-setting-up-database-mirroring-using-windows-authentication-transact-sql) en lugar de la interfaz gráfica de usuario de SQL Server Management Studio. El uso de T-SQL garantizará que el punto de conexión de creación de reflejo de la base de datos se cree con DNN en lugar de VNN. 

Por ejemplo, si el nombre DNS de DNN es `dnnlsnr` y el punto de conexión de creación de reflejo de la base de datos es 7022, el siguiente fragmento de código de T-SQL configura el asociado de creación de reflejo de la base de datos: 

```sql
ALTER DATABASE AdventureWorks
    SET PARTNER =
    'TCP://dnnlsnr:7022'
GO 
```

Para el acceso de cliente, la propiedad **Socio de conmutación por error** puede administrar la conmutación por error de la creación de reflejo de la base de datos, pero no la conmutación por error de la FCI. 

## <a name="msdtc"></a>MSDTC

La FCI puede participar en transacciones distribuidas coordinadas mediante el Coordinador de transacciones distribuidas de Microsoft (MSDTC). Aunque tanto el MSDTC en clúster como el MSDTC local son compatibles con el DNN de la FCI, en Azure, sigue siendo necesario un equilibrador de carga para el MSDTC en clúster. El DNN definido en la FCI no reemplaza el requisito de Azure Load Balancer del MSDTC en clúster en Azure. 

## <a name="filestream"></a>Secuencia de archivos

Aunque FileStream es compatible con una base de datos de una FCI, no se admite el acceso a FileStream o a FileTable mediante las API del sistema de archivos con DNN. 

## <a name="linked-servers"></a>Servidores vinculados

Se admite el uso de un servidor vinculado con un DNN de FCI. Use el DNN directamente para configurar un servidor vinculado o un alias de red para asignar el VNN al DNN. 


Por ejemplo, para crear un servidor vinculado con el nombre DNS del DNN `dnnlsnr` para la instancia con nombre `insta1`, utilice el siguiente comando de Transact-SQL (T-SQL):

```sql
USE [master]   
GO   

EXEC master.dbo.sp_addlinkedserver    
    @server = N'dnnlsnr\inst1',    
    @srvproduct=N'SQL Server' ;   
GO 
```

Como alternativa, puede crear el servidor vinculado con el nombre de red virtual (VNN) en su lugar, pero tendrá que definir un alias de red para asignar el VNN al DNN. 

Por ejemplo, para el nombre de instancia `insta1`, el nombre VNN `vnnname` y el nombre DNN `dnnlsnr`, utilice el siguiente comando de Transact-SQL (T-SQL) para crear un servidor vinculado mediante el VNN:

```sql
USE [master]
GO   

EXEC master.dbo.sp_addlinkedserver   
    @server = N'vnnname\inst1',    
    @srvproduct=N'SQL Server' ;   
GO 

```

A continuación, cree un alias de red para asignar `vnnname\insta1` a `dnnlsnr\insta1`. 



## <a name="frequently-asked-questions"></a>Preguntas más frecuentes


- ¿Qué versión de SQL Server proporciona compatibilidad con el DNN? 

   SQL Server 2019 CU2 y versiones posteriores.

- ¿Cuál es el tiempo de conmutación por error esperado cuando se utiliza el DNN?

   En el caso del DNN, el tiempo de conmutación por error será solo el tiempo de conmutación por error de la FCI, sin más tiempo adicional (como el tiempo de sondeo cuando se utiliza Azure Load Balancer).

- ¿Hay algún requisito de versión para que los clientes de SQL admitan DNN con OLEDB y ODBC?

   Se recomienda compatibilidad con la cadena de conexión `MultiSubnetFailover=True` para el DNN. Está disponible a partir de SQL Server 2012 (11.x).

- ¿Hay algún cambio de configuración de SQL Server que necesite para utilizar el DNN? 

   SQL Server no requiere ningún cambio en la configuración para utilizar el DNN, pero es posible que algunas características de SQL Server requieran más atención. 

- ¿Admite el DNN clústeres de varias subredes?

   Sí. El clúster enlaza el DNN en DNS con las direcciones IP físicas de todos los nodos del clúster, independientemente de la subred. El cliente SQL prueba todas las direcciones IP del nombre DNS independientemente de la subred. 



## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, consulte:

- [Clúster de conmutación por error de Windows Server con SQL Server en máquinas virtuales de Azure](hadr-windows-server-failover-cluster-overview.md)
- [Instancias de clúster de conmutación por error con SQL Server en Azure Virtual Machines](failover-cluster-instance-overview.md)
- [Información general de las instancias de clúster de conmutación por error](/sql/sql-server/failover-clusters/windows/always-on-failover-cluster-instances-sql-server)
- [Configuración de alta disponibilidad y recuperación ante desastres para SQL Server en máquinas virtuales de Azure](hadr-cluster-best-practices.md)

