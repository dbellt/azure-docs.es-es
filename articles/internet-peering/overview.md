---
title: Configuración del emparejamiento con Microsoft
titleSuffix: Azure
description: Introducción al emparejamiento
services: internet-peering
author: prmitiki
ms.service: internet-peering
ms.topic: overview
ms.date: 12/15/2020
ms.author: prmitiki
ms.openlocfilehash: 99cfad67742c5f1261811469b3c41f5bb3bad1ba
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108145076"
---
# <a name="internet-peering-overview"></a>Introducción al emparejamiento de Internet

El emparejamiento consiste en la interconexión entre la red global de Microsoft (AS8075) y la red del usuario con el fin de intercambiar tráfico de Internet desde y hacia los servicios en línea de Microsoft y los servicios de Microsoft Azure. Los operadores o los proveedores de servicios pueden solicitar la conexión con Microsoft en todas las ubicaciones de Edge. Microsoft revisa cada solicitud para asegurarse de que cumple con nuestras directivas de emparejamiento. Puede configurar un emparejamiento con la red de Microsoft de dos maneras:

* **Emparejamiento directo:**

    El emparejamiento se establece a través de conexiones físicas directas entre la red de Microsoft en Microsoft Edge y la red del usuario. Las sesiones del Protocolo de puerta de enlace de borde (BGP) se configuran en estas conexiones según nuestra directiva de enrutamiento y mediante un acuerdo negociado previamente. A esto también se le conoce como PNI.

* **Emparejamiento de Exchange:**

    Esto hace referencia a las conexiones de emparejamiento público estándar en los intercambios de Internet (IX). Las conexiones físicas entre la red de Microsoft y la red del usuario se realizan a través de un tejido de conmutación que controla IX. Las sesiones del Protocolo de puerta de enlace de borde se configuran mediante el espacio de direcciones IP que proporciona IX.

## <a name="benefits-of-peering-with-microsoft"></a>Ventajas del emparejamiento con Microsoft
* Reduzca los costos de tránsito con la entrega de tráfico de Microsoft mediante el emparejamiento con Microsoft.
* Mejore el rendimiento de sus clientes reduciendo los saltos de red y la latencia en la red de Microsoft Edge.
* Proteja el tráfico del cliente frente a errores en la red o en la red del proveedor de tránsito mediante el emparejamiento con Microsoft en ubicaciones redundantes.
* Obtenga información sobre las métricas de rendimiento de sus conexiones de emparejamiento y use esta información para solucionar problemas de la red.

## <a name="benefits-of-using-azure-to-set-up-peering"></a>Ventajas de usar Azure para configurar el emparejamiento

Puede solicitar el emparejamiento con Microsoft mediante Azure PowerShell o el portal. La configuración del emparejamiento de esta forma se administra como un recurso de Azure y proporciona las siguientes ventajas:
* Pasos simplificados y automatizables para configurar y administrar el emparejamiento con Microsoft.
* Una manera rápida y sencilla de ver y administrar todos los emparejamientos en un solo lugar.
* Seguimiento de los datos de estado y de ancho de banda de todas las conexiones.
* Puede usar la misma suscripción para acceder a Azure Cloud Services.

Si ya ha establecido emparejamientos con Microsoft, se hace referencia a ellos como **emparejamientos heredados**. Puede optar por administrar tales emparejamientos como recursos de Azure para aprovechar las ventajas anteriores. Para enviar una nueva solicitud de emparejamiento o convertir el emparejamiento heredado en un recurso de Azure, siga los vínculos de la sección **Pasos siguientes**.

## <a name="peering-policy"></a>Directiva de emparejamiento
Microsoft tiene una directiva de emparejamiento selectiva, pero generalmente abierta. Los dispositivos del mismo nivel se seleccionan en función del rendimiento, la capacidad o donde existen ventajas mutuas, y están sujetos a determinados requisitos técnicos, comerciales y legales. Para más información, consulte [Directiva de emparejamiento](policy.md).

## <a name="faq"></a>Preguntas más frecuentes
Para ver las preguntas más frecuentes sobre el emparejamiento, consulte [Preguntas más frecuentes sobre el emparejamiento de Internet](faqs.md).

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre los pasos para configurar el emparejamiento directo con Microsoft, consulte [Tutorial del emparejamiento directo](walkthrough-direct-all.md).
* Para más información sobre los pasos para configurar el emparejamiento de intercambio con Microsoft, consulte [Tutorial del emparejamiento de Exchange](walkthrough-exchange-all.md).
* Aprenda sobre las demás [funcionalidades de red](../networking/fundamentals/networking-overview.md) clave de Azure.