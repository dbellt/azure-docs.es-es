---
title: Solicitud de la clave de autorización para ExpressRoute
description: Pasos para solicitar una clave de autorización para ExpressRoute.
ms.topic: include
ms.date: 03/15/2021
ms.openlocfilehash: 44ca81e25dda61ab32ea3455a1f228e134952582
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107945579"
---
<!-- used in tutorial-expressroute-global-reach-private-cloud.md and create-ipsec-tunnel.md -->

1. En Azure Portal, vaya a la nube privada de Azure VMware Solution. Seleccione **Administrar** > **Conectividad** > **ExpressRoute** y seleccione **+ Solicitar una clave de autorización**.

   :::image type="content" source="../media/expressroute-global-reach/start-request-authorization-key.png" alt-text="Captura de pantalla que muestra cómo solicitar una clave de autorización de ExpressRoute." border="true" lightbox="../media/expressroute-global-reach/start-request-authorization-key.png":::

1. Proporciónele un nombre y luego seleccione **Crear**.

   La clave puede tardar unos 30 segundos en crearse. Una vez creada, la nueva clave aparece en la lista de claves de autorización para la nube privada.

   :::image type="content" source="../media/expressroute-global-reach/show-global-reach-auth-key.png" alt-text="Captura de pantalla que muestra la clave de autorización de ExpressRoute Global Reach.":::
  
1. Copie la clave de autorización y el identificador de ExpressRoute. Necesitará la información para finalizar el emparejamiento. La clave de autorización desaparece tras un tiempo, por lo que debe copiarla en cuanto aparezca.

