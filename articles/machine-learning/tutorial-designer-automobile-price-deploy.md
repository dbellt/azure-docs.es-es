---
title: 'Tutorial: Implementación de modelos de ML con el diseñador'
titleSuffix: Azure Machine Learning
description: Cree una solución de análisis predictivo en el diseñador de Azure Machine Learning. Entrene, puntúe e implemente un modelo de Machine Learning mediante módulos del tipo arrastrar y colocar.
author: likebupt
ms.author: keli19
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
ms.date: 05/25/2021
ms.custom: designer
ms.openlocfilehash: 73cc893815cf591812c95c8d8b092a4b58697917
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110457655"
---
# <a name="tutorial-deploy-a-machine-learning-model-with-the-designer"></a>Tutorial: Implementación de un modelo de Machine Learning con el diseñador


Puede implementar el modelo predictivo desarrollado en la [parte uno del tutorial](tutorial-designer-automobile-price-train-score.md) para que otros usuarios puedan usarlo. En la parte 1, ha entrenado el modelo. Ahora, es el momento de generar predicciones basadas en los datos especificados por el usuario. En esta parte del tutorial, va a realizar lo siguiente:

> [!div class="checklist"]
> * Crear una canalización de inferencia en tiempo real.
> * Crear un clúster de inferencia.
> * Implementación del punto de conexión en tiempo real.
> * Prueba del punto de conexión en tiempo real.

## <a name="prerequisites"></a>Requisitos previos

Complete la [parte uno del tutorial](tutorial-designer-automobile-price-train-score.md) para aprender a entrenar y puntuar un modelo de Machine Learning en el diseñador.

[!INCLUDE [machine-learning-missing-ui](../../includes/machine-learning-missing-ui.md)]

## <a name="create-a-real-time-inference-pipeline"></a>Crear una canalización de inferencia en tiempo real

Para implementar la canalización, antes debe convertir la canalización de entrenamiento en una canalización de inferencia en tiempo real. Este proceso quita los módulos de entrenamiento y agrega entradas y salidas de servicios web para administrar las solicitudes.

### <a name="create-a-real-time-inference-pipeline"></a>Crear una canalización de inferencia en tiempo real

1. Seleccione, encima del lienzo de la canalización, **Create inference pipeline** > **Real-time inference pipeline** (Crear canalización de inferencia > Canalización de inferencia en tiempo real).

    :::image type="content" source="./media/tutorial-designer-automobile-price-deploy/tutorial2-create-inference-pipeline.png" alt-text="Captura de pantalla que muestra dónde encontrar el botón Crear canalización":::

    La canalización debería tener este aspecto: 

   ![Captura de pantalla que muestra la configuración esperada de la canalización después de prepararla para la implementación](./media/tutorial-designer-automobile-price-deploy/real-time-inference-pipeline.png)

    Al seleccionar **Create inference pipeline** (Crear canalización de inferencia), suceden varias cosas:
    
    * El modelo entrenado se almacena como un módulo **Conjunto de datos** en la paleta de módulos. Puede encontrarlo en **My Datasets** (Mis conjuntos de datos).
    * Se quitan los módulos de entrenamiento como **Train Model** (Entrenar modelo) y **Split Data** (Dividir datos).
    * El modelo entrenado guardado se vuelve a agregar a la canalización.
    * Se agregan los módulos **Web Service Input** (Entrada de servicio web) y **Web Service Output** (Salida de servicio web). Estos módulos muestran dónde entran los datos del usuario en la canalización y dónde se devuelven.

    > [!NOTE]
    > De forma predeterminada, la **entrada del servicio web** esperará el mismo esquema de datos que los datos de salida del módulo que se conecta al mismo puerto de bajada que ellos. En este ejemplo, la **entrada del servicio web** y los **datos de precios de automóviles (sin procesar)** se conectan al mismo módulo de bajada, por lo que la **entrada de servicio web** espera el mismo esquema de datos que los **datos de precios de automóviles (sin procesar)** y la columna de variable de destino `price` se incluye en el esquema.
    > Sin embargo, al puntuar los datos, normalmente no conocerá los valores de las variables de destino. En tal caso, puede quitar la columna de variable de destino de la canalización de inferencia mediante el módulo **Seleccionar columnas de conjunto de datos**. Asegúrese de que la salida de **Seleccionar columnas de conjunto de datos** que quita la columna de variable de destino está conectada al mismo puerto que la salida del módulo **Entrada del servicio web**.

1. Seleccione **Enviar** y use el mismo destino de proceso y el mismo experimento que usó en la parte 1.

    Si esta es la primera ejecución, la canalización puede tardar hasta 20 minutos en finalizar la ejecución. La configuración del proceso predeterminada tiene un tamaño de nodo mínimo de 0, lo que significa que el diseñador debe asignar recursos después de estar inactivo. Las ejecuciones de canalización repetidas tardarán menos en terminar, dado que los recursos del proceso ya están asignados. Además, el diseñador usa resultados almacenados en la caché en cada módulo para mejorar aún más la eficiencia.

1. Seleccione **Implementar**.

## <a name="create-an-inferencing-cluster"></a>Creación de un clúster de inferencia

En el cuadro de diálogo que aparece, puede seleccionar entre los clústeres de Azure Kubernetes Service (AKS) existentes aquel en el que quiere implementar el modelo. Si no tiene un clúster de AKS, use los pasos siguientes para crear uno.

1. Seleccione **Compute** (Proceso) en el cuadro de diálogo que aparece, para ir a la página **Compute** (Proceso).

1. En la cinta de opciones de navegación, seleccione **Inference Clusters** (Clústeres de inferencia) >  **+ Nuevo**.

    ![Captura de pantalla que muestra cómo ir al panel del nuevo clúster de inferencia](./media/tutorial-designer-automobile-price-deploy/new-inference-cluster.png)
   
1. En el panel del clúster de inferencia, configure un nuevo servicio de Kubernetes.

1. Escriba *aks-compute* en **Compute name** (Nombre de proceso).
    
1. Seleccione una región cercana que esté disponible para la **región**.

1. Seleccione **Crear**.

    > [!NOTE]
    > La creación de un servicio AKS tarda unos 15 minutos. Puede comprobar el estado de aprovisionamiento en la página **Inference Clusters** (Clústeres de inferencia).
    >

## <a name="deploy-the-real-time-endpoint"></a>Implementación del punto de conexión en tiempo real

Después de que el servicio AKS haya terminado de aprovisionarse, vuelva a la canalización de inferencia en tiempo real para finalizar la implementación.

1. Seleccione **Implementar** encima del lienzo.

1. Seleccione **Deploy new real-time endpoint** (Implementar nuevo punto de conexión en tiempo real). 

1. Seleccione el clúster de AKS que ha creado.

    :::image type="content" source="./media/tutorial-designer-automobile-price-deploy/setup-endpoint.png" alt-text="Captura de pantalla que muestra cómo configurar un nuevo punto de conexión en tiempo real":::

    También puede cambiar la configuración **avanzada** del punto de conexión en tiempo real.
    
    |Configuración avanzada|Descripción|
    |---|---|
    |Enable Application Insights diagnostics and data collection (Habilitar la recopilación de datos y el diagnóstico de Application Insights)| Si se va a habilitar Azure Application Insights para recopilar datos de los puntos de conexión implementados. </br> Valor predeterminado: false |
    |Scoring timeout (Tiempo de espera de la puntuación)| Tiempo de espera en milisegundos para forzar la puntuación de las llamadas al servicio web.</br>De forma predeterminada: 60000|
    |Auto scale enabled (Escalado automático habilitado)|   Indica si se debe habilitar la escalabilidad automática para el servicio web.</br>Valor predeterminado: true|
    |Min replicas (Número mínimo de réplicas)| Número mínimo de contenedores que se van a usar al escalar automáticamente este servicio web.</br>De forma predeterminada: 1|
    |Max replicas (Número máximo de réplicas)| Número máximo de contenedores que se van a usar al escalar automáticamente este servicio web.</br> De forma predeterminada: 10|
    |Target utilization (Uso de destino)|Uso objetivo (en un porcentaje de 100) que el escalador automático debe intentar mantener para este servicio web.</br> De forma predeterminada: 70|
    |Refresh period (Período de actualización)|Frecuencia (en segundos) con la que la escalabilidad automática intenta escalar este servicio web.</br> De forma predeterminada: 1|
    |CPU reserve capacity (Capacidad de reserva de CPU)|Número de núcleos de CPU que se asigna a este servicio web.</br> De forma predeterminada: 0,1|
    |Memory reserve capacity (Capacidad de reserva de memoria)|Cantidad de memoria (en GB) que se va a asignar a este servicio web.</br> De forma predeterminada: 0.5|
        

1. Seleccione **Implementar**. 

    Aparece una notificación de finalización correcta en la parte superior del lienzo una vez que termina la implementación. Esto puede llevar unos minutos.

> [!TIP]
> También puede realizar la implementación en **Instancia de Azure Container** (ACI) mediante la selección de **Instancia de Azure Container** en **Tipo de proceso** en el cuadro de configuración de puntos de conexión en tiempo real.
> Instancia de Azure Container se usa con fines de pruebas o desarrollo. Use ACI con cargas de trabajo basadas en CPU a pequeña escala que requieran menos de 48 GB de RAM.

## <a name="test-the-real-time-endpoint"></a>Prueba del punto de conexión en tiempo real

Una vez finalizada la implementación, puede ver el punto de conexión en tiempo real; para ello, vaya a la página **Endpoints** (Puntos de conexión).

1. En la página **Puntos de conexión**, seleccione el punto de conexión que implementó.

    En la pestaña **Detalles**, puede ver más información, como el identificador URI de REST, la definición de Swagger, el estado y las etiquetas.

    En la pestaña **Consumir**, encontrará código de consumo de ejemplo y claves de seguridad. Además, podrá establecer los métodos de autenticación.

    En la pestaña **Registros de implementación**, puede encontrar los registros de implementación detallados del punto de conexión en tiempo real.

1. Para probar el punto de conexión, vaya a la pestaña **Prueba**. Aquí puede especificar los datos de prueba y seleccionar **Probar** para verificar la salida del punto de conexión.

Para más información sobre cómo utilizar el servicio web, consulte [Consumo de un modelo implementado como servicio web](how-to-consume-web-service.md).

## <a name="limitations"></a>Limitaciones

### <a name="update-inference-pipeline"></a>Actualizar canalización de inferencia

Si realiza cualquier modificación en la canalización de entrenamiento, debe volver a enviarla, **actualizar** la canalización de inferencia y volver a ejecutar esta última canalización.

Tenga en cuenta que en la canalización de inferencia solo se actualizarán los modelos entrenados, la transformación de datos no se actualizará.

Para usar la transformación actualizada en la canalización de inferencia, es preciso registrar la salida de la transformación del módulo de transformación como conjunto de datos.

![Captura de pantalla en la que se muestra cómo registrar un conjunto de datos de transformación](./media/tutorial-designer-automobile-price-deploy/register-transformation-dataset.png)

Luego, reemplace de forma manual el módulo **TD-** en la canalización de inferencia con el conjunto de datos registrado.

![Captura de pantalla que muestra cómo reemplazar el módulo de la transformación](./media/tutorial-designer-automobile-price-deploy/replace-td-module.png)

A continuación, puede enviar la canalización de inferencia con el modelo y la transformación actualizados e implementarla.

### <a name="deploy-real-time-endpoint"></a>Implementar un punto de conexión en tiempo real

Debido a la limitación del acceso al almacén de datos, si la canalización de inferencia contiene el módulo **Importar datos** o **Exportar datos**, se quitará automáticamente cuando se implemente en el punto de conexión en tiempo real.

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [aml-ui-cleanup](../../includes/aml-ui-cleanup.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha aprendido los pasos clave para crear, implementar y usar un modelo de Machine Learning en el diseñador. Para más información sobre cómo puede utilizar el diseñador, consulte los vínculos siguientes:

+ [Ejemplos del diseñador](samples-designer.md): aprenda a utilizar el diseñador para resolver otros tipos de problemas.
+ [Habilitación de Azure Machine Learning Studio en una Azure Virtual Network](how-to-enable-studio-virtual-network.md).
