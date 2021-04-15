---
title: 'ML Studio (clásico): Migración a Azure Machine Learning - Consumo de puntos de conexión de canalización'
description: Integre los puntos de conexión de canalización con las aplicaciones cliente en Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio-classic
ms.topic: how-to
author: xiaoharper
ms.author: zhanxia
ms.date: 03/08/2021
ms.openlocfilehash: fc7e222d400c2ded602e3d26d504896302fff014
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107311794"
---
# <a name="consume-pipeline-endpoints-from-client-applications"></a>Consumo de puntos de conexión de canalización de aplicaciones cliente

En este artículo, aprenderá a integrar aplicaciones cliente con puntos de conexión de Azure Machine Learning. Para obtener más información sobre cómo escribir código de aplicación, consulte [Consumir un modelo de Azure Machine Learning que está implementado como un servicio web](../how-to-consume-web-service.md).

Este artículo forma parte de la serie de migración de Studio (clásico) a Azure Machine Learning. Para obtener más información sobre cómo migrar a Azure Machine Learning, consulte el [artículo de información general sobre la migración](migrate-overview.md).

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Un área de trabajo de Azure Machine Learning. [Cree un área de trabajo de Azure Machine Learning](../how-to-manage-workspace.md#create-a-workspace).
- Un [punto de conexión en tiempo real o un punto de conexión de canalización de Azure Machine Learning](migrate-rebuild-web-service.md).


## <a name="consume-a-real-time-endpoint"></a>Consumo de un punto de conexión en tiempo real 

Si ha implementado el modelo como un **punto de conexión en tiempo real**, puede encontrar su punto de conexión REST y el código de consumo generado previamente en C#, Python y R:

1. Vaya a Estudio de Azure Machine Learning en [ml.azure.com](https://ml.azure.com).
1. Haga clic en la pestaña **Puntos de conexión**.
1. Seleccione el punto de conexión en tiempo real.
1. Seleccione **Consumir**.

> [!NOTE]
> También puede encontrar la especificación de Swagger del punto de conexión en la pestaña **Detalles**. Utilice esta definición para comprender el esquema del punto de conexión. Para obtener más información sobre la definición Swagger, consulte la [documentación oficial de Swagger](https://swagger.io/docs/specification/2-0/what-is-swagger/).


## <a name="consume-a-pipeline-endpoint"></a>Consumir un punto de conexión de canalización

Hay dos maneras de consumir un punto de conexión de canalización:

- Llamadas a la API de REST
- Integración con Azure Data Factory

### <a name="use-rest-api-calls"></a>Uso de llamadas a la API de REST

Llame al punto de conexión REST desde la aplicación cliente. Puede usar la especificación Swagger del punto de conexión para comprender su esquema:

1. Vaya a Estudio de Azure Machine Learning en [ml.azure.com](https://ml.azure.com).
1. Haga clic en la pestaña **Puntos de conexión**.
1. Seleccione **Pipeline endpoints** (Puntos de conexión de canalización).
1. Seleccione el punto de conexión de la canalización.
1. En el panel de **Información general sobre los puntos de conexión de las canalizaciones**, seleccione el vínculo que aparece debajo de la **documentación del punto de conexión REST**.

### <a name="use-azure-data-factory"></a>Usar Azure Data Factory

Puede llamar a la canalización de Azure Machine Learning como un paso en una canalización de Azure Data Factory. Para más información, consulte [Ejecución de canalizaciones de Azure Machine Learning en Azure Data Factory](../../data-factory/transform-data-machine-learning-service.md).


## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a encontrar el esquema y el código de ejemplo de los puntos de conexión de la canalización. Para obtener más información sobre cómo consumir los puntos de conexión de la aplicación cliente, consulte [Consumir un modelo de Azure Machine Learning que está implementado como un servicio web](../how-to-consume-web-service.md).

Consulte el resto de los artículos de la serie sobre migración a Azure Machine Learning: 
1. [Información general sobre la migración](migrate-overview.md).
1. [Migración de un conjunto de datos de Studio a Azure Machine Learning](migrate-register-dataset.md).
1. [Recompilación de una canalización de entrenamiento de Studio (clásico)](migrate-rebuild-experiment.md).
1. [Recompilación de un servicio web de Studio (clásico)](migrate-rebuild-web-service.md).
1. **Integración de un servicio web de Azure Machine Learning con aplicaciones cliente**.
1. [Migración de los módulos Execute R Script (Ejecutar script R) de Studio (Clásico)](migrate-execute-r-script.md).