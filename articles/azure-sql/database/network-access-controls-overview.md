---
title: Controles de acceso a la red
titleSuffix: Azure SQL Database & Azure Synapse Analytics
description: Información general sobre cómo administrar y controlar el acceso a la red para Azure SQL Database y Azure Synapse Analytics.
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: sqldbrb=3, devx-track-azurepowershell
ms.devlang: ''
ms.topic: conceptual
author: rohitnayakmsft
ms.author: rohitna
ms.reviewer: vanto
ms.date: 03/09/2020
ms.openlocfilehash: 1640cfae4925c1ef3699ea72778e392e4249db9b
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110666607"
---
# <a name="azure-sql-database-and-azure-synapse-analytics-network-access-controls"></a>Controles de acceso a la red de Azure SQL Database y Azure Synapse Analytics

Al crear un servidor SQL lógico en [Azure Portal](single-database-create-quickstart.md) para Azure SQL Database y Azure Synapse Analytics, el resultado es un punto de conexión público con el formato *nombreDelServidor.database.windows.net*.

Puede usar los siguientes controles de acceso a la red para permitir el acceso a una base de datos de forma selectiva mediante el punto de conexión público:

- Allow Azure Services (Permitir servicios de Azure): cuando se establece en ACTIVADO, otros recursos dentro de los límites de Azure, por ejemplo, una máquina virtual de Azure, pueden acceder a SQL Database.
- Reglas de firewall de IP: use esta característica para permitir explícitamente las conexiones desde una dirección IP específica, por ejemplo, desde máquinas locales.

También puede permitir el acceso privado a la base de datos desde [redes virtuales](../../virtual-network/virtual-networks-overview.md) a través de:

- Reglas de firewall de la red virtual: Use esta característica para permitir el tráfico desde una red virtual específica dentro de los límites de Azure
- Private Link: Use esta característica para crear un punto de conexión privado para un [servidor SQL lógico](logical-servers.md) dentro de una red virtual específica

> [!IMPORTANT]
> Este artículo *no* se aplica a **Instancia administrada de SQL**. Para obtener más información sobre la configuración de red, consulte [Conexión a Instancia administrada de Azure SQL](../managed-instance/connect-application-instance.md).

Vea el vídeo siguiente para obtener una explicación de alto nivel sobre estos controles de acceso y lo que hacen:

> [!VIDEO https://channel9.msdn.com/Shows/Data-Exposed/Data-Exposed--SQL-Database-Connectivity-Explained/player?WT.mc_id=dataexposed-c9-niner]

## <a name="allow-azure-services"></a>Allow Azure Services (Permitir servicios de Azure)

De manera predeterminada, durante la creación de un servidor SQL lógico en [Azure Portal](single-database-create-quickstart.md), esta opción se deja en **DESACTIVADO**. Esta opción aparece cuando se permite la conectividad con el punto de conexión de servicio público.

También puede cambiar esta opción desde el panel de firewall después de crear el servidor SQL lógico, como sigue.
  
![Captura de pantalla de la administración del firewall del servidor][2]

Cuando se establece en **ACTIVADO**, el servidor permite la comunicación de todos los recursos dentro de los límites de Azure, que pueden o no formar parte de la suscripción.

En muchos casos, el valor **ACTIVADO** es más permisivo de lo que desea la mayoría de los clientes. Es posible que desee establecer esta opción en **DESACTIVADO** y reemplazarla por reglas de firewall de IP más restrictivas o reglas de firewall de red virtual. 

Sin embargo, esto afecta a las siguientes características que se ejecutan en máquinas virtuales de Azure que no forman parte de la red virtual y, por tanto, se conectan a la base de datos a través de una dirección IP de Azure:

### <a name="import-export-service"></a>Import Export Service

El servicio de importación y exportación no funciona cuando **Permitir el acceso a servicios de Azure** se establece en **DESACTIVADO**. Sin embargo, puede solucionar el problema [al ejecutar manualmente sqlpackage.exe desde una máquina virtual de Azure o realizar la exportación](./database-import-export-azure-services-off.md) directamente en el código mediante la API de DACFx.

### <a name="data-sync"></a>Sincronización de datos

Para usar la característica de sincronización de datos con **Permitir el acceso a servicios de Azure** establecido en **DESACTIVADO**, debe crear entradas de reglas de firewall individuales para [agregar direcciones IP](firewall-create-server-level-portal-quickstart.md) de la **etiqueta de servicio SQL** para la región que hospeda la base de datos **central**.
Agregue estas reglas de firewall de nivel de servidor a los servidores que hospedan las bases de datos **central** y **miembro** (que pueden estar en regiones diferentes)

Use el siguiente script de PowerShell para generar las direcciones IP correspondientes a la etiqueta de servicio de SQL para la región Oeste de EE. UU.

```powershell
PS C:\>  $serviceTags = Get-AzNetworkServiceTag -Location eastus2
PS C:\>  $sql = $serviceTags.Values | Where-Object { $_.Name -eq "Sql.WestUS" }
PS C:\> $sql.Properties.AddressPrefixes.Count
70
PS C:\> $sql.Properties.AddressPrefixes
13.86.216.0/25
13.86.216.128/26
13.86.216.192/27
13.86.217.0/25
13.86.217.128/26
13.86.217.192/27
```

> [!TIP]
> Get-AzNetworkServiceTag devuelve el intervalo global de la etiqueta de servicio SQL a pesar de que se especificó el parámetro Location. Asegúrese de filtrar para usarlo en la región que hospeda la base de datos central que usa el grupo de sincronización.

Tenga en cuenta que la salida del script de PowerShell está en notación de enrutamiento de interdominios sin clases (CIDR). Esto se debe convertir a un formato de dirección IP inicial y final mediante [Get-IPrangeStartEnd.ps1](https://gallery.technet.microsoft.com/scriptcenter/Start-and-End-IP-addresses-bcccc3a9) similar al siguiente:

```powershell
PS C:\> Get-IPrangeStartEnd -ip 52.229.17.93 -cidr 26
start        end
-----        ---
52.229.17.64 52.229.17.127
```

Puede utilizar este script de PowerShell adicional para convertir todas las direcciones IP de CIDR al formato de dirección IP inicial y final.

```powershell
PS C:\>foreach( $i in $sql.Properties.AddressPrefixes) {$ip,$cidr= $i.split('/') ; Get-IPrangeStartEnd -ip $ip -cidr $cidr;}
start          end
-----          ---
13.86.216.0    13.86.216.127
13.86.216.128  13.86.216.191
13.86.216.192  13.86.216.223
```

Ahora puede agregarlas como reglas de firewall independientes y, a continuación, establecer la opción **Permitir que los servicios de Azure accedan al servidor** en OFF.

## <a name="ip-firewall-rules"></a>Reglas de firewall de IP

El firewall basado en IP es una característica del servidor SQL lógico en Azure, que impide todo acceso al servidor hasta que [agregue direcciones IP](firewall-create-server-level-portal-quickstart.md) explícitamente de las máquinas cliente.

## <a name="virtual-network-firewall-rules"></a>Reglas de firewall de red virtual

Además de las reglas de IP, el firewall del servidor permite definir *reglas de red virtual*.  
Para más información, consulte [Uso de reglas y puntos de conexión de servicio de red virtual para Azure SQL Database y SQL Data Warehouse](vnet-service-endpoint-rule-overview.md) o vea este vídeo:

> [!VIDEO https://channel9.msdn.com/Shows/Data-Exposed/Data-Exposed--Demo--Vnet-Firewall-Rules-for-SQL-Database/player?WT.mc_id=dataexposed-c9-niner]

### <a name="azure-networking-terminology"></a>Terminología de redes en Azure

Tenga en cuenta los siguientes términos de redes en Azure a medida que explora las reglas de firewall de red virtual

**Red virtual:** puede tener redes virtuales asociadas a la suscripción de Azure.

**Subred:** una red virtual contiene **subredes**. Cualquier máquina virtual (VM) de Azure que tenga se asignará a las subredes. Una subred puede contener varias máquinas virtuales u otros nodos de proceso. Los nodos de proceso que se encuentran fuera de la red virtual no pueden tener acceso a su red virtual a menos que configure la seguridad para que permita el acceso.

**Punto de conexión de servicio de red virtual:** Un [punto de conexión de servicio de red virtual](../../virtual-network/virtual-network-service-endpoints-overview.md) es una subred cuyos valores de propiedad incluyen uno o más nombres formales de tipo de servicio de Azure. En este artículo nos interesa el nombre de tipo de **Microsoft.Sql**, que hace referencia al servicio de Azure denominado SQL Database.

**Regla de red virtual:** Una regla de red virtual para el servidor es una subred que se muestra en la lista de control de acceso (ACL) del servidor. Para estar en la ACL de su base de datos en SQL Database, la subred debe contener el nombre de tipo **Microsoft.Sql**. Una regla de red virtual indica al servidor que acepte las comunicaciones procedentes de todos los nodos que están en la subred.

## <a name="ip-vs-virtual-network-firewall-rules"></a>Reglas de IP frente a Reglas de firewall de red virtual

El firewall de Azure SQL Database le permite especificar intervalos de direcciones IP desde los que se aceptan las comunicaciones en SQL Database. Este enfoque es preciso para las direcciones IP estables que están fuera de la red privada de Azure. Sin embargo, las máquinas virtuales de la red privada de Azure se configuran con direcciones IP *dinámicas*. Las direcciones IP dinámicas pueden cambiar cuando se reinicia la máquina virtual y, a su vez, invalidar la regla de firewall basada en IP. Sería una locura especificar una dirección IP dinámica en una regla de firewall, en un entorno de producción.

Para superar esta limitación, puede obtener una dirección IP *estática* para la máquina virtual. Para obtener más información, consulte [Creación de una máquina virtual con una dirección IP pública estática mediante Azure Portal](../../virtual-network/virtual-network-deploy-static-pip-arm-portal.md). Sin embargo, el enfoque de IP estática puede resultar difícil de administrar, y es costoso si se realiza a escala.

Las reglas de red virtual son alternativas más fáciles de establecer y su acceso, de administrar desde una subred específica que contenga las máquinas virtuales.

> [!NOTE]
> Aún no se puede disponer de SQL Database en una subred. Si su servidor fuera un nodo de una subred de la red virtual, todos los nodos de la red virtual podrían comunicarse con su instancia de SQL Database. En este caso, las máquinas virtuales podrían comunicarse con SQL Database sin necesitar ninguna regla IP ni regla de red virtual.

## <a name="private-link"></a>Private Link

Private Link le permite conectarse a un servidor mediante un **punto de conexión privado**. Un punto de conexión privado es una dirección IP privada dentro de una [red virtual](../../virtual-network/virtual-networks-overview.md) y una subred específicas.

## <a name="next-steps"></a>Pasos siguientes

- Para consultar un inicio rápido sobre cómo crear una regla de firewall por IP de nivel de servidor, consulte [Creación de una base de datos en SQL Database](single-database-create-quickstart.md).

- Para un inicio rápido sobre la creación de una regla de firewall de red virtual de nivel de servidor, consulte [Puntos de conexión de servicio de red virtual y reglas para Azure SQL Database](vnet-service-endpoint-rule-overview.md).

- Si desea obtener ayuda para conectarse a una base de SQL Database desde aplicaciones de código abierto o de terceros, consulte [Ejemplos de código de inicio rápido de cliente para SQL Database](/previous-versions/azure/ee336282(v=azure.100)).

- Para información sobre los puertos adicionales que puede necesitar abrir, vea la sección **SQL Database: fuera frente a dentro**: de la sección [Puertos más allá de 1433 para ADO.NET 4.5 y SQL Database](adonet-v12-develop-direct-route-ports.md)

- Para información general sobre la connectividad de Azure SQL Database, consulte [Arquitectura de conectividad de Azure SQL](connectivity-architecture.md).

- Para obtener información general sobre la seguridad de Azure SQL Database, vea [Protección de bases de datos SQL](security-overview.md).

<!--Image references-->
[1]: media/quickstart-create-single-database/new-server2.png
[2]: media/quickstart-create-single-database/manage-server-firewall.png
