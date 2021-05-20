---
title: archivo de inclusión
description: archivo de inclusión
services: data-factory
author: chez-charlie
ms.service: data-factory
ms.topic: include
ms.date: 11/16/2020
ms.author: chez
ms.custom: include file
ms.openlocfilehash: 5efc74ec150dcd18181404d93dc2142552a2c792
ms.sourcegitcommit: 2cb7772f60599e065fff13fdecd795cce6500630
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108807449"
---
Azure Data Factory es un servicio multiinquilino que tiene los siguientes límites predeterminados para asegurarse de que las suscripciones de cliente están protegidas frente a las cargas de trabajo de los demás. Para elevar los límites al máximo de la suscripción, póngase en contacto con el servicio de soporte técnico.

### <a name="version-2"></a>versión 2

| Recurso | Límite predeterminado | Límite máximo |
| -------- | ------------- | ------------- |
| Factorías de datos por cada grupo de recursos | 800 | 800 |
| Número total de entidades como canalizaciones, conjuntos de datos, desencadenadores, servicios vinculados, puntos de conexión privados y entornos de ejecución de integración de una factoría de datos | 5\.000 | [Póngase en contacto con el servicio de soporte técnico](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/). |
| Núcleos de CPU totales para Azure-SSIS Integration Runtime en una suscripción | 256 | [Póngase en contacto con el servicio de soporte técnico](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/). |
| Ejecuciones de canalizaciones simultáneas por factoría de datos compartida entre todas las canalizaciones de la factoría | 10 000  | 10 000 |
| Ejecuciones de actividades externas simultáneas por suscripción por [región de Azure Integration Runtime](../articles/data-factory/concepts-integration-runtime.md#azure-ir-location)<br><small>Las actividades externas se administran en el entorno de ejecución de integración, pero se ejecutan en servicios vinculados, incluidos Databricks, procedimiento almacenado, HDInsights, Web, etc. Este límite no se aplica al entorno de ejecución de integración autohospedado.</small> | 3,000 | 3,000 |
| Ejecuciones de actividades de canalización simultáneas por suscripción por [región de Azure Integration Runtime](../articles/data-factory/concepts-integration-runtime.md#azure-ir-location) <br><small>Las actividades de canalización se ejecutan en el entorno de ejecución de integración, lo que incluye Lookup, GetMetadata y Delete. Este límite no se aplica al entorno de ejecución de integración autohospedado.</small> | 1,000 | 1,000                                                        |
| Operaciones de creación simultáneas por suscripción por [región de Azure Integration Runtime](../articles/data-factory/concepts-integration-runtime.md#azure-ir-location)<br><small>Se incluye la prueba de la conexión, el examen de las listas de carpetas y tablas y la vista previa de los datos. Este límite no se aplica al entorno de ejecución de integración autohospedado.</small> | 200 | 200                                                          |
| Uso de unidades de integración de datos simultáneas<sup>1</sup> por suscripción por [región de Azure Integration Runtime](../articles/data-factory/concepts-integration-runtime.md#integration-runtime-location)| Grupo de regiones 1<sup>2</sup>: 6,000<br>Grupo de regiones 2<sup>2</sup>: 3000<br>Grupo de regiones 3<sup>2</sup>: 1500 | Grupo de regiones 1<sup>2</sup>: 6,000<br/>Grupo de regiones 2<sup>2</sup>: 3000<br/>Grupo de regiones 3<sup>2</sup>: 1500 |
| Número máximo de actividades por canalización, lo que incluye actividades internas de contenedores | 40 | 40 |
| Número máximo de entornos vinculados de ejecución de integración que pueden crearse en un único entorno de ejecución de integración autohospedado | 100 | [Póngase en contacto con el servicio de soporte técnico](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/). |
| Parámetros máximos por canalización | 50 | 50 |
| Elementos ForEach | 100 000 | 100 000 |
| Paralelismo de ForEach | 20 | 50 |
| Número máximo de ejecuciones en cola por canalización | 100 | 100 |
| Caracteres por expresión | 8192 | 8192 |
| Intervalo mínimo del desencadenador de ventana de saltos de tamaño constante | 15 minutos | 15 minutos |
| Número máximo de tiempos de expiración de ejecuciones de actividad de canalización | 7 días | 7 días |
| Bytes por objeto para objetos de canalización<sup>3</sup> | 200 KB | 200 KB |
| Bytes por objeto para objetos de conjunto de datos y de servicio vinculados<sup>3</sup> | 100 KB | 2000 KB |
| Bytes por carga para cada ejecución de actividad<sup>4</sup> | 896 KB | 896 KB |
| Unidades de integración de datos<sup>1</sup> por ejecución de la actividad de copia | 256 | 256 |
| Llamadas API de escritura | 1200/h | 1200/h<br/><br/> Este límite lo impone Azure Resource Manager, no Azure Data Factory. |
| Llamadas API de lectura | 12 500/h | 12 500/h<br/><br/> Este límite lo impone Azure Resource Manager, no Azure Data Factory. |
| Supervisión de consultas por minuto | 1,000 | 1,000 |
| Tiempo máximo de la sesión de depuración de flujo de datos | 8 h | 8 h |
| Número simultáneo de flujos de datos por entorno de ejecución de integración | 50 | [Póngase en contacto con el servicio de soporte técnico](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/). |
| Número simultáneo de sesiones de depuración de flujo de datos por usuario y fábrica | 3 | 3 |
| Límite de TTL de Azure IR de flujo de datos | 4 h |  4 h |
| Límite de tamaño de la entidad de metadatos en una fábrica | 2 GB | [Póngase en contacto con el servicio de soporte técnico](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/). |

<sup>1</sup> La unidad de integración de datos (DIU) se usa en una operación de copia de nube a nube, obtenga más información en [Unidades de integración de datos (versión 2)](../articles/data-factory/copy-activity-performance.md#data-integration-units). Para obtener información sobre facturación, vea [Precios de Azure Data Factory](https://azure.microsoft.com/pricing/details/data-factory/).

<sup>2</sup> [Azure Integration Runtime](../articles/data-factory/concepts-integration-runtime.md#azure-integration-runtime) está [ disponible globalmente](https://azure.microsoft.com/global-infrastructure/services/) para garantizar el cumplimiento de los datos, la eficacia y los costos de salida de red reducidos. 

| Grupo de regiones | Regions |
| -------- | ------ |
| Grupo de regiones 1 | Centro de EE. UU., Este de EE. UU., Este de EE. UU. 2, Norte de Europa, Oeste de Europa, Oeste de EE. UU., Oeste de EE. UU. 2 |
| Grupo de regiones 2 | Este de Australia, Sudeste de Australia, Sur de Brasil, Centro de la India, Este de Japón, Centro-norte de EE. UU., Centro-sur de EE. UU., Sudeste de Asia, Centro-oeste de EE. UU. |
| Grupo de regiones 3 | Otras regiones |

<sup>3</sup> Los objetos de canalización, de conjunto de datos y de servicio vinculado representan una agrupación lógica de la carga de trabajo. Los límites de estos objetos no se corresponden con la cantidad de datos que se pueden mover y procesar con Azure Data Factory. Data Factory está diseñado para escalarse a fin de manejar petabytes de datos.

<sup>4</sup> La carga de cada ejecución de actividad incluye la configuración de la actividad, las configuraciones de los conjuntos de datos asociados y los servicios vinculados (si los hay), y una pequeña parte de las propiedades del sistema que se generan por tipo de actividad. Los límites de tamaño de esta carga no se corresponden con la cantidad de datos que se pueden mover y procesar con Azure Data Factory. Obtenga más información sobre los [síntomas y recomendaciones](../articles/data-factory/data-factory-troubleshoot-guide.md#payload-is-too-large) si se alcanza este límite.

### <a name="version-1"></a>versión 1

| **Recurso** | **Límite predeterminado** | **Límite máximo** |
| --- | --- | --- |
| Canalizaciones dentro de una factoría de datos |2,500 |[Póngase en contacto con el servicio de soporte técnico](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/). |
| Conjuntos de datos dentro de una factoría de datos |5\.000 |[Póngase en contacto con el servicio de soporte técnico](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/). |
| Fragmentos simultáneos por conjunto de datos |10 |10 |
| Bytes por objeto para objetos de canalización<sup>1</sup> |200 KB |200 KB |
| Bytes por objeto para objetos de conjunto de datos y de servicio vinculado<sup>1</sup> |100 KB |2000 KB |
| Núcleos de clúster a petición de Azure HDInsight con una suscripción<sup>2</sup> |60 |[Póngase en contacto con el servicio de soporte técnico](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/). |
| Unidades de movimiento de datos en la nube por ejecución de actividad de copia<sup>3</sup> |32 |32 |
| Número de reintentos de ejecuciones de la actividad Canalización |1,000 |MaxInt (32 bits) |

<sup>1</sup> Los objetos de canalización, de conjunto de datos y de servicio vinculado representan una agrupación lógica de la carga de trabajo. Los límites de estos objetos no se corresponden con la cantidad de datos que se pueden mover y procesar con Azure Data Factory. Data Factory está diseñado para escalarse a fin de manejar petabytes de datos.

<sup>2</sup>Los núcleos de HDInsight a petición se asignan fuera de la suscripción que contiene Data Factory. Como resultado, el límite anterior es el límite de núcleos aplicado por Data Factory para los núcleos de HDInsight a petición. Es diferente al límite de núcleos asociado a la suscripción de Azure.

<sup>3</sup> La unidad de movimiento de datos en la nube (DMU) para la versión 1 se usa en una operación de copia de nube a nube, obtenga más información en [Unidades de movimiento de datos de nube (versión 1)](../articles/data-factory/v1/data-factory-copy-activity-performance.md#cloud-data-movement-units). Para obtener información sobre facturación, vea [Precios de Azure Data Factory](https://azure.microsoft.com/pricing/details/data-factory/).

| **Recurso** | **Límite inferior predeterminado** | **Límite mínimo** |
| --- | --- | --- |
| Intervalo de programación |15 minutos |15 minutos |
| Intervalo entre reintentos |1 segundo |1 segundo |
| Valor de tiempo de espera de reintento |1 segundo |1 segundo |

#### <a name="web-service-call-limits"></a>Límites de llamadas de servicio web
Azure Resource Manager tiene límites para las llamadas de API. Se pueden realizar llamadas API a un ritmo que esté dentro de los [límites de la API del Administrador de recursos de Azure](../articles/azure-resource-manager/management/azure-subscription-service-limits.md#resource-group-limits).
