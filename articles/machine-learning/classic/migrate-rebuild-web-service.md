---
title: 'ML Studio (clásico): Migración a Azure Machine Learning - Recompilación de servicios web'
description: Recompile servicios Web de Studio (Classic) como puntos de conexión de canalización en Azure Machine Learning
services: machine-learning
ms.service: machine-learning
ms.subservice: studio-classic
ms.topic: how-to
author: xiaoharper
ms.author: zhanxia
ms.date: 03/08/2021
ms.openlocfilehash: 35ee5bf22aa88c18bade0ebdcd961b7687d24e7f
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107311828"
---
# <a name="rebuild-a-studio-classic-web-service-in-azure-machine-learning"></a>Recompilación de un servicio web de Studio (clásico) en Azure Machine Learning

En este artículo, aprenderá a recompilar un servicio web de Studio (clásico) como **punto de conexión** en Azure Machine Learning.

Los puntos de conexión de canalización de Azure Machine Learning se usan para realizar predicciones, volver a entrenar modelos o ejecutar cualquier canalización genérica. El punto de conexión REST permite ejecutar canalizaciones desde cualquier plataforma. 

Este artículo forma parte de la serie de migración de Studio (clásico) a Azure Machine Learning. Para obtener más información sobre cómo migrar a Azure Machine Learning, consulte el [artículo de información general sobre la migración](migrate-overview.md).

> [!NOTE]
> Esta serie sobre la migración se centra en el diseñador de arrastrar y colocar. Para obtener más información sobre la implementación de modelos mediante programación, consulte [Implementación de modelos de aprendizaje automático en Azure](../how-to-deploy-and-where.md).

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Un área de trabajo de Azure Machine Learning. [Cree un área de trabajo de Azure Machine Learning](../how-to-manage-workspace.md#create-a-workspace).
- Una canalización de entrenamiento de Azure Machine Learning. Para más información, consulte [Recompilación de un experimento de Studio (clásico) en Azure Machine Learning](migrate-rebuild-experiment.md).

## <a name="real-time-endpoint-vs-pipeline-endpoint"></a>Punto de conexión en tiempo real y punto de conexión de canalización

Los servicios web de Studio (clásico) se han reemplazado por **puntos de conexión** en Azure Machine Learning. Use la tabla siguiente para elegir el tipo de punto de conexión que desea usar:

|Servicio web de Studio (clásico)| Reemplazo en Azure Machine Learning
|---|---|
|Servicio web de solicitud/respuesta (predicción en tiempo real)|Punto de conexión en tiempo real|
|Servicio web de lotes (predicción por lotes)|Punto de conexión de canalización|
|Servicio web de nuevo entrenamiento (nuevo entrenamiento)|Punto de conexión de canalización| 


## <a name="deploy-a-real-time-endpoint"></a>Implementar un punto de conexión en tiempo real

En Studio (clásico), usaba un **servicio web REQUEST/RESPOND** para implementar un modelo para las predicciones en tiempo real. En Azure Machine Learning, se usa un **punto de conexión en tiempo real**.

Hay varias maneras de implementar un modelo en Azure Machine Learning. Una de las maneras más sencillas es usar el diseñador para automatizar el proceso de implementación. Siga estos pasos para implementar un modelo como un punto de conexión en tiempo real:

1. Ejecute la canalización de entrenamiento finalizada al menos una vez.
1. Cuando finalice la ejecución, en la parte superior del lienzo, seleccione **Create inference pipeline** > **Real-time inference pipeline** (Crear canalización de inferencia > Canalización de inferencia en tiempo real).

    ![crear canalización de inferencia en tiempo real](./media/migrate-rebuild-web-service/create-inference-pipeline.png)
        
    El diseñador convierte la canalización de entrenamiento en una canalización de inferencia en tiempo real. También se produce una conversión similar en Studio (clásico).

    En el diseñador, el paso de conversión también [registra el modelo entrenado en el área de trabajo de Azure Machine Learning](../how-to-deploy-and-where.md#registermodel).

1. Seleccione **Enviar** para ejecutar la canalización de inferencia en tiempo real y compruebe que se ejecuta correctamente.

1. Después de comprobar la canalización de inferencia, seleccione **Implementar**.

1. Escriba un nombre para el punto de conexión y un tipo de proceso.

    En la tabla siguiente se describen las opciones de proceso de implementación en el diseñador:

    | Destino de proceso | Se usa para | Descripción | Creación |
    | ----- |  ----- | ----- | -----  |
    |[Azure Kubernetes Service (AKS)](../how-to-deploy-azure-kubernetes-service.md) |Inferencia en tiempo real|Implementaciones de producción a gran escala. Tiempo de respuesta rápido y escalado automático del servicio.| Creado por el usuario. Para más información, consulte el artículo sobre [creación de destinos de proceso](../how-to-create-attach-compute-studio.md#inference-clusters). |
    |[Azure Container Instances ](../how-to-deploy-azure-container-instance.md)|Pruebas o desarrollo | Cargas de trabajo basadas en CPU a pequeña escala que requieren menos de 48 GB de RAM.| Azure Machine Learning lo crea automáticamente.

### <a name="test-the-real-time-endpoint"></a>Prueba del punto de conexión en tiempo real

Una vez finalizada la implementación, puede ver más detalles y probar el punto de conexión:

1. Haga clic en la pestaña **Puntos de conexión**.
1. Seleccione el punto de conexión.
1. Seleccione la pestaña **Prueba**.
    
    ![Captura de pantalla que muestra la pestaña Puntos de conexión con el botón para probar el punto de conexión](./media/migrate-rebuild-web-service/test-realtime-endpoint.png)

## <a name="publish-a-pipeline-endpoint-for-batch-prediction-or-retraining"></a>Publicación de un punto de conexión de canalización para un nuevo entrenamiento o la predicción por lotes

También puede usar la canalización de entrenamiento para crear un **punto de conexión de canalización** en lugar de un punto de conexión en tiempo real. Use **puntos de conexión de canalización** para realizar nuevos entrenamientos o predicciones por lotes.

Los puntos de conexión de canalización reemplazan los **puntos de conexión de ejecución por lotes** de Studio (clásico) y los **servicios web de nuevo entrenamiento**.

### <a name="publish-a-pipeline-endpoint-for-batch-prediction"></a>Publicación de un punto de conexión de canalización para la predicción por lotes

La publicación de un punto de conexión de predicción por lotes es similar a la del punto de conexión en tiempo real.

Siga estos pasos para publicar un punto de conexión de canalización para la predicción por lotes:

1. Ejecute la canalización de entrenamiento finalizada al menos una vez.

1. Cuando finalice la ejecución, en la parte superior del lienzo, seleccione **Create inference pipeline** > **Batch inference pipeline** (Crear canalización de inferencia > Canalización de inferencias por lotes).

    ![Captura de pantalla que muestra el botón para crear una canalización de inferencia en una canalización de entrenamiento](./media/migrate-rebuild-web-service/create-inference-pipeline.png)
        
    El diseñador convierte la canalización de entrenamiento en una canalización de inferencia por lotes. También se produce una conversión similar en Studio (clásico).

    En el diseñador, este paso también [registra el modelo entrenado en el área de trabajo de Azure Machine Learning](../how-to-deploy-and-where.md#registermodel).

1. Seleccione **Enviar** para ejecutar la canalización de inferencia por lotes y compruebe que se ejecuta correctamente.

1. Después de comprobar la canalización de inferencia, seleccione **Publicar**.
 
1. Cree un nuevo punto de conexión de canalización o seleccione uno existente.
    
    Un nuevo punto de conexión de canalización crea un nuevo punto de conexión REST para la canalización. 

    Si selecciona un punto de conexión de canalización existente, no se sobrescribirá la canalización existente. En su lugar, Azure Machine Learning crea una versión de cada canalización en el punto de conexión. Puede especificar la versión que se ejecutará en la llamada REST. También debe establecer una canalización predeterminada si la llamada REST no especifica una versión.


 ### <a name="publish-a-pipeline-endpoint-for-retraining"></a>Publicación de un punto de conexión de canalización para el nuevo entrenamiento

Para publicar un punto de conexión de canalización para el nuevo entrenamiento, ya debe tener un borrador de canalización que entrene un modelo. Para más información sobre cómo compilar una canalización de entrenamiento, consulte [Recompilación de un experimento de Studio (clásico)](migrate-rebuild-experiment.md).

Para volver a usar el punto de conexión de la canalización para el nuevo entrenamiento, debe crear un **parámetro de canalización** para el conjunto de datos de entrada. Esto le permite establecer de forma dinámica el conjunto de datos de entrenamiento, para que pueda volver a entrenar el modelo.

Siga estos pasos para publicar el punto de conexión de canalización del nuevo entrenamiento:

1. Ejecute la canalización de entrenamiento al menos una vez.
1. Una vez finalizada la ejecución, seleccione el módulo del conjunto de datos.
1. En el panel de detalles del módulo, seleccione **Set as pipeline parameter** (Establecer como parámetro de canalización).
1. Proporcione un nombre descriptivo, como "InputDataset".    

    ![Captura de pantalla que muestra cómo crear un parámetro de canalización](./media/migrate-rebuild-web-service/create-pipeline-parameter.png)

    De este modo crea un parámetro de canalización para el conjunto de datos de entrada. Cuando llame al punto de conexión de la canalización para el entrenamiento, puede especificar un nuevo conjunto de datos para volver a entrenar el modelo.

1. Seleccione **Publicar**.

    ![Captura de pantalla que resalta el botón para publicar en una canalización de entrenamiento](./media/migrate-rebuild-web-service/create-retraining-pipeline.png)


## <a name="call-your-pipeline-endpoint-from-the-studio"></a>Llamada al punto de conexión de la canalización desde Studio

Después de crear la inferencia de lotes o el punto de conexión de la canalización de nuevo entrenamiento, puede llamar al punto de conexión directamente desde el explorador.

1. Vaya a la pestaña **Canalizaciones** y seleccione **Puntos de conexión de canalización**.
1. Seleccione el punto de conexión de canalización que desea ejecutar.
1. Seleccione **Submit** (Enviar).

    Puede especificar cualquier parámetro de canalización después de seleccionar **Enviar**.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a recompilar un servicio web de Studio (clásico) en Azure Machine Learning. El siguiente paso consiste en [integrar el servicio web con las aplicaciones cliente](migrate-rebuild-integrate-with-client-app.md).


Consulte los demás artículos de la serie de migración de Studio (clásico):

1. [Información general sobre la migración](migrate-overview.md).
1. [Migración de un conjunto de datos de Studio a Azure Machine Learning](migrate-register-dataset.md).
1. [Recompilación de una canalización de entrenamiento de Studio (clásico)](migrate-rebuild-experiment.md).
1. **Recompilación de un servicio web de Studio (clásico)**.
1. [Integración de un servicio web de Azure Machine Learning con aplicaciones cliente](migrate-rebuild-integrate-with-client-app.md).
1. [Migración de los módulos Execute R Script (Ejecutar script R) de Studio (Clásico)](migrate-execute-r-script.md).
