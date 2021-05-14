---
title: Administración de registros DNS en Azure DNS mediante Azure PowerShell | Microsoft Docs
description: Administración de conjuntos de registros y registros DNS en DNS de Azure al hospedar dominios en DNS de Azure. Todos los comandos de PowerShell para operaciones en conjuntos de registros y registros.
services: dns
documentationcenter: na
author: rohinkoul
ms.service: dns
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.custom: H1Hack27Feb2017
ms.workload: infrastructure-services
ms.date: 04/28/2021
ms.author: rohink
ms.openlocfilehash: 1b3e4df12a7b8a08d3f7edc8e63c9b6225a3e7bc
ms.sourcegitcommit: 49bd8e68bd1aff789766c24b91f957f6b4bf5a9b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108228089"
---
# <a name="manage-dns-records-and-recordsets-in-azure-dns-using-azure-powershell"></a>Administración de conjuntos de registros y registros de DNS en Azure DNS mediante Azure PowerShell

> [!div class="op_single_selector"]
> * [Azure Portal](dns-operations-recordsets-portal.md)
> * [CLI de Azure clásica](./dns-operations-recordsets-cli.md)
> * [CLI de Azure](dns-operations-recordsets-cli.md)
> * [PowerShell](dns-operations-recordsets.md)

En este artículo se muestra cómo administrar registros DNS para su zona DNS mediante Azure PowerShell. Los registros de DNS también se pueden administrar mediante la [CLI de Azure](dns-operations-recordsets-cli.md) multiplataforma o [Azure Portal](dns-operations-recordsets-portal.md).

En los ejemplos de este artículo se supone que ya ha [instalado Azure PowerShell, iniciado sesión y creado una zona DNS](dns-operations-dnszones.md).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="introduction"></a>Introducción

Antes de crear registros DNS en Azure DNS, es necesario que comprenda cómo Azure DNS los organiza en conjuntos de registros DNS.

[!INCLUDE [dns-about-records-include](../../includes/dns-about-records-include.md)]

Para más información sobre los registros DNS en Azure DNS, consulte [DNS zones and records](dns-zones-records.md) (Zonas y registros DNS).


## <a name="create-a-new-dns-record"></a>Creación de un nuevo registro DNS

Para crear un nuevo conjunto de registros, debe tener un nombre y un tipo diferentes a los registros existentes. Si el nuevo registro tiene el mismo nombre y tipo que un registro existente, tendrá que agregarlo al [conjunto de registros existente](#add-a-record-to-an-existing-record-set).

### <a name="create-a-records-in-a-new-record-set"></a>Creación de registros "D" en un nuevo conjunto de registros

Los conjuntos de registros se crean mediante el cmdlet `New-AzDnsRecordSet`. Al crear un conjunto de registros, deberá especificar el nombre del conjunto de registros, la zona, el período de vida (TTL), el tipo de registro y los registros que se crearán.

Los parámetros para agregar registros a un conjunto de registros varían según el tipo de conjunto de registros. Por ejemplo, cuando se usa un conjunto de registros de tipo "A", deberá especificar la dirección IP mediante el parámetro `-IPv4Address`. Los distintos tipos de registro tendrán parámetros adicionales.

En el ejemplo siguiente se crea un conjunto de registros con el nombre relativo `www` en la zona DNS `contoso.com`. El nombre completo del conjunto de registros es `www.contoso.com`. El tipo de registro es "D" y el valor de TTL es de 3600 segundos. El conjunto de registros contiene un único registro con la dirección IP "1.2.3.4".

```azurepowershell-interactive
New-AzDnsRecordSet -Name "www" -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzDnsRecordConfig -IPv4Address "1.2.3.4") 
```

Para crear un conjunto de registros en el "vértice" de una zona (en este caso, "contoso.com"), use el nombre del conjunto de registros "\@" (excluidas las comillas):

```azurepowershell-interactive
New-AzDnsRecordSet -Name "@" -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzDnsRecordConfig -IPv4Address "1.2.3.4") 
```

Si necesita crear un conjunto de registros que contenga más de un registro, primero cree una matriz local y agregue los registros, luego pase la matriz a `New-AzDnsRecordSet` de la forma siguiente:

```azurepowershell-interactive
$aRecords = @()
$aRecords += New-AzDnsRecordConfig -IPv4Address "1.2.3.4"
$aRecords += New-AzDnsRecordConfig -IPv4Address "2.3.4.5"
New-AzDnsRecordSet -Name www –ZoneName "contoso.com" -ResourceGroupName MyResourceGroup -Ttl 3600 -RecordType A -DnsRecords $aRecords
```

Se pueden usar [metadatos del conjunto de registros](dns-zones-records.md#tags-and-metadata) para asociar datos específicos de la aplicación con cada conjunto de registros como pares clave-valor. En el ejemplo siguiente se muestra cómo crear un conjunto de registros con dos entradas de metadatos: "dept=finance" y "environment=production".

```azurepowershell-interactive
New-AzDnsRecordSet -Name "www" -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzDnsRecordConfig -IPv4Address "1.2.3.4") -Metadata @{ dept="finance"; environment="production" } 
```

Azure DNS también admite conjuntos de registros "vacíos" que pueden funcionar como marcador de posición para reservar un nombre DNS antes de crear registros DNS. Los conjuntos de registros vacíos son visibles en el panel de control de Azure DNS, pero aparecen en los servidores de nombres de Azure DNS. En el ejemplo siguiente se crea un conjunto de registros vacío:

```azurepowershell-interactive
New-AzDnsRecordSet -Name "www" -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords @()
```

## <a name="create-records-of-other-types"></a>Creación de registros de otros tipos

Después de haber visto de forma detallada como crear registros "A", en los siguientes ejemplos se muestra cómo crear registros de otros tipos compatibles con Azure DNS.

En cada caso, se muestra cómo crear un nuevo conjunto de registros que contiene un único registro. Los ejemplos anteriores para registros "A" se pueden adaptar para crear conjuntos de registros de otros tipos que contengan varios registros, con metadatos, o crear conjuntos de registros vacíos.

No hay ningún ejemplo para crear un conjunto de registros SOA, ya que los SOA se crean y eliminan con cada zona DNS. Los registros SOA no pueden crearse ni eliminarse por separado. Sin embargo, el registro SOA se puede [modificar](#to-modify-an-soa-record), como se muestra en un ejemplo más adelante.

### <a name="create-an-aaaa-record-set-with-a-single-record"></a>Creación de un conjunto de registros AAAA con un único registro

```azurepowershell-interactive
New-AzDnsRecordSet -Name "test-aaaa" -RecordType AAAA -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzDnsRecordConfig -Ipv6Address "2607:f8b0:4009:1803::1005") 
```

### <a name="create-a-caa-record-set-with-a-single-record"></a>Creación de un conjunto de registros CAA con un único registro

```azurepowershell-interactive
New-AzDnsRecordSet -Name "test-caa" -RecordType CAA -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzDnsRecordConfig -Caaflags 0 -CaaTag "issue" -CaaValue "ca1.contoso.com") 
```

### <a name="create-a-cname-record-set-with-a-single-record"></a>Creación de un conjunto de registros CNAME con un único registro

> [!NOTE]
> Los estándares DNS no permiten registros CNAME en el vértice de una zona (`-Name '@'`), ni permiten conjuntos de registros que contengan más de un registro.
> 
> Para más información, consulte [Registros CNAME](dns-zones-records.md#cname-records).


```azurepowershell-interactive
New-AzDnsRecordSet -Name "test-cname" -RecordType CNAME -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzDnsRecordConfig -Cname "www.contoso.com") 
```

### <a name="create-an-mx-record-set-with-a-single-record"></a>Creación de un conjunto de registros MX con un único registro

En este ejemplo, se utiliza el nombre de conjunto de registros "\@" para crear el registro MX en el vértice de la zona (por ejemplo, "contoso.com").


```azurepowershell-interactive
New-AzDnsRecordSet -Name "@" -RecordType MX -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzDnsRecordConfig -Exchange "mail.contoso.com" -Preference 5) 
```

### <a name="create-an-ns-record-set-with-a-single-record"></a>Creación de un conjunto de registros NS con un único registro

```azurepowershell-interactive
New-AzDnsRecordSet -Name "test-ns" -RecordType NS -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzDnsRecordConfig -Nsdname "ns1.contoso.com") 
```

### <a name="create-a-ptr-record-set-with-a-single-record"></a>Creación de un conjunto de registros PTR con un único registro

En este caso, "my-arpa-zone.com" representa la zona de búsqueda inversa ARPA que representa el intervalo IP. Cada registro PTR establecido en esta zona se corresponde con una dirección IP dentro de este intervalo IP. El nombre de registro "10" es el último octeto de la dirección IP dentro del intervalo IP que representa dicho registro.

```azurepowershell-interactive
New-AzDnsRecordSet -Name 10 -RecordType PTR -ZoneName "my-arpa-zone.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzDnsRecordConfig -Ptrdname "myservice.contoso.com") 
```

### <a name="create-an-srv-record-set-with-a-single-record"></a>Creación de un conjunto de registros SRV con un único registro

Al crear un [conjunto de registros SRV](dns-zones-records.md#srv-records), especifique el *\_servicio* y el *\_protocolo* en el nombre del conjunto de registros. No hay necesidad de incluir "\@" en el nombre del conjunto de registros al crear un conjunto de registros SRV en el vértice de la zona.

```azurepowershell-interactive
New-AzDnsRecordSet -Name "_sip._tls" -RecordType SRV -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzDnsRecordConfig -Priority 0 -Weight 5 -Port 8080 -Target "sip.contoso.com") 
```


### <a name="create-a-txt-record-set-with-a-single-record"></a>Crear un conjunto de registros TXT con un único registro

En el ejemplo siguiente se muestra cómo crear un registro TXT. Para más información sobre la longitud de cadena máxima admitida en registros TXT, consulte [Registros TXT](dns-zones-records.md#txt-records).

```azurepowershell-interactive
New-AzDnsRecordSet -Name "test-txt" -RecordType TXT -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzDnsRecordConfig -Value "This is a TXT record") 
```


## <a name="get-a-record-set"></a>Recuperación de un conjunto de registros

Para recuperar un conjunto de registros existente, use `Get-AzDnsRecordSet`. Este cmdlet devuelve un objeto local que representa el conjunto de registros en Azure DNS.

Al igual que con `New-AzDnsRecordSet`, el nombre del conjunto de registros dado debe ser un nombre *relativo*, lo que significa que debe excluir el nombre de zona. También se debe especificar el tipo de registro y la zona que contiene el conjunto de registros.

En el ejemplo siguiente se muestra cómo recuperar un conjunto de registros. En este ejemplo, la zona se especifica mediante los parámetros `-ZoneName` y `-ResourceGroupName`.

```azurepowershell-interactive
$rs = Get-AzDnsRecordSet -Name "www" -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup"
```

En su lugar, también puede especificar la zona mediante un objeto de zona, que se pasa mediante el parámetro `-Zone`.

```azurepowershell-interactive
$zone = Get-AzDnsZone -Name "contoso.com" -ResourceGroupName "MyResourceGroup"
$rs = Get-AzDnsRecordSet -Name "www" -RecordType A -Zone $zone
```

## <a name="list-record-sets"></a>Enumeración de conjuntos de registros

También puede usar `Get-AzDnsZone` para mostrar conjuntos de registros de una zona y omitir uno de los parámetros `-Name` o `-RecordType`, o ambos.

En el ejemplo siguiente se devuelven todos los conjuntos de registros de la zona:

```azurepowershell-interactive
$recordsets = Get-AzDnsRecordSet -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup"
```

En el ejemplo siguiente se muestra cómo puede recuperar todos los conjuntos de registros de un tipo dado especificando el tipo de registro al omitir el nombre del conjunto de registros:

```azurepowershell-interactive
$recordsets = Get-AzDnsRecordSet -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup"
```

Para recuperar todos los conjuntos de registros con un nombre específico de todos los tipos de registros, debe recuperar todos los conjuntos de registros y luego filtrar los resultados:

```azurepowershell-interactive
$recordsets = Get-AzDnsRecordSet -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" | where {$_.Name.Equals("www")}
```

En todos los ejemplos anteriores, la zona se puede especificar mediante los parámetros `-ZoneName` y `-ResourceGroupName` (como se muestra) o por medio de un objeto de zona:

```azurepowershell-interactive
$zone = Get-AzDnsZone -Name "contoso.com" -ResourceGroupName "MyResourceGroup"
$recordsets = Get-AzDnsRecordSet -Zone $zone
```

## <a name="add-a-record-to-an-existing-record-set"></a>Adición de un registro a un conjunto de registros existente

Para agregar un registro a un conjunto de registros existente, siga estos tres pasos:

1. Obtenga el conjunto de registros existente.

    ```azurepowershell-interactive
    $rs = Get-AzDnsRecordSet -Name www –ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -RecordType A
    ```

1. Agregue el nuevo registro al conjunto de registros local.

    ```azurepowershell-interactive
    Add-AzDnsRecordConfig -RecordSet $rs -Ipv4Address "5.6.7.8"
    ```

3. Actualice los cambios para que se reflejen en el servicio Azure DNS. 

    ```azurepowershell-interactive
    Set-AzDnsRecordSet -RecordSet $rs
    ```

Al usar , `Set-AzDnsRecordSet` *se sustituye* el conjunto de registros existente en Azure DNS (y todos los registros que contiene) por el conjunto de registros especificado. Se usan [comprobaciones de ETag](dns-zones-records.md#etags) para garantizar que los cambios simultáneos no se sobrescriban. Puede usar el modificador `-Overwrite` opcional para suprimir estas comprobaciones.

Esta secuencia de operaciones también se puede *canalizar*, es decir, pasar el objeto de conjunto de registros mediante la canalización en lugar de como un parámetro.

```azurepowershell-interactive
Get-AzDnsRecordSet -Name "www" –ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -RecordType A | Add-AzDnsRecordConfig -Ipv4Address "5.6.7.8" | Set-AzDnsRecordSet
```

Los ejemplos anteriores muestran cómo agregar un registro "A" a un conjunto de registros existente de tipo "A". Se usa una secuencia parecida de operaciones para agregar registros a conjunto de registros de otros tipos, reemplazando el parámetro `-Ipv4Address` de `Add-AzDnsRecordConfig` por otros parámetros específicos de cada tipo de registro. Los parámetros de cada tipo de registro son los mismos que para el cmdlet `New-AzDnsRecordConfig`, como se muestra en otros ejemplos de tipos de registros anteriormente.

Los conjuntos de registros de tipo "CNAME" o "SOA" no pueden contener más de un registro. Esta restricción surge de los estándares de DNS. No se trata de una limitación de Azure DNS.

## <a name="remove-a-record-from-an-existing-record-set"></a>Eliminación de un registro de un conjunto de registros existente

El proceso para quitar un registro de un conjunto de registros es similar al proceso para agregar un registro a un conjunto de registros existente:

1. Obtenga el conjunto de registros existente.

    ```azurepowershell-interactive
    $rs = Get-AzDnsRecordSet -Name www –ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -RecordType A
    ```

2. Quite el registro del objeto del conjunto de registros local. Todos los parámetros del registro que se va a eliminar deben coincidir exactamente con los del registro existente.

    ```azurepowershell-interactive
    Remove-AzDnsRecordConfig -RecordSet $rs -Ipv4Address "5.6.7.8"
    ```

3. Confirme el cambio al servicio Azure DNS. Use el modificador `-Overwrite` para suprimir las [comprobaciones de ETag](dns-zones-records.md#etags) para cambios simultáneos.

    ```azurepowershell-interactive
    Set-AzDnsRecordSet -RecordSet $Rs
    ```

El uso de la secuencia anterior para quitar el último registro de un conjunto de registros no elimina el conjunto de registros, sino que deja un conjunto de registros vacío. Para quitar completamente un conjunto de registros, consulte [Eliminación de un conjunto de registros](#delete-a-record-set).

Lo mismo que para agregar registros a un conjunto de registros, la secuencia de operaciones para quitar un conjunto de registros también se puede canalizar:

```azurepowershell-interactive
Get-AzDnsRecordSet -Name www –ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -RecordType A | Remove-AzDnsRecordConfig -Ipv4Address "5.6.7.8" | Set-AzDnsRecordSet
```

Se admiten diferentes tipos de registros pasando los parámetros adecuados específicos del tipo a `Remove-AzDnsRecordSet`. Los parámetros de cada tipo de registro son los mismos que para el cmdlet `New-AzDnsRecordConfig`, como se muestra en otros ejemplos de tipos de registros anteriormente.


## <a name="modify-an-existing-record-set"></a>Modificación de un conjunto de registros existente

Los pasos para modificar un conjunto de registros existente son parecidos a los que se realizan al agregar o quitar registros de un conjunto de registros:

1. Recupere el conjunto de registros existente mediante `Get-AzDnsRecordSet`.
2. Para modificar el objeto de conjunto de registros local:
    * Agregue o quite registros.
    * Cambie los parámetros de registros existentes.
    * Cambie los metadatos del conjunto de registros y el período de vida (TTL).
3. Confirme los cambios mediante el cmdlet `Set-AzDnsRecordSet` . Esta acción *reemplaza* el conjunto de registros existente de Azure DNS por el conjunto de registros especificado.

Al usar `Set-AzDnsRecordSet`, se emplean [comprobaciones de ETag](dns-zones-records.md#etags) para garantizar que los cambios simultáneos no se sobrescriban. Puede usar el modificador `-Overwrite` opcional para suprimir estas comprobaciones.

### <a name="to-update-a-record-in-an-existing-record-set"></a>Para actualizar un registro en un conjunto de registros existente

En este ejemplo, se cambiará la dirección IP de un registro A existente:

```azurepowershell-interactive
$rs = Get-AzDnsRecordSet -name "www" -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup"
$rs.Records[0].Ipv4Address = "9.8.7.6"
Set-AzDnsRecordSet -RecordSet $rs
```

### <a name="to-modify-an-soa-record"></a>Para modificar un registro SOA

No puede agregar ni quitar registros del conjunto de registros SOA creado automáticamente en el vértice de zona (`-Name "@"`, comillas incluidas). Sin embargo, puede modificar cualquiera de los parámetros del registro SOA (excepto "Host") y del conjunto de registros TTL.

En el ejemplo siguiente se muestra cómo cambiar la propiedad *Email* del registro SOA:

```azurepowershell-interactive
$rs = Get-AzDnsRecordSet -Name "@" -RecordType SOA -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup"
$rs.Records[0].Email = "admin.contoso.com"
Set-AzDnsRecordSet -RecordSet $rs
```

### <a name="to-modify-ns-records-at-the-zone-apex"></a>Para modificar los registros NS en el vértice de zona

El conjunto de registros NS en el vértice de zona se crea automáticamente con cada zona DNS. Este conjunto de registros contiene los nombres de los servidores de nombres de Azure DNS asignados a la zona.

Puede agregar más servidores de nombres a este conjunto de registros NS, para admitir dominios de hospedaje conjunto con más de un proveedor DNS. También puede modificar el TTL y los metadatos de este conjunto de registros. Sin embargo, no puede quitar ni modificar los servidores de nombres de Azure DNS rellenados previamente.

Esta restricción se aplica solo al conjunto de registros NS en el vértice de zona. Otros conjuntos de registros NS de su zona (como los que se usan para delegar zonas secundarias) se pueden modificar sin restricciones.

En el ejemplo siguiente se muestra cómo agregar otro servidor de nombres al conjunto de registros NS en el vértice de zona:

```azurepowershell-interactive
$rs = Get-AzDnsRecordSet -Name "@" -RecordType NS -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup"
Add-AzDnsRecordConfig -RecordSet $rs -Nsdname ns1.myotherdnsprovider.com
Set-AzDnsRecordSet -RecordSet $rs
```

### <a name="to-modify-record-set-metadata"></a>Para modificar los metadatos del conjunto de registros:

Se pueden usar [metadatos del conjunto de registros](dns-zones-records.md#tags-and-metadata) para asociar datos específicos de la aplicación con cada conjunto de registros como pares clave-valor.

En el ejemplo siguiente se muestra cómo modificar los metadatos de un conjunto de registros existente:

```azurepowershell-interactive
# Get the record set
$rs = Get-AzDnsRecordSet -Name www -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup"

# Add 'dept=finance' name-value pair
$rs.Metadata.Add('dept', 'finance') 

# Remove metadata item named 'environment'
$rs.Metadata.Remove('environment')  

# Commit changes
Set-AzDnsRecordSet -RecordSet $rs
```

## <a name="delete-a-record-set"></a>Eliminación de un conjunto de registros

Los conjuntos de registros pueden eliminarse mediante el cmdlet `Remove-AzDnsRecordSet` . Al eliminar un conjunto de registros también se eliminan todos los registros que contiene.

> [!NOTE]
> No se pueden eliminar los conjuntos de registros SOA y NS en el vértice de zona (`-Name '@'`).  Los DNS de Azure se crean automáticamente cuando se genera la zona y se eliminan automáticamente cuando se elimina.

En el ejemplo siguiente se muestra cómo eliminar un conjunto de registros. En este ejemplo, el nombre del conjunto de registros, el tipo de conjunto de registros, el nombre de zona y el grupo de recursos se especifican de manera explícita.

```azurepowershell-interactive
Remove-AzDnsRecordSet -Name "www" -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup"
```

En su lugar, el conjunto de registros se puede especificar por nombre y tipo, y la zona se puede especificar mediante un objeto:

```azurepowershell-interactive
$zone = Get-AzDnsZone -Name "contoso.com" -ResourceGroupName "MyResourceGroup"
Remove-AzDnsRecordSet -Name "www" -RecordType A -Zone $zone
```

Como tercera opción, el propio conjunto de registros se puede especificar mediante un objeto de conjunto de registros:

```azurepowershell-interactive
$rs = Get-AzDnsRecordSet -Name www -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup"
Remove-AzDnsRecordSet -RecordSet $rs
```

Cuando especifica que se elimine el conjunto de registros eliminar mediante un objeto de conjunto de registros, se usan [comprobaciones de ETag](dns-zones-records.md#etags) para garantizar que no se eliminen los cambios simultáneos. Puede usar el modificador `-Overwrite` opcional para suprimir estas comprobaciones.

El objeto del conjunto de registros también puede canalizarse en lugar de pasarse como un parámetro:

```azurepowershell-interactive
Get-AzDnsRecordSet -Name www -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" | Remove-AzDnsRecordSet
```

## <a name="confirmation-prompts"></a>Mensajes de confirmación

Los cmdlets `New-AzDnsRecordSet`, `Set-AzDnsRecordSet` y `Remove-AzDnsRecordSet` todos admiten mensajes de confirmación.

Cada cmdlet pide confirmación si la variable de preferencia de PowerShell `$ConfirmPreference` tiene un valor de `Medium` o inferior. Dado que el valor predeterminado de `$ConfirmPreference` es `High`, estos mensajes no aparecen cuando se usa la configuración predeterminada de PowerShell.

Puede invalidar el valor actual de `$ConfirmPreference` mediante el parámetro `-Confirm`. Si especifica `-Confirm` o `-Confirm:$True`, el cmdlet solicita confirmación antes de ejecutarse. Si especifica `-Confirm:$False`, el cmdlet no le pide una confirmación. 

Para más información sobre `-Confirm` y `$ConfirmPreference`, consulte [About Preference Variables](/powershell/module/microsoft.powershell.core/about/about_preference_variables) (Acerca de las variables de preferencias).

## <a name="next-steps"></a>Pasos siguientes

Más información sobre [zonas y registros en Azure DNS](dns-zones-records.md).
<br>
Aprenda a [proteger las zonas y los registros](dns-protect-zones-recordsets.md) cuando se usa Azure DNS.
<br>
Revise la [documentación de referencia de PowerShell para Azure DNS](/powershell/module/az.dns).