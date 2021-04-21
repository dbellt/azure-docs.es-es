---
title: Azure Advisor for MariaDB
description: Conozca las recomendaciones de Azure Advisor para MariaDB.
author: alau-ms
ms.author: alau
ms.service: mariadb
ms.topic: conceptual
ms.date: 04/12/2021
ms.openlocfilehash: 60a0a5763085de38dbfd4dabcb65dc24b299b29a
ms.sourcegitcommit: dddd1596fa368f68861856849fbbbb9ea55cb4c7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107368544"
---
# <a name="azure-advisor-for-mariadb"></a>Azure Advisor for MariaDB
Conozca cómo Azure Advisor se aplica a Azure Database for MariaDB y obtenga respuestas a preguntas comunes.
## <a name="what-is-azure-advisor-for-mariadb"></a>¿Qué es Azure Advisor for MariaDB?
El sistema Azure Advisor emplea telemetría para emitir recomendaciones de rendimiento y confiabilidad para la base de datos MariaDB. 

Algunas recomendaciones son comunes a varias ofertas de productos, mientras que otras se basan en optimizaciones específicas de un producto.
## <a name="where-can-i-view-my-recommendations"></a>¿Dónde puedo ver mis recomendaciones?
Las recomendaciones están disponibles en la barra lateral de navegación de **Información general** en Azure Portal. Aparecerá una versión preliminar como notificación de banner y los detalles se pueden ver en la sección **Notificaciones** que se encuentra justo debajo de los grafos de uso de recursos.

:::image type="content" source="./media/concepts-azure-advisor-recommendations/advisor-example.png" alt-text="Captura de pantalla de Azure Portal que muestra una recomendación de Azure Advisor.":::

## <a name="recommendation-types"></a>Tipos de recomendaciones
Azure Database for MariaDB clasifica por orden de prioridad los siguientes tipos de recomendaciones:
* **Rendimiento**: para mejorar la velocidad del servidor de MariaDB. Incluye el uso de CPU y del disco, la presión de memoria y los parámetros de servidor específicos del producto. Para obtener más información, consulte las [recomendaciones sobre rendimiento de Advisor](../advisor/advisor-performance-recommendations.md).
* **Confiabilidad**: para garantizar y mejorar la continuidad de las bases de datos críticas para la empresa. Incluye recomendaciones sobre límites de almacenamiento y conexión. Para obtener más información, consulte las [recomendaciones de confiabilidad de Advisor](../advisor/advisor-high-availability-recommendations.md).
* **Costo**: ayuda a optimizar y reducir el gasto general de Azure. Incluye recomendaciones del dimensionamiento correcto del servidor. Para obtener más información, consulte las [recomendaciones sobre el costo de Advisor](../advisor/advisor-cost-recommendations.md).

## <a name="understanding-your-recommendations"></a>Descripción de las recomendaciones
* **Programación diaria**: en el caso de las bases de datos MariaDB de Azure, se comprueba la telemetría del servidor y se emiten recomendaciones según una programación diaria. Si realiza un cambio en la configuración del servidor, las recomendaciones existentes permanecerán visibles hasta que se vuelva a examinar la telemetría al día siguiente. 
* **Historial de rendimiento**: algunas de nuestras recomendaciones se basan en el historial de rendimiento. Estas recomendaciones solo aparecerán después de que un servidor haya estado funcionando con la misma configuración durante 7 días. Nos permite detectar patrones de uso intensivo (por ejemplo, una actividad de CPU elevada o un volumen alto de conexiones) durante un período de tiempo prolongado. Si aprovisiona un nuevo servidor o cambia a una nueva configuración de núcleo virtual, estas recomendaciones se pausarán temporalmente. Así se evita que la telemetría heredada desencadene recomendaciones en un servidor recién reconfigurado. Sin embargo, también significa que es posible que las recomendaciones basadas en el historial de rendimiento no se identifiquen inmediatamente.

## <a name="next-steps"></a>Pasos siguientes
Para más información, consulte [Información general sobre Azure Advisor](../advisor/advisor-overview.md).
