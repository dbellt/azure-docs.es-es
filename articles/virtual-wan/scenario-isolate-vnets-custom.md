---
title: 'Escenario: Aislamiento personalizado para redes virtuales'
titleSuffix: Azure Virtual WAN
description: Obtenga información sobre los escenarios de aislamiento personalizado de enrutamiento de Virtual WAN para evitar que un conjunto específico de redes virtuales pueda comunicarse con otro conjunto específico de redes virtuales, pero es necesario que las redes virtuales se comuniquen con todas las ramas.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 04/27/2021
ms.author: cherylmc
ms.openlocfilehash: 56a0fe561d026f1b01f27cf3015c31820e8110bb
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108162048"
---
# <a name="scenario-custom-isolation-for-vnets"></a>Escenario: Aislamiento personalizado para redes virtuales

Al trabajar con el enrutamiento de centros virtuales de Virtual WAN, hay bastantes escenarios disponibles. En un escenario de aislamiento personalizado para redes virtuales, el objetivo es evitar que un conjunto específico de redes virtuales pueda comunicarse con otro conjunto específico de redes virtuales. Sin embargo, es necesario que las redes virtuales tengan acceso a todas las ramas (VPN, ER y VPN de usuario). Para más información sobre el enrutamiento de centros virtuales, vea [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).

## <a name="design"></a><a name="design"></a>Diseño

Para averiguar el número de tablas de enrutamiento que va a necesitar, puede crear una matriz de conectividad. En el caso de este escenario, será similar al siguiente, donde cada celda representa si un origen (fila) puede comunicarse con un destino (columna):

| From | A:| *Redes virtuales Azules* | *Redes virtuales Rojas* | *Ramas*|
|---|---|---|---|---|
| **Redes virtuales Azules** |   &#8594;|   Directo     |           |  Directo |
| **Redes virtuales Rojas**  |   &#8594;|              |   Directo  |  Directo |
| **Ramas**   |   &#8594;|   Directo     |   Directo  |  Directo |

En cada una de las celdas de la tabla anterior se describe si una conexión de Virtual WAN (el lado "De" del flujo, los encabezados de fila) se comunica con un destino (el lado "A" del flujo, los encabezados de columna en cursiva). En este escenario no hay ningún firewall ni dispositivo virtual de red, por lo que la comunicación fluye directamente mediante Virtual WAN (de ahí la palabra "Directo" en la tabla).

El número de patrones de fila diferentes será el número de tablas de enrutamiento que se necesitará en este escenario. En este caso, tres tablas de enrutamiento que llamaremos **RT_BLUE** y **RT_RED** para las redes virtuales y **Predeterminada** para las ramas. Recuerde que las ramas siempre tienen que estar asociadas a la tabla de enrutamiento Predeterminada.

Las ramas deberán aprender los prefijos de las redes virtuales Rojas y Azules, para que todas las redes virtuales se propaguen a la tabla Predeterminada (además de **RT_BLUE** o **RT_RED**). Las redes virtuales Azules y Rojas deberán aprender los prefijos de las ramas, para que las ramas se propaguen también a ambas tablas de enrutamiento **RT_BLUE** y **RT_RED**. En consecuencia, este es el diseño final:

* Redes virtuales Azules:
  * Tabla de enrutamiento asociada: **RT_BLUE**
  * Propagación a tablas de enrutamiento: **RT_BLUE** y **Predeterminada**
* Redes virtuales Rojas:
  * Tabla de enrutamiento asociada: **RT_RED**
  * Propagación a tablas de enrutamiento: **RT_RED** y **Predeterminada**
* Ramas:
  * Tabla de enrutamiento asociada: **Valor predeterminado**
  * Propagación a tablas de enrutamiento: **RT_BLUE**, **RT_RED** y **Predeterminada**

> [!NOTE]
> Como todas las ramas deben estar asociadas a la tabla de enrutamiento Predeterminada, así como propagarse al mismo conjunto de tablas de enrutamiento, todas las ramas tendrán el mismo perfil de conectividad. En otras palabras, el concepto Rojo/Azul para redes virtuales no se puede aplicar a ramas.

> [!NOTE]
> Si Virtual WAN se implementó en varias regiones, tendrá que crear las tablas de enrutamiento **RT_BLUE** y **RT_RED** en cada centro de conectividad y las rutas de cada conexión de red virtual se deben propagar a las tablas de enrutamiento de cada centro de conectividad mediante etiquetas de propagación.

Para más información sobre el enrutamiento de centros virtuales, vea [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).

## <a name="workflow"></a><a name="architecture"></a>Flujo de trabajo

En la **ilustración 1**, hay conexiones de red virtual azules y rojas.

* Las redes virtuales conectadas en azul pueden comunicarse entre sí, así como conectarse con todas las ramas (VPN/ER/P2S).
* Las redes virtuales rojas pueden comunicarse entre sí, así como comunicarse con todas las ramas (VPN/ER/P2S).

Tenga en cuenta los siguientes pasos al configurar el enrutamiento.

1. Cree dos tablas de rutas personalizadas en Azure Portal, **RT_BLUE** y **RT_RED**.
2. En el caso de la tabla de enrutamiento **RT_BLUE**, para la configuración siguiente:
   * **Asociación**: seleccione todas las redes virtuales Azules.
   * **Propagación**: en el caso de las ramas, seleccione la opción para las ramas, e indique que las conexiones de rama (VPN/ER/P2S) propagarán las rutas a esta tabla de enrutamiento.
3. Repita los mismos pasos para la tabla de enrutamiento **RT_RED** para las redes virtuales Rojas y las ramas (VPN/ER/P2S).

Esto hará que la configuración de enrutamiento cambie como se muestra en la figura siguiente.

**Ilustración 1**

:::image type="content" source="./media/routing-scenarios/custom-isolation/custom.png" alt-text="ilustración 1":::

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre Virtual WAN, consulte las [preguntas más frecuentes](virtual-wan-faq.md).
* Para obtener más información sobre el enrutamiento de centros virtuales, vea [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).
