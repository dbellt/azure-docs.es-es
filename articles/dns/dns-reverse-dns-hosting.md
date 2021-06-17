---
title: Alojamiento de zonas de búsqueda inversa DNS en Azure DNS
description: Aprenda a usar DNS de Azure para hospedar las zonas de búsqueda inversa de DNS para los intervalos IP
author: rohinkoul
ms.service: dns
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 04/29/2021
ms.author: rohink
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: ae3343a4b441ff47621d4ad5a6252768e2d5ef63
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110705009"
---
# <a name="host-reverse-dns-lookup-zones-in-azure-dns"></a>Alojamiento de zonas de búsqueda inversa DNS en Azure DNS

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

En este artículo se explica cómo hospedar las zonas de búsqueda inversa DNS para los intervalos IP asignados en Azure DNS. Los intervalos IP representados por las zonas de búsqueda inversa deben asignarse a su organización, normalmente por su ISP.

Para configurar el DNS inverso para una dirección IP de propiedad de Azure asignada al servicio de Azure, consulte [Configuración de DNS inversos para servicios hospedados en Azure](dns-reverse-dns-for-azure-services.md).

Antes de leer este artículo, debe familiarizarse con la [información general de DNS inverso](dns-reverse-dns-overview.md) y su compatibilidad en Azure.

En este artículo, aprenderá a crear su primera zona de búsqueda inversa DNS y su primer registro de DNS con Azure Portal, Azure PowerShell, CLI de Azure clásica y CLI de Azure.

## <a name="create-a-reverse-lookup-dns-zone"></a>Creación de una zona de búsqueda inversa DNS

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. En la parte superior izquierda de la pantalla, seleccione **Crear un recurso** y busque **Zona DNS**. Seleccione **Crear**.

      :::image type="content" source="./media/dns-operations-dnszones-portal/search-dns-zone.png" alt-text="Captura de pantalla de la búsqueda para crear un recurso para la zona DNS inversa.":::

1. En la página **Crear zona DNS**, escriba o seleccione los siguientes parámetros:

    | Configuración | Detalles |
    | --- | --- |
    | **Suscripción** | Seleccione la suscripción en la que se creará la zona DNS.|
    | **Grupos de recursos** | Seleccione o cree un grupo de recursos. Para más información sobre los grupos de recursos, lea el artículo [Información general de Azure Resource Manager](../azure-resource-manager/management/overview.md?toc=%2fazure%2fdns%2ftoc.json#resource-groups).|
    | **Nombre** | Escriba un nombre para la zona DNS. El nombre de la zona está diseñado de forma diferente para los prefijos IPv4 e IPv6. Use las instrucciones para [IPv4](#ipv4) o [IPv6](#ipv6) a fin de asignar un nombre a la zona.  |
    | **Ubicación** | Seleccione la ubicación del grupo de recursos. La ubicación ya estará seleccionada si usa un grupo de recursos creado anteriormente. |

1. Seleccione **Revisar y crear** y luego seleccione **Crear** una vez superada la validación.

### <a name="ipv4"></a>IPv4

El nombre de una zona de búsqueda inversa IPv4 se basa en el intervalo IP que representa. Tendrá el siguiente formato: `<IPv4 network prefix in reverse order>.in-addr.arpa`. Para obtener ejemplos, consulte [Introducción a DNS inverso](dns-reverse-dns-overview.md#ipv4) para IPv4.

> [!NOTE]
> Cuando crea zonas de búsqueda inversa de DNS sin clases en Azure DNS, debe usar un guion (`-`) en lugar de una barra diagonal (`/`) en el nombre de la zona.
>
> Por ejemplo, para el intervalo IP de 192.0.2.128/26, usará `128-26.2.0.192.in-addr.arpa` como nombre de zona en lugar de `128/26.2.0.192.in-addr.arpa`.
>
> Aunque los estándares DNS admiten ambos métodos, Azure DNS no admite los nombres de zonas DNS que contienen el carácter de barra diagonal (`/`).

En el ejemplo siguiente se muestra cómo crear una zona DNS inversa de Clase C denominada `2.0.192.in-addr.arpa` en Azure DNS mediante Azure Portal:

:::image type="content" source="./media/dns-reverse-dns-hosting/ipv4-arpa-zone.png" alt-text="Captura de pantalla de la creación de una zona DNS arpa de IPv4.":::

Los ejemplos siguientes muestran cómo completar esta tarea mediante Azure PowerShell y la CLI de Azure.

#### <a name="powershell"></a>PowerShell

```azurepowershell-interactive
New-AzDnsZone -Name 2.0.192.in-addr.arpa -ResourceGroupName mydnsresourcegroup
```

#### <a name="azure-classic-cli"></a>CLI de Azure clásica

```azurecli
azure network dns zone create mydnsresourcegroup 2.0.192.in-addr.arpa
```

#### <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
az network dns zone create -g mydnsresourcegroup -n 2.0.192.in-addr.arpa
```

### <a name="ipv6"></a>IPv6

El nombre de una zona de búsqueda inversa IPv6 debe tener el formato siguiente: `<IPv6 network prefix in reverse order>.ip6.arpa`.  Para obtener ejemplos, consulte [Introducción a DNS inverso](dns-reverse-dns-overview.md#ipv6) para IPv6.


En el ejemplo siguiente se muestra cómo crear una zona de búsqueda inversa DNS IPv6 denominada `0.0.0.0.d.c.b.a.8.b.d.0.1.0.0.2.ip6.arpa` en DNS de Azure mediante Azure Portal:

:::image type="content" source="./media/dns-reverse-dns-hosting/ipv6-arpa-zone.png" alt-text="Captura de pantalla de la creación de una zona DNS arpa de IPv6.":::

Los ejemplos siguientes muestran cómo completar esta tarea mediante Azure PowerShell y la CLI de Azure.

#### <a name="powershell"></a>PowerShell

```powershell
New-AzDnsZone -Name 0.0.0.0.d.c.b.a.8.b.d.0.1.0.0.2.ip6.arpa -ResourceGroupName mydnsresourcegroup
```

#### <a name="azure-classic-cli"></a>CLI de Azure clásica

```azurecli
azure network dns zone create mydnsresourcegroup 0.0.0.0.d.c.b.a.8.b.d.0.1.0.0.2.ip6.arpa
```

#### <a name="azure-cli"></a>Azure CLI

```azurecli
az network dns zone create -g mydnsresourcegroup -n 0.0.0.0.d.c.b.a.8.b.d.0.1.0.0.2.ip6.arpa
```

## <a name="delegate-a-reverse-dns-lookup-zone"></a>Delegación de una zona de búsqueda inversa DNS

Una vez creada la zona de búsqueda inversa DNS, debe asegurarse de que la zona se delega de la zona primaria. La delegación de DNS permite que el proceso de resolución de nombres DNS encuentre los servidores de nombres que hospedan la zona de búsqueda inversa DNS. Esos servidores de nombres entonces pueden responder a consultas DNS inversas para las direcciones IP en el intervalo de direcciones.

Para las zonas de búsqueda directa, el proceso de delegación de una zona DNS se describe en [Delegación de un dominio en DNS de Azure](dns-delegate-domain-azure-dns.md). La delegación de zonas de búsqueda inversa funciona del mismo modo. La única diferencia es que debe configurar los servidores de nombres con el ISP. El ISP administra el intervalo IP, por eso tiene que actualizar los servidores de nombres en lugar del registrador de nombres de dominio.

## <a name="create-a-dns-ptr-record"></a>Creación de un registro PTR DNS

### <a name="ipv4"></a>IPv4

En el ejemplo siguiente se explica el proceso de creación de un registro PTR en una zona DNS inversa mediante Azure Portal. Para obtener más información sobre los tipos de registro o cómo modificar los registros existentes, consulte [Administración de registros y conjuntos de registros DNS](dns-operations-recordsets-portal.md).

1. En la parte superior de la página de información general *Zona DNS*, seleccione **+ Conjunto de registros** para abrir la página *Agregar conjunto de registros*.

    :::image type="content" source="./media/dns-reverse-dns-hosting/create-record-set-ipv4.png" alt-text="Captura de pantalla de creación de un conjunto de registros de puntero de IPv4.":::

1. El nombre del conjunto de registros para un registro PTR será el resto de la dirección IPv4 en orden inverso. 

    En este ejemplo, los tres primeros octetos ya están rellenados como parte del nombre de la zona `.2.0.192`. Es por ello que solo se proporciona el último octeto en el cuadro de **nombre**. Por ejemplo, dará al conjunto de registros el nombre **15** para un recurso cuya dirección IP es `192.0.2.15`.  

    :::image type="content" source="./media/dns-reverse-dns-hosting/create-ipv4-ptr.png" alt-text="Captura de pantalla de creación de un registro de puntero de IPv4.":::

1. En *Tipo*, seleccione **PTR**.

1. En *NOMBRE DE DOMINIO*, escriba el nombre de dominio completo (FQDN) del recurso que usa la dirección IP.

1. Seleccione **Aceptar** para crear el registro DNS.

Los ejemplos siguientes muestran cómo completar esta tarea mediante Azure PowerShell y la CLI de Azure.

#### <a name="powershell"></a>PowerShell

```azurepowershell-interactive
New-AzDnsRecordSet -Name 15 -RecordType PTR -ZoneName 2.0.192.in-addr.arpa -ResourceGroupName mydnsresourcegroup -Ttl 3600 -DnsRecords (New-AzDnsRecordConfig -Ptrdname "dc1.contoso.com")
```
#### <a name="azure-classic-cli"></a>CLI de Azure clásica

```azurecli
azure network dns record-set add-record mydnsresourcegroup 2.0.192.in-addr.arpa 15 PTR --ptrdname dc1.contoso.com  
```

#### <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
az network dns record-set ptr add-record -g mydnsresourcegroup -z 2.0.192.in-addr.arpa -n 15 --ptrdname dc1.contoso.com
```

### <a name="ipv6"></a>IPv6

En el ejemplo siguiente, se explica el proceso de creación de un registro PTR para IPv6. Para obtener más información sobre los tipos de registro o cómo modificar los registros existentes, consulte [Administración de registros y conjuntos de registros DNS](dns-operations-recordsets-portal.md).

1. En la parte superior del panel *Zona DNS*, seleccione **+ Conjunto de registros** para abrir el panel *Agregar conjunto de registros*.

   :::image type="content" source="./media/dns-reverse-dns-hosting/create-record-set-ipv6.png" alt-text="Captura de pantalla de creación de un conjunto de registros de puntero de IPv6.":::

1. El nombre del conjunto de registros para un registro PTR será el resto de la dirección IPv6 en orden inverso. No puede contener ninguna compresión de ceros. 

    En este ejemplo, los primeros 64 bits de IPv6 se rellenan como parte del nombre de zona (0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2.ip6.arpa). Por lo tanto, solo se proporcionan los últimos 64 bits en el cuadro de **nombre**. Los últimos 64 bits de la dirección IP se introducen en orden inverso, con un punto como delimitador entre cada número hexadecimal. Podrá denominar al conjunto de registros **e.5.0.4.9.f.a.1.c.b.0.1.4.2.5.f** para un recurso cuya dirección IP es 2001:0db8:abdc:0000:f524:10bc:1af9:405e.

    :::image type="content" source="./media/dns-reverse-dns-hosting/create-ipv6-ptr.png" alt-text="Captura de pantalla de creación de un registro de puntero de IPv6.":::

1. En *Tipo*, seleccione **PTR**.  

1. En *NOMBRE DE DOMINIO*, escriba el nombre de dominio completo del recurso que usa la dirección IP.

1. Seleccione **Aceptar** para crear el registro DNS.

Los ejemplos siguientes muestran cómo completar esta tarea mediante PowerShell y la CLI de Azure.

#### <a name="powershell"></a>PowerShell

```azurepowershell-interactive
New-AzDnsRecordSet -Name "e.5.0.4.9.f.a.1.c.b.0.1.4.2.5.f" -RecordType PTR -ZoneName 0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2.ip6.arpa -ResourceGroupName mydnsresourcegroup -Ttl 3600 -DnsRecords (New-AzDnsRecordConfig -Ptrdname "dc2.contoso.com")
```

#### <a name="azure-classic-cli"></a>CLI de Azure clásica

```azurecli
azure network dns record-set add-record mydnsresourcegroup 0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2.ip6.arpa e.5.0.4.9.f.a.1.c.b.0.1.4.2.5.f PTR --ptrdname dc2.contoso.com 
```
 
#### <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
az network dns record-set ptr add-record -g mydnsresourcegroup -z 0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2.ip6.arpa -n e.5.0.4.9.f.a.1.c.b.0.1.4.2.5.f --ptrdname dc2.contoso.com
```

## <a name="view-records"></a>Visualización de los registros

Para ver los registros que creó, vaya a la zona DNS en Azure Portal. En la parte inferior del panel **Zona DNS**, puede consultar los registros correspondientes a la zona DNS. Debería ver los registros NS y SOA predeterminados, además de cualquier registro que se haya creado. Los registros NS y SOA se crean en cada zona. 

### <a name="ipv4"></a>IPv4

La página **Zona DNS** mostrará el registro PTR de IPv4:

:::image type="content" source="./media/dns-reverse-dns-hosting/view-ipv4-ptr-record.png" alt-text="Captura de pantalla del registro de puntero de IPv4 en la página de información general." lightbox="./media/dns-reverse-dns-hosting/view-ipv4-ptr-record-expanded.png":::

En los ejemplos siguientes se muestra cómo ver los registros PTR mediante Azure PowerShell y la CLI de Azure.

#### <a name="powershell"></a>PowerShell

```azurepowershell-interactive
Get-AzDnsRecordSet -ZoneName 2.0.192.in-addr.arpa -ResourceGroupName mydnsresourcegroup
```

#### <a name="azure-classic-cli"></a>CLI de Azure clásica

```azurecli
azure network dns record-set list mydnsresourcegroup 2.0.192.in-addr.arpa
```

#### <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
az network dns record-set list -g mydnsresourcegroup -z 2.0.192.in-addr.arpa
```

### <a name="ipv6"></a>IPv6

El panel **Zona DNS** se muestran los registros PTR de IPv6:

:::image type="content" source="./media/dns-reverse-dns-hosting/view-ipv6-ptr-record.png" alt-text="Captura de pantalla del registro de puntero de IPv6 en la página de información general." lightbox="./media/dns-reverse-dns-hosting/view-ipv6-ptr-record-expanded.png":::

En los ejemplos siguientes se muestra cómo ver los registros mediante PowerShell o la CLI de Azure.

#### <a name="powershell"></a>PowerShell

```powershell
Get-AzDnsRecordSet -ZoneName 0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2.ip6.arpa -ResourceGroupName mydnsresourcegroup
```

#### <a name="azure-classic-cli"></a>CLI de Azure clásica

```azurecli
azure network dns record-set list mydnsresourcegroup 0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2.ip6.arpa
```

#### <a name="azure-cli"></a>Azure CLI

```azurecli
az network dns record-set list -g mydnsresourcegroup -z 0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2.ip6.arpa
```

## <a name="faq"></a>Preguntas más frecuentes

### <a name="can-i-host-reverse-dns-lookup-zones-for-my-isp-assigned-ip-blocks-on-azure-dns"></a>¿Puedo hospedar zonas de búsqueda inversa DNS para mis bloques IP asignados por el ISP en DNS de Azure?

Sí. El hospedaje de zonas de búsqueda inversa (ARPA) para sus propios intervalos de IP en DNS de Azure está totalmente permitido.

Cree la zona de búsqueda inversa en Azure DNS tal como se explica en este artículo. A continuación, trabaje con el ISP para [delegar la zona](dns-domain-delegation.md). Después puede administrar los registros PTR de cada búsqueda inversa igual que otros tipos de registros.

### <a name="how-much-does-hosting-my-reverse-dns-lookup-zone-cost"></a>¿Cuánto cuesta el hospedaje de mi zona de búsqueda inversa DNS?

El hospedaje de una zona de búsqueda inversa DNS para su bloque de direcciones IP asignadas por el ISP en DNS de Azure se cobra según las [tarifas estándar de DNS de Azure](https://azure.microsoft.com/pricing/details/dns/).

### <a name="can-i-host-reverse-dns-lookup-zones-for-both-ipv4-and-ipv6-addresses-in-azure-dns"></a>Puedo hospedar zonas de búsqueda inversa DNS para direcciones IPv4 e IPv6 en DNS de Azure?

Sí. En este artículo se explica cómo crear zonas de búsqueda inversa DNS de IPv4 e IPv6 en DNS de Azure.

### <a name="can-i-import-an-existing-reverse-dns-lookup-zone"></a>¿Puedo importar una zona de búsqueda inversa DNS existente?

Sí. Puede usar la CLI de Azure para importar las zonas DNS existentes a Azure DNS. Este método funciona para las zonas de búsqueda directa y las zonas de búsqueda inversa.

Para más información, consulte [Importación y exportación de un archivo de zona DNS mediante la CLI de Azure](dns-import-export.md).

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre los registros de DNS inverso, consulte información sobre la [búsqueda de DNS inverso en Wikipedia](https://en.wikipedia.org/wiki/Reverse_DNS_lookup).

* Aprenda a [administrar registros de DNS inversos para servicios de Azure](dns-reverse-dns-for-azure-services.md).
