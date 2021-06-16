---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: include
ms.date: 05/26/2021
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: ab341181db71a8df5dde27311e9169f9477c70f8
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110578291"
---
1. Busque la red de WAN virtual que ha creado. En la página de la red WAN virtual, en la sección **Conectividad**, seleccione **Centros de conectividad**.
2. En la página **Centros de conectividad**, seleccione **+ Nuevo centro de conectividad** para abrir la página **Crear centro de conectividad virtual**.

   :::image type="content" source="./media/virtual-wan-tutorial-hub-include/basics.png" alt-text="Captura de pantalla que muestra el panel Crear centro de conectividad virtual con la pestaña Aspectos básicos seleccionada." border="false":::
3. En la página **Crear centro de conectividad virtual**, en la pestaña **Aspectos básicos** rellene los siguientes campos:

   * **Región** (anteriormente se conocía como "ubicación")
   * **Nombre**
   * **Espacio de direcciones privadas del centro de conectividad**: el espacio de direcciones mínimo para crear un centro de conectividad es /24. Si usa cualquier valor comprendido entre /25 y /32, se producirá un error durante el proceso de creación. No es necesario planear explícitamente el espacio de direcciones de subred para los servicios del centro de conectividad virtual. Como Azure Virtual WAN es un servicio administrado, crea las subredes adecuadas en el centro de conectividad virtual para las diferentes puertas de enlace y servicios (por ejemplo, puertas de enlace de VPN, puertas de enlace de ExpressRoute, puertas de enlace de VPN de punto a sitio del usuario, firewall, enrutamiento, etc.).
4. Seleccione **Siguiente: De sitio a sitio**.

   :::image type="content" source="./media/virtual-wan-tutorial-hub-include/site-to-site.png" alt-text="Captura de pantalla que muestra el panel Crear centro de conectividad virtual con Site to site (De sitio a sitio) seleccionado." border="false":::

5. En la pestaña **De sitio a sitio**, rellene los siguientes campos:

   * Seleccione **Sí** para crear una VPN de sitio a sitio.
   * El campo Número AS no se puede modificar.
   * En la lista desplegable, seleccione el valor **Unidades de escalado de puerta de enlace**. La unidad de escalado permite elegir el rendimiento agregado de la puerta de enlace de VPN que se va a crear en el centro de conectividad virtual para conectar los sitios. Si elige 1 unidad de escalado = 500 Mbps, esto implica que se crearán dos instancias de redundancia, cada una con un rendimiento máximo de 500 Mbps. Por ejemplo, si tuviera cinco ramas, cada una con 10 Mbps en la rama, necesitará un agregado de 50 Mbps en el extremo principal. El planeamiento de la capacidad agregada de la puerta de enlace de VPN de Azure debe realizarse después de evaluar la capacidad necesaria para admitir el número de ramas en el centro de conectividad.
6. Seleccione **Revisar y crear** para validar.
7. Seleccione **Crear** para crear un centro de conectividad. Pasados 30 minutos haga clic en **Actualizar** para ver el centro de conectividad en la página **Centros**. Seleccione **Ir al recurso** para ir al recurso.
