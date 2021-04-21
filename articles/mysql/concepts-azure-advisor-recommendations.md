---
title: Azure Advisor for MySQL
description: Conozca las recomendaciones de Azure Advisor para MySQL.
author: alau-ms
ms.author: alau
ms.service: mysql
ms.topic: conceptual
ms.date: 04/08/2021
ms.openlocfilehash: 4b3d750c930defbfcb7db6d4d67210e001e3ce8b
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107315331"
---
# <a name="azure-advisor-for-mysql"></a>Azure Advisor for MySQL
Obtenga información sobre cómo se aplica Azure Advisor a Azure Database for MySQL obtenga respuestas a preguntas habituales.
## <a name="what-is-azure-advisor-for-mysql"></a>¿Qué es Azure Advisor for MySQL?
El sistema Azure Advisor usa telemetría para emitir recomendaciones sobre el rendimiento y la confiabilidad de la base de datos MySQL. Las recomendaciones de Advisor se dividen entre nuestras ofertas de base de datos MySQL:
* Servidor único de Azure Database for MySQL
* Azure Database for MySQL con la opción Servidor flexible

Algunas recomendaciones son comunes a varias ofertas de productos, mientras que otras se basan en optimizaciones específicas de un producto.
## <a name="where-can-i-view-my-recommendations"></a>¿Dónde puedo ver mis recomendaciones?
Las recomendaciones están disponibles en la barra lateral de navegación de **Información general** en Azure Portal. Aparecerá una versión preliminar como notificación de banner y los detalles se pueden ver en la sección **Notificaciones** que se encuentra justo debajo de los gráficos de uso de recursos.

:::image type="content" source="./media/concepts-azure-advisor-recommendations/advisor-example.png" alt-text="Captura de pantalla de Azure Portal que muestra una recomendación de Azure Advisor.":::

## <a name="recommendation-types"></a>Tipos de recomendaciones
Azure Database for MySQL prioriza los siguientes tipos de recomendaciones:
* **Rendimiento**: para mejorar la velocidad del servidor MySQL. Incluye el uso de CPU, la presión de memoria, la agrupación de conexiones, el uso del disco y los parámetros de servidor específicos del producto. Para obtener más información, consulte las [recomendaciones sobre rendimiento de Advisor](../advisor/advisor-performance-recommendations.md).
* **Confiabilidad**: para garantizar y mejorar la continuidad de las bases de datos críticas para la empresa. Incluye recomendaciones de límite de almacenamiento y límite de conexión. Para obtener más información, consulte las [recomendaciones de confiabilidad de Advisor](../advisor/advisor-high-availability-recommendations.md).
* **Costo**: ayuda a optimizar y reducir el gasto general de Azure. Incluye recomendaciones del dimensionamiento correcto del servidor. Para obtener más información, consulte las [recomendaciones sobre el costo de Advisor](../advisor/advisor-cost-recommendations.md).

## <a name="understanding-your-recommendations"></a>Descripción de las recomendaciones
* **Programación diaria**: en el caso de las bases de datos de Azure MySQL, se comprueba la telemetría del servidor y se emiten recomendaciones a diario. Si realiza un cambio en la configuración del servidor, las recomendaciones existentes permanecerán visibles hasta que se vuelva a examinar la telemetría al día siguiente. 
* **Historial de rendimiento**: algunas de nuestras recomendaciones se basan en el historial de rendimiento. Estas recomendaciones solo aparecerán después de que un servidor haya estado funcionando con la misma configuración durante 7 días. Nos permite detectar patrones de uso intensivo (por ejemplo, una actividad de CPU elevada o un volumen alto de conexión) durante un período de tiempo prolongado. Si aprovisiona un nuevo servidor o cambia a una nueva configuración de núcleo virtual, estas recomendaciones se pausarán temporalmente. Así se evita que la telemetría heredada desencadene recomendaciones en un servidor recién reconfigurado. Sin embargo, esto también significa que es posible que las recomendaciones basadas en el historial de rendimiento no se identifiquen inmediatamente.

## <a name="next-steps"></a>Pasos siguientes
Para más información, consulte [Información general sobre Azure Advisor](../advisor/advisor-overview.md).
