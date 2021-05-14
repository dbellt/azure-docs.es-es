---
title: Selección de ruta de acceso de Azure en varios vínculos de ISP
titleSuffix: Azure Virtual WAN
description: En este artículo, aprenderá de qué manera Azure Virtual WAN puede incluir información de vínculos para dirigir el tráfico a través de varios vínculos mediante la selección de ruta de acceso de Azure.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 04/27/2021
ms.author: cherylmc
ms.openlocfilehash: a2de840d07a99ed7c135b75ef24144da642654bb
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108166206"
---
# <a name="azure-path-selection-across-multiple-isp-links"></a>Selección de ruta de acceso de Azure en varios vínculos de ISP

Azure Virtual WAN proporciona a un usuario la funcionalidad para incluir información de vínculos en un sitio VPN, lo que permite escenarios en los que la solución de dispositivo VPN/SD-WAN puede programar directivas específicas de la rama para dirigir el tráfico a través de varios vínculos a Azure. Esto se denomina **selección de ruta de acceso de Azure**.

## <a name="architecture"></a>Architecture

Para entender cómo funciona la selección de rutas de acceso de Azure, vamos a usar el ejemplo de un sitio VPN de Virtual WAN y una conexión de sitio a sitio.

Un sitio VPN representa el dispositivo SD-WAN/VPN local con información como la dirección IP pública, el modelo y el nombre del dispositivo, etc. El sitio VPN local real puede tener varios vínculos de ISP que también pueden incluirse en la información del sitio VPN de Virtual WAN. Esto le permite ver la información de vínculos en Azure.

Una conexión IPsec de sitio a sitio que llega a la VPN de Virtual WAN finaliza en las instancias de VPN Gateway dentro de un centro de conectividad virtual. Una conexión de sitio a sitio representa la conectividad entre el sitio VPN y Azure VPN Gateway. Consta de una o varias conexiones de vínculos. Cada conexión de vínculo consta de dos túneles, cada uno de los cuales termina en una instancia única de VPN Gateway de Azure Virtual WAN. Se pueden configurar hasta cuatro conexiones de vínculo en la conexión de sitio a sitio, lo que permite tener hasta ocho túneles dentro de una conexión de sitio a sitio. Azure admite hasta 2000 túneles que finalizan dentro de una única puerta de enlace de VPN de WAN virtual.

:::image type="content" source="./media/path-selection-multiple-links/multi-link-site.png" alt-text="Diagrama de vínculo múltiple":::

En esta ilustración se muestra un vínculo múltiple en un sitio que se conecta a Azure Virtual WAN. En este diagrama:

* Hay dos vínculos de ISP en la rama local (dispositivo VPN/SD-WAN). Cada vínculo de ISP corresponde a una conexión de vínculo.

* Se supone que el dispositivo VPN/SD-WAN del administrador del cliente local admite IKEv1 o IKEv2 IPsec.

* Cada conexión de sitio a sitio de Azure Virtual WAN se compone de conexiones de vínculo dentro de sí misma. Una conexión admite hasta cuatro conexiones de vínculo. Azure cobra una cuota por unidad de conexión de Virtual WAN. No se aplica ningún cargo por las conexiones de vínculo.

* Cada conexión de vínculo, a su vez, consta de dos túneles IPsec que pueden finalizar en dos instancias diferentes de VPN Gateway de Virtual WAN. Las puertas de enlace están configuradas como puertas de enlace activas-activas con fines de resistencia. Cada conexión de vínculo debe tener una dirección IP única y una IP de emparejamiento de BGP. En el diagrama, el túnel 0 puede terminar en la instancia 0 y el túnel 1 puede terminar en la instancia 1.

* Los dispositivos de rama que proporcionan selección de rutas de acceso pueden habilitar la directiva adecuada en la solución de administración de ramas para dirigir el tráfico a través de varios vínculos a Azure. Por ejemplo, el vínculo ISP 1 se puede usar para el tráfico de mayor prioridad y el vínculo de ISP 2 se puede usar como reserva.

* Es importante tener en cuenta que la VPN del centro de conectividad virtual usa ECMP (enrutamiento de rutas de acceso múltiples de costo equivalente) en todos los túneles de terminación.

## <a name="next-steps"></a>Pasos siguientes

Consulte las [preguntas más frecuentes sobre Azure](virtual-wan-faq.md).