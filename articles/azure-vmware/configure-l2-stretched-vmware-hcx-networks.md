---
title: Configuración de DHCP en redes de VMware HCX extendido L2
description: Aprenda a enviar solicitudes DHCP desde máquinas virtuales de Azure VMware Solution a un servidor DHCP que no es de NSX-T.
ms.topic: how-to
ms.date: 05/28/2021
ms.openlocfilehash: fb72fef74d83c2e860cb0d2ad661a91c40197f5a
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111746976"
---
# <a name="configure-dhcp-on-l2-stretched-vmware-hcx-networks"></a>Configuración de DHCP en redes de VMware HCX extendido L2

DHCP no funciona para las máquinas virtuales (VM) de la red elástica L2 de VMware HCX cuando el servidor DHCP está en el centro de datos local.  De forma predeterminada, NSX impide que todas las solicitudes DHCP atraviesen la red elástica L2. Si desea enviar solicitudes DHCP desde las máquinas virtuales de Azure VMware Solution a un servidor DHCP que no es de NSX-T, deberá configurar DHCP en redes de VMware HCX con extensión L2.

1. (Opcional) Si necesita buscar el nombre de segmento de la extensión L2:

   1. Inicie sesión en la instancia local de vCenter y en **Inicio**, seleccione **HCX**.

   1. Seleccione **Network Extension** (Extensión de red) en **Services** (Servicios).

   1. Seleccione la extensión de red que desea que admita las solicitudes DHCP de Azure VMware Solution en el entorno local.

   1. Tome nota del nombre de la red de destino.

      :::image type="content" source="media/manage-dhcp/hcx-find-destination-network.png" alt-text="Captura de pantalla de una extensión de red en el cliente VMware vSphere" lightbox="media/manage-dhcp/hcx-find-destination-network.png":::

1. En NSX-T Manager, seleccione **Networking** > **Segments** > **Segment Profiles** (Redes > Segmentos > Perfiles de segmento).

1. Seleccione **Add Segment Profile** (Agregar perfil de segmento) y luego **Segment Security** (Seguridad de segmento).

   :::image type="content" source="media/manage-dhcp/add-segment-profile.png" alt-text="Captura de pantalla de cómo agregar un perfil de segmento en NSX-T" lightbox="media/manage-dhcp/add-segment-profile.png":::
1. Proporcione un nombre y una etiqueta y, a continuación, establezca el conmutador **BPDU Filter** (Filtro BPDU) en ON (Activado) y todos los conmutadores de DHCP en OFF (Desactivado).

   :::image type="content" source="media/manage-dhcp/add-segment-profile-bpdu-filter-dhcp-options.png" alt-text="Captura de pantalla que muestra el conmutador de filtro BPDU activado y los conmutadores de DHCP desactivados" lightbox="media/manage-dhcp/add-segment-profile-bpdu-filter-dhcp-options.png":::
    
   :::image type="content" source="media/manage-dhcp/edit-segment-security.png" alt-text="Captura de pantalla del campo Seguridad del segmento" lightbox="media/manage-dhcp/edit-segment-security.png":::
