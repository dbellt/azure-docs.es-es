---
title: ¿Qué son los puntos de conexión? (Versión preliminar)
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo los puntos de conexión de Azure Machine Learning (versión preliminar) simplifican las implementaciones de aprendizaje automático.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: seramasu
author: rsethur
ms.reviewer: laobri
ms.date: 05/25/2021
ms.openlocfilehash: 472af8cdb377ea5eb1be15ebcfadabb38311545a
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110382913"
---
# <a name="what-are-azure-machine-learning-endpoints-preview"></a>¿Qué son los puntos de conexión de Azure Machine Learning (versión preliminar)?

[!INCLUDE [preview disclaimer](../../includes/machine-learning-preview-generic-disclaimer.md)]

Use los puntos de conexión de Azure Machine Learning (versión preliminar) para simplificar las implementaciones de modelos en implementaciones en tiempo real y de inferencias por lotes. Los puntos de conexión proporcionan una interfaz unificada para invocar y administrar implementaciones de modelos entre tipos de proceso.

En este artículo, aprenderá lo siguiente:
> [!div class="checklist"]
> * Puntos de conexión
> * Implementaciones
> * Puntos de conexión en línea administrados
> * Puntos de conexión en línea de AKS
> * Puntos de conexión de inferencias por lotes

## <a name="what-are-endpoints-and-deployments-preview"></a>¿Qué son los puntos de conexión y las implementaciones (versión preliminar)?

Después de entrenar un modelo de Machine Learning, debe implementarlo para que otros usuarios puedan usarlo para realizar inferencias. En Azure Machine Learning, puede usar **puntos de conexión** (versión preliminar) e **implementaciones** (versión preliminar) para hacerlo.

:::image type="content" source="media/concept-endpoints/endpoint-concept.png" alt-text="Diagrama que muestra un punto de conexión que divide el tráfico hacia dos implementaciones":::

Un **punto de conexión** es un punto de conexión HTTPS al que los clientes pueden llamar para recibir la salida de inferencia (puntuación) de un modelo entrenado. Proporciona: 
- Autenticación mediante autenticación basada en "claves y tokens" 
- Terminación de SSL 
- Asignación de tráfico entre implementaciones 
- Un URI de puntuación estable (endpoint-name.region.inference.ml.azure.com)


Una **implementación** es un conjunto de recursos de proceso que hospedan el modelo que realiza la inferencia real. Contiene: 
- Detalles del modelo (código, modelo, entorno) 
- Configuración de recursos y escalado de proceso 
- Configuración avanzada (como la configuración de solicitud y sondeo)

Un único punto de conexión puede contener varias implementaciones. Los puntos de conexión y las implementaciones son recursos de Azure Resource Manager independientes que aparecerán en Azure Portal.

Azure Machine Learning usa el concepto de puntos de conexión y de implementaciones para implementar diferentes tipos de puntos de conexión: [**puntos de conexión en línea**](#what-are-online-endpoints-preview) y [**puntos de conexión por lotes**](#what-are-batch-endpoints-preview).

### <a name="multiple-developer-interfaces"></a>Varias interfaces de desarrollador

Cree y administre puntos de conexión por lotes y en línea con varias herramientas de desarrollo:
- CLI
- Azure Resource Manager y API REST
- Portal web de Azure Machine Learning Studio
- Azure Portal (TI o administrador)
- Compatibilidad con canalizaciones de MLOps de CI/CD mediante la interfaz de la CLI y las interfaces de REST y Azure Resource Manager

## <a name="what-are-online-endpoints-preview"></a>¿Qué son los puntos de conexión en línea (versión preliminar)?

Los **puntos de conexión en línea** (versión preliminar) son puntos de conexión que se usan para las inferencias en línea (en tiempo real). En comparación con los **puntos de conexión por lotes**, los **puntos de conexión en línea** contienen **implementaciones** que están listas para recibir datos de los clientes y que pueden enviar respuestas en tiempo real.

### <a name="online-endpoints-requirements"></a>Requisitos de los puntos de conexión en línea

Para crear un punto de conexión en línea, debe especificar lo siguiente:
- Archivos de modelo (o especificar un modelo registrado en el área de trabajo) 
- Script de puntuación: código necesario para realizar la puntuación o inferencia
- Entorno: una imagen de Docker con dependencias de Conda o un archivo dockerfile 
- Instancia de proceso y configuración de escalado 

Aprenda a implementar puntos de conexión en línea desde la [CLI](how-to-deploy-managed-online-endpoints.md) y el [portal web de Studio](how-to-use-managed-online-endpoint-studio.md).

### <a name="test-and-deploy-locally-for-faster-debugging"></a>Prueba e implementación local para una depuración más rápida

Implemente localmente para probar los puntos de conexión sin implementarlos en la nube. Azure Machine Learning crea una imagen de Docker local que imita la imagen de Azure Machine Learning. Azure Machine Learning creará y ejecutará las implementaciones de forma local y almacenará en caché la imagen para lograr unas iteraciones rápidas.

### <a name="native-bluegreen-deployment"></a>Implementación nativa azul-verde 

Recuerde que un único punto de conexión puede tener varias implementaciones. El punto de conexión en línea puede realizar el equilibrio de carga para asignar cualquier porcentaje de tráfico a cada implementación.

La asignación de tráfico se puede usar para realizar implementaciones azul/verde de lanzamiento seguras equilibrando las solicitudes entre las diferentes instancias.

:::image type="content" source="media/concept-endpoints/traffic-allocation.png" alt-text="Captura de pantalla que muestra la interfaz con el control deslizante para establecer la asignación de tráfico entre las implementaciones":::

Aprenda a [implementar de forma segura en puntos de conexión en línea](how-to-safely-rollout-managed-endpoints.md).

### <a name="application-insights-integration"></a>Integración de Application Insights

Todos los puntos de conexión en línea se integran con Application Insights para supervisar los Acuerdos de Nivel de Servicio y diagnosticar problemas. 

No obstante, los [puntos de conexión en línea administrados](#managed-online-endpoints-vs-aks-online-endpoints-preview) también incluyen integración lista para su uso con los registros y métricas de Azure.

### <a name="security"></a>Seguridad

- Autenticación: Tokens de clave y de Azure Machine Learning
- Identidad administrada: asignada por el usuario y asignada por el sistema (solo puntos de conexión en línea administrados)
- SSL de forma predeterminada para la invocación de puntos de conexión


## <a name="managed-online-endpoints-vs-aks-online-endpoints-preview"></a>Puntos de conexión en línea administrados frente a puntos de conexión en línea de AKS (versión preliminar)

Hay dos tipos de puntos de conexión en línea: **puntos de conexión en línea administrados** (versión preliminar) y **puntos de conexión en línea de AKS** (versión preliminar). En la tabla siguiente se resaltan algunas de sus principales diferencias.

|  | Puntos de conexión en línea administrados | Puntos de conexión en línea de AKS |
|-|-|-|
| **Usuarios recomendados** | Usuarios que desean una implementación de modelo administrada y una experiencia de MLOps mejorada | Usuarios que prefieren Azure Kubernetes Service (AKS) y pueden administrar los requisitos de infraestructura |
| **Administración de la infraestructura** | Aprovisionamiento de proceso administrado, escalado, actualizaciones de imágenes del sistema operativo del host y reforzamiento de la seguridad | Responsabilidad del usuario |
| **Compute type (Tipo de proceso)** | Administrado (AmlCompute) | AKS |
| **Supervisión inmediata** | [Supervisión de Azure](how-to-monitor-online-endpoints.md) <br> (incluye métricas clave como la latencia y el rendimiento) | No compatible |
| **Registro inmediato** | [Registros y análisis de registros de Azure en el nivel de punto de conexión](how-to-deploy-managed-online-endpoints.md#optional-integrate-with-log-analytics) | Configuración manual en el nivel del clúster |
| **Application Insights** | Compatible | Compatible |
| **Identidad administrada** | [Compatible](tutorial-deploy-managed-endpoints-using-system-managed-identity.md) | No compatible |
| **Virtual Network (VNET)** | No admitido (versión preliminar pública) | Configuración manual en el nivel del clúster |
| **Visualización de costos** | [Nivel de punto de conexión y de implementación](how-to-view-online-endpoints-costs.md) | Nivel de clúster |

### <a name="managed-online-endpoints"></a>Puntos de conexión en línea administrados

Los puntos de conexión en línea administrados pueden ayudar a simplificar el proceso de implementación. Los puntos de conexión en línea administrados proporcionan las siguientes ventajas con respecto a los puntos de conexión en línea de AKS:

- Infraestructura administrada
    - Aprovisiona automáticamente el proceso y hospeda el modelo (el usuario solo tiene que especificar el tipo de máquina virtual y la configuración del escalado). 
    - Realiza automáticamente actualizaciones y revisiones en la imagen del sistema operativo del host subyacente.
    - Recuperación automática de nodos en caso de un error del sistema

:::image type="content" source="media/concept-endpoints/log-analytics-and-azure-monitor.png" alt-text="Captura de pantalla que muestra un gráfico de Azure Monitor con la latencia del punto de conexión":::

- Supervisión y registros
    - Supervise la disponibilidad, el rendimiento y el Acuerdo de Nivel de Servicio del modelo mediante la [integración nativa con Azure Monitor](how-to-monitor-online-endpoints.md).
    - Depure las implementaciones mediante los registros y la integración nativa con Azure Log Analytics.

- Identidad administrada
    -  Uso de [identidades administradas para acceder a recursos protegidos desde el script de puntuación](tutorial-deploy-managed-endpoints-using-system-managed-identity.md)

:::image type="content" source="media/concept-endpoints/endpoint-deployment-costs.png" alt-text="Captura de pantalla del gráfico de costos de un punto de conexión y de la implementación":::

- Visualización de costos 
    - Los puntos de conexión en línea administrados le permiten [supervisar el costo en los niveles de punto de conexión y de implementación](how-to-view-online-endpoints-costs.md).

Para ver un tutorial paso a paso, consulte [Implementación de puntos de conexión en línea administrados](how-to-deploy-managed-online-endpoints.md).

## <a name="what-are-batch-endpoints-preview"></a>¿Qué son los puntos de conexión por lotes (versión preliminar)?

Los **puntos de conexión por lotes** (versión preliminar) son puntos de conexión que se usan para realizar la inferencia por lotes en grandes volúmenes de datos durante un período de tiempo.  Los **puntos de conexión por lotes** reciben punteros hacia los datos y ejecutan trabajos de forma asincrónica para procesar los datos en paralelo en los clústeres de proceso. Los puntos de conexión por lotes almacenan salidas en un almacén de datos para su posterior análisis.

Aprenda a [implementar y usar puntos de conexión por lotes con la CLI de Azure](how-to-use-batch-endpoint.md).

### <a name="no-code-mlflow-model-deployments"></a>Implementaciones de modelos de MLflow sin código

Use la experiencia de creación de puntos de conexión por lotes sin código para los [modelos de MLflow](how-to-use-mlflow.md) para crear automáticamente scripts de puntuación y entornos de ejecución.  

Para los puntos de conexión por lotes que usan modelos de MLflow, debe especificar lo siguiente:
- Archivos de modelo (o especificar un modelo registrado en el área de trabajo)
- Destino de proceso

Sin embargo, si **no** va a implementar un modelo de MLflow, debe proporcionar requisitos adicionales:
- Script de puntuación: código necesario para realizar la puntuación o inferencia
- Entorno: una imagen de Docker con dependencias de Conda


### <a name="managed-cost-with-autoscaling-compute"></a>Costo administrado con un proceso de escalado automático

La invocación de un punto de conexión por lotes desencadena un trabajo asincrónico de inferencia por lotes. Los recursos de proceso se aprovisionan automáticamente cuando se inicia el trabajo y se desasignan automáticamente cuando se completa. Por lo tanto, solo paga por el proceso cuando lo usa.

Puede [invalidar la configuración de recursos de proceso](how-to-use-batch-endpoint.md#overwrite-settings) (por ejemplo, el recuento de instancias) y la configuración avanzada (como el tamaño de lote mínimo, el umbral de error, etc) para cada trabajo de inferencia por lotes individual para acelerar la ejecución y reducir el costo.

### <a name="flexible-data-sources-and-storage"></a>Almacenamiento y orígenes de datos flexibles

Puede usar las siguientes opciones para los datos de entrada al invocar un punto de conexión por lotes:

- Conjuntos de datos registrados de Azure Machine Learning: para más información, consulte [Creación de conjuntos de datos de Azure Machine Learning](how-to-train-with-datasets.md).
- Datos en la nube: URI de datos públicos o ruta de acceso a datos del almacén de datos. Para más información, consulte [Conexión a los datos con Azure Machine Learning Studio](how-to-connect-data-ui.md).
- Datos almacenados localmente

Especifique la ubicación de salida del almacenamiento en cualquier almacén de datos y ruta de acceso. De forma predeterminada, los puntos de conexión por lotes almacenan su salida en el almacén de blobs predeterminado del área de trabajo, organizados según el nombre del trabajo (un GUID generado por el sistema).

### <a name="security"></a>Seguridad

- Autenticación previa: tokens de Azure Active Directory
- SSL de forma predeterminada para la invocación de puntos de conexión

## <a name="next-steps"></a>Pasos siguientes

- [Implementación de puntos de conexión en línea administrados con la CLI de Azure](how-to-deploy-managed-online-endpoints.md)
- [Implementación de puntos de conexión por lotes con la CLI de Azure](how-to-use-batch-endpoint.md)
- [Uso de puntos de conexión en línea administrados con Studio](how-to-use-managed-online-endpoint-studio.md)
- [Supervisión de los puntos de conexión en línea administrados](how-to-monitor-online-endpoints.md)
- [Visualización de los costos de los puntos de conexión en línea](how-to-view-online-endpoints-costs.md)
