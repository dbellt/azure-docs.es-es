---
title: Versiones de software de VMware
description: Versiones de software de VMware compatibles con Azure VMware Solution.
ms.topic: include
ms.date: 03/31/2021
ms.openlocfilehash: a6441b55bbc6a8f694c50bbf022a6a2ae52d60bf
ms.sourcegitcommit: 99fc6ced979d780f773d73ec01bf651d18e89b93
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/31/2021
ms.locfileid: "106097973"
---
<!-- Used in faq.md and concepts-private-clouds-clusters.md -->


Las versiones de software de VMware utilizadas en las nuevas implementaciones de clústeres de nubes privadas de Azure VMware Solution son:

| Software              |    Versión   |
| :---                  |     :---:    |
| VCSA/vSphere/ESXi |    6.7 U3l    | 
| ESXi                  |    6.7 U3l    | 
| vSAN                  |    6.7 U3l    |
| NSX-T <br />**NOTA:** : La única versión admitida de NSX es NSX-T.               |      3.1.1     |


Si se agregan nuevos clústeres a una nube privada existente, se aplica la versión de software que se está ejecutando actualmente. Para obtener más información, consulte los [requisitos de la versión de software de VMware](https://docs.vmware.com/en/VMware-HCX/services/user-guide/GUID-54E5293B-8707-4D29-BFE8-EE63539CC49B.html).

Las actualizaciones del conjunto de productos de software de la nube privada mantienen el software dentro de una de las versiones más recientes del conjunto de productos de software de VMware. Las versiones de software de la nube privada pueden diferir de las versiones más recientes de los componentes de software individuales (ESXi, NSX-T, vCenter, vSAN). Podrá encontrar las directivas y procesos de actualización generales del software de la plataforma Azure VMware Solution descritos en [Actualizaciones y mejoras de la nube privada](../concepts-upgrades.md).

