---
title: Traiga su propio aprendizaje automático a Azure Sentinel | Microsoft Docs
description: En este artículo se explica cómo crear y usar sus propios algoritmos de aprendizaje automático para el análisis de datos en Azure Sentinel.
services: sentinel
cloud: na
documentationcenter: na
author: yelevin
manager: rkarlin
ms.assetid: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/23/2020
ms.author: yelevin
ms.openlocfilehash: e912cabd27bac52792ffb2d71027170f4e53f310
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109784528"
---
# <a name="bring-your-own-machine-learning-ml-into-azure-sentinel"></a>Traiga su propio aprendizaje automático a Azure Sentinel

El aprendizaje automático es uno de los principales fundamentos de Azure Sentinel y uno de los principales atributos que lo diferencian. Azure Sentinel ofrece el aprendizaje automático en varias experiencias: integrado en el motor de correlación de [Fusion](fusion.md) y en los cuadernos de Jupyter Notebook, así como en la nueva plataforma Traiga su propio aprendizaje automático (BYO ML). 

Los modelos de detección de aprendizaje automático se pueden adaptar a entornos individuales y a cambios en el comportamiento del usuario para reducir [falsos positivos](false-positives.md) e identificar amenazas que no se podrían encontrar con un enfoque tradicional. Muchas organizaciones de seguridad son conscientes del valor del aprendizaje automático para la seguridad, aunque solo algunas tienen la suerte de contar con profesionales con experiencia en ambas materias: seguridad y aprendizaje automático. Hemos diseñado la plataforma que se presenta aquí para poder acompañar a los profesionales y organizaciones de seguridad en su recorrido hacia la adopción del aprendizaje automático. Las organizaciones no familiarizadas con el aprendizaje automático, o que no cuentan con la necesaria experiencia, pueden obtener un valor de protección significativo de las funcionalidades de aprendizaje automático integradas de Azure Sentinel.

:::image type="content" source="./media/bring-your-own-ml/machine-learning-framework.png" alt-text="plataforma de aprendizaje automático":::

## <a name="what-is-the-bring-your-own-machine-learning-byo-ml-platform"></a>¿Qué es la plataforma Traiga su propio aprendizaje automático (BYO-ML)?

Para aquellas organizaciones que tienen recursos de aprendizaje automático y desean crear modelos de aprendizaje automático personalizados para sus necesidades empresariales exclusivas, ofrecemos la **plataforma BYO-ML**. La plataforma utiliza el entorno [Azure Databricks](/azure/databricks/scenarios/what-is-azure-databricks)/[Apache Spark](http://spark.apache.org/) y Jupyter Notebook para generar el entorno de aprendizaje automático. Ofrece los siguientes componentes:

- un paquete BYO-ML, que incluye bibliotecas para ayudarle a acceder a los datos y a insertar los resultados en Log Analytics, de forma que pueda integrar los resultados con la detección, la investigación y la búsqueda. 

- Plantillas de algoritmos de aprendizaje automático que puede personalizar para ajustarse a problemas de seguridad específicos de su organización. 

- Cuadernos de ejemplo para entrenar el modelo y programar la puntuación de este. 

Además de todo esto, puede traer sus propios modelos de aprendizaje automático y/o su propio entorno de Spark para integrarlos con Azure Sentinel.

Con la plataforma BYO-ML, puede empezar a crear sus propios modelos de aprendizaje automático: 

- El cuaderno con datos de ejemplo le ayuda a obtener una experiencia práctica completa, sin preocuparse por el control de los datos de producción.

- El paquete integrado con el entorno de Spark reduce los desafíos y dificultades en la administración de la infraestructura.

- Las bibliotecas admiten los movimientos de datos. Los cuadernos de entrenamiento y puntuación muestran la experiencia completa y sirven como plantilla que el usuario puede adaptar a su entorno.

### <a name="use-cases"></a>Casos de uso
 
La plataforma y el paquete BYO-ML reducen significativamente el tiempo y el esfuerzo que necesitará para crear sus propias detecciones de aprendizaje automático y liberar la funcionalidad necesaria para afrontar problemas de seguridad específicos en Azure Sentinel. La plataforma admite los siguientes casos de uso:

**Entrenamiento de un algoritmo de aprendizaje automático para obtener un modelo personalizado:** puede tomar un algoritmo de aprendizaje automático existente (que haya compartido Microsoft o la comunidad de usuarios) y entrenarlo fácilmente con sus propios datos para obtener un modelo de aprendizaje automático personalizado que se adapte mejor a sus datos y entorno.

**Modificación de una plantilla de algoritmo de aprendizaje automático para la obtención de un modelo personalizado:** puede modificar una plantilla de algoritmo de aprendizaje automático (que haya compartido Microsoft o la comunidad de usuarios) y entrenar el algoritmo modificado con sus propios datos para derivar un modelo personalizado que se ajuste a su problema concreto.

**Creación de su propio modelo:** cree su propio modelo desde cero con las utilidades y la plataforma BYO-ML de Azure Sentinel.

**Integración con el entorno de Databricks/Spark:** integre el entorno existente de Databricks/Spark en Azure Sentinel y use las bibliotecas y plantillas de BYO-ML para crear modelos de aprendizaje automático para su situación concreta.

**Importación de su propio modelo de aprendizaje automático:** puede importar sus propios modelos de aprendizaje automático y usar la plataforma y utilidades de BYO-ML para integrarlos con Azure Sentinel.

**Uso compartido de un algoritmo de aprendizaje automático:** comparta un algoritmo de aprendizaje automático para que la comunidad lo adopte y lo adapte.

**Uso del aprendizaje automático para habilitar SecOps:** use su propio modelo de aprendizaje automático personalizado y los resultados para la búsqueda, las detecciones, la investigación y la respuesta.

En este artículo se muestran los componentes de la plataforma BYO-ML y cómo aprovechar la plataforma y el algoritmo de acceso a recursos anómalo para ofrecer una detección personalizada de aprendizaje automático con Azure Sentinel.

## <a name="azure-databricksspark-environment"></a>Entorno de Azure Databricks/Spark

[Apache Spark™](http://spark.apache.org/) ha supuesto un gran avance en la simplificación de los macrodatos al ofrecer una plataforma unificada para la creación de canalizaciones de datos. Azure Databricks ha llevado esto aún más lejos al proporcionar una plataforma en la nube que no necesita administración creada en torno a Spark. Le recomendamos que use Databricks para su plataforma BYO-ML, de modo que pueda centrarse en buscar respuestas que tengan un efecto inmediato en su negocio, en lugar de tener que abordar las canalizaciones de datos y los problemas de la plataforma.
Si ya dispone de Databricks o de cualquier otro entorno de Spark, y prefiere usar la configuración ya existente, el paquete BYO-ML funcionará también correctamente. 

## <a name="byo-ml-package"></a>Paquete BYO-ML

El paquete BYO-ML incluye los procedimientos recomendados y la experiencia en investigación de Microsoft sobre el aprendizaje automático en relación con la seguridad. En este paquete, se proporciona la siguiente lista de utilidades, cuadernos y plantillas de algoritmos para problemas de seguridad.

| Nombre de archivo | Descripción |
| --------- | ----------- |
| azure_sentinel_utilities.whl | Contiene utilidades para la lectura de blobs de Azure y la escritura en Log Analytics. |
| AnomalousRASampleData | Notebook muestra el uso del modelo de acceso a recursos anómalo en Azure Sentinel con datos de ejemplo de entrenamiento y prueba generados. |
| AnomalousRATraining.ipynb | Cuaderno para entrenar el algoritmo, crear los modelos y guardarlos. |
| AnomalousRAScoring.ipynb | Cuaderno para programar el modelo que se va a ejecutar, visualizar el resultado y escribir la puntuación de nuevo en Azure Sentinel. |
|

La primera plantilla de algoritmo de aprendizaje automático que se ofrece es para la [detección del acceso a recursos anómalo](https://github.com/Azure/Azure-Sentinel/tree/master/BYOML). Se basa en un algoritmo de filtrado colaborativo y se entrena con los registros de acceso de los recursos compartidos de archivos de Windows (eventos de seguridad con el identificador de evento 5140). La información clave que necesita para este modelo en el registro es el emparejamiento de los usuarios y los recursos a los que se ha accedido. 

## <a name="example-walkthrough-anomalous-file-share-access-detection"></a>Tutorial de ejemplo: Detección del acceso a recursos compartidos de archivos anómalo 

Ahora que está familiarizado con los componentes clave de la plataforma BYO-ML, este es un ejemplo que muestra el uso de la plataforma y los componentes para proporcionar una detección de aprendizaje automático personalizada.

### <a name="setup-the-databricksspark-environment"></a>Configuración del entorno Databricks/Spark

Deberá configurar su propio entorno de Databricks si no dispone ya de uno. Consulte el documento de [inicio rápido de Databricks](/azure/databricks/scenarios/quickstart-create-databricks-workspace-portal?tabs=azure-portal) para obtener instrucciones.

### <a name="auto-export-instruction"></a>Instrucción de exportación automática

Para crear modelos de Machine Learning personalizados basados en sus propios datos en Azure Sentinel, deberá exportar los datos de Log Analytics a un recurso de Blob Storage o de Event Hubs, de forma que el modelo de aprendizaje automático pueda acceder a él desde Databricks. Aprenda a [ingerir datos en Azure Sentinel](connect-data-sources.md).

En este ejemplo, debe tener los datos de entrenamiento para el registro de acceso a los recursos compartidos de archivos en Azure Blob Storage. El formato de los datos está documentado en el cuaderno y las bibliotecas.

Puede exportar automáticamente los datos desde Log Analytics mediante la [interfaz de la línea de comandos (CLI) de Azure](/cli/azure/monitor/log-analytics). 

Debe tener asignado el rol **Colaborador** en el área de trabajo de Log Analytics, la cuenta de almacenamiento y el recurso de EventHub para poder ejecutar los comandos. 

A continuación se muestra un conjunto de comandos de ejemplo para configurar la exportación automática:

```azurecli

az –version

# Login with Azure CLI
 az login

# List all Log Analytics clusters
 az monitor log-analytics cluster list

# Set to specific subscription
 az account set --subscription "SUBSCRIPTION_NAME"
 
# Export to Storage - all tables
 az monitor log-analytics workspace data-export create --resource-group "RG_NAME" --workspace-name "WS_NAME" -n LAExportCLIStr --destination “DESTINATION_NAME" --enable "true" --tables SecurityEvent
 
# Export to EventHub - all tables
 az monitor log-analytics workspace data-export create --resource-group "RG_NAME" --workspace-name "WS_NAME" -n LAExportCLIEH --destination “DESTINATION_NAME" --enable "true" --tables SecurityEvent Heartbeat"]

# List export settings
az monitor log-analytics workspace data-export list --resource-group "RG_NAME" --workspace-name "WS_NAME"

# Delete export setting
 az monitor log-analytics workspace data-export delete --resource-group "RG_NAME" --workspace-name "WS_NAME" --name "NAME"
```

### <a name="export-custom-data"></a>Exportación de datos personalizados

En el caso de los datos personalizados que no admiten la exportación automática de Log Analytics, puede usar una aplicación lógica u otras soluciones para trasladar los datos. Puede consultar el blog y el script de [Exportación de datos de Log Analytics a Blob Storage](https://techcommunity.microsoft.com/t5/azure-monitor/log-analytics-data-export-preview/ba-p/1783530).

### <a name="correlate-with-data-outside-of-azure-sentinel"></a>Correlación con datos fuera de Azure Sentinel

También puede traer datos desde fuera de Azure Sentinel a Blob Storage o Event Hubs y correlacionarlos con los datos de Azure Sentinel para crear sus modelos de Machine Learning. 
 
### <a name="copy-and-install-the-related-packages"></a>Copia e instalación de los paquetes relacionados

Copie el paquete BYO-ML del repositorio de GitHub para Azure Sentinel mencionado anteriormente en el entorno de Databricks. Después, abra los cuadernos y siga las instrucciones del cuaderno para instalar las bibliotecas necesarias en los clústeres.

### <a name="model-training-and-scoring"></a>Entrenamiento y puntuación del modelo

Siga las instrucciones de los dos cuadernos para cambiar las configuraciones de acuerdo con su propio entorno y recursos, siga los pasos para entrenar y crear el modelo y, finalmente, programe el modelo para puntuar los registros de acceso de los recursos compartidos de archivos entrantes.

### <a name="write-results-to-log-analytics"></a>Escritura de resultados en Log Analytics

Una vez que obtenga la puntuación programada, puede usar el módulo en el cuaderno de puntuación para escribir los resultados de puntuación en el área de trabajo de Log Analytics asociada a la instancia de Azure Sentinel.

### <a name="check-results-in-azure-sentinel"></a>Comprobación de resultados en Azure Sentinel

Para ver los resultados puntuados junto con los detalles de registro relacionados, vuelva al portal de Azure Sentinel. En **Registros** > Registros personalizados, verá los resultados en la tabla **AnomalousResourceAccessResult_CL** (o en una con su propio nombre de tabla personalizado). Puede utilizar estos resultados para mejorar su experiencia de investigación y búsqueda.

:::image type="content" source="./media/bring-your-own-ml/anomalous-resource-access-logs.png" alt-text="registros de acceso a recursos anómalo":::

### <a name="build-custom-analytics-rule-with-ml-results"></a>Creación de una regla de análisis personalizado con resultados de aprendizaje automático

Una vez que haya confirmado que los resultados de aprendizaje automático están en la tabla de registros personalizados y esté satisfecho con la fidelidad de las puntuaciones, puede crear una detección basada en los resultados. Vaya a **Analytics** desde el portal de Azure Sentinel y [cree una nueva regla de detección](tutorial-detect-threats-custom.md). En el siguiente ejemplo se muestra la consulta utilizada para crear la detección.

:::image type="content" source="./media/bring-your-own-ml/create-byo-ml-analytics-rule.png" alt-text="creación de una regla de análisis personalizada para las detecciones de B Y O M L":::

### <a name="view-and-respond-to-incidents"></a>Visualización de incidentes y respuesta a estos
Una vez que haya configurado la regla de análisis según los resultados del aprendizaje automático, si hay resultados por encima del umbral establecido en la consulta, se generará un incidente y se mostrará en la página **Incidentes** en Azure Sentinel. 

## <a name="next-steps"></a>Pasos siguientes

En este documento, ha aprendido a usar la plataforma BYO-ML de Azure Sentinel para crear o importar sus propios algoritmos de aprendizaje automático para analizar datos y detectar amenazas.

- Consulte publicaciones sobre aprendizaje automático y muchos otros temas pertinentes en el [blog de Azure Sentinel](https://aka.ms/azuresentinelblog).
