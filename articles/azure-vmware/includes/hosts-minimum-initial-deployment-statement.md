---
title: Implementación inicial mínima de hosts
description: La implementación inicial mínima es de tres hosts.
ms.topic: include
ms.date: 04/28/2021
ms.openlocfilehash: 49eb2fdd105b02a52d0cba555a78e0d44d133bf6
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108179551"
---
<!-- Used in production-ready-deployment-steps.md and concepts-private-clouds-clusters.md -->

De manera predeterminada, hay un clúster vSAN para cada nube privada que se crea. Los clústeres se pueden agregar, eliminar y escalar.  El número mínimo de hosts por clúster y la implementación inicial es tres. 

Los clústeres de prueba están disponibles con fines de evaluación y están limitados a tres hosts. Hay un único clúster de prueba por nube privada en el que puede escalarlo mediante un único host durante el período de evaluación.

Se usa vSphere y NSX-T Manager para administrar la mayoría de los otros aspectos de la configuración o del funcionamiento de los clústeres. Todo el almacenamiento local de cada host de un clúster está bajo el control de vSAN.

>[!TIP]
>Si necesita aumentar el número de implementación inicial, siempre puede extender el clúster y agregar clústeres adicionales más adelante.
