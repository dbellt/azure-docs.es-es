---
title: 'Inicio rápido: Creación de una zona y un registro de Azure DNS: Azure PowerShell'
titleSuffix: Azure DNS
description: Obtenga información sobre cómo crear una zona y un registro DNS en Azure DNS. Este es un inicio rápido paso a paso para crear y administrar su primera zona y su primer registro DNS con Azure PowerShell.
services: dns
author: rohinkoul
ms.author: rohink
ms.date: 04/23/2021
ms.topic: quickstart
ms.service: dns
ms.custom:
- devx-track-azurepowershell
- mode-api
ms.openlocfilehash: 431afacd8e8097a60c32e5e286026f33dbfa7d8c
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107949999"
---
# <a name="quickstart-create-an-azure-dns-zone-and-record-using-azure-powershell"></a>Inicio rápido: Creación de una zona y un registro de Azure DNS mediante Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

En este inicio rápido, creará su primera zona y su primer registro DNS mediante Azure PowerShell. También puede llevar a cabo estos pasos con [Azure Portal](dns-getstarted-portal.md) o la [CLI de Azure](dns-getstarted-cli.md). 

Una zona DNS se usa para hospedar los registros DNS de un dominio concreto. Para iniciar el hospedaje de su dominio en DNS de Azure, debe crear una zona DNS para ese nombre de dominio. Cada registro DNS del dominio se crea luego en esta zona DNS. Finalmente, para publicar la zona DNS en Internet, debe configurar los servidores de nombres para el dominio. A continuación, se describen cada uno de estos pasos.

:::image type="content" source="media/dns-getstarted-portal/environment-diagram.png" alt-text="Diagrama del entorno de implementación de DNS mediante el uso de Azure PowerShell." border="false":::

Azure DNS también admite la creación de dominios privados. Para obtener instrucciones paso a paso sobre cómo crear la primera zona DNS privada y registrarla, consulte [Introducción al uso de PowerShell con las zonas privadas de Azure DNS](private-dns-getstarted-powershell.md).

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Azure PowerShell instalado localmente o Azure Cloud Shell

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-the-resource-group"></a>Creación del grupo de recursos

Antes de crear la zona DNS, cree un grupo de recursos que la contenga:

```powershell
New-AzResourceGroup -name MyResourceGroup -location "eastus"
```

## <a name="create-a-dns-zone"></a>Creación de una zona DNS

Una zona DNS se crea mediante el cmdlet `New-AzDnsZone` . En el ejemplo siguiente, se crea una zona DNS llamada *contoso.xyz* en el grupo de recursos *MyResourceGroup*. Utilice el ejemplo para crear una zona DNS, sustituyendo los valores por los suyos.

```powershell
New-AzDnsZone -Name contoso.xyz -ResourceGroupName MyResourceGroup
```

## <a name="create-a-dns-record"></a>Creación de un registro de DNS

Los conjuntos de registros se crean mediante el cmdlet `New-AzDnsRecordSet`. En el ejemplo siguiente se crea un registro con el nombre relativo "www" en la zona DNS "contoso.xyz" del grupo de recursos "MyResourceGroup". El nombre completo del conjunto de registros es "www.contoso.xyz". El tipo de registro es "A", con la dirección IP 10.10.10.10 y el valor de TTL es de 3600 segundos.

```powershell
New-AzDnsRecordSet -Name www -RecordType A -ZoneName contoso.xyz -ResourceGroupName MyResourceGroup -Ttl 3600 -DnsRecords (New-AzDnsRecordConfig -IPv4Address "10.10.10.10")
```

## <a name="view-records"></a>Visualización de los registros

Para enumerar los registros DNS de su zona, use lo siguiente:

```powershell
Get-AzDnsRecordSet -ZoneName contoso.xyz -ResourceGroupName MyResourceGroup
```

## <a name="test-the-name-resolution"></a>Probar la resolución de nombres

Ahora que tiene una zona DNS de prueba, con un registro "A" de prueba, puede probar la resolución de nombres con una herramienta llamada *nslookup*. 

**Para probar la resolución de nombres DNS:**

1. Ejecute el siguiente cmdlet para obtener la lista de servidores de nombres para la zona:

   ```azurepowershell
   Get-AzDnsRecordSet -ZoneName contoso.xyz -ResourceGroupName MyResourceGroup -RecordType ns
   ```

1. Copie uno de los nombres de servidor de la salida del paso anterior.

1. Abra un símbolo del sistema y ejecute el comando siguiente:

   ```
   nslookup www.contoso.xyz <name server name>
   ```

   Por ejemplo:

   ```
   nslookup www.contoso.xyz ns1-08.azure-dns.com.
   ```

   Debe ver algo parecido a la pantalla siguiente:

   ![La captura de pantalla muestra una ventana del símbolo del sistema con un comando de búsqueda ns y los valores de Servidor, Dirección, Nombre y Dirección.](media/dns-getstarted-portal/nslookup.PNG)

El nombre de host **www\.contoso.xyz** se resuelve en **10.10.10.10**, que es como lo ha configurado. Este resultado confirma que la resolución de nombres funciona correctamente.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, puede eliminar todos los recursos creados en este inicio rápido mediante la eliminación del grupo de recursos:

```powershell
Remove-AzResourceGroup -Name MyResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha creado su primera zona y su primer registro DNS con Azure PowerShell, puede crear registros para una aplicación web de un dominio personalizado.

> [!div class="nextstepaction"]
> [Creación de registros DNS para una aplicación web en un dominio personalizado](./dns-web-sites-custom-domain.md)
