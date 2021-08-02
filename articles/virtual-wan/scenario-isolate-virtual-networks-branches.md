---
title: 'Escenario: Aislamiento personalizado para redes virtuales y sucursales'
titleSuffix: Azure Virtual WAN
description: Obtenga información sobre escenarios de enrutamiento de Virtual WAN para evitar que redes virtuales y ramas seleccionadas puedan comunicarse entre sí.
services: virtual-wan
author: wellee
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 04/27/2021
ms.author: wellee
ms.openlocfilehash: c5c85d2d24a6e1c49dd2787eeb14caabd0e3f941
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110579248"
---
# <a name="scenario-custom-isolation-for-virtual-networks-and-branches"></a>Escenario: Aislamiento personalizado para redes virtuales y sucursales

Al trabajar con el enrutamiento de centros virtuales de Virtual WAN, hay bastantes escenarios disponibles. En un escenario de aislamiento personalizado para redes virtuales (VNet) y sucursales, el objetivo es evitar que un conjunto específico de redes virtuales se comunique con otro. Del mismo modo, solo se permite que las sucursales (VPN/ER/VPN de usuario) se comuniquen con determinados conjuntos de redes virtuales.

También se presenta el requisito adicional de que Azure Firewall debe inspeccionar el tráfico entre sucursales y redes virtuales, pero **no** el tráfico entre redes virtuales.  

Para obtener más información sobre el enrutamiento de centros virtuales, vea [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).

## <a name="design"></a><a name="design"></a>Diseño

Para averiguar el número de tablas de enrutamiento que va a necesitar, puede crear una matriz de conectividad. En el caso de este escenario, será similar al siguiente, donde cada celda representa si un origen (fila) puede comunicarse con un destino (columna):

| From | A:| *Redes virtuales Azules* | *Redes virtuales Rojas* | *Sucursales rojas*| *Sucursales azules*| 
|---|---|---|---|---|---|
| **Redes virtuales Azules** |   &#8594;|   Directo     |           |   |  AzFW|
| **Redes virtuales Rojas**  |   &#8594;|              |   Directo  |  AzFW  | 
| **Sucursales rojas**   |   &#8594;|   |   AzFW  |  Directo | Directo
| **Sucursales azules**| &#8594;| AzFW  |   |Directo   | Directo

En cada una de las celdas de la tabla anterior se describe si una conexión de Virtual WAN (el lado "De" del flujo, los encabezados de fila) se comunica con un destino (el lado "A" del flujo, los encabezados de columna en cursiva). **Directo** implica que el tráfico fluye directamente a través de Virtual WAN, mientras que **AzFW** implica que Azure Firewall inspecciona el tráfico antes de que se reenvíe al destino. Una entrada en blanco significa que el flujo está bloqueado en la configuración.

En este caso, las dos tablas de rutas de las redes virtuales son necesarias para alcanzar el objetivo de aislamiento de las redes virtuales sin Azure Firewall en la ruta de acceso. Llamaremos a estas tablas de rutas **RT_BLUE** y **RT_RED**.

Además, las sucursales siempre deben estar asociadas a la tabla de rutas **predeterminada**. Para asegurarse de que Azure Firewall inspeccione el tráfico hacia las sucursales y desde estas, se agregaron rutas estáticas en las tablas de rutas **predeterminada**, **RT_RED** y **RT_BLUE** que apuntan el tráfico a Azure Firewall, y se configuraron reglas de red para permitir el tráfico deseado. Además, nos hemos asegurado de que las sucursales **no** se propaguen a **RT_BLUE** y **RT_RED**.

En consecuencia, este es el diseño final:

* Redes virtuales Azules:
  * Tabla de enrutamiento asociada: **RT_BLUE**
  * Propagación a tablas de rutas: **RT_BLUE**
* Redes virtuales Rojas:
  * Tabla de enrutamiento asociada: **RT_RED**
  * Propagación a tablas de rutas: **RT_RED** 
* Ramas:
  * Tabla de enrutamiento asociada: **Valor predeterminado**
  * Propagación a tablas de rutas: **Valor predeterminado**
* Rutas estáticas:
    * **Tabla de rutas predeterminada**: Espacios de direcciones de Virtual Network con el próximo salto Azure Firewall
    * **RT_RED**: 0.0.0.0/0 con el próximo salto Azure Firewall
    * **RT_BLUE**: 0.0.0.0/0 con el próximo salto Azure Firewall
* Reglas de red de firewall:
    * **REGLA DE PERMISO** **Prefijo de origen**: prefijos de dirección de sucursales azules **Prefijo de destino**: Prefijos de redes virtuales azules 
    * **REGLA DE PERMISO** **Prefijo de origen**: prefijos de dirección de sucursales rojas **Prefijo de destino**: prefijos de redes virtuales rojas

> [!NOTE]
> Como todas las ramas deben estar asociadas a la tabla de enrutamiento Predeterminada, así como propagarse al mismo conjunto de tablas de enrutamiento, todas las ramas tendrán el mismo perfil de conectividad. En otras palabras, el concepto Rojo/Azul para redes virtuales no se puede aplicar a ramas. Sin embargo, para lograr el enrutamiento personalizado para las sucursales, podemos reenviar el tráfico desde las sucursales a Azure Firewall.

> [!NOTE]
> De manera predeterminada, Azure Firewall deniega el tráfico en un modelo de confianza cero. Si no hay ninguna regla explícita de **PERMISO** que coincida con el paquete inspeccionado, Azure Firewall anulará el paquete.

Para más información sobre el enrutamiento de centros virtuales, vea [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).



## <a name="workflow"></a><a name="architecture"></a>Flujo de trabajo

En la **figura 1**, hay redes virtuales azules y rojas, así como sucursales que pueden acceder a redes virtuales azules o rojas.

* Las redes virtuales conectadas en azul pueden comunicarse entre sí, y pueden conectarse con todas las sucursales azules (VPN/ER/P2S). En el diagrama, la sucursal azul es el sitio VPN de sitio a sitio.
* Las redes virtuales conectadas en rojo pueden comunicarse entre sí, y pueden conectarse con todas las sucursales rojas (VPN/ER/P2S). En el diagrama, la sucursal roja son los usuarios de VPN de punto a sitio.

Tenga en cuenta los siguientes pasos al configurar el enrutamiento.

1. Cree dos tablas de rutas personalizadas en Azure Portal, **RT_BLUE** y **RT_RED** para personalizar el tráfico a esas redes virtuales.
2. En la tabla de rutas **RT_BLUE**, aplique la configuración siguiente para asegurarse de que las redes virtuales azules aprenden los prefijos de dirección de todas las demás redes virtuales azules:
   * **Asociación**: seleccione todas las redes virtuales Azules.
   * **Propagación**: seleccione todas las redes virtuales Azules.
3. Repita los mismos pasos para la tabla de rutas **RT_RED** para las redes virtuales rojas.
4. Aprovisione una instancia de Azure Firewall en Virtual WAN. Para obtener más información sobre Azure Firewall en el centro de Virtual WAN, consulte [Configuración de Azure Firewall en un centro de Virtual WAN](howto-firewall.md).
5. Agregue una ruta estática a la tabla de rutas **predeterminada** del centro virtual que dirige todo el tráfico destinado a los espacios de direcciones de redes virtuales (tanto azules como rojas) a Azure Firewall. Este paso garantiza que todos los paquetes de las sucursales se envíen a Azure Firewall para su inspección.
    * Ejemplo: **Prefijo de destino**:  10.0.0.0/8 **Próximo salto**: Azure Firewall
    >[!NOTE]
    > Este paso también puede realizarse mediante Firewall Manager al seleccionar la opción "Secure Private Traffic" (Tráfico privado seguro). Esto agregará una ruta para todas las direcciones IP privadas RFC1918 aplicables a redes virtuales y sucursales. Tendrá que agregar manualmente las sucursales o redes virtuales que no cumplan con RFC1918. 

6. Agregue una ruta estática a **RT_RED** y **RT_BLUE** que dirija todo el tráfico a Azure Firewall. Este paso garantiza que las redes virtuales no puedan acceder directamente a las sucursales. Este paso no se puede realizar a través de Firewall Manager porque estas redes virtuales no están asociadas a la tabla de rutas predeterminada.
    * Ejemplo: **Prefijo de destino**: 0.0.0.0/0 **Próximo salto**: Azure Firewall

    > [!NOTE]
    > El enrutamiento se realiza mediante la coincidencia de prefijo más larga (LPM). Como resultado, las rutas estáticas 0.0.0.0/0 **no** se preferirán por sobre los prefijos exactos que existen en **BLUE_RT** y **RED_RT**. Como consecuencia, Azure Firewall no inspeccionará el tráfico de la red virtual.

Esto hará que la configuración de enrutamiento cambie como se muestra en la figura siguiente.

**Figura 1**
[ ![Figura 1](./media/routing-scenarios/custom-branch-vnet/custom-branch.png)](./media/routing-scenarios/custom-branch-vnet/custom-branch.png#lightbox)

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre Virtual WAN, consulte las [preguntas más frecuentes](virtual-wan-faq.md).
* Para obtener más información sobre el enrutamiento de centros virtuales, vea [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).
