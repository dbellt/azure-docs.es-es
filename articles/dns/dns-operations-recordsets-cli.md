---
title: Administre registros de DNS en la DNS de Azure mediante la CLI de Azure.
description: Administración de conjuntos de registros y registros DNS en DNS de Azure al hospedar dominios en DNS de Azure.
author: rohinkoul
ms.assetid: 5356a3a5-8dec-44ac-9709-0c2b707f6cb5
ms.service: dns
ms.devlang: azurecli
ms.topic: how-to
ms.custom: H1Hack27Feb2017, devx-track-azurecli
ms.workload: infrastructure-services
ms.date: 04/28/2021
ms.author: rohink
ms.openlocfilehash: 52e82e52ba0476aba02b4d083537e085181bbde9
ms.sourcegitcommit: 49bd8e68bd1aff789766c24b91f957f6b4bf5a9b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108228077"
---
# <a name="manage-dns-records-and-recordsets-in-azure-dns-using-the-azure-cli"></a>Administración de conjuntos de registros y registros de DNS en Azure DNS mediante la CLI de Azure

> [!div class="op_single_selector"]
> * [Azure Portal](dns-operations-recordsets-portal.md)
> * [CLI de Azure](dns-operations-recordsets-cli.md)
> * [PowerShell](dns-operations-recordsets.md)

En este artículo se muestra cómo administrar registros DNS para su zona DNS mediante la CLI de Azure entre plataformas. La CLI de Azure está disponible para Windows, Mac y Linux. También puede administrar sus registros de DNS mediante [Azure PowerShell](dns-operations-recordsets.md) o [Azure Portal](dns-operations-recordsets-portal.md).

En los ejemplos de este artículo se supone que ya ha [instalado la CLI de Azure, iniciado sesión y creado una zona DNS](dns-operations-dnszones-cli.md).

## <a name="introduction"></a>Introducción

Antes de crear registros DNS en Azure DNS, es necesario que comprenda cómo Azure DNS los organiza en conjuntos de registros DNS.

[!INCLUDE [dns-about-records-include](../../includes/dns-about-records-include.md)]

Para más información sobre los registros DNS en Azure DNS, consulte [DNS zones and records](dns-zones-records.md) (Zonas y registros DNS).

## <a name="create-a-dns-record"></a>Creación de un registro DNS

Para crear un registro DNS, use el comando `az network dns record-set <record-type> add-record` (donde `<record-type>` es el tipo de registro, es decir, a, srv, txt, etc.) Para obtener ayuda, consulte `az network dns record-set --help`.

Al crear un registro, tiene que especificar la siguiente información: 

* Definición de un nombre de grupo de recursos
* Nombre de zona
* Nombre del conjunto de registros
* Tipo de registro

El nombre del conjunto de registros especificado debe ser un nombre *relativo*; es decir, debe excluir el nombre de zona. Si el conjunto de registros aún no existe, este comando lo creará automáticamente. Sin embargo, este comando agregará el registro que especifique si el conjunto de registros ya existe.

Si se crea un conjunto de registros, se utiliza el valor predeterminado del período de vida (3600). Para obtener instrucciones sobre cómo usar diferentes TTL, consulte [Creación de un conjunto de registros de DNS](#create-a-dns-record-set).

En el ejemplo siguiente se crea un registro A denominado *www* en la zona *contoso.com* del grupo de recursos *MyResourceGroup*. La dirección IP del registro A es *1.2.3.4*.

```azurecli-interactive
az network dns record-set a add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name www --ipv4-address 1.2.3.4
```

Para crear un conjunto de registros en el vértice de la zona (en este caso, "contoso.com"), utilice el nombre de registro "\@", incluidas las comillas:

```azurecli-interactive
az network dns record-set a add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name "@" --ipv4-address 1.2.3.4
```

## <a name="create-a-dns-record-set"></a>Creación de un conjunto de registros de DNS

En los ejemplos anteriores, bien se agregó el registro de DNS en un conjunto de registros existente, bien se creó el conjunto de registros *implícitamente*. También puede crear el conjunto de registros *explícitamente* antes de agregarle registros. La DNS de Azure admite conjuntos de registros "vacíos" que pueden funcionar como marcador de posición para reservar un nombre DNS antes de crear registros de DNS. Los conjuntos de registros vacíos son visibles en el panel de control de Azure DNS, pero no aparecen en los servidores de nombres de Azure DNS.

Los conjuntos de registros se crean con el comando `az network dns record-set <record-type> create`. Para obtener ayuda, consulte `az network dns record-set <record-type> create --help`.

Al crear el conjunto de registros explícitamente, podrá especificar las propiedades de este, como el [período de vida (TTL)](dns-zones-records.md#time-to-live) y los metadatos. Se pueden usar [metadatos del conjunto de registros](dns-zones-records.md#tags-and-metadata) para asociar datos específicos de la aplicación con cada conjunto de registros como pares clave-valor.

En el ejemplo siguiente se crea un conjunto de registros vacío de tipo "A" con un TTL de 60 segundos mediante el parámetro `--ttl` (forma abreviada `-l`):

```azurecli-interactive
az network dns record-set a create --resource-group myresourcegroup --zone-name contoso.com --name www --ttl 60
```

En el siguiente ejemplo, se crea un conjunto de registros con dos entradas de metadatos, "dept=finance" y "environment=production", mediante el parámetro `--metadata`:

```azurecli-interactive
az network dns record-set a create --resource-group myresourcegroup --zone-name contoso.com --name www --metadata "dept=finance" "environment=production"
```

Tras haber creado un conjunto de registros vacío, se podrán añadir el registro mediante `azure network dns record-set <record-type> add-record`, como se describe en [Creación de un registro de DNS](#create-a-dns-record).

## <a name="create-records-of-other-types"></a>Creación de registros de otros tipos

Después de haber visto de forma detallada cómo crear registros "A", en los siguientes ejemplos se muestra cómo crear registros de otros tipos compatibles con DNS de Azure.

Los parámetros utilizados para especificar los datos del registro varían según el tipo del registro. Por ejemplo, para un registro de tipo "A", especifique la dirección IPv4 con el parámetro `--ipv4-address <IPv4 address>`. Es posible obtener una lista de los parámetros de cada tipo de registro con `az network dns record-set <record-type> add-record --help`.

En cada caso, se muestra cómo crear un único registro. Se agrega el registro al conjunto de registros existente o a uno creado de forma implícita. Para obtener más información sobre cómo crear conjuntos de registros y definir explícitamente los parámetros de un conjunto de registros, consulte [Creación de un conjunto de registros de DNS](#create-a-dns-record-set).

No hay ningún ejemplo para crear un conjunto de registros SOA, ya que los SOA se crean y eliminan con cada zona DNS. Los registros SOA no pueden crearse ni eliminarse por separado. Sin embargo, el registro SOA se puede [modificar](#to-modify-an-soa-record), como se muestra en un ejemplo más adelante.

### <a name="create-an-aaaa-record"></a>Creación de un registro AAAA

```azurecli-interactive
az network dns record-set aaaa add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name test-aaaa --ipv6-address 2607:f8b0:4009:1803::1005
```

### <a name="create-a-caa-record"></a>Creación de un registro CAA

```azurecli-interactive
az network dns record-set caa add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name test-caa --flags 0 --tag "issue" --value "ca1.contoso.com"
```

### <a name="create-a-cname-record"></a>Creación de un registro CNAME

> [!NOTE]
> Los estándares DNS no permiten registros CNAME en el vértice de una zona (`--Name "@"`), ni permiten conjuntos de registros que contengan más de un registro.
> 
> Para más información, consulte [Registros CNAME](dns-zones-records.md#cname-records).

```azurecli-interactive
az network dns record-set cname set-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name test-cname --cname www.contoso.com
```

### <a name="create-an-mx-record"></a>Creación de un registro MX

En este ejemplo, se utiliza el nombre de conjunto de registros "\@" para crear el registro MX en el vértice de la zona (en este caso, "contoso.com").

```azurecli-interactive
az network dns record-set mx add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name "@" --exchange mail.contoso.com --preference 5
```

### <a name="create-an-ns-record"></a>Creación de un registro NS

```azurecli-interactive
az network dns record-set ns add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name test-ns --nsdname ns1.contoso.com
```

### <a name="create-a-ptr-record"></a>Creación de un registro PTR

En este caso "my-arpa-zone.com" representa la zona ARPA que representa el intervalo IP. Cada registro PTR establecido en esta zona se corresponde con una dirección IP dentro de este intervalo IP.  El nombre de registro "10" es el último octeto de la dirección IP dentro del intervalo IP que representa dicho registro.

```azurecli-interactive
az network dns record-set ptr add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name my-arpa.zone.com --ptrdname myservice.contoso.com
```

### <a name="create-an-srv-record"></a>Creación de un registro SRV

Al crear un [conjunto de registros SRV](dns-zones-records.md#srv-records), especifique el *\_servicio* y el *\_protocolo* en el nombre del conjunto de registros. No hay necesidad de incluir "\@" en el nombre del conjunto de registros al crear un conjunto de registros SRV en el vértice de la zona.

```azurecli-interactive
az network dns record-set srv add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name _sip._tls --priority 10 --weight 5 --port 8080 --target sip.contoso.com
```

### <a name="create-a-txt-record"></a>Creación de un registro TXT

En el ejemplo siguiente se muestra cómo crear un registro TXT. Para más información sobre la longitud de cadena máxima admitida en registros TXT, consulte [Registros TXT](dns-zones-records.md#txt-records).

```azurecli-interactive
az network dns record-set txt add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name test-txt --value "This is a TXT record"
```

## <a name="get-a-record-set"></a>Recuperación de un conjunto de registros

Para recuperar un conjunto de registros existente, use `az network dns record-set <record-type> show`. Para obtener ayuda, consulte `az network dns record-set <record-type> show --help`.

Al crear un registro o conjunto de registros, el nombre del conjunto de registros especificado debe ser un nombre *relativo*. Este nombre no incluye el nombre de zona. También debe especificar el tipo de registro, la zona que contiene el conjunto de registros y el grupo de recursos que contiene la zona.

En el ejemplo siguiente se recupera el registro *www* de tipo A de la zona *contoso.com*, que se encuentra en el grupo de recursos *MyResourceGroup*:

```azurecli-interactive
az network dns record-set a show --resource-group myresourcegroup --zone-name contoso.com --name www
```

## <a name="list-record-sets"></a>Enumeración de conjuntos de registros

Puede mostrar todos los registros de una zona DNS con el comando `az network dns record-set list` . Para obtener ayuda, consulte `az network dns record-set list --help`.

En este ejemplo, se devuelve todos los conjuntos de registros de la zona *contoso.com*, que se encuentra en el grupo de recursos *MyResourceGroup*:

```azurecli-interactive
az network dns record-set list --resource-group myresourcegroup --zone-name contoso.com
```

En este ejemplo, se devuelven todos los conjuntos de registros que coinciden con el tipo de registro especificado (en este caso, los registros "A"):

```azurecli-interactive
az network dns record-set a list --resource-group myresourcegroup --zone-name contoso.com 
```

## <a name="add-a-record-to-an-existing-record-set"></a>Adición de un registro a un conjunto de registros existente

Puede usar `az network dns record-set <record-type> add-record` tanto para crear un registro en un nuevo conjunto de registros como para agregar un registro a un conjunto de registros existente.

Para obtener más información, consulte las secciones [Creación de un registro de DNS](#create-a-dns-record) y [Creación de registros de otros tipos](#create-records-of-other-types).

## <a name="remove-a-record-from-an-existing-record-set"></a>Quite un registro de un conjunto de registros existente.

Para quitar un registro de DNS de un conjunto de registros existente, use `az network dns record-set <record-type> remove-record`. Para obtener ayuda, consulte `az network dns record-set <record-type> remove-record -h`.

Este comando elimina un registro de DNS de un conjunto de registros. Si se elimina el último registro de un conjunto de registros, el conjunto de registros propiamente dicho también se elimina. Si, por el contrario, desea mantener el registro vacío, use la opción `--keep-empty-record-set`.

Al usar el comando `az network dns record-set <record-type> add-record`, debe especificar el registro que se va a eliminar y la zona de la que se va a eliminar. Estos parámetros se describen en las secciones [Creación de un registro de DNS](#create-a-dns-record) y [Creación de registros de otros tipos](#create-records-of-other-types).

En el ejemplo siguiente se elimina el registro A con el valor "1.2.3.4" del conjunto de registros con el nombre *www* de la zona *contoso.com*, que se encuentra en el grupo de recursos *MyResourceGroup*.

```azurecli-interactive
az network dns record-set a remove-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name "www" --ipv4-address 1.2.3.4
```

## <a name="modify-an-existing-record-set"></a>Modificación de un conjunto de registros existente

Cada conjunto de registros contiene un [time-to-live (TTL)](dns-zones-records.md#time-to-live), [metadatos](dns-zones-records.md#tags-and-metadata)y los registros de DNS. En las siguientes secciones se explican cómo modificar cada una de estas propiedades.

### <a name="to-modify-an-a-aaaa-caa-mx-ns-ptr-srv-or-txt-record"></a>Modificación de un registro A, AAAA, CAA, MX, NS, PTR, SRV o TXT

Para modificar un registro existente de tipo A, AAAA, CAA, MX, NS, PTR, SRV o TXT, debe agregar primero un nuevo registro y, después, eliminar el existente. Para obtener instrucciones detalladas sobre cómo eliminar y agregar registros, consulte las secciones anteriores de este artículo.

En el ejemplo siguiente se muestra cómo modificar un registro "A", de la dirección IP 1.2.3.4 a la 5.6.7.8:

```azurecli-interactive
az network dns record-set a add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name www --ipv4-address 5.6.7.8
az network dns record-set a remove-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name www --ipv4-address 1.2.3.4
```

No puede agregar, quitar ni modificar los registros del conjunto de registros NS creado automáticamente en el vértice de zona (`--Name "@"`, comillas incluidas). En el caso de este conjunto de registros, los únicos cambios permitidos son modificar el TTL del conjunto de registros y los metadatos.

### <a name="to-modify-a-cname-record"></a>Para modificar un registro CNAME

A diferencia de la mayoría de otros tipos de registros, un conjunto de registros CNAME solo puede contener un único registro.  Por eso no puede reemplazar el valor actual agregando un nuevo registro y quitando el existente, como en otros tipos de registros.

En su lugar, para modificar un registro CNAME, use `az network dns record-set cname set-record`. Para obtener ayuda, consulte `az network dns record-set cname set-record --help`.

En el ejemplo se modifica el conjunto de registros CNAME *www* de la zona *contoso.com*, que se encuentra en el grupo de recursos *MyResourceGroup*, para que apunte a "www.fabrikam.net" en lugar de a su valor existente:

```azurecli-interactive
az network dns record-set cname set-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name test-cname --cname www.fabrikam.net
``` 

### <a name="to-modify-an-soa-record"></a>Para modificar un registro SOA

A diferencia de la mayoría de otros tipos de registros, un conjunto de registros SOA solo puede contener un único registro.  Por eso no puede reemplazar el valor actual agregando un nuevo registro y quitando el existente, como en otros tipos de registros.

En su lugar, para modificar el registro SOA, use `az network dns record-set soa update`. Para obtener ayuda, consulte `az network dns record-set soa update --help`.

En el ejemplo siguiente se muestra cómo establecer la propiedad "email" del registro SOA de la zona *contoso.com*:

```azurecli-interactive
az network dns record-set soa update --resource-group myresourcegroup --zone-name contoso.com --email admin.contoso.com
```

### <a name="to-modify-ns-records-at-the-zone-apex"></a>Para modificar los registros NS en el vértice de zona

El conjunto de registros NS en el vértice de zona se crea automáticamente con cada zona DNS. Este conjunto de registros contiene los nombres de los servidores de nombres de Azure DNS asignados a la zona.

Puede agregar más servidores de nombres a este conjunto de registros NS, para admitir dominios de hospedaje conjunto con más de un proveedor DNS. También puede modificar el TTL y los metadatos de este conjunto de registros. Sin embargo, no puede quitar ni modificar los servidores de nombres de Azure DNS rellenados previamente.

Esta restricción se aplica solo al conjunto de registros NS en el vértice de zona. Otros conjuntos de registros NS de su zona (como los que se usan para delegar zonas secundarias) se pueden modificar sin restricciones.

En el ejemplo siguiente se muestra cómo agregar otro servidor de nombres al conjunto de registros NS en el vértice de zona:

```azurecli-interactive
az network dns record-set ns add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name "@" --nsdname ns1.myotherdnsprovider.com 
```

### <a name="to-modify-the-ttl-of-an-existing-record-set"></a>Para modificar el TTL de un conjunto de registros existente

Para modificar el TTL de un conjunto de registros existente, utilice `azure network dns record-set <record-type> update`. Para obtener ayuda, consulte `azure network dns record-set <record-type> update --help`.

En el ejemplo siguiente se muestra cómo modificar el TTL de un conjunto de registros, en este caso a 60 segundos:

```azurecli-interactive
az network dns record-set a update --resource-group myresourcegroup --zone-name contoso.com --name www --set ttl=60
```

### <a name="to-modify-the-metadata-of-an-existing-record-set"></a>Para modificar los metadatos de un conjunto de registros existente

Se pueden usar [metadatos del conjunto de registros](dns-zones-records.md#tags-and-metadata) para asociar datos específicos de la aplicación con cada conjunto de registros como pares clave-valor. Para modificar los metadatos de un conjunto de registros existente, utilice `az network dns record-set <record-type> update`. Para obtener ayuda, consulte `az network dns record-set <record-type> update --help`.

En el ejemplo siguiente se muestra cómo crear un conjunto de registros con dos entradas de metadatos: "dept=finance" y "environment=production". Los metadatos existentes se *sustituyen* por los valores especificados.

```azurecli-interactive
az network dns record-set a update --resource-group myresourcegroup --zone-name contoso.com --name www --set metadata.dept=finance metadata.environment=production
```

## <a name="delete-a-record-set"></a>Eliminación de un conjunto de registros

Los conjuntos de registros pueden eliminarse mediante el comando `az network dns record-set <record-type> delete`. Para obtener ayuda, consulte `azure network dns record-set <record-type> delete --help`. Al eliminar un conjunto de registros también se eliminan todos los registros que contiene.

> [!NOTE]
> No se pueden eliminar los conjuntos de registros SOA y NS en el vértice de zona (`--name "@"`).  Se crean automáticamente cuando se crea la zona y se eliminan automáticamente cuando se elimina.

En el ejemplo siguiente se elimina el conjunto de registros con el nombre *www* de tipo A de la zona *contoso.com*, la cual se encuentra en el grupo de recursos *MyResourceGroup*:

```azurecli-interactive
az network dns record-set a delete --resource-group myresourcegroup --zone-name contoso.com --name www
```

Se le pide que confirme la operación de eliminación. Para suprimir este mensaje, use el modificador `--yes`.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre [zonas y registros en Azure DNS](dns-zones-records.md).
<br>
Aprenda a [proteger las zonas y los registros](dns-protect-zones-recordsets.md) cuando se usa Azure DNS.
