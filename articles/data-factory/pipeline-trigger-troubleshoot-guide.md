---
title: Solución de problemas relacionados con orquestaciones y desencadenadores de canalizaciones en Azure Data Factory
description: Use distintos métodos para solucionar problemas de desencadenadores de canalizaciones en Azure Data Factory.
author: ssabat
ms.service: data-factory
ms.date: 04/01/2021
ms.topic: troubleshooting
ms.author: susabat
ms.reviewer: susabat
ms.openlocfilehash: d9827eab8c9d6187c78a979591f2c7ee0cad99e7
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108741894"
---
# <a name="troubleshoot-pipeline-orchestration-and-triggers-in-azure-data-factory"></a>Solución de problemas relacionados con orquestaciones y desencadenadores de canalizaciones en Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Una ejecución de canalización en Azure Data Factory define una instancia de dicha ejecución. Por ejemplo, supongamos que tiene una canalización que se ejecuta a las 8:00 a. m., 9:00 a. m. y 10:00 a. m. En este caso, hay tres ejecuciones de canalización independientes. Cada canalización ejecutar tiene un identificador de canalización único. Un id. de ejecución es un identificador único global (GUID) que define esa ejecución de canalización concreta.

Normalmente las instancias de ejecuciones de canalización se crean al pasar argumentos a parámetros que se definen en las canalizaciones. Puede ejecutar una canalización manualmente o mediante un desencadenador. Consulte [Ejecución y desencadenadores de canalización en Azure Data Factory](concepts-pipeline-execution-triggers.md) para más información.

## <a name="common-issues-causes-and-solutions"></a>Problemas comunes, causas y soluciones

### <a name="an-azure-functions-app-pipeline-throws-an-error-with-private-endpoint-connectivity"></a>La canalización de una aplicación de Azure Functions produce un error con la conectividad de puntos de conexión privados
 
Data Factory y una aplicación de funciones se ejecutan en un punto de conexión privado en Azure. Está intentando ejecutar una canalización que interactúa con la aplicación de funciones. Ha probado tres métodos diferentes, pero uno devuelve el error "solicitud incorrecta" y los otros dos métodos devuelven "error 103 prohibido".

**Causa**

Data Factory no admite actualmente un conector de punto de conexión privado para las aplicaciones de funciones. Azure Functions rechaza las llamadas porque está configurado para permitir solo las conexiones de un vínculo privado.

**Resolución**

Cree un punto de conexión de **PrivateLinkService** y especifique el DNS de la aplicación de funciones.

### <a name="a-pipeline-run-is-canceled-but-the-monitor-still-shows-progress-status"></a>Se cancela una ejecución de la canalización, pero el monitor sigue mostrando el mantenimiento en curso

**Causa**

Cuando se cancela una ejecución de canalización, la supervisión de la canalización sigue mostrando el estado En curso. Esto sucede debido a un problema de caché del explorador web. También es posible que no tenga los filtros de supervisión correctos.

**Resolución**

Actualice el explorador y aplique los filtros de supervisión adecuados.
 
### <a name="you-see-a-delimitedtextmorecolumnsthandefined-error-when-copying-a-pipeline"></a>Aparece un error "DelimitedTextMoreColumnsThanDefined" al copiar una canalización
 
 **Causa**
 
Si una carpeta que está copiando contiene archivos con esquemas diferentes, como un número variable de columnas, delimitadores diferentes, valores de carácter de comillas o algún problema de datos, la canalización de Data Factory podría devolver este error:

`
Operation on target Copy_sks  failed: Failure happened on 'Sink' side.
ErrorCode=DelimitedTextMoreColumnsThanDefined,
'Type=Microsoft.DataTransfer.Common.Shared.HybridDeliveryException,
Message=Error found when processing 'Csv/Tsv Format Text' source '0_2020_11_09_11_43_32.avro' with row number 53: found more columns than expected column count 27.,
Source=Microsoft.DataTransfer.Common,'
`

**Resolución**

Seleccione la opción **Binary Copy** (Copia binaria) al crear la actividad de copia. De esta manera, para realizar copias masivas o migrar los datos de un lago de datos a otro, Data Factory no abrirá los archivos para leer el esquema. En su lugar, Data Factory tratará cada archivo como un archivo binario y cópielo en la otra ubicación.

### <a name="a-pipeline-run-fails-when-you-reach-the-capacity-limit-of-the-integration-runtime-for-data-flow"></a>Se produce un error de ejecución de canalización cuando se alcanza el límite de capacidad del entorno de ejecución de integración del flujo de datos

**Problema**

Mensaje de error:

`
Type=Microsoft.DataTransfer.Execution.Core.ExecutionException,Message=There are substantial concurrent MappingDataflow executions which is causing failures due to throttling under Integration Runtime 'AutoResolveIntegrationRuntime'.
`

**Causa**

Ha alcanzado el límite de capacidad del entorno de ejecución de integración. Podría estar ejecutando una gran cantidad de flujos de datos mediante el mismo entorno de ejecución de integración de manera simultánea. Para más información, consulte [Límites, cuotas y restricciones de suscripción y servicios de Azure](../azure-resource-manager/management/azure-subscription-service-limits.md#version-2)).

**Resolución**
 
- Ejecute las canalizaciones en distintos momentos del desencadenador.
- Cree un nuevo entorno de ejecución de integración y divida sus canalizaciones entre varios entornos de ejecución de integración.

### <a name="a-pipeline-run-error-while-invoking-rest-api-in-a-web-activity"></a>Error de ejecución de canalización al invocar la API REST en una actividad web

**Problema**

Mensaje de error:

`
Operation on target Cancel failed: {“error”:{“code”:”AuthorizationFailed”,”message”:”The client ‘<client>’ with object id ‘<object>’ does not have authorization to perform action ‘Microsoft.DataFactory/factories/pipelineruns/cancel/action’ over scope ‘/subscriptions/<subscription>/resourceGroups/<resource group>/providers/Microsoft.DataFactory/factories/<data factory name>/pipelineruns/<pipeline run id>’ or the scope is invalid. If access was recently granted, please refresh your credentials.”}}
`

**Causa**

Las canalizaciones pueden utilizar la actividad web para llamar a los métodos de la API REST de ADF si y solo si se asigna el rol de colaborador al miembro de Azure Data Factory. Primero debe configurar la incorporación de la identidad administrada de Azure Data Factory al rol de seguridad de colaborador. 

**Resolución**

Antes de usar la API REST de Azure Data Factory en la pestaña Configuración de una actividad web, se debe configurar la seguridad. Las canalizaciones de Azure Data Factory pueden utilizar la actividad web para llamar a los métodos de la API REST de ADF si y solo si se asigna el rol de *colaborador* a la identidad administrada de Azure Data Factory. Comience abriendo Azure Portal y haciendo clic en el vínculo **Todos los recursos** en el menú de la izquierda. Seleccione **Azure Data Factory** para agregar la identidad administrada de ADF con el rol de colaborador; para ello, haga clic en el botón **Agregar** en el cuadro *Agregar una asignación de roles*.


### <a name="how-to-check-and-branch-on-activity-level-success-and-failure-in-pipelines"></a>Comprobación y ramificación en el éxito y el error del nivel de actividad en las canalizaciones

**Causa**

La orquestación de Azure Data Factory permite la lógica condicional y permite al usuario tomar diferentes rutas de acceso en función del resultado de una actividad anterior. Permite cuatro rutas de acceso condicionales: **Upon Success** (ruta predeterminada) (en caso de ejecución correcta), **Upon Failure** (en caso de error), **Upon Completion** (en caso de finalización) y **Upon Skip** (en caso de omisión). 

Azure Data Factory evalúa el resultado de todas las actividades de nivel de hoja. Los resultados de la canalización son correctos solo si todas las actividades del nivel de hoja se realizan correctamente. Si se omitió una actividad del nivel hoja, se evalúa su actividad primaria en su lugar. 

**Resolución**

* Implemente comprobaciones en el nivel de actividad con las indicaciones de [Control de errores de canalización](https://techcommunity.microsoft.com/t5/azure-data-factory/understanding-pipeline-failures-and-error-handling/ba-p/1630459).
* Use Azure Logic Apps para supervisar las canalizaciones a intervalos regulares con las indicaciones de [Consulta por factoría](/rest/api/datafactory/pipelineruns/querybyfactory).
* [Supervisión visual de la canalización](./monitor-visually.md)

### <a name="how-to-monitor-pipeline-failures-in-regular-intervals"></a>Supervisión de errores de canalización en intervalos regulares

**Causa**

Quizá necesite supervisar las canalizaciones de Data Factory con errores a intervalos, por ejemplo, de 5 minutos. Puede consultar y filtrar las ejecuciones de canalización de una factoría de datos con el punto de conexión. 

**Resolución**
* Puede configurar una aplicación lógica de Azure para consultar todas las canalizaciones con error cada 5 minutos, como se describe en [Consulta por factoría](/rest/api/datafactory/pipelineruns/querybyfactory). A continuación, puede notificar los incidentes en nuestro sistema de incidencias.
* [Supervisión visual de la canalización](./monitor-visually.md)

### <a name="degree-of-parallelism--increase-does-not-result-in-higher-throughput"></a>El aumento del grado de paralelismo no deriva en un rendimiento mayor.

**Causa** 

El grado de paralelismo de *ForEach* es realmente el grado máximo de paralelismo. No se puede garantizar que se produzca un número específico de ejecuciones al mismo tiempo, pero este parámetro garantizará que nunca se supere el valor establecido. Debe considerar esto como un límite que debe utilizar para controlar el acceso simultáneo a sus orígenes y receptores.

Hechos conocidos sobre *ForEach*
 * ForEach tiene una propiedad denominada número de lote(n), donde el valor predeterminado es 20 y el máximo es 50.
 * El número de lote, n, se usa para construir n colas. 
 * Cada cola se ejecuta secuencialmente, pero puede tener varias colas ejecutándose en paralelo.
 * Las colas se crean previamente. Esto significa que no hay ningún reequilibrio de las colas durante el tiempo de ejecución.
 * En cualquier momento, hay al menos un elemento que se está procesando en cada cola. Esto significa que, como máximo, se procesan n elementos en un momento dado.
 * El tiempo total de procesamiento de ForEach es igual al tiempo de procesamiento de la cola más larga. Esto significa que la actividad de ForEach depende de cómo se construyen las colas.
 
**Resolución**

 * No debe utilizar la actividad *SetVariable* dentro de *For Each* que se ejecute en paralelo.
 * Teniendo en cuenta la forma en que se crean las colas, el cliente puede mejorar el rendimiento de foreach estableciendo varios *foreach*, donde cada valor de *foreach* tenga elementos con un tiempo de procesamiento similar. 
 * Esto garantizará que las ejecuciones largas se procesen en paralelo y no de forma secuencial.

 ### <a name="pipeline-status-is-queued-or-stuck-for-a-long-time"></a>El estado de la canalización está en cola o bloqueado durante mucho tiempo
 
 **Causa**
 
 Esto puede ocurrir por diversos motivos: por alcanzar los límites de simultaneidad, debido a interrupciones del servicio, por errores de red, etc.
 
 **Resolución**
 
* Límite de simultaneidad: si la canalización tiene una directiva de simultaneidad, compruebe que no haya ninguna ejecución de canalización anterior en curso. La simultaneidad máxima de la canalización permitida en Azure Data Factory es de 10 canalizaciones. 
* Límites de supervisión: vaya al lienzo de creación de ADF, seleccione la canalización y determine si tiene una propiedad de simultaneidad asignada. Si es así, vaya a la vista Supervisión y asegúrese de que no haya nada en curso en los últimos 45 días. Si hay algo en curso, puede cancelarlo y debería iniciarse la nueva ejecución de canalización.
* Problemas transitorios: es posible que la ejecución se haya visto afectada por un problema transitorio de la red, errores de credenciales, interrupciones de servicios, etc.  Si esto ocurre, Azure Data Factory cuenta con un proceso de recuperación interno que supervisa todas las ejecuciones y las inicia en caso de que se produzca algún problema. Este proceso se produce cada hora, así que si la ejecución se bloquea durante más de una hora, deberá crear un caso de soporte técnico.
 
### <a name="longer-start-up-times-for-activities-in-adf-copy-and-data-flow"></a>Tiempos de inicio más largos para actividades en la copia de ADF y en Data Flow

**Causa**

Esto puede ocurrir si no ha implementado la característica de período de vida de Data Flow o si no ha optimizado SHIR.

**Resolución**

* Si cada actividad de copia tarda hasta dos minutos en iniciarse y el problema se produce principalmente en una unión de red virtual (frente a Azure IR), puede tratarse de un problema de rendimiento de la copia. Para revisar los pasos para la solución de problemas, vaya a [Mejora del rendimiento de la actividad de copia.](./copy-activity-performance-troubleshooting.md)
* Puede usar la característica de período de vida para reducir el tiempo de inicio del clúster en las actividades de flujo de datos. Revise [Entorno de ejecución de integración de Data Flow.](./control-flow-execute-data-flow-activity.md#data-flow-integration-runtime)

 ### <a name="hitting-capacity-issues-in-shirself-hosted-integration-runtime"></a>Problemas de capacidad en SHIR (Integration Runtime autohospedado)
 
 **Causa**
 
Esto puede ocurrir si no ha escalado verticalmente SHIR en función de la carga de trabajo.

**Resolución**

* Si encuentra un problema de capacidad de SHIR, actualice la máquina virtual para aumentar el nodo con el fin de equilibrar las actividades. Si recibe un mensaje de error sobre un error o un error general de IR autohospedado, una actualización de IR autohospedado o problemas de conectividad de IR autohospedado, que pueden generar una cola larga, vaya a [Solución de problemas del entorno de ejecución de integración autohospedado](./self-hosted-integration-runtime-troubleshoot-guide.md).

### <a name="error-messages-due-to-long-queues-for-adf-copy-and-data-flow"></a>Mensajes de error debidos a colas largas en la copia de ADF y en Data Flow

**Causa**

Los mensajes de error de colas largas pueden aparecer por varias razones. 

**Resolución**
* Si recibe un mensaje de error de cualquier origen o destino a través de conectores, que puede generar una cola larga, vaya a la [guía de solución de problemas de conectores](./connector-troubleshoot-guide.md).
* Si recibe un mensaje de error sobre el flujo de datos de asignación, que puede generar una cola larga, consulte la [guía de solución de problemas de flujos de datos](./data-flow-troubleshoot-guide.md).
* Si recibe un mensaje de error sobre otras actividades, como Databricks, las actividades personalizadas o HDI, que pueden generar una cola larga, vaya a la [guía de solución de problemas de actividades](./data-factory-troubleshoot-guide.md).
* Si recibe un error al ejecutar paquetes de SSIS, que pueden generar una cola larga, vaya a la [guía de solución de problemas de ejecución de paquetes de Azure-SSIS](./ssis-integration-runtime-ssis-activity-faq.md) y a la [guía de solución de problemas de administración de Integration Runtime](./ssis-integration-runtime-management-troubleshoot.md).


## <a name="next-steps"></a>Pasos siguientes

Para obtener ayuda para solucionar problemas, pruebe estos recursos:

*  [Blog de Data Factory](https://azure.microsoft.com/blog/tag/azure-data-factory/)
*  [Solicitud de características de Data Factory](https://feedback.azure.com/forums/270578-data-factory)
*  [Vídeos de Azure](https://azure.microsoft.com/resources/videos/index/?sort=newest&services=data-factory)
*  [Página de preguntas y respuestas de Microsoft](/answers/topics/azure-data-factory.html)
*  [Información de Twitter sobre Data Factory](https://twitter.com/hashtag/DataFactory)
