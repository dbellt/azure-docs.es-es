---
title: Adición de un segmento de red de NSX-T
description: Pasos para agregar un segmento de red de NSX-T para Azure VMware Solution.
ms.topic: include
ms.date: 03/13/2021
ms.openlocfilehash: 477533887629a88f72426a33ed38278636e72fef
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110064472"
---
<!-- Used in configure-dhcp-azure-vmware-solution.md and tutorial-nsx-t-network-segment.md -->

1. In NSX-T Manager, seleccione **Networking** > **Segments** (Redes > Segmentos) y, a continuación, seleccione **Add Segment** (Agregar segmento). 

   :::image type="content" source="../media/nsxt/nsxt-segments-overview.png" alt-text="Captura de pantalla en la que se muestra cómo agregar un nuevo segmento":::

1. Escriba un nombre para el segmento.

1. Seleccione la puerta de enlace de nivel 1 (TNTxx-T1) para **Connected Gateway** (Puerta de enlace conectada) y deje el campo **Type** (Tipo) como Flexible.

1. Seleccione la superposición preconfigurada **Transport Zone** (Zona de transporte) (TNTxx-OVERLAY-TZ) y, a continuación, seleccione **Set Subnets** (Establecer subredes). 

   :::image type="content" source="../media/nsxt/nsxt-create-segment-specs.png" alt-text="Establezca el nombre del segmento, la puerta de enlace conectada y el tipo y la zona de transporte; a continuación, seleccione Set Subnet (Establecer subred).":::

1. Escriba la dirección IP de la puerta de enlace y, a continuación, seleccione **Add** (Agregar). 

   >[!IMPORTANT]
   >La dirección IP debe estar en un bloque de direcciones RFC1918 que no se superponga, lo que garantiza la conexión a las máquinas virtuales en el nuevo segmento.

   :::image type="content" source="../media/nsxt/nsxt-create-segment-gateway.png" alt-text="Establezca la dirección IP de la puerta de enlace para el nuevo segmento y, a continuación, seleccione ADD (AGREGAR)":::.

1. Seleccione **Apply** (Aplicar) y, a continuación, seleccione **Save** (Guardar).

1. Seleccione **No** para rechazar la opción para continuar con la configuración del segmento. 

   :::image type="content" source="../media/nsxt/nsxt-create-segment-continue-no.png" alt-text="Para seguir configurando el segmento de red recién creado, seleccione NO.":::

1. Confirme la presencia del nuevo segmento de red. En este ejemplo, **ls01** es el nuevo segmento de red.

   1. En NSX-T Manager, seleccione **Networking** > **Segments** (Redes > Segmentos). 

      :::image type="content" source="../media/nsxt/nsxt-new-segment-overview-2.png" alt-text="Confirme que el nuevo segmento de red esté presente en NSX-T.":::

   1. En vCenter, seleccione **Networking > SDDC-Datacenter** (Redes > SDDC: centro de datos).

      :::image type="content" source="../media/nsxt/vcenter-with-ls01-2.png" alt-text="Confirme que el nuevo segmento de red está presente en vCenter.":::