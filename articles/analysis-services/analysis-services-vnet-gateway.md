---
title: Configuración de Azure Analysis Services para orígenes de datos de red virtual | Microsoft Docs
description: Aprenda a configurar un servidor de Azure Analysis Services para que use una puerta de enlace para orígenes de datos en Azure Virtual Network (VNet).
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 04/27/2021
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: c3a0ed0f2c9a52d10d206075d229cd62cd3909db
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108145760"
---
# <a name="use-gateway-for-data-sources-on-an-azure-virtual-network-vnet"></a>Uso de la puerta de enlace para orígenes de datos en una red virtual de Azure (VNet)

En este artículo se describe la propiedad **AlwaysUseGateway** del servidor de Azure Analysis Services que se usa cuando los orígenes de datos se encuentran en una [red virtual de Azure (VNET)](../virtual-network/virtual-networks-overview.md).

## <a name="server-access-to-vnet-data-sources"></a>Acceso del servidor a los orígenes de datos de una red virtual

Si el acceso a sus orígenes de datos se realiza a través de una red virtual, el servidor de Azure Analysis Services debe conectarse, en su propio entorno, a esos orígenes de datos como si estuvieran en el entorno local. Puede configurar la propiedad de servidor **AlwaysUseGateway** para especificar el servidor y acceder a todos los datos de orígenes de datos mediante una [puerta de enlace local](analysis-services-gateway.md). 

Los orígenes de datos de Instancia administrada de Azure SQL se ejecutan dentro de la red virtual de Azure con una dirección IP privada. Si el punto de conexión público está habilitado en la instancia, no se requiere una puerta de enlace. Si el punto de conexión público no está habilitado, se requiere una puerta de enlace de datos local y la propiedad AlwaysUseGateway debe establecerse en true.

> [!NOTE]
> Esta propiedad es efectiva únicamente cuando se instala y configura una [puerta de enlace de datos local](analysis-services-gateway.md). La puerta de enlace puede estar en la red virtual.

## <a name="configure-alwaysusegateway-property"></a>Configuración de la propiedad AlwaysUseGateway

1. En SSMS > servidor > **Propiedades** > **General**, seleccione **Show Advanced (All) Properties** (Mostrar propiedades avanzadas [todas]).
2. En **ASPaaS\AlwaysUseGateway**, seleccione **true**.

    ![Propiedad AlwaysUseGateway](media/analysis-services-vnet-gateway/aas-ssms-always-property.png)


## <a name="see-also"></a>Consulte también
[Conexión a orígenes de datos locales](analysis-services-gateway.md)   
[Instalación y configuración de una puerta de enlace de datos local](analysis-services-gateway-install.md)   
[Azure Virtual Network (VNET)](../virtual-network/virtual-networks-overview.md)   

