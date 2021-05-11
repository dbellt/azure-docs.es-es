---
title: Enrutamiento del tráfico a través de NVA mediante una configuración personalizada
titleSuffix: Azure Virtual WAN
description: Obtenga información sobre escenarios de enrutamiento de Virtual WAN con el fin de enrutar tráfico a través de aplicaciones virtuales de red (NVA). En este escenario, el tráfico se enruta a través de NVA usando una NVA diferente para el tráfico de Internet y una configuración personalizada.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 04/27/2021
ms.author: cherylmc
ms.custom: fasttrack-edit
ms.openlocfilehash: 153e3dd64546dfa713abefcff106652fc061a6a2
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108165828"
---
# <a name="scenario-route-traffic-through-nvas-by-using-custom-settings"></a>Escenario: Enrutamiento del tráfico a través de NVA mediante una configuración personalizada

Al trabajar con el enrutamiento de centros virtuales de Azure Virtual WAN, dispone de varias opciones. Este artículo se centra en el enrutamiento del tráfico a través de una aplicación virtual de red (NVA) para la comunicación entre redes virtuales y ramas, y usar otra NVA para el tráfico vinculado a Internet. Para obtener más información, consulte [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).

## <a name="design"></a>Diseño

* **Radios** para redes virtuales conectadas al centro virtual. (Por ejemplo, VNet 1, VNet 2 y VNet 3 en el diagrama más adelante en este artículo).
* **Red virtual de servicio** para las redes virtuales en las que los usuarios han implementado una NVA para inspeccionar el tráfico que no procede de Internet, y posiblemente con servicios comunes a los que se accede mediante radios. (Por ejemplo, VNet 4 en el diagrama más adelante en este artículo).
* **Red virtual perimetral** para las redes virtuales en las que los usuarios han implementado una NVA que se usará para inspeccionar el tráfico enlazado a Internet. (Por ejemplo, VNet 5 en el diagrama más adelante en este artículo).
* **Centros** para los centros de Virtual WAN administrados por Microsoft.

En la tabla siguiente se resumen las conexiones que se admiten en este escenario:

| De          | En|Radios|VNet de servicio|Ramas|Internet|
|---|:---:|:---:|:---:|:---:|:---:|
| **Radios**| ->| Directamente |Directamente | a través de la red virtual de servicio |a través de la red virtual perimetral |
| **VNet de servicio**| ->| Directamente |N/D| Directamente | |
| **Ramas** | ->| a través de la red virtual de servicio |Directamente| Directamente |  |

En cada una de las celdas de la matriz se describe si la conectividad fluye directamente a través de Virtual WAN o de una de las redes virtuales con una NVA.

Tenga en cuenta los siguientes detalles:

* Radios:
  * los radios llegarán a otros radios directamente a través de los concentradores de Virtual WAN.
  * Los radios obtendrán conectividad a las ramas a través de una ruta estática que apunta a la VNet de servicio. Desconocen los prefijos específicos de las ramas, porque son más específicos e invalidan el resumen.
  * Los radios enviarán el tráfico de Internet a la red virtual perimetral mediante un emparejamiento de VNet directo.
* las ramas llegarán a los radios a través de un enrutamiento estático que apunta a la VNet de servicio. No conocen los prefijos específicos de las redes virtuales que invalidan la ruta estática resumida.
* La red virtual de servicio será similar a una red virtual de servicios compartidos que debe ser accesible desde todas las redes virtuales y todas las ramas.
* No es necesario que la red virtual perimetral tenga conectividad a través de Virtual WAN, porque el único tráfico que admitirá proviene de emparejamientos de redes virtuales directos. Pero para simplificar la configuración, use el mismo modelo de conectividad que para la red virtual perimetral.

Hay tres patrones de conectividad distintos, lo que se traduce en tres tablas de enrutamiento. Las asociaciones a las distintas redes virtuales son las siguientes:

* Radios:
  * Tabla de enrutamiento asociada: **RT_V2B**
  * Propagación a tablas de enrutamiento: **RT_V2B** y **RT_SHARED**
* Redes virtuales de NVA (red virtual de servicio y red virtual DMZ):
  * Tabla de enrutamiento asociada: **RT_SHARED**
  * Propagación a tablas de enrutamiento: **RT_SHARED**
* Ramas:
  * Tabla de enrutamiento asociada: **Valor predeterminado**
  * Propagación a tablas de enrutamiento: **RT_SHARED** y **Predeterminado**

> [!NOTE]
> Asegúrese de que las VNet de radios no se propagan a la etiqueta predeterminada. Esto garantiza que el tráfico de las ramas a las redes virtuales de radio se reenviará a las NVA.

Estas rutas estáticas garantizan que el tráfico hacia y desde la red virtual y la rama atraviesa la NVA en la red virtual de servicio (VNet 4):

| Descripción | Tabla de rutas | Ruta estática              |
| ----------- | ----------- | ------------------------- |
| Ramas    | RT_V2B      | 10.2.0.0/16-> vnet4conn  |
| Radios de NVA  | Valor predeterminado     | 10.1.0.0/16 -> vnet4conn  |

Ahora puede usar Virtual WAN para seleccionar la conexión correcta a la que enviar los paquetes. También debe usar Virtual WAN para seleccionar la acción correcta que se realizará al recibir esos paquetes. Para ello, use las tablas de rutas de conexión como se indica a continuación:

| Descripción | Conexión | Ruta estática            |
| ----------- | ---------- | ----------------------- |
| VNet2Branch | vnet4conn  | 10.2.0.0/16 -> 10.4.0.5 |
| Branch2VNet | vnet4conn  | 10.1.0.0/16 -> 10.4.0.5 |

Para obtener más información, consulte [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).

## <a name="architecture"></a>Architecture

En el diagrama siguiente se muestra la arquitectura descrita anteriormente en el artículo.

En el diagrama hay un concentrador; **Concentrador 1**.

* El **Concentrador 1** está conectado directamente a las VNet de NVA **VNet 4** y **VNet 5**.

* Se espera que el tráfico entre las redes virtuales 1, 2 y 3, y las ramas pase por la **NVA de la red virtual 4** 10.4.0.5.

* Se espera que todo el tráfico vinculado a Internet desde las redes virtuales 1, 2 y 3 pase por la **NVA VNet 5** 10.5.0.5.

:::image type="content" source="./media/routing-scenarios/nva-custom/architecture.png" alt-text="Diagrama de una arquitectura de red.":::

## <a name="workflow"></a><a name="workflow"></a>Flujo de trabajo

:::image type="content" source="./media/routing-scenarios/nva-custom/workflow.png" alt-text="Diagrama del flujo de trabajo." lightbox="./media/routing-scenarios/nva-custom/workflow.png":::

Para configurar el enrutamiento mediante la NVA, tenga en cuenta los pasos siguientes:

1. Para que el tráfico vinculado a Internet pase por la VNet 5, es necesario que las redes virtuales 1, 2 y 3 se conecten directamente a la 5 a través del emparejamiento de red virtual. También es necesario configurar una ruta definida por el usuario en las redes virtuales para 0.0.0.0/0 y el próximo salto 10.5.0.5.

   Si no quiere conectar las redes virtuales 1, 2 y 3 a la red virtual 5 y prefiere usar la NVA en la red virtual 4 para enrutar el tráfico 0.0.0.0/0 de las ramas (conexiones VPN o ExpressRoute locales), vaya al [flujo de trabajo alternativo](#alternate).

   Pero si quiere que el tráfico entre las redes virtuales pase por la NVA, tendrá que desconectar las redes virtuales 1, 2 y 3 del concentrador virtual, y conectarlas o apilarlas sobre la red virtual 4 de radio de la NVA. En la WAN virtual, el tráfico entre redes virtuales se transmite mediante el concentrador Virtual WAN o una instancia de Azure Firewall del concentrador Virtual WAN (concentrador seguro). Si las redes virtuales se emparejan directamente mediante emparejamiento de VNet, se pueden comunicar de forma directa si omiten el tránsito por el concentrador virtual.

1. En Azure Portal, vaya al centro virtual y cree una tabla de rutas personalizada con el nombre **RT_Shared**. En esta tabla, las rutas se obtienen a través de la propagación desde todas las redes virtuales y las conexiones de rama. Puede ver esta tabla vacía en el diagrama siguiente.

   * **Rutas:** no es necesario agregar ninguna ruta estática.

   * **Asociación:** seleccione las redes virtuales 4 y 5, lo que significa que sus conexiones se asocian a la tabla de rutas **RT_Shared**.

   * **Propagación:** como quiere que todas las ramas y conexiones de red virtual propaguen sus rutas dinámicamente a esta tabla de rutas, seleccione las ramas y todas las redes virtuales.

1. Cree una tabla de rutas personalizada con el nombre **RT_V2B** para dirigir el tráfico desde las redes virtuales 1, 2 y 3 a las ramas.

   * **Rutas:** agregue una entrada de ruta estática agregada para las ramas, con el próximo salto como la conexión de la red virtual 4. Configure una ruta estática en la conexión de la red virtual 4 para los prefijos de rama. Indique el próximo salto como la dirección IP específica de la NVA en la red virtual 4.

   * **Asociación:** seleccione las **redes virtuales 1, 2 y 3**. Esto hará que las conexiones de red virtual 1, 2 y 3 se asociarán a esta tabla de rutas y podrán aprender las rutas (estáticas y dinámicas mediante propagación) en esta tabla de rutas.

   * **Propagación:** Las conexiones propagan las rutas a las tablas de rutas. La selección de las redes virtuales 1, 2 y 3 habilita la propagación de rutas desde estas redes virtuales hasta esta tabla de rutas. No es necesario propagar las rutas desde las conexiones de rama a **RT_V2B**, ya que el tráfico de la red virtual de rama pasa por la NVA en la red virtual 4.
  
1. Edite la tabla de rutas predeterminada, **DefaultRouteTable**.

   Todas las conexiones VPN, Azure ExpressRoute y VPN de usuario están asociadas a la tabla de rutas predeterminada. Todas las conexiones VPN, ExpressRoute y VPN de usuario propagan rutas al mismo conjunto de tablas de rutas.

   * **Rutas:** agregue una entrada de ruta estática agregada para las redes virtuales 1, 2 y 3, con el próximo salto como la conexión de la red virtual 4. Configure una ruta estática en la conexión de la red virtual 4 para los prefijos agregados de las redes virtuales 1, 2 y 3. Indique el próximo salto como la dirección IP específica de la NVA en la red virtual 4.

   * **Asociación:** asegúrese de que la opción para las ramas (VPN/ER/P2S) esté seleccionada, y de que las conexiones de rama locales estén asociadas a la tabla de rutas predeterminada.

   * **Propagación:** asegúrese de que la opción para las ramas (VPN/ER/P2S) esté seleccionada, y de que las conexiones de rama locales estén asociadas a la tabla de rutas predeterminada.

## <a name="alternate-workflow"></a><a name="alternate"></a>Flujo de trabajo alternativo

En este flujo de trabajo, las redes virtuales 1, 2 y 3 no se conectan a la red virtual 5. En su lugar, se usa la NVA en la red virtual 4 para enrutar el tráfico 0.0.0.0/0 de las ramas (conexiones VPN o ExpressRoute locales).

:::image type="content" source="./media/routing-scenarios/nva-custom/alternate.png" alt-text="Diagrama del flujo de trabajo alternativo." lightbox="./media/routing-scenarios/nva-custom/alternate.png":::

Para configurar el enrutamiento mediante la NVA, tenga en cuenta los pasos siguientes:

1. En Azure Portal, vaya al concentrador virtual y cree una tabla de rutas personalizada llamada  **RT_NVA** para dirigir el tráfico por la NVA 10.4.0.5  

   * **Rutas:**  no es necesaria ninguna acción.

   * **Asociación:**  Seleccione **VNet 4**. Esto implica que la conexión de la red virtual 4 se asociará a esta tabla de rutas y podrá aprender sus rutas (estáticas y dinámicas mediante propagación).

   * **Propagación:**  las conexiones propagan las rutas a las tablas de rutas. La selección de las redes virtuales 1, 2 y 3 habilita la propagación de rutas desde estas redes virtuales hasta esta tabla de rutas. Al seleccionar ramas (VPN/ER/P2S), se habilita la propagación de rutas desde las ramas o conexiones locales a esta tabla de rutas. Todas las conexiones VPN, ExpressRoute y VPN de usuario propagan rutas al mismo conjunto de tablas de rutas.

1. Cree una tabla de rutas personalizada llamada **RT_VNET** para dirigir el tráfico de las redes virtuales 1, 2 y 3 a las ramas o a Internet (0.0.0.0/0) mediante la NVA de la red virtual 4. El tráfico entre las redes virtuales será y por medio de la NVA de la red virtual 4. Si quiere que el tráfico pase por la NVA, desconecte las redes virtuales 1, 2 y 3, y conéctelas mediante el emparejamiento de VNet a la red virtual 4.

   * **Rutas:**  

     * agregue una ruta agregada "10.2.0.0/16" con el próximo salto como conexión de la red virtual 4 para el tráfico que va desde las redes virtuales 1, 2 y 3 hacia las ramas. En la conexión de la red virtual 4, configure una ruta para "10.2.0.0/16" e indique el próximo salto para que sea la dirección IP específica de la NVA en la red virtual 4.

     * Agregue una ruta "0.0.0.0/0" con el próximo salto como la conexión de la red virtual 4. Se agrega "0.0.0.0/0" para implicar el envío de tráfico a Internet. No implica prefijos de dirección específicos que pertenezcan a redes virtuales o ramas. En la conexión de la red virtual 4, configure una ruta para "0.0.0.0/0" e indique el próximo salto para que sea la dirección IP específica de la NVA en la red virtual 4.

   * **Asociación:**  seleccione las **redes virtuales 1, 2 y 3**. Esto hará que las conexiones de red virtual 1, 2 y 3 se asociarán a esta tabla de rutas y podrán aprender las rutas (estáticas y dinámicas mediante propagación) en esta tabla de rutas.

   * **Propagación:**  las conexiones propagan las rutas a las tablas de rutas. La selección de las redes virtuales 1, 2 y 3 habilita la propagación de rutas desde estas redes virtuales hasta esta tabla de rutas. Asegúrese de que la opción para ramas (VPN/ER/P2S) no está seleccionada. Esto garantiza que las conexiones locales no puedan llegar directamente a las redes virtuales 1, 2 y 3.

1. Edite la tabla de rutas predeterminada,  **DefaultRouteTable**.

   Todas las conexiones VPN, Azure ExpressRoute y VPN de usuario están asociadas a la tabla de rutas predeterminada. Todas las conexiones VPN, ExpressRoute y VPN de usuario propagan rutas al mismo conjunto de tablas de rutas.

   * **Rutas:**  

     * agregue una ruta agregada "10.1.0.0/16" para las **redes virtuales 1, 2 y 3** con el próximo salto como la **conexión de la red virtual 4**.

     * Agregue una ruta "0.0.0.0/0" con el próximo salto como la **conexión de la red virtual 4**. Se agrega "0.0.0.0/0" para implicar el envío de tráfico a Internet. No implica prefijos de dirección específicos que pertenezcan a redes virtuales o ramas. En el paso anterior de la conexión de la red virtual 4, ya había configurado una ruta para "0.0.0.0/0", con el próximo salto para que sea la dirección IP específica de la NVA en la red virtual 4.

   * **Asociación:**  asegúrese de que la opción para las ramas **(VPN/ER/P2S)** está seleccionada. Esto garantiza que las conexiones de rama locales se asocien a la tabla de rutas predeterminada. Todas las conexiones VPN, Azure ExpressRoute y VPN de usuario solo se asocian a la tabla de rutas predeterminada.

   * **Propagación desde:**  asegúrese de que la opción para las ramas **(VPN/ER/P2S)** está seleccionada. Esto garantiza que las conexiones locales propagan las rutas a la tabla de rutas predeterminada. Todas las conexiones VPN, ExpressRoute y VPN de usuario propagan rutas al "mismo conjunto de tablas de rutas".

   >[!Note]
   >
   > * Los usuarios del portal deben habilitar la propagación a la ruta predeterminada en las conexiones (VPN/ER/P2S/VNet) para que la ruta 0.0.0.0/0 surta efecto.
   > * Los usuarios de PS/CLI/REST deben establecer la marca "enableinternetsecurity" en true para que la ruta 0.0.0.0/0 surta efecto.
   > * La Conexión de Virtual Network no es compatible con la dirección IP de próximo salto "múltiple o única" para la aplicación virtual de red "igual" en una VNet de radio "si" una de las rutas con IP de próximo salto se indica como dirección IP pública o 0.0.0.0/0 (Internet)

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre Virtual WAN, consulte las [preguntas más frecuentes](virtual-wan-faq.md).
* Para obtener más información sobre el enrutamiento de centros virtuales, vea [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).
