---
title: Implementación inicial mínima de hosts
description: La implementación inicial mínima es de tres hosts.
ms.topic: include
ms.date: 04/27/2021
ms.openlocfilehash: 72f0ad6c02c7ef24301276f1c6f68b300c86c0a4
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108126325"
---
<!-- Used in production-ready-deployment-steps.md and concepts-private-clouds-clusters.md -->


De manera predeterminada, hay un clúster vSAN para cada nube privada que se crea. Los clústeres se pueden agregar, eliminar y escalar.  El número mínimo de hosts por clúster es tres. Puede agregar más hosts de uno en uno, hasta un máximo de 16 por clúster. El número máximo de clústeres por nube privada es 12.  La implementación inicial de Azure VMware Solution incluye tres hosts. 

Los clústeres de prueba están disponibles con fines de evaluación y están limitados a tres hosts. Hay un solo clúster de prueba por cada nube privada. Puede escalar un clúster de prueba en un solo host durante el período de evaluación.

Se usa vSphere y NSX-T Manager para administrar la mayoría de los otros aspectos de la configuración o del funcionamiento de los clústeres. Todo el almacenamiento local de cada host de un clúster está bajo el control de vSAN.

>[!TIP]
>Si necesita aumentar el número de implementación inicial, siempre puede extender el clúster y agregar clústeres adicionales más adelante.