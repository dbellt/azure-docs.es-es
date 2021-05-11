---
title: Importar y exportar un archivo de zona de dominio de la CLI de Azure
titleSuffix: Azure DNS
description: Aprenda a importar y exportar un archivo de zona DNS en Azure DNS (Sistema de nombres de dominio) mediante la CLI de Azure
services: dns
author: rohinkoul
ms.service: dns
ms.date: 04/29/2021
ms.author: rohink
ms.topic: how-to
ms.openlocfilehash: 949e497057646507fcaa04907d032beb5b8ef47c
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108316900"
---
# <a name="import-and-export-a-dns-zone-file-using-the-azure-cli"></a>Importación y exportación de un archivo de zona DNS mediante la CLI de Azure

En este artículo, aprenderá a importar y exportar un archivo de zona DNS en Azure DNS mediante la CLI de Azure.

## <a name="introduction-to-dns-zone-migration"></a>Introducción a la migración de zona DNS

Un archivo de zona DNS es un archivo de texto que contiene la información de cada registro de Sistema de nombres de dominio (DNS) de la zona. Sigue un formato estándar, por lo que es adecuado para transferir registros DNS entre distintos sistemas DNS. El uso de un archivo de zona representa una manera rápida y cómoda de importar zonas DNS en Azure DNS. También puede exportar un archivo de zona desde Azure DNS para usarlo con otros sistemas DNS.

DNS de Azure admite la importación y la exportación de archivos de zona mediante la interfaz de la línea de comandos (CLI) de Azure. La importación de archivos de zona **no** se admite actualmente con Azure PowerShell o Azure Portal.

La CLI de Azure es una herramienta de línea de comandos multiplataforma que se usa para administrar servicios de Azure. Está disponible para las plataformas Windows, Mac y Linux en la [página de descargas de Azure](https://azure.microsoft.com/downloads/).

## <a name="obtain-your-existing-dns-zone-file"></a>Obtención del archivo de zona DNS existente

Antes de importar un archivo de zona DNS a DNS de Azure, debe obtener una copia del archivo de zona. El origen de este archivo varía en función de dónde se hospede la zona DNS.

* Si la zona DNS está hospedada actualmente por un servicio de asociados, tendrá una manera de descargar el archivo de zona DNS. Entre los servicios asociados se incluyen el registrador de dominios, el proveedor de hospedaje de DNS dedicado o un proveedor de nube alternativo.
* Si la zona DNS se hospeda en DNS de Windows, la carpeta predeterminada para los archivos de zona es **%systemroot%\system32\dns**. También se muestra la ruta de acceso completa de cada archivo de zona en la pestaña **General** de la consola de DNS.
* Si la zona DNS se hospeda con BIND, la ubicación del archivo de zona para cada zona se especifica en el archivo de configuración de BIND **named.conf**.

## <a name="import-a-dns-zone-file-into-azure-dns"></a>Importación de un archivo de zona DNS a DNS de Azure

Si aún no existe, al importar un archivo de zona se crea una nueva zona de Azure DNS. Si la zona ya existe, los conjuntos de registros del archivo de zona se combinarán con los conjuntos de registros existentes.

### <a name="merge-behavior"></a>Comportamiento de combinación

* De forma predeterminada, los nuevos conjuntos de registros se combinan con los conjuntos de registros existentes. Los registros idénticos dentro de un conjunto de registros combinado no se duplican.
* Cuando se combinan conjuntos de registros, se usa el período de vida (TTL) de los conjuntos de registros existentes.
* Los parámetros de inicio de autoridad (SOA), a excepción de `host`, siempre se toman del archivo de zona importado. El conjunto de registros de servidor de nombres en el vértice de la zona usará siempre el TTL tomado del archivo de zona importado.
* Un registro CNAME importado no reemplaza a un registro CNAME existente con el mismo nombre.  
* Cuando se produce un conflicto entre un registro CNAME y otro registro con el mismo nombre de tipo diferente, se usa el registro existente.

### <a name="additional-information-about-importing"></a>Información adicional sobre la importación

En las notas siguientes, se ofrecen detalles técnicos adicionales sobre el proceso de importación de zona.

* La directiva `$TTL` es opcional y se admite. Cuando no se indica ninguna directiva `$TTL`, los registros sin TTL explícito se importan con un TTL predeterminado de 3600 segundos. Cuando dos registros del mismo conjunto de registros especifican diferentes TTL, se usa el valor más bajo.
* La directiva `$ORIGIN` es opcional y se admite. Cuando no se establece ninguna directiva `$ORIGIN`, el valor predeterminado que se usa es el nombre de la zona según lo especificado en la línea de comandos, incluido el final en ".".
* No se admiten las directivas `$INCLUDE` y `$GENERATE`.
* Se admiten los tipos de registro siguientes: A, AAAA, CAA, CNAME, MX, NS, SOA, SRV y TXT.
* DNS de Azure crea automáticamente el registro SOA cuando se crea una zona. Cuando se importa un archivo de zona, todos los parámetros SOA se toman del archivo de zona *excepto* el parámetro `host`. Este parámetro usa el valor proporcionado por Azure DNS porque debe hacer referencia al servidor de nombres principal proporcionado por Azure DNS.
* DNS de Azure también crea automáticamente el conjunto de registros de servidor de nombres en el ápice de zona al crear la zona. Solo se importa el TTL de este conjunto de registros. Estos registros contienen los nombres de servidores de nombres proporcionados por DNS de Azure. No se sobrescriben los datos del registro con los valores contenidos en el archivo de zona importado.
* Durante la versión preliminar pública, DNS de Azure admite solamente registros TXT de cadena única. Los registros TXT de varias cadenas se concatenarán y truncarán a 255 caracteres.

### <a name="cli-format-and-values"></a>Valores y formato de la CLI

El formato del comando de CLI de Azure para importar una zona DNS es:

```azurecli-interactive-interactive
az network dns zone import -g <resource group> -n <zone name> -f <zone file name>
```

Valores:

* `<resource group>` es el nombre del grupo de recursos para la zona en DNS de Azure.
* `<zone name>` es el nombre de la zona.
* `<zone file name>` es la ruta de acceso y el nombre del archivo de zona que se va a importar.

Si no existe aún una zona con este nombre en el grupo de recursos, se creará automáticamente. En una zona existente, los conjuntos de registros importados se combinarán con los conjuntos de registros existentes. 

### <a name="import-a-zone-file"></a>Importación de un archivo de zona

Para importar un archivo de zona para la zona **contoso.com**.

1. Cree un grupo de recursos si no tiene ninguno.

    ```azurecli-interactive
    az group create --resource-group myresourcegroup -l westeurope
    ```

1. Para importar la zona **contoso.com** del archivo **contoso.com.txt** a una nueva zona DNS en el grupo de recursos **myresourcegroup**, ejecutará el comando `az network dns zone import`.

    Con este comando se carga el archivo de zona y se analiza. El comando ejecuta una serie de operaciones en el servicio Azure DNS para crear la zona y todos los conjuntos de registros de la zona. El comando notificará el progreso en la ventana de la consola, junto con los errores o las advertencias. Puesto que los conjuntos de registros se crean en serie, puede tardar unos minutos en importar un archivo de zona de gran tamaño.

    ```azurecli-interactive
    az network dns zone import -g myresourcegroup -n contoso.com -f contoso.com.txt
    ```

### <a name="verify-the-zone"></a>Comprobación de la zona

Puede usar cualquiera de los métodos siguientes para comprobar la zona DNS después de importar el archivo:

* Para ver los registros en una lista, use el siguiente comando de la CLI de Azure:

    ```azurecli-interactive
    az network dns record-set list -g myresourcegroup -z contoso.com
    ```

* También puede mostrar una lista de los registros mediante el siguiente comando de la CLI de Azure `az network dns record-set ns list`.
* Use `nslookup` para comprobar la resolución de nombres de los registros. Como la zona aún no está delegada, debe especificar explícitamente los servidores de nombres de Azure DNS correctos. En el ejemplo siguiente se muestra cómo recuperar los nombres de servidores de nombres asignados a la zona.

    ```azurecli-interactive
    az network dns record-set ns list -g myresourcegroup -z contoso.com  --output json 
    ```

    ```json
    [
      {
       .......
       "name": "@",
        "nsRecords": [
          {
            "additionalProperties": {},
            "nsdname": "ns1-03.azure-dns.com."
          },
          {
            "additionalProperties": {},
            "nsdname": "ns2-03.azure-dns.net."
          },
          {
            "additionalProperties": {},
            "nsdname": "ns3-03.azure-dns.org."
          },
          {
            "additionalProperties": {},
            "nsdname": "ns4-03.azure-dns.info."
          }
        ],
        "resourceGroup": "myresourcegroup",
        "ttl": 86400,
        "type": "Microsoft.Network/dnszones/NS"
      }
    ]
    ```

    Use el símbolo del sistema de Windows para consultar el registro "www" con el comando `nslookup`.

    ```cmd
    nslookup www.contoso.com ns1-03.azure-dns.com

        Server: ns1-01.azure-dns.com
        Address:  40.90.4.1

        Name:www.contoso.com
        Addresses:  134.170.185.46
        134.170.188.221
    ```

### <a name="update-dns-delegation"></a>Actualización de la delegación de DNS

Una vez que haya comprobado que la zona se ha importado correctamente, debe actualizar la delegación de DNS para que apunte a los servidores de nombres DNS de Azure. Para más información, consulte [Actualización de la delegación DNS](dns-domain-delegation.md).

## <a name="export-a-dns-zone-file-from-azure-dns"></a>Exportación de un archivo de zona DNS de DNS de Azure

Para exportar una zona DNS, use el siguiente comando de la CLI de Azure:

```azurecli-interactive
az network dns zone export -g <resource group> -n <zone name> -f <zone file name>
```

Valores:

* `<resource group>` es el nombre del grupo de recursos para la zona en DNS de Azure.
* `<zone name>` es el nombre de la zona.
* `<zone file name>` es la ruta de acceso y el nombre del archivo de zona que se va a exportar.

Al igual que con la importación de zona, en primer lugar necesita iniciar sesión, elegir su suscripción y configurar la CLI de Azure para que use el modo de Resource Manager.

### <a name="to-export-a-zone-file"></a>Para exportar un archivo de zona

Para exportar la zona DNS de Azure existente **contoso.com** en el grupo de recursos **myresourcegroup** al archivo **contoso.com.txt** (en la carpeta actual), ejecute `azure network dns zone export`. Este comando llama al servicio DNS de Azure para enumerar los conjuntos de registros de la zona y exportar los resultados a un archivo de zona compatible con BIND.

```azurecli-interactive
az network dns zone export -g myresourcegroup -n contoso.com -f contoso.com.txt
```

## <a name="next-steps"></a>Pasos siguientes

* En esta guía se explica cómo [administrar conjuntos de registros y registros](./dns-getstarted-cli.md) en la zona DNS.

* Aprenda a cómo [delegar el dominio a Azure DNS](dns-domain-delegation.md).
