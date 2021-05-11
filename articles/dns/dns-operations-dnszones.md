---
title: Administración de zonas DNS en Azure DNS - PowerShell | Microsoft Docs
description: Puede administrar zonas DNS con Azure PowerShell. Este artículo describe cómo actualizar, eliminar y crear zonas DNS en Azure DNS.
services: dns
documentationcenter: na
author: rohinkoul
ms.service: dns
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/27/2021
ms.author: rohink
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 02c6518a1ccfaa678c42369ae22f67670aaf0566
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108203276"
---
# <a name="how-to-manage-dns-zones-using-powershell"></a>Cómo administrar zonas DNS con PowerShell

> [!div class="op_single_selector"]
> * [Portal](dns-operations-dnszones-portal.md)
> * [PowerShell](dns-operations-dnszones.md)
> * [CLI de Azure clásica](./dns-operations-dnszones-cli.md)
> * [CLI de Azure](dns-operations-dnszones-cli.md)

En este artículo se muestra cómo administrar sus zonas DNS mediante Azure PowerShell. También se pueden administrar las zonas DNS mediante la [CLI de Azure](dns-operations-dnszones-cli.md) multiplataforma o Azure Portal.

Esta guía trata específicamente con las zonas DNS públicas. Para obtener información acerca del uso de Azure PowerShell para administrar zonas privadas en Azure DNS, consulte [Get started with Azure DNS Private Zones using Azure CLI 2.0](private-dns-getstarted-powershell.md) (Introducción a Azure DNS Private Zones con Azure PowerShell).

[!INCLUDE [dns-create-zone-about](../../includes/dns-create-zone-about-include.md)]

[!INCLUDE [dns-powershell-setup](../../includes/dns-powershell-setup-include.md)]

## <a name="create-a-dns-zone"></a>Creación de una zona DNS

Una zona DNS se crea mediante el cmdlet `New-AzDnsZone` .

En el ejemplo siguiente, se crea una zona DNS llamada *contoso.com* en el grupo de recursos *MyDNSResourceGroup*:

```azurepowershell-interactive
New-AzDnsZone -Name contoso.com -ResourceGroupName MyDNSResourceGroup
```

En el ejemplo siguiente se muestra cómo crear una zona DNS con dos [etiquetas Azure Resource Manager](dns-zones-records.md#tags), *project = demo* y *env = test*:

```azurepowershell-interactive
New-AzDnsZone -Name contoso.com -ResourceGroupName MyDNSResourceGroup -Tag @{ project="demo"; env="test" }
```

## <a name="get-a-dns-zone"></a>Recuperación de una zona DNS

Para recuperar una zona DNS, use el cmdlet `Get-AzDnsZone` . Esta operación devuelve un objeto de la zona DNS correspondiente a una zona existente en DNS de Azure. El objeto contiene datos sobre la zona (por ejemplo, el número de conjuntos de registros), pero no contiene los conjuntos de registros (consulte `Get-AzDnsRecordSet`).

```azurepowershell-interactive
Get-AzDnsZone -Name contoso.com –ResourceGroupName MyDNSResourceGroup

Name                  : contoso.com
ResourceGroupName     : myresourcegroup
Etag                  : 00000003-0000-0000-8ec2-f4879750d201
Tags                  : {project, env}
NameServers           : {ns1-01.azure-dns.com., ns2-01.azure-dns.net., ns3-01.azure-dns.org.,
                        ns4-01.azure-dns.info.}
NumberOfRecordSets    : 2
MaxNumberOfRecordSets : 5000
```

## <a name="list-dns-zones"></a>Enumeración de zonas DNS

Si se omite el nombre de la zona de `Get-AzDnsZone`, puede enumerar todas las zonas en un grupo de recursos. Esta operación devuelve una matriz de objetos de la zona.

```azurepowershell-interactive
$zoneList = Get-AzDnsZone -ResourceGroupName MyDNSResourceGroup
$zoneList
```

Si se omite tanto el nombre de zona como el nombre del grupo de recursos de `Get-AzDnsZone`, puede enumerar todas las zonas de la suscripción de Azure.

```azurepowershell-interactive
$zoneList = Get-AzDnsZone
$zoneList
```

## <a name="update-a-dns-zone"></a>Actualización de una zona DNS

Los cambios en los recursos de una zona DNS se pueden realizar mediante `Set-AzDnsZone`. Este cmdlet no actualiza ninguno de los conjuntos de registros de DNS de la zona (consulte [Administración de registros DNS](dns-operations-recordsets.md)). Solo se utiliza para actualizar las propiedades de los recursos de la zona. Las propiedades de zona que se pueden escribir están actualmente limitadas a las ["etiquetas" Azure Resource Manager para el recurso de la zona](dns-zones-records.md#tags).

Utilice una de las dos opciones siguientes para actualizar una zona DNS:

### <a name="specify-the-zone-using-the-zone-name-and-resource-group"></a>Especificar la zona con la utilización del nombre de zona y el grupo de recursos

Este método reemplaza las etiquetas de zona existentes con los valores especificados.

```azurepowershell-interactive
Set-AzDnsZone -Name contoso.com -ResourceGroupName MyDNSResourceGroup -Tag @{ project="demo"; env="test" }
```

### <a name="specify-the-zone-using-a-zone-object"></a>Especificar la zona utilizando un objeto $zone

Este método recupera el objeto de zona existente, modifica las etiquetas y, a continuación, confirma los cambios. De este modo, se pueden conservar las etiquetas existentes.

```azurepowershell-interactive
# Get the zone object
$zone = Get-AzDnsZone -Name contoso.com -ResourceGroupName MyDNSResourceGroup

# Remove an existing tag
$zone.Tags.Remove("project")

# Add a new tag
$zone.Tags.Add("status","approved")

# Commit changes
Set-AzDnsZone -Zone $zone
```

Al usar `Set-AzDnsZone` con un objeto $zone, se usan las [comprobaciones de ETag](dns-zones-records.md#etags) para garantizar que no se sobrescriben los cambios simultáneos. Puede usar el modificador `-Overwrite` opcional para suprimir estas comprobaciones.

## <a name="delete-a-dns-zone"></a>Eliminación de una zona DNS

Las zonas DNS se pueden eliminar mediante el cmdlet `Remove-AzDnsZone`.

> [!NOTE]
> Si se elimina una zona DNS, también se eliminan todos los registros DNS dentro de la zona. No se puede deshacer esta operación. Si la zona DNS está en uso, los servicios con la zona provocarán un error cuando se elimina la zona.
>
>Para protegerse contra la eliminación accidental de zona, consulte [Proteger registros y zonas DNS](dns-protect-zones-recordsets.md).


Utilice una de las dos opciones siguientes para eliminar una zona DNS:

### <a name="specify-the-zone-using-the-zone-name-and-resource-group-name"></a>Especificar la zona con la utilización del nombre de zona y el nombre del grupo de recursos

```azurepowershell-interactive
Remove-AzDnsZone -Name contoso.com -ResourceGroupName MyDNSResourceGroup
```

### <a name="specify-the-zone-using-a-zone-object"></a>Especificar la zona utilizando un objeto $zone

Puede especificar la zona que desee eliminar mediante un objeto `$zone` devuelto por `Get-AzDnsZone`.

```azurepowershell-interactive
$zone = Get-AzDnsZone -Name contoso.com -ResourceGroupName MyDNSResourceGroup
Remove-AzDnsZone -Zone $zone
```

También se puede canalizar el objeto de la zona en lugar de pasarlo como parámetro:

```azurepowershell-interactive
Get-AzDnsZone -Name contoso.com -ResourceGroupName MyDNSResourceGroup | Remove-AzDnsZone

```

Como con `Set-AzDnsZone`, especificar la zona usando un objeto `$zone` permite realizar las comprobaciones de ETag para asegurarse de que no se eliminen los cambios simultáneos. Use el modificador `-Overwrite` para suprimir estas comprobaciones.

## <a name="confirmation-prompts"></a>Mensajes de confirmación

Los cmdlets `New-AzDnsZone`, `Set-AzDnsZone` y `Remove-AzDnsZone` todos admiten mensajes de confirmación.

Ambos `New-AzDnsZone` y `Set-AzDnsZone` piden confirmación si la variable de preferencia de PowerShell `$ConfirmPreference` tiene un valor de `Medium` o inferior. Dado que la eliminación de una zona DNS puede provocar condiciones no deseadas, el cmdlet `Remove-AzDnsZone` pide confirmación si la variable `$ConfirmPreference` de PowerShell tiene cualquier valor distinto de `None`.

Puesto que el valor predeterminado para `$ConfirmPreference` es `High`, solo `Remove-AzDnsZone` pide confirmación de forma predeterminada.

Puede invalidar el valor actual de `$ConfirmPreference` mediante el parámetro `-Confirm`. Si especifica `-Confirm` o `-Confirm:$True`, el cmdlet solicita confirmación antes de ejecutarse. Si especifica `-Confirm:$False`, el cmdlet no le pide la confirmación.

Para más información sobre `-Confirm` y `$ConfirmPreference`, consulte [About Preference Variables](/powershell/module/microsoft.powershell.core/about/about_preference_variables) (Acerca de las variables de preferencias).

## <a name="next-steps"></a>Pasos siguientes

En esta guía se explica cómo [administrar conjuntos de registros y registros](dns-operations-recordsets.md) en la zona DNS.
<br>
Aprenda a cómo [delegar el dominio a Azure DNS](dns-domain-delegation.md).
<br>
Revise la [documentación de referencia de PowerShell para Azure DNS](/powershell/module/Az.dns).