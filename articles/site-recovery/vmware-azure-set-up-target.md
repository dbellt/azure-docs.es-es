---
title: Preparación del entorno de destino para la replicación de máquinas virtuales de VMware en Azure Site Recovery
description: En este artículo se describe cómo configurar el entorno de Azure de destino para la replicación de máquinas virtuales de VMware en Azure.
services: site-recovery
author: Sharmistha-Rai
manager: gaggupta
ms.service: site-recovery
ms.topic: article
ms.author: sharrai
ms.date: 05/27/2021
ms.openlocfilehash: ad52303ed03f5894c09308639ed5281854deae40
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110576594"
---
# <a name="prepare-the-target-environment-for-disaster-recovery-of-vmware-vms-or-physical-servers-to-azure"></a>Preparación del entorno de destino para la recuperación ante desastres de máquinas virtuales de VMware o servidores físicos en Azure

En este artículo se describe cómo configurar el entorno de Azure de destino para empezar a replicar máquinas virtuales de VMware o servidores físicos en Azure.

## <a name="prerequisites"></a>Prerrequisitos

En este artículo se da por supuesto lo siguiente:
- Ha creado un almacén de Recovery Services en [Azure Portal](https://portal.azure.com "Portal de Azure") para proteger las máquinas de origen.
- Ha configurado el entorno local para replicar las [máquinas virtuales de VMware](vmware-azure-set-up-source.md) o los [servidores físicos](physical-azure-set-up-source.md) en Azure.

## <a name="prepare-target"></a>Preparación del destino

Después de completar el **Paso 1: Selección del objetivo de protección** y el **Paso 2: Preparación del origen**, se le dirigirá al **Paso 3: Destino**

![Preparación del destino](./media/vmware-azure-set-up-target/prepare-target-vmware-to-azure.png)

1. **Subscription** (Suscripción): en el menú desplegable, seleccione la suscripción en la que quiere replicar las máquinas virtuales o los servidores físicos.
2. **Modelo de implementación:** seleccione el modelo de implementación (clásico o Resource Manager)

Según el modelo de implementación elegido, se ejecuta una validación para asegurarse de que tiene al menos una red virtual en la suscripción de destino en la que replicar y conmutar por error las máquinas virtuales o los servidores físicos.

Una vez completadas las validaciones correctamente, haga clic en Aceptar para ir al paso siguiente.

Si no tiene una red virtual, puede crearla haciendo clic en el botón **+ Agregar** en la parte superior de la página.

## <a name="next-steps"></a>Pasos siguientes
[Configuración de las opciones de replicación](vmware-azure-set-up-replication.md).
