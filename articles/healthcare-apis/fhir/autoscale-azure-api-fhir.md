---
title: Escalado automático para Azure API for FHIR
description: En este artículo se describe la característica de escalado automático para Azure API for FHIR.
author: stevewohl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: conceptual
ms.date: 07/26/2021
ms.author: zxue
ms.openlocfilehash: cc5fd2dd91c1edc39886938d91ebb54bcbc3ef0d
ms.sourcegitcommit: e6de87b42dc320a3a2939bf1249020e5508cba94
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 07/27/2021
ms.locfileid: "114713399"
---
# <a name="autoscale-for-azure-api-for-fhir"></a>Escalado automático para Azure API for FHIR 

La Azure API for FHIR como servicio administrado permite a los clientes conservar con datos de atención sanitaria compatibles con FHIR e intercambiar los datos de forma segura a través de la API de servicio. Para dar cabida a diferentes cargas de trabajo de transacciones, los clientes pueden usar escalado manual o escalado automático.

## <a name="what-is-autoscale"></a>¿Qué es el escalado automático?

De forma predeterminada, el Azure API for FHIR se establece en escala manual. Esta opción funciona bien cuando las cargas de trabajo de transacción son conocidas y coherentes. Los clientes pueden ajustar el rendimiento a través del portal hasta `RU/s` 10 000 y enviar una solicitud para aumentar el límite. 

Con la escalabilidad automática, los clientes pueden ejecutar varias cargas de trabajo y el rendimiento se escala vertical y `RU/s` verticalmente automáticamente sin ajustes manuales.

## <a name="how-to-enable-autoscale"></a>¿Cómo habilitar el escalado automático?

Para habilitar la característica de escalado automático, puede crear una única vale de soporte técnico para solicitarla. El equipo de soporte técnico de Microsoft habilitará la característica de escalado automático en función de la prioridad de soporte técnico.

> [!NOTE]
> La característica de escalado automático no está disponible en el Azure Portal.

## <a name="how-to-adjust-the-maximum-throughput-rus"></a>¿Cómo ajustar las RU/s de rendimiento máximo?

Cuando el escalado automático está habilitado, el sistema calcula y establece el valor `Tmax` inicial. La escalabilidad se rige por el valor de rendimiento máximo, o `RU/s` , y se ejecuta entre `Tmax` `0.1 *Tmax` (o 10 % `Tmax` ) y `Tmax RU/s` . 

Puede aumentar el valor máximo `RU/s` o e ir tan alto como admite el `Tmax` servicio. Cuando el servicio está ocupado, el rendimiento `RU/s` se escala verticalmente hasta el `Tmax` valor . Cuando el servicio está inactivo, el rendimiento se escala `RU/s` verticalmente hasta un valor del 10 `Tmax` %.
 
También puede reducir el valor `RU/s` máximo `Tmax` o . Cuando se reduce el valor máximo , el valor mínimo que puede establecer en es: , redondeado al `RU/s` `MAX (4000, highest max RU/s ever provisioned / 10, current storage in GB * 400)` 1000 más `RU/s` cercano.

* **Ejemplo 1:** tiene datos de 1 GB y el mayor `RU/s` aprovisionado es 10 000. El valor mínimo es Max (**4000**, 10 000/10, 1x400) = 4000. Se usa el **primer número, 4000**.
* **Ejemplo 2:** tiene datos de 20 GB y el máximo aprovisionado `RU/s` es 100 000. El valor mínimo es Max (4000, **100 000/10,** 20x400) = 10 000. Se usa el segundo número, **100 000/10 =10 000.**
* **Ejemplo 3:** tiene datos de 80 GB y las RU/s aprovisionadas más altas son 300 000. El valor mínimo es Max (4000, 300 000/10, **80x400**) = 32 000. Se usa el **tercer número, 80x400=32 000.**

Puede ajustar el valor máximo o a través del portal si es un número válido y no es mayor `RU/s` `Tmax` que 10 000 `RU/s` . Puede crear una vale de soporte técnico para solicitar un valor superior a `Tmax` 10 000.

## <a name="what-is-the-cost-impact-of-autoscale"></a>¿Cuál es el impacto en el costo de la escalado automático?

La característica de escalado automático incurre en costos debido a la administración automática de las unidades de rendimiento aprovisionadas. Este aumento de costos no se aplica a los costos de almacenamiento y tiempo de ejecución. Para obtener información sobre los precios, [vea Azure API for FHIR precios.](https://azure.microsoft.com/pricing/details/azure-api-for-fhir/)

>[!div class="nextstepaction"]
>[Acerca de la API de Azure para FHIR](overview.md)
