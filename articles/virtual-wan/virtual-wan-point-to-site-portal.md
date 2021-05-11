---
title: 'Tutorial: Uso de Azure Virtual WAN para crear una conexión de punto a sitio a Azure'
description: En este tutorial, aprenderá a usar Azure Virtual WAN para crear una conexión VPN de usuario (de punto a sitio) a Azure.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: tutorial
ms.date: 04/27/2021
ms.author: cherylmc
ms.openlocfilehash: a2943d83bd29251551bdf82e906e068717595e33
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108164244"
---
# <a name="tutorial-create-a-user-vpn-connection-using-azure-virtual-wan"></a>Tutorial: Creación de una conexión VPN de usuario mediante Azure Virtual WAN

Este tutorial muestra cómo usar Virtual WAN para conectarse a los recursos de Azure a través de una conexión VPN de OpenVPN o IPsec/IKE (IKEv2). Este tipo de conexión requiere que se configure el cliente VPN en el equipo cliente. Para más información sobre Virtual WAN, consulte la [Introducción a Virtual WAN](virtual-wan-about.md).

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una instancia de Virtual WAN
> * Creación de una configuración de P2S
> * Crear un centro virtual
> * Elegir grupos de direcciones de clientes
> * Especificación de los servidores DNS
> * Generar un paquete de configuración de cliente VPN
> * Configuración de clientes VPN
> * Visualizar la instancia de Virtual WAN

![Diagrama de Virtual WAN](./media/virtual-wan-about/virtualwanp2s.png)

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [Before beginning](../../includes/virtual-wan-before-include.md)]

## <a name="create-a-virtual-wan"></a><a name="wan"></a>Creación de una instancia de Virtual WAN

[!INCLUDE [Create a virtual WAN](../../includes/virtual-wan-create-vwan-include.md)]

## <a name="create-a-p2s-configuration"></a><a name="p2sconfig"></a>Creación de una configuración de P2S

Una configuración de punto a sitio (P2S) define los parámetros para conectar clientes remotos.

[!INCLUDE [Create P2S configuration](../../includes/virtual-wan-p2s-configuration-include.md)]

## <a name="create-virtual-hub-and-gateway"></a><a name="hub"></a>Creación de un centro virtual y una puerta de enlace

[!INCLUDE [Create hub](../../includes/virtual-wan-p2s-hub-include.md)]

## <a name="choose-p2s-client-address-pools"></a><a name="chooseclientpools"></a> Elegir grupos de direcciones de cliente de P2S

[!INCLUDE [Choose pools](../../includes/virtual-wan-allocating-p2s-pools.md)]

## <a name="specify-dns-server"></a><a name="dns"></a>Especificación del servidor DNS

Este valor se puede configurar al crear el centro, o bien se puede modificar posteriormente. Para modificarlo, busque el centro de virtual. En **VPN de usuario (punto a sitio)** , seleccione **Configurar** y escriba las direcciones IP del servidor de DNS en los cuadros de texto **Servidores DNS personalizados**. Puede especificar un máximo de cinco servidores DNS.

   :::image type="content" source="media/virtual-wan-point-to-site-portal/custom-dns.png" alt-text="DNS personalizado" lightbox="media/virtual-wan-point-to-site-portal/custom-dns-expand.png":::

## <a name="generate-vpn-client-profile-package"></a><a name="download"></a>Generación de un paquete de perfiles de cliente VPN

Genere y descargue el paquete de perfiles de cliente VPN para configurar los clientes VPN.

[!INCLUDE [Download profile](../../includes/virtual-wan-p2s-download-profile-include.md)]

## <a name="configure-vpn-clients"></a><a name="configure-client"></a>Configuración de clientes VPN

Use el paquete de perfiles descargado para configurar los clientes VPN de acceso remoto. El procedimiento para cada sistema operativo es diferente. Siga las instrucciones que se aplican al sistema.
Una vez que haya terminado de configurar el cliente, puede conectarse.

[!INCLUDE [Configure clients](../../includes/virtual-wan-p2s-configure-clients-include.md)]

## <a name="view-your-virtual-wan"></a><a name="viewwan"></a>Visualización de la instancia de Virtual WAN

1. Vaya a la instancia de Virtual WAN.
1. En la página **Información general**, cada punto del mapa representa un centro de conectividad.
1. En la sección de **centros de conectividad y conexiones**, puede ver estado del centro de conectividad, sitio, región, estado de la conexión VPN y bytes de entrada y salida.

## <a name="clean-up-resources"></a><a name="cleanup"></a>Limpieza de recursos

Cuando ya no necesite los recursos que ha creado, elimínelos. Algunos recursos de Virtual WAN deben eliminarse en un orden determinado debido a las dependencias. La eliminación puede tardar unos 30 minutos.

[!INCLUDE [Delete resources](../../includes/virtual-wan-resource-cleanup.md)]

## <a name="next-steps"></a>Pasos siguientes

Más información sobre Virtual WAN en:

> [!div class="nextstepaction"]
> * [Preguntas más frecuentes sobre Virtual WAN](virtual-wan-faq.md)
