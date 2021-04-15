---
title: archivo de inclusión
description: archivo de inclusión
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 10/22/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: c538c510eeafacd1596fdeb1b4593919e6236cf0
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "92487084"
---
1. En [Azure Portal](https://portal.azure.com), en **Buscar recursos, servicios y documentos (G+/)** escriba **puerta de enlace de red local**. Busque la **puerta de enlace de red local** en **Marketplace**, en los resultados de la búsqueda y selecciónela. Se abre la página **Crear puerta de enlace de red local**.
1. En la página **Crear puerta de enlace de red local**, especifique los valores de la puerta de enlace de red local.

   :::image type="content" source="./media/vpn-gateway-add-local-network-gateway-portal-ip-empty/create-ip.png" alt-text="Creación de una puerta de enlace de red local con una dirección IP":::

   * **Nombre:** especifique el nombre del objeto de puerta de enlace de red local.
   * **Punto de conexión:** Seleccione el tipo de punto de conexión para el dispositivo VPN local: **dirección IP** o **FQDN (nombre de dominio completo)** .
      * **Dirección IP**: si tiene asignada una dirección IP pública estática de su proveedor de servicios de Internet para el dispositivo VPN, seleccione la opción Dirección IP y rellene la dirección IP como se indica en el ejemplo. Esta es la dirección IP pública del dispositivo VPN al que desea que Azure VPN Gateway se conecte. Si no tiene la dirección IP en este momento, puede usar los valores que se muestran en el ejemplo, pero deberá volver y reemplazar la dirección IP del marcador de posición por la dirección IP pública de su dispositivo VPN. Si no lo hace, Azure no podrá conectarse.
   * **Espacio de direcciones** hace referencia a los intervalos de direcciones de la red que representa esta red local. Puede agregar varios intervalos de espacios de direcciones. Asegúrese de que los intervalos que especifique aquí no se superpongan con los de otras redes a las que quiera conectarse. Azure enrutará el intervalo de direcciones que especifique a la dirección IP del dispositivo VPN local. *Use sus propios valores aquí, y no los mostrados en el ejemplo, si quiere conectarse a su sitio local*.
   * **Configurar BGP:** usar solo al configurar BGP. En caso contrario, no seleccione esta opción.
   * **Subscription** (Suscripción): compruebe que se muestra la suscripción correcta.
   * **Grupos de recursos:** seleccione el grupo de recursos que quiere usar. Puede crear un grupo de recursos nuevo o seleccionar uno ya creado.
   * **Ubicación:** La ubicación es la misma que **Región** en otros valores. seleccione la ubicación en la que se creará este objeto. Puede seleccionar la misma ubicación en la que reside la red, pero no es obligatorio.

1. Cuando haya terminado de especificar los valores, seleccione el botón **Crear** en la parte inferior de la página para crear la puerta de enlace de red local.
