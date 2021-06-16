---
title: 'DNS inversos para servicios de Azure: Azure DNS'
description: Con esta ruta de aprendizaje, empiece a configurar búsquedas inversas de DNS para los servicios hospedados en Azure.
services: dns
documentationcenter: na
author: rohinkoul
ms.service: dns
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/29/2021
ms.author: rohink
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: e9f952f0bb534ddad767280991ab0829ff9fa1b8
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110696954"
---
# <a name="configure-reverse-dns-for-services-hosted-in-azure"></a>Configuración de DNS inversos para servicios hospedados en Azure

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

En este artículo se explica cómo configurar búsquedas inversas de DNS para servicios hospedados en Azure.

Los servicios de Azure utilizan direcciones IP asignadas por Azure y propiedad de Microsoft. Estos registros de DNS inversos (registros PTR) deben crearse en las correspondientes zonas de búsqueda inversa DNS propiedad de Microsoft.

Este escenario difiere de la capacidad de [hospedar las zonas de búsqueda inversa DNS](dns-reverse-dns-hosting.md) para los intervalos IP asignados en Azure DNS. En este caso, los intervalos IP representados por la zona de búsqueda inversa deben asignarse a su organización, normalmente por su ISP.

Antes de leer este artículo, debe familiarizarse con el [DNS inverso en Azure DNS](dns-reverse-dns-overview.md).

En Azure DNS, los recursos de proceso, como las máquinas virtuales, los conjuntos de escalado de máquinas virtuales y los clústeres de Service Fabric tienen direcciones IP públicas. Las búsquedas inversas DNS se configuran mediante la propiedad 'ReverseFqdn' de la dirección IP pública.

Actualmente no se admite DNS inverso para Azure App Service ni Application Gateway.

## <a name="validation-of-reverse-dns-records"></a>Validación de registros de DNS inversos

Un tercero no debe tener acceso para crear registros DNS inversos para la asignación del servicio de Azure a sus dominios de DNS. Este es el motivo por el que Azure solo permite crear un registro DNS inverso si el nombre de dominio es el mismo o se resuelve en una dirección IP pública de la misma suscripción. Esta restricción también se aplica a Cloud Service.

Esta validación solo se realiza cuando se establece o modifica el registro de DNS inverso. No se realiza la revalidación periódica.

Por ejemplo, supongamos que el recurso de la dirección IP pública tiene el nombre DNS `contosoapp1.northus.cloudapp.azure.com` y la dirección IP `23.96.52.53`. El FQDN inverso de la dirección IP pública se puede especificar de la siguiente manera:

* Nombre DNS de la dirección IP pública: `contosoapp1.northus.cloudapp.azure.com`.
* Nombre DNS de una dirección PublicIpAddress diferente de la misma suscripción, como: `contosoapp2.westus.cloudapp.azure.com`.
* Nombre DNS personalizado, como: `app1.contoso.com`. Siempre que el nombre se configure *primero* como un CNAME que apunte a `contosoapp1.northus.cloudapp.azure.com`. El nombre también puede apuntar a una dirección IP pública diferente de la misma suscripción.
* Nombre DNS personalizado, como: `app1.contoso.com`. Siempre que este nombre se configure *primero* como un registro A que apunte a la dirección IP 23.96.52.53. El nombre también puede apuntar a otra dirección IP de la misma suscripción.

Se aplican las mismas restricciones a DNS inverso para Cloud Services.

## <a name="reverse-dns-for-public-ip-address-resources"></a>DNS inverso para recursos de dirección IP pública

Esta sección proporciona instrucciones detalladas acerca de cómo configurar DNS inverso para recursos de dirección IP pública en el modelo de implementación de Resource Manager. Puede usar Azure PowerShell, la CLI clásica de Azure o la CLI de Azure realizar esta tarea. Actualmente no se admite la configuración de DNS inverso para un recurso de dirección IP pública en Azure Portal.

Actualmente, Azure solo admite un DNS inverso para los recursos de dirección IPv4 pública.

### <a name="add-reverse-dns-to-an-existing-publicipaddresses"></a>Incorporación de DNS inversos a PublicIpAddresses existentes

#### <a name="azure-powershell"></a>Azure PowerShell

Para actualizar un DNS inverso a una PublicIpAddress existente:

```azurepowershell-interactive
$pip = Get-AzPublicIpAddress -Name "PublicIp" -ResourceGroupName "MyResourceGroup"
$pip.DnsSettings.ReverseFqdn = "contosoapp1.westus.cloudapp.azure.com."
Set-AzPublicIpAddress -PublicIpAddress $pip
```

Para agregar un DNS inverso a una PublicIpAddress existente que no tenga aún un nombre DNS, debe especificar también un nombre DNS:

```azurepowershell-interactive
$pip = Get-AzPublicIpAddress -Name "PublicIp" -ResourceGroupName "MyResourceGroup"
$pip.DnsSettings = New-Object -TypeName "Microsoft.Azure.Commands.Network.Models.PSPublicIpAddressDnsSettings"
$pip.DnsSettings.DomainNameLabel = "contosoapp1"
$pip.DnsSettings.ReverseFqdn = "contosoapp1.westus.cloudapp.azure.com."
Set-AzPublicIpAddress -PublicIpAddress $pip
```

#### <a name="azure-classic-cli"></a>CLI de Azure clásica

Para agregar un DNS inverso a una PublicIpAddress existente:

```azurecli
azure network public-ip set -n PublicIp -g MyResourceGroup -f contosoapp1.westus.cloudapp.azure.com.
```

Para agregar un DNS inverso a una PublicIpAddress existente que no tenga aún un nombre DNS, debe especificar también un nombre DNS:

```azurecli-interactive
azure network public-ip set -n PublicIp -g MyResourceGroup -d contosoapp1 -f contosoapp1.westus.cloudapp.azure.com.
```

#### <a name="azure-cli"></a>Azure CLI

Para agregar un DNS inverso a una PublicIpAddress existente:

```azurecli-interacgive
az network public-ip update --resource-group MyResourceGroup --name PublicIp --reverse-fqdn contosoapp1.westus.cloudapp.azure.com.
```

Para agregar un DNS inverso a una PublicIpAddress existente que no tenga aún un nombre DNS, debe especificar también un nombre DNS:

```azurecli-interactive
az network public-ip update --resource-group MyResourceGroup --name PublicIp --reverse-fqdn contosoapp1.westus.cloudapp.azure.com --dns-name contosoapp1
```

### <a name="create-a-public-ip-address-with-reverse-dns"></a>Creación de una dirección IP pública con un DNS inverso

Para crear una nueva PublicIpAddress con la propiedad de DNS inverso ya especificada:

#### <a name="azure-powershell"></a>Azure PowerShell

```azurepowershell-interactive
New-AzPublicIpAddress -Name "PublicIp" -ResourceGroupName "MyResourceGroup" -Location "WestUS" -AllocationMethod Dynamic -DomainNameLabel "contosoapp2" -ReverseFqdn "contosoapp2.westus.cloudapp.azure.com."
```

#### <a name="azure-classic-cli"></a>CLI de Azure clásica

```azurecli
azure network public-ip create -n PublicIp -g MyResourceGroup -l westus -d contosoapp3 -f contosoapp3.westus.cloudapp.azure.com.
```

#### <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
az network public-ip create --name PublicIp --resource-group MyResourceGroup --location westcentralus --dns-name contosoapp1 --reverse-fqdn contosoapp1.westcentralus.cloudapp.azure.com
```

### <a name="view-reverse-dns-for-an-existing-publicipaddress"></a>Visualización de DNS inverso para una PublicIpAddress existente

Para ver el valor configurado para una PublicIpAddress existente:

#### <a name="azure-powershell"></a>Azure PowerShell

```azurepowershell-interactive
Get-AzPublicIpAddress -Name "PublicIp" -ResourceGroupName "MyResourceGroup"
```

#### <a name="azure-classic-cli"></a>CLI de Azure clásica

```azurecli
azure network public-ip show -n PublicIp -g MyResourceGroup
```

#### <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
az network public-ip show --name PublicIp --resource-group MyResourceGroup
```

### <a name="remove-reverse-dns-from-existing-public-ip-addresses"></a>Eliminación de DNS inversos de direcciones IP públicas existentes

Para quitar una propiedad de DNS inverso de una PublicIpAddress existente:

#### <a name="azure-powershell"></a>Azure PowerShell

```azurepowershell-interactive
$pip = Get-AzPublicIpAddress -Name "PublicIp" -ResourceGroupName "MyResourceGroup"
$pip.DnsSettings.ReverseFqdn = ""
Set-AzPublicIpAddress -PublicIpAddress $pip
```

#### <a name="azure-classic-cli"></a>CLI de Azure clásica

```azurecli
azure network public-ip set -n PublicIp -g MyResourceGroup –f ""
```

#### <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
az network public-ip update --resource-group MyResourceGroup --name PublicIp --reverse-fqdn ""
```

## <a name="configure-reverse-dns-for-cloud-services"></a>Configuración de DNS inverso para Cloud Services

Esta sección proporciona instrucciones detalladas sobre cómo configurar DNS inverso para Cloud Services en el modelo de implementación clásico, mediante Azure PowerShell. No se admite la configuración de DNS inverso para Cloud Services mediante Azure Portal, la CLI clásica de Azure ni la CLI de Azure.

### <a name="add-reverse-dns-to-existing-cloud-services"></a>Adición de DNS inverso a Cloud Services existentes

Para agregar un registro DNS inverso a un servicio en la nube existente:

```powershell
Set-AzureService –ServiceName "contosoapp1" –Description "App1 with Reverse DNS" –ReverseDnsFqdn "contosoapp1.cloudapp.net."
```

### <a name="create-a-cloud-service-with-reverse-dns"></a>Creación de un servicio en la nube con un DNS inverso

Para crear un nuevo servicio en la nube con la propiedad de DNS inverso ya especificada:

```powershell
New-AzureService –ServiceName "contosoapp1" –Location "West US" –Description "App1 with Reverse DNS" –ReverseDnsFqdn "contosoapp1.cloudapp.net."
```

### <a name="view-reverse-dns-for-existing-cloud-services"></a>Visualización de DNS inverso para las instancias de Cloud Services existentes

Para ver la propiedad de DNS inverso de un servicio en la nube existente:

```powershell
Get-AzureService "contosoapp1"
```

### <a name="remove-reverse-dns-from-existing-cloud-services"></a>Eliminación de DNS inverso de instancias de Cloud Services existentes

Para quitar un propiedad de DNS inverso de un servicio en la nube existente:

```powershell
Set-AzureService –ServiceName "contosoapp1" –Description "App1 with Reverse DNS" –ReverseDnsFqdn ""
```

## <a name="faq"></a>Preguntas más frecuentes

### <a name="how-much-do-reverse-dns-records-cost"></a>¿Cuánto cuestan los registros de DNS inversos?

Son gratuitos. No existe ningún costo adicional para las consultas o los registros de DNS inversos.

### <a name="will-my-reverse-dns-records-resolve-from-the-internet"></a>¿Se resolverán mis registros de DNS inversos desde Internet?

Sí. Después de configurar la propiedad de DNS inverso para el servicio de Azure, Azure administra todas las delegaciones y zonas DNS necesarias para garantizar que se resuelve para todos los usuarios de Internet.

### <a name="are-default-reverse-dns-records-created-for-my-azure-services"></a>¿Se crean registros de DNS inverso predeterminados para mis servicios de Azure?

No. El DNS inverso es una característica opcional. Si decide no configurarla, no se crea ningún registro de DNS inverso predeterminado.

### <a name="what-is-the-format-for-the-fully-qualified-domain-name-fqdn"></a>¿Cuál es el formato del nombre de dominio completo (FQDN)?

Los FQDN se especifican en orden progresivo y deben terminar con un punto (por ejemplo, "app1.contoso.com.").

### <a name="what-happens-if-the-validation-check-for-the-reverse-dns-ive-specified-fails"></a>¿Qué ocurre si no se supera la comprobación de validación para el DNS inverso que he especificado?

Cuando se produce un error en la comprobación de validación de DNS inverso, se produce un error en la operación para configurar el registro de DNS inverso. Corrija el valor de DNS inverso según sea necesario y vuelva a intentarlo.

### <a name="can-i-configure-reverse-dns-for-azure-app-service"></a>¿Puedo configurar DNS inverso para Azure App Service?

No. No se admite el DNS inverso para Azure App Service.

### <a name="can-i-configure-multiple-reverse-dns-records-for-my-azure-service"></a>¿Puedo configurar varios registros de DNS inversos para mi servicio de Azure?

No. Azure admite un único registro de DNS inverso por cada servicio en la nube de Azure o PublicIpAddress.

### <a name="can-i-configure-reverse-dns-for-ipv6-publicipaddress-resources"></a>¿Puedo configurar DNS inverso para los recursos de PublicIpAddress de IPv6?

No. Azure admite un DNS inverso en la actualidad solo para los recursos de PublicIpAddress de IPv4 y Cloud Services.

### <a name="can-i-send-emails-to-external-domains-from-my-azure-compute-services"></a>¿Puedo enviar correos electrónicos a dominios externos desde mis instancias de Azure Compute Services?

La capacidad técnica para enviar correos electrónicos directamente desde una implementación de Azure depende del tipo de suscripción. Independientemente del tipo de suscripción, Microsoft recomienda el uso de servicios de retransmisión de correo de confianza para enviar correo saliente. Para obtener más información, consulte [Seguridad de Azure mejorada para el envío de correos electrónicos: actualización de noviembre de 2017](../virtual-network/troubleshoot-outbound-smtp-connectivity.md).

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre los registros de DNS inverso, consulte información sobre la [búsqueda de DNS inverso en Wikipedia](https://en.wikipedia.org/wiki/Reverse_DNS_lookup).
* Aprenda cómo [hospedar la zona de búsqueda inversa para el intervalo IP asignado por el ISP en DNS de Azure](dns-reverse-dns-for-azure-services.md).
