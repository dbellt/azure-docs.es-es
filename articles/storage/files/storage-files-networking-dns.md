---
title: Configuración del reenvío de DNS para Azure Files | Microsoft Docs
description: Obtenga información sobre cómo configurar el reenvío de DNS para Azure Files.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 3/19/2020
ms.author: rogarana
ms.subservice: files
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 8dc6ac712110210414f66446827569f51f396c1f
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110669813"
---
# <a name="configuring-dns-forwarding-for-azure-files"></a>Configuración del reenvío de DNS para Azure Files
Azure Files le permite crear puntos de conexión privados para las cuentas de almacenamiento que contienen los recursos compartidos de archivos. Aunque son útiles para muchas aplicaciones diferentes, los puntos de conexión privados lo son especialmente para conectarse a los recursos compartidos de archivos de Azure desde la red local mediante una conexión VPN o ExpressRoute con emparejamiento privado. 

Para que las conexiones a la cuenta de almacenamiento pasen por el túnel de red, el nombre de dominio completo (FQDN) de la cuenta de almacenamiento debe resolverse en la dirección IP privada del punto de conexión privado. Para ello, debe reenviar el sufijo del punto de conexión de almacenamiento (`core.windows.net` para las regiones de la nube pública) al servicio DNS privado de Azure accesible desde la red virtual. En esta guía se muestra cómo configurar el reenvío de DNS para que se resuelva correctamente en la dirección IP del punto de conexión privado de la cuenta de almacenamiento.

Se recomienda encarecidamente leer [Planeamiento de una implementación de Azure Files](storage-files-planning.md) y [Consideraciones de redes para Azure Files](storage-files-networking-overview.md) antes de seguir los pasos que se describen en este artículo.

## <a name="overview"></a>Información general
Azure Files proporciona dos tipos principales de puntos de conexión para el acceso a los recursos compartidos de archivos de Azure: 
- Puntos de conexión públicos, que tienen una dirección IP pública y a los que se puede acceder desde cualquier parte del mundo.
- Puntos de conexión privados, que existen dentro de una red virtual y tienen una dirección IP privada en el espacio de direcciones de esa red virtual.

Los puntos de conexión públicos y privados existen en la cuenta de almacenamiento de Azure. Una cuenta de almacenamiento es una construcción de administración que representa un grupo compartido de almacenamiento en el que puede implementar varios recursos compartidos de archivos u otros recursos de almacenamiento, como contenedores de blobs o colas.

Cada cuenta de almacenamiento tiene un nombre de dominio completo (FQDN). En el caso de las regiones de la nube pública, este FQDN sigue el patrón `storageaccount.file.core.windows.net`, donde `storageaccount` es el nombre de la cuenta de almacenamiento. Cuando se realizan solicitudes con este nombre, como montar el recurso compartido en la estación de trabajo mediante SMB, el sistema operativo hace una búsqueda DNS para resolver el nombre de dominio completo en una dirección IP que pueda usar para enviar las solicitudes de SMB.

De forma predeterminada, `storageaccount.file.core.windows.net` se resuelve en la dirección IP del punto de conexión público. El punto de conexión público de una cuenta de almacenamiento se hospeda en un clúster de almacenamiento de Azure que hospeda muchos otros puntos de conexión públicos de las cuentas de almacenamiento. Cuando se crea un punto de conexión privado, una zona DNS privada se vincula a la red virtual a la que se agregó, con una asignación de registros CNAME `storageaccount.file.core.windows.net` a una entrada de registros D para la dirección IP privada del punto de conexión privado de la cuenta de almacenamiento. Esto le permite usar el FQDN `storageaccount.file.core.windows.net` dentro de la red virtual y hacer que se resuelva en la dirección IP del punto de conexión privado.

Dado que nuestro objetivo final es acceder a los recursos compartidos de archivos de Azure hospedados en la cuenta de almacenamiento desde el entorno local mediante un túnel de red, como una conexión VPN o ExpressRoute, debe configurar los servidores DNS locales para que reenvíen las solicitudes realizadas al servicio Azure Files al servicio DNS privado de Azure. Para ello, debe configurar el *reenvío condicional* de `*.core.windows.net` (o el sufijo del punto de conexión de almacenamiento adecuado para las nubes nacionales de EE. UU., Alemania o China) en un servidor DNS hospedado en la red virtual de Azure. A continuación, este servidor DNS reenviará de forma recursiva la solicitud al servicio DNS privado de Azure, que resolverá el nombre de dominio completo de la cuenta de almacenamiento en la dirección IP privada adecuada.

Para configurar el reenvío de DNS para Azure Files, es necesario ejecutar una máquina virtual para hospedar un servidor DNS para reenviar las solicitudes; sin embargo, este es un paso puntual para todos los recursos compartidos de archivos de Azure hospedados en la red virtual. Además, no es un requisito exclusivo de Azure Files: cualquier servicio de Azure que admita puntos de conexión privados a los que quiera acceder desde el entorno local puede usar el reenvío de DNS que se va a configurar en esta guía: Azure Blob Storage, SQL Azure, Cosmos DB, etc. 

En esta guía se muestran los pasos para configurar el reenvío de DNS para el punto de conexión de almacenamiento de Azure, así que, además de Azure Files, las solicitudes de resolución de nombres DNS para todos los demás servicios de almacenamiento de Azure (Azure Blob Storage, Azure Table Storage, Azure Queue Storage, etc.) se reenviarán al servicio DNS privado de Azure. Si así lo desea, también se pueden agregar puntos de conexión adicionales para otros servicios de Azure. También se configurará el reenvío de DNS a los servidores DNS locales, lo que permitirá que los recursos en la nube de la red virtual (por ejemplo, un servidor DFS-N) resuelvan los nombres de las máquinas locales. 

## <a name="prerequisites"></a>Prerrequisitos
Para poder configurar el reenvío de DNS a Azure Files, debe haber completado los pasos siguientes:

- Una cuenta de almacenamiento que contenga un recurso compartido de archivos de Azure que le gustaría montar. Para aprender a crear una cuenta de almacenamiento y un recurso compartido de archivos de Azure, consulte [Creación de un recurso compartido de archivos de Azure](storage-how-to-create-file-share.md).
- Un punto de conexión privado para la cuenta de almacenamiento. Para información sobre cómo crear un punto de conexión privado para Azure Files, consulte [Creación de un punto de conexión privado](storage-files-networking-endpoints.md#create-a-private-endpoint).
- La [versión más reciente](/powershell/azure/install-az-ps) del módulo de Azure PowerShell.

> [!Important]  
> En esta guía se da por supuesto que está usando el servidor DNS en Windows Server en el entorno local. Todos los pasos descritos en esta guía son posibles con cualquier servidor DNS, no solo con el de Windows.

## <a name="manually-configuring-dns-forwarding"></a>Configuración manual del reenvío de DNS
Si ya tiene colocados servidores DNS en la red virtual de Azure, o si simplemente prefiere implementar sus propias máquinas virtuales como servidores DNS por cualquier metodología que use la organización, puede configurar DNS manualmente con los cmdlets de PowerShell integrados del servidor DNS.

En los servidores DNS locales, cree un reenviador condicional mediante `Add-DnsServerConditionalForwarderZone`. Este reenviador condicional debe implementarse en todos los servidores DNS locales para que sea eficaz a la hora de reenviar correctamente el tráfico a Azure. No olvide reemplazar `<azure-dns-server-ip>` por las direcciones IP adecuadas para su entorno.

```powershell
$vnetDnsServers = "<azure-dns-server-ip>", "<azure-dns-server-ip>"

$storageAccountEndpoint = Get-AzContext | `
    Select-Object -ExpandProperty Environment | `
    Select-Object -ExpandProperty StorageEndpointSuffix

Add-DnsServerConditionalForwarderZone `
        -Name $storageAccountEndpoint `
        -MasterServers $vnetDnsServers
```

En los servidores DNS de la red virtual de Azure, también deberá colocar un reenviador de forma que las solicitudes de la zona DNS de la cuenta de almacenamiento se dirijan al servicio DNS privado de Azure, encabezado por la dirección IP reservada `168.63.129.16`. (Si ejecuta los comandos en una sesión de PowerShell diferente, recuerde rellenar `$storageAccountEndpoint`).

```powershell
Add-DnsServerConditionalForwarderZone `
        -Name $storageAccountEndpoint `
        -MasterServers "168.63.129.16"
```

## <a name="using-the-azure-files-hybrid-module-to-configure-dns-forwarding"></a>Uso del módulo híbrido de Azure Files para configurar el reenvío de DNS
Para que la configuración del reenvío de DNS sea lo más sencilla posible, hemos proporcionado automatización en el módulo híbrido de Azure Files. Los cmdlets que se proporcionan para manipular DNS en este módulo le ayudarán a implementar servidores DNS en la red virtual de Azure y a actualizar los servidores DNS locales para el reenvío a estos. 

Si no ha usado nunca el módulo híbrido de Azure Files, primero debe instalarlo en su estación de trabajo. Descargue la [última versión](https://github.com/Azure-Samples/azure-files-samples/releases) del módulo híbrido de PowerShell de Azure Files:

```PowerShell
# Unzip the downloaded file
Expand-Archive -Path AzFilesHybrid.zip

# Change the execution policy to unblock importing AzFilesHybrid.psm1 module
Set-ExecutionPolicy -ExecutionPolicy Unrestricted

# Navigate to where AzFilesHybrid is unzipped and stored and run to copy the files into your path
.\AzFilesHybrid\CopyToPSPath.ps1 

# Import AzFilesHybrid module
Import-Module -Name AzFilesHybrid
```

La implementación de la solución de reenvío de DNS consta de dos pasos: la creación de un conjunto de reglas de reenvío de DNS, que define a qué servicios de Azure quiere reenviar las solicitudes, y la implementación real de los reenviadores DNS. 

En el ejemplo siguiente se reenvían solicitudes a la cuenta de almacenamiento, incluidas solicitudes a Azure Files, Azure Blob Storage, Azure Table Storage y Azure Queue Storage. Si lo desea, puede agregar a la regla el reenvío para servicios de Azure adicionales mediante el parámetro `-AzureEndpoints` del cmdlet `New-AzDnsForwardingRuleSet`. No olvide reemplazar `<virtual-network-resource-group>`, `<virtual-network-name>` y `<subnet-name>` por los valores correctos para su entorno.

```PowerShell
# Create a rule set, which defines the forwarding rules
$ruleSet = New-AzDnsForwardingRuleSet -AzureEndpoints StorageAccountEndpoint

# Deploy and configure DNS forwarders
New-AzDnsForwarder `
        -DnsForwardingRuleSet $ruleSet `
        -VirtualNetworkResourceGroupName "<virtual-network-resource-group>" `
        -VirtualNetworkName "<virtual-network-name>" `
        -VirtualNetworkSubnetName "<subnet-name>"
```

Además, puede que le resulte útil o necesario proporcionar varios parámetros adicionales:

| Nombre de parámetro | Tipo | Descripción |
|----------------|------|-------------|
| `DnsServerResourceGroupName` | `string` | De forma predeterminada, los servidores DNS se implementarán en el mismo grupo de recursos que la red virtual. Si no es lo que quiere, este parámetro le permite seleccionar un grupo de recursos alternativo en el que implementarlos. |
| `DnsForwarderRootName` | `string` | De forma predeterminada, los servidores DNS que se implementan en Azure tienen los nombres `DnsFwder-*`, donde el asterisco se rellena mediante un iterador. Este parámetro cambia la raíz de ese nombre (es decir, `DnsFwder`). |
| `VmTemporaryPassword` | `SecureString` | De forma predeterminada, se elige una contraseña aleatoria para la cuenta predeterminada temporal que tenga una máquina virtual antes de que se una a un dominio. Una vez unida a un dominio, la cuenta predeterminada está deshabilitada. |
| `DomainToJoin` | `string` | El dominio al que se unen las máquinas virtuales de DNS. De forma predeterminada, este dominio se elige en función del dominio del equipo en el que se ejecutan los cmdlets. |
| `DnsForwarderRedundancyCount` | `int` | El número de máquinas virtuales de DNS que se van a implementar para la red virtual. De forma predeterminada, `New-AzDnsForwarder` implementa dos servidores DNS en la red virtual de Azure, en un conjunto de disponibilidad, para garantizar la redundancia. Este número se puede modificar según sea necesario. |
| `OnPremDnsHostNames` | `HashSet<string>` | Una lista especificada manualmente de nombres de host DNS locales en la que crear reenviadores. Este parámetro es útil si no desea aplicar reenviadores en todos los servidores DNS locales, como cuando tiene un intervalo de clientes con nombres DNS especificados manualmente. |
| `Credential` | `PSCredential` | Credencial que se va a usar al actualizar los servidores DNS. Resulta útil cuando la cuenta de usuario con la que ha iniciado sesión no tiene permisos para modificar la configuración de DNS. |
| `SkipParentDomain` | `SwitchParameter` | De forma predeterminada, los reenviadores DNS se aplican al dominio de nivel superior que existe en el entorno. Por ejemplo, si `northamerica.corp.contoso.com` es un dominio secundario de `corp.contoso.com`, se creará el reenviador para los servidores DNS asociados a `corp.contoso.com`. Este parámetro hará que los reenviadores se creen en `northamerica.corp.contoso.com`. |

## <a name="confirm-dns-forwarders"></a>Confirmación de reenviadores DNS
Antes de realizar pruebas para ver si los reenviadores de DNS se han aplicado correctamente, se recomienda borrar la caché de DNS en la estación de trabajo local mediante `Clear-DnsClientCache`. Para comprobar si puede resolver correctamente el nombre de dominio completo de la cuenta de almacenamiento, use `Resolve-DnsName` o `nslookup`.

```powershell
# Replace storageaccount.file.core.windows.net with the appropriate FQDN for your storage account.
# Note the proper suffix (core.windows.net) depends on the cloud your deployed in.
Resolve-DnsName -Name storageaccount.file.core.windows.net
```

Si la resolución de nombres se realiza correctamente, verá que la dirección IP resuelta coincide con la dirección IP de la cuenta de almacenamiento.

```Output
Name                              Type   TTL   Section    NameHost
----                              ----   ---   -------    --------
storageaccount.file.core.windows. CNAME  29    Answer     csostoracct.privatelink.file.core.windows.net
net

Name       : storageaccount.privatelink.file.core.windows.net
QueryType  : A
TTL        : 1769
Section    : Answer
IP4Address : 192.168.0.4
```

Si ya ha configurado una conexión VPN o ExpressRoute, también puede usar `Test-NetConnection` para ver que se puede crear correctamente una conexión TCP a la cuenta de almacenamiento.

```PowerShell
Test-NetConnection -ComputerName storageaccount.file.core.windows.net -CommonTCPPort SMB
```

## <a name="see-also"></a>Consulte también
- [Planeamiento de una implementación de Azure Files](storage-files-planning.md)
- [Consideraciones de redes para Azure Files](storage-files-networking-overview.md)
- [Configuración de puntos de conexión de red de Azure Files](storage-files-networking-endpoints.md)
