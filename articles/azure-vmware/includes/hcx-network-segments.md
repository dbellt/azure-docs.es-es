---
title: Segmentos de red de VMware HCX
description: Hay cuatro redes necesarias para VMware HCX.
ms.topic: include
ms.date: 03/13/2021
ms.openlocfilehash: f2f5597471355bf4d74be7fbe370e6fa58770bf1
ms.sourcegitcommit: b28e9f4d34abcb6f5ccbf112206926d5434bd0da
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107251510"
---
<!-- Used in avs-production-ready-deployment.md and tutorial-deploy-vmware-hcx.md -->

Existen diferentes maneras de configurar los segmentos de red de VMware HCX en el entorno local. A continuación, se describe una configuración simple que admite un caso de uso de producción pequeña o piloto.  Al diseñar para cientos o miles de cargas de trabajo, es posible que sea necesario cambiar esta configuración, en función de las necesidades de la migración.  

Como preparación para que la implementación de VMware HCX admita el caso de uso de producción pequeña o piloto, identifique lo siguiente:

- **Red de administración:** al implementar VMware HCX en un entorno local, deberá identificar una red de administración para VMware HCX.  Normalmente, es la misma red de administración que usa el clúster de VMware local.  Como mínimo, identifique **dos** direcciones IP en este segmento de red para VMware HCX. Es posible que necesite números mayores, en función de la escala de la implementación más allá del caso de uso pequeño o piloto.

  > [!NOTE]
  > La preparación de entornos grandes, en lugar de usar la red de administración usada para el clúster de VMware local, cree una red /26 y presente dicha red como un grupo de puertos a su clúster de VMware local.  Luego puede crear hasta 10 mallas de servicio y 60 controles extensores de red (-1 por malla de servicio). Se pueden expandir **ocho** redes por cada extensor de red mediante nubes privadas de soluciones de Azure VMware Solution.
  >

- **Red de vínculo superior:** al implementar VMware HCX en un entorno local, deberá identificar una red de vínculo superior para VMware HCX. Use la misma red que va a usar para la red de administración. 

- **Red de vMotion:** al implementar VMware HCX en un entorno local, deberá identificar una red de vMotion para VMware HCX.  Normalmente, es la misma red que usa el clúster de VMware local para vMotion.  Como mínimo, identifique **dos** direcciones IP en este segmento de red para VMware HCX. Es posible que necesite números mayores, en función de la escala de la implementación más allá del caso de uso pequeño o piloto.

   La red de vMotion debe exponerse en un conmutador virtual distribuido o vSwitch0. Si no es así, modifique el entorno para adaptarlo.

   > [!NOTE]
   > Muchos entornos de VMware utilizan segmentos de red no enrutados para vMotion, que no plantea ningún problema.
  
- **Red de replicación:** al implementar VMware HCX local, deberá definir una red de replicación. Utilice la misma red que usa para las redes de administración y de vínculo superior.  Si los hosts del clúster local usan una red VMkernel de replicación dedicada, reserve **dos** direcciones IP en este segmento de red y use la red VMkernel de replicación para la red de replicación.
