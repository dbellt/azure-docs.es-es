---
title: Ejemplos de plantilla de Azure Resource Manager para una red virtual | Microsoft Docs
description: Conozca las diferentes plantillas de Azure Resource Manager disponibles con las que se pueden implementar redes virtuales de Azure.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: mtillman
editor: ''
tags: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: sample
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 04/22/2019
ms.author: kumud
ms.openlocfilehash: 580cd2c72374a1c54664c8bef5c9d11c79336e88
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111949678"
---
# <a name="azure-resource-manager-template-samples-for-virtual-network"></a>Ejemplos de plantilla de Azure Resource Manager para una red virtual

En la tabla siguiente se incluyen vínculos a ejemplos de plantilla de Azure Resource Manager. Las plantillas se pueden implementar mediante Azure [Portal](../azure-resource-manager/templates/deploy-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json), la [CLI](../azure-resource-manager/templates/deploy-cli.md?toc=%2fazure%2fvirtual-network%2ftoc.json) de Azure o Azure [PowerShell](../azure-resource-manager/templates/deploy-powershell.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Para aprender a crear sus propias plantillas, consulte [Creación e implementación de la primera plantilla de Azure Resource Manager](../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) y [Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager](../azure-resource-manager/templates/syntax.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

Para la sintaxis y las propiedades de JSON que se usan en una plantilla, consulte [Tipos de recursos de Microsoft.Network](/azure/templates/microsoft.network/allversions).

| Tarea | Descripción |
|----|----|
|[Creación de una red virtual con dos subredes](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.network/vnet-two-subnets/)| Crea una red virtual con dos subredes.|
|[Enrutamiento del tráfico por una aplicación virtual de red](https://github.com/Azure/azure-quickstart-templates/tree/master/201-userdefined-routes-appliance)| Crea una red virtual con tres subredes. Implementa una máquina virtual en cada una de las subredes. Crea una tabla de rutas que contiene las rutas para dirigir el tráfico de una subred a otra a través de la máquina virtual de la tercera subred. Asocia la tabla de rutas a una de las subredes.|
|[Creación de un punto de conexión de servicio de red virtual para Azure Storage](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.network/vnet-2subnets-service-endpoints-storage-integration)|Crea una nueva red virtual con dos subredes y una interfaz de red en cada subred. Habilita un punto de conexión de servicio en Azure Storage para una de las subredes y protege una nueva cuenta de almacenamiento en dicha subred.|
|[Conexión de dos redes virtuales](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vnet-to-vnet-peering)| Crea dos redes virtuales y un emparejamiento de redes virtuales entre ellas.|
|[Creación de una máquina virtual con varias direcciones IP](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.compute/vm-multiple-ipconfig)| Crea una máquina virtual Windows o Linux con varias direcciones IP.|
|[Configuración de una red virtual de doble pila de IPv4 + IPv6](https://github.com/Azure/azure-quickstart-templates/tree/master/demos/ipv6-in-vnet)|Implementa la red virtual de doble pila (IPv4 + IPv6) con dos máquinas virtuales y una instancia de Azure Basic Load Balancer con las direcciones IP públicas IPv4 e IPv6. |