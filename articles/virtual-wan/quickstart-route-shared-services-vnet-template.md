---
title: 'Inicio rápido: Enrutamiento a servicios compartidos mediante una plantilla de ARM'
titleSuffix: Azure Virtual WAN
description: Aprenda a configurar rutas para acceder a una red virtual de servicios compartidos con una carga de trabajo a la que quiere que accedan cada red virtual y rama mediante una plantilla de Azure Resource Manager (ARM).
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: quickstart
ms.date: 03/05/2021
ms.author: cherylmc
ms.custom: subject-armqs
ms.openlocfilehash: ff23bf626fe252547d3449cfe318c073e9cc9b6c
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112026316"
---
# <a name="quickstart-route-to-shared-services-vnets-using-an-arm-template"></a>Inicio rápido: Enrutamiento a redes virtuales de servicios compartidos mediante una plantilla de ARM

Este inicio rápido se describe cómo usar una plantilla de Azure Resource Manager (ARM) para configurar rutas de acceso a una red virtual de servicios compartidos con las cargas de trabajo a las que desea que accedan cada red virtual y rama (VPN/ER/P2S). Algunos ejemplos de estas cargas de trabajo compartidas podrían incluir máquinas virtuales con servicios como controladores de dominio o recursos compartidos de archivos, o bien servicios de Azure expuestos internamente mediante un [punto de conexión privado de Azure](../private-link/private-endpoint-overview.md).

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2fquickstarts%2fmicrosoft.network%2fvirtual-wan-with-route-tables%2fazuredeploy.json)

## <a name="prerequisites"></a>Prerrequisitos

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.
* Esta configuración requiere los datos del certificado de clave pública. En el artículo se proporcionan datos de ejemplo. Sin embargo, solo se proporcionan para cumplir los requisitos para crear una puerta de enlace de P2S de la plantilla. Una vez finalizada la plantilla e implementados los recursos, debe actualizar este campo con sus propios datos de certificado para que la configuración funcione. Consulte [Certificados de VPN del usuario](certificates-point-to-site.md#cer).

## <a name="review-the-template"></a><a name="review"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/virtual-wan-with-route-tables). La plantilla de este artículo es demasiado larga para mostrarse aquí. Para ver la plantilla, consulte [azuredeploy.json](https://github.com/Azure/azure-quickstart-templates/blob/master/quickstarts/microsoft.network/virtual-wan-with-route-tables/azuredeploy.json).

En este inicio rápido creará una implementación de varios centros de conectividad de Azure Virtual WAN con todas las puertas de enlace y conexiones de red virtual. La lista de parámetros de entrada se ha reducido al mínimo a propósito. El esquema de direccionamiento IP se puede cambiar si se modifican las variables de la plantilla. El escenario se explica con más detalle en el artículo [Escenario: Enrutamiento a redes virtuales de servicios compartidos](scenario-shared-services-vnet.md).

:::image type="content" source="./media/routing-scenarios/shared-service-vnet/shared-services.png" alt-text="Arquitectura de implementación" lightbox="./media/routing-scenarios/shared-service-vnet/shared-services.png":::

Con esta plantilla se crea un entorno de Azure Virtual WAN totalmente funcional con los siguientes recursos:

* Dos centros de conectividad distintos en regiones diferentes.
* Cuatro redes virtuales de Azure.
* Dos conexiones de red virtual para cada centro de conectividad de VWAN.
* Una puerta de enlace de VPN de punto a sitio (P2S) en cada centro de conectividad.
* Una puerta de enlace de VPN de sitio a sitio (S2S) en cada centro de conectividad.
* Una puerta de enlace de ExpressRoute en cada centro de conectividad.
* Tablas de rutas personalizadas RT_SHARED en cada concentrador.
* Una etiqueta LBL_RT_SHARED para agrupar las tablas de rutas RT_SHARED.

En la plantilla se definen varios recursos de Azure:

* [**Microsoft.Network/virtualwans**](/azure/templates/microsoft.network/virtualwans)
* [**Microsoft.Network/virtualhubs**](/azure/templates/microsoft.network/virtualhubs)
* [**Microsoft.Network/virtualnetworks**](/azure/templates/microsoft.network/virtualnetworks)
* [**Microsoft.Network/hubvirtualnetworkconnections**](/azure/templates/microsoft.network/virtualhubs/hubvirtualnetworkconnections)
* [**Microsoft.Network/hubroutetables**](/azure/templates/microsoft.network/virtualhubs/hubRouteTables)
* [**Microsoft.Network/p2svpngateways**](/azure/templates/microsoft.network/p2svpngateways)
* [**Microsoft.Network/vpngateways**](/azure/templates/microsoft.network/vpngateways)
* [**Microsoft.Network/expressroutegateways**](/azure/templates/microsoft.network/expressroutegateways)
* [**Microsoft.Network/vpnserverconfigurations**](/azure/templates/microsoft.network/vpnServerConfigurations)

>[!NOTE]
> Con esta plantilla de ARM no se crean los recursos del lado del cliente necesarios para la conectividad híbrida. Después de implementar la plantilla, tiene que crear y configurar los clientes VPN de punto a sitio y las ramas de VPN (sitios locales), y conectar los circuitos de ExpressRoute.
>

Para buscar más plantillas, consulte [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Network&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a><a name="deploy"></a>Implementación de la plantilla

Para implementar esta plantilla correctamente, en lugar de otros métodos debe usar el botón para implementar en Azure y Azure Portal, por las siguientes razones:

* Para crear la configuración de punto a sitio, debe cargar los datos del certificado raíz. El campo de datos no acepta los datos de certificado cuando se usa PowerShell o la CLI.
* Esta plantilla no funciona correctamente con Cloud Shell debido a la carga de datos del certificado.
* También puede modificar fácilmente la plantilla y los parámetros en el portal para acomodar intervalos de direcciones IP y otros valores.

1. Haga clic en **Implementar en Azure**.

   [![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2fquickstarts%2fmicrosoft.network%2fvirtual-wan-with-route-tables%2fazuredeploy.json)
1. Para ver la plantilla, haga clic en **Editar plantilla**. En esta página puede ajustar algunos de los valores, como el espacio de direcciones o el nombre de determinados recursos. Haga clic en **Guardar** para guardar los cambios o en **Descartar**.
1. En la página plantilla, escriba los valores. Para esta plantilla, se necesitan los datos del certificado público de punto a sitio. Si este artículo es para hacer un ejercicio, puede usar los siguientes datos de este archivo .cer como datos de ejemplo para ambos centros de conectividad. Con la plantilla en ejecución y la implementación completa, para usar la configuración de punto a sitio, debe reemplazar esta información con los [datos del certificado](certificates-point-to-site.md#cer) de clave pública para su propia implementación.

   ```certificate-data
   MIIC5zCCAc+gAwIBAgIQGxd3Av1q6LJDZ71e3TzqcTANBgkqhkiG9w0BAQsFADAW
   MRQwEgYDVQQDDAtQMlNSb290Q2VydDAeFw0yMDExMDkyMjMxNTVaFw0yMTExMDky
   MjUxNTVaMBYxFDASBgNVBAMMC1AyU1Jvb3RDZXJ0MIIBIjANBgkqhkiG9w0BAQEF
   AAOCAQ8AMIIBCgKCAQEA33fFra/E0YmGuXLKmYcdvjsYpKwQmw8DjjDkbwhE9jcc
   Dp50e7F1P6Rxo1T6Hm3dIhEji+0QkP4Ie0XPpw0eW77+RWUiG9XJxGqtJ3Q4tyRy
   vBfsHORcqMlpV3VZOXIxrk+L/1sSm2xAc2QGuOqKaDNNoKmjrSGNVAeQHigxbTQg
   zCcyeuhFxHxAaxpW0bslK2hEZ9PhuAe22c2SHht6fOIDeXkadzqTFeV8wEZdltLr
   6Per0krxf7N2hFo5Cfz0KgWlvgdKLL7dUc9cjHo6b6BL2pNbLh8YofwHQOQbwt6H
   miAkEnx1EJ5N8AWuruUTByR2jcWyCnEAUSH41+nk4QIDAQABozEwLzAOBgNVHQ8B
   Af8EBAMCAgQwHQYDVR0OBBYEFJMgnJSYHH5AJ+9XB11usKRwjbjNMA0GCSqGSIb3
   DQEBCwUAA4IBAQBOy8Z5FBd/nvgDcjvAwNCw9h5RHzgtgQqDP0qUjEqeQv3ALeC+
   k/F2Tz0OWiPEzX5N+MMrf/jiYsL2exXuaPWCF5U9fu8bvs89GabHma8MGU3Qua2x
   Imvt0whWExQMjoyU8SNUi2S13fnRie9ZlSwNh8B/OIUUEtVhQsd4OfuZZFVH4xGp
   ibJMSMe5JBbZJC2tCdSdTLYfYJqrLkVuTjynXOjmz2JXfwnDNqEMdIMMjXzlNavR
   J8SNtAoptMOK5vAvlySg4LYtFyXkl0W0vLKIbbHf+2UszuSCijTUa3o/Y1FoYSfi
   eJH431YTnVLuwdd6fXkXFBrXDhjNsU866+hE
   ```

1. Cuando haya terminado de especificar los valores, seleccione **Revisar y crear**.
1. En la página **Revisar y crear**, cuando se supere la validación, seleccione **Crear**.
1. La implementación tarda unos 75 minutos. Puede ver el progreso en la página **Overview** (Información general) de la plantilla. La implementación continúa aunque se cierre el portal.

   :::image type="content" source="./media/quickstart-route-shared-services-template/template.png" alt-text="Ejemplo de implementación completa":::

## <a name="validate-the-deployment"></a><a name="validate"></a>Validación de la implementación

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione **Grupos de recursos** en el panel izquierdo.
1. Seleccione el grupo de recursos que creó en la sección anterior. En la página **Overview** (Información general), verá algo similar a este ejemplo: :::image type="content" source="./media/quickstart-route-shared-services-template/resources.png" alt-text="Ejemplo de recursos" lightbox="./media/quickstart-route-shared-services-template/resources.png":::

1. Haga clic en la WAN virtual para ver los centros de conectividad. En la página de la WAN virtual, haga clic en cada centro de conectividad para ver las conexiones y otra información de este.
   :::image type="content" source="./media/quickstart-route-shared-services-template/hub.png" alt-text="Ejemplo de centros de conectividad" lightbox="./media/quickstart-route-shared-services-template/hub.png":::

## <a name="complete-the-hybrid-configuration"></a><a name="complete"></a>Para completar la configuración híbrida

Con la plantilla no se configuran todas las opciones necesarias para una red híbrida. En función de sus requisitos, debe realizar lo siguiente:

* [Configuración de las ramas de VPN: sitios locales](virtual-wan-site-to-site-portal.md#site)
* [Configuración de VPN de punto a sitio](virtual-wan-point-to-site-portal.md)
* [Conexión de los circuitos de ExpressRoute](virtual-wan-expressroute-portal.md)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite los recursos que ha creado, elimínelos. Algunos recursos de Virtual WAN deben eliminarse en un orden determinado debido a las dependencias. La eliminación puede tardar unos 30 minutos.

[!INCLUDE [Delete resources](../../includes/virtual-wan-resource-cleanup.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Configuración de VPN de punto a sitio](virtual-wan-point-to-site-portal.md)

> [!div class="nextstepaction"]
> [Configuración de las ramas de VPN: sitios locales](virtual-wan-site-to-site-portal.md#site)

> [!div class="nextstepaction"]
> [Conexión de los circuitos de ExpressRoute](virtual-wan-expressroute-portal.md)
