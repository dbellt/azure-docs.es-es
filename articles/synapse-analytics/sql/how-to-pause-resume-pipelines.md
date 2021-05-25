---
title: Pausa y reanudación de grupos de SQL dedicados con canalizaciones de Synapse
description: Aprenda a automatizar la pausa y reanudación de un grupo de SQL dedicado con canalizaciones de Synapse en Azure Synapse Analytics.
author: julieMSFT
ms.author: jrasnick
ms.service: synapse-analytics
ms.topic: how-to
ms.date: 02/05/2021
ms.custom: template-how-to
ms.openlocfilehash: b75f1e7df44840618accdbcc3b8d95eba9d2e228
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108776762"
---
# <a name="pause-and-resume-dedicated-sql-pools-with-synapse-pipelines"></a>Pausa y reanudación de grupos de SQL dedicados con canalizaciones de Synapse

Pausar y reanudar grupos de SQL dedicados se puede automatizar mediante canalizaciones de Synapse en Azure Synapse Analytics. La pausa y la reanudación se pueden usar para ahorrar costos para un grupo de SQL dedicado. Esta solución se puede incluir fácilmente en un proceso de orquestación de datos existente. 

Los pasos siguientes le guiarán a través de la configuración de pausas y reanudaciones automatizadas.

1. Creación de una canalización
1. Configure los parámetros en la canalización.
1. Identifique la lista de grupos de SQL dedicados en el área de trabajo de Synapse.
1. Filtre los grupos de SQL dedicados que no quiera pausar o reanudar en la lista. 
1. Recorra en bucle cada grupo de SQL dedicado y:
    1. Compruebe el estado del grupo de SQL dedicado.
    1. Evalúe el estado del grupo de SQL dedicado.
    1. Pause o reanude el grupo de SQL dedicado.

Estos pasos se establecen en una canalización simple en Synapse:

![Canalización de Synapse simple](./media/how-to-pause-resume-pipelines/simple-pipeline.png)


En función de la naturaleza del entorno, es posible que no se aplique todo el proceso descrito aquí y que simplemente desee elegir los pasos adecuados. El proceso descrito aquí se puede usar para pausar o reanudar todas las instancias en un entorno de desarrollo, prueba o prueba de concepto. En el caso de un entorno de producción, es más probable que programe la pausa o la reanudación en una instancia por instancia, por lo que solo necesitará los pasos 5a a 5c.

En los pasos anteriores se usan las API REST para Synapse y Azure SQL:

- [Operaciones de grupos de SQL dedicados](/rest/api/synapse/sqlpools)
 
- [API REST de Azure SQL Database](/rest/api/sql)

Las canalizaciones de Synapse permiten la automatización de la pausa y la reanudación, pero puede ejecutar estos comandos a petición a través de la herramienta o aplicación que prefiera.

## <a name="prerequisites"></a>Requisitos previos

- Un [área de trabajo de Azure Synapse](../get-started-create-workspace.md) existente.
- Al menos un [grupo de SQL dedicado](../get-started-analyze-sql-pool.md).
- El área de trabajo debe tener asignado el rol de colaborador de Azure. Consulte [Concesión del rol Colaborador de Azure a los administradores de Synapse en el área de trabajo](https://review.docs.microsoft.com/en-us/azure/synapse-analytics/security/how-to-set-up-access-control?branch=pr-en-us-146232#step-5-grant-synapse-administrators-the-azure-contributor-role-on-the-workspace).

## <a name="step-1-create-a-pipeline-in-synapse-studio"></a>Paso 1: Creación de una canalización en Synapse Studio
1. Vaya al área de trabajo y abra Synapse Studio. 
1. Seleccione el icono **Integrar** y, a continuación, seleccione el signo **+** para crear una canalización. 
1. Asigne a la canalización el nombre PauseResume.

    ![Creación de una canalización en Synapse Studio](./media/how-to-pause-resume-pipelines/create-pipeline.png) 

## <a name="step-2-create-pipeline-parameters"></a>Paso 2: Creación de parámetros de canalización 

La canalización que creará estará controlada por parámetros. Los parámetros permiten crear una canalización genérica que puede usar en varias suscripciones, grupos de recursos o grupos de SQL dedicados. Seleccione la pestaña **Parámetros** situada en la parte inferior de la pantalla de la canalización. Seleccione **+Nuevo** para crear cada uno de los parámetros siguientes:

    
|Nombre  |Tipo  |Valor predeterminado  |Descripción|
|---------|---------|---------|-----------|
|ResourceGroup    |string        |Synapse          |Nombre del grupo de recursos para los grupos de SQL dedicados|
|SubscriptionID   |string        |<SubscriptionID> |Identificador de suscripción para el grupo de recursos|
|WorkspaceName    |string        |Synapse          |Nombre del área de trabajo|
|SQLPoolName      |string        |SQLPool1         |Nombre del grupo de SQL dedicado|
|PauseorResume    |string        |Pausar            |Estado deseado al final de la ejecución de canalización|

![Parámetros de la canalización en Synapse Studio.](./media/how-to-pause-resume-pipelines/pipeline-parameters.png)

## <a name="step-3-create-list-of-dedicated-sql-pools"></a>Paso 3: Creación de una lista de grupos de SQL dedicados
 
 Configure una actividad **web**. Usará esta actividad para crear la lista de grupos de SQL dedicados mediante una llamada a la solicitud de API REST Grupos de SQL dedicados: lista por servidor. La salida es una cadena JSON que contiene una lista de los grupos de SQL dedicados del área de trabajo. La cadena JSON se pasa a la siguiente actividad.
1. En **Actividades** > **General**, arrastre una actividad **web** al lienzo de canalización como la primera fase de la canalización.  
1. En la pestaña **General,** asigne a esta fase el nombre GET List. 
1. Seleccione la pestaña **Configuración**, haga clic en el espacio de entrada de **dirección URL** y, a continuación, seleccione **Agregar contenido dinámico**. Copie y pegue la solicitud GET que se ha parametrizado mediante la función de cadena @concat siguiente en el cuadro de contenido dinámico. Seleccione **Finalizar**.
El siguiente código es una solicitud GET sencilla:

    ```HTTP
    GET https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Synapse/workspaces/{workspace-name}/sqlPools?api-version=2019-06-01-preview
    ```
    
    Solicitud GET que se ha parametrizado mediante la función de cadena @concat:
    
    ```HTTP
    @concat('https://management.azure.com/subscriptions/',pipeline().parameters.SubscriptionID,'/resourceGroups/',pipeline().parameters.ResourceGroup,'/providers/Microsoft.Synapse/workspaces/',pipeline().parameters.WorkspaceName,'/sqlPools?api-version=2019-06-01-preview')
    ``` 
1. Seleccione la lista desplegable de **Método** y seleccione **GET**.  
1. Seleccione **Avanzado** para expandir el contenido. Seleccione **MSI** como tipo de autenticación. En Recurso, escriba `https://management.azure.com/`. 
    > [!IMPORTANT]
    > Para todas las actividades web o llamadas web de la API REST, debe asegurarse de que la canalización de Synapse se autentique en un grupo de SQL dedicado. La [identidad administrada](../../data-factory/control-flow-web-activity.md#managed-identity) es necesaria para ejecutar estas llamadas a la API REST. 
        
        
    ![Lista de actividades web para grupos de SQL dedicados](./media/how-to-pause-resume-pipelines/web-activity-list-sql-pools.png)



## <a name="step-4-filter-dedicated-sql-pools"></a>Paso 4: Filtrado de grupos de SQL dedicados
Quite los grupos de SQL dedicados que no desea pausar ni reanudar. Use una actividad de filtro que filtre los valores pasados desde la actividad Obtener lista.  En este ejemplo, se extraen los registros de la matriz que no tienen "prod" en el nombre. Aplique otras condiciones según sea necesario. Por ejemplo, filtre por la SKU o el nombre del área de trabajo de Synapse para asegurarse de que solo se identifican los grupos de SQL dedicados válidos.
1. Seleccione y arrastre la actividad **Filtro** en **Iteración y condicionales** al lienzo de canalización.    
![Filtrado de grupos de SQL dedicados](./media/how-to-pause-resume-pipelines/filter-sql-pools.png)    
1. Conecte la actividad web Obtener lista a la actividad Filtro. Seleccione la pestaña verde de la actividad web y arrástrela al cuadro Filtro.
1. Escriba `@activity('Get list').output.value` en **Elementos** donde GET List es el nombre de la actividad web anterior.
1. Escriba `@not(endswith(item().name,'prod'))` en **Condición**. A continuación, los registros restantes de la matriz se pasan a la siguiente actividad.

## <a name="step-5-create-a-foreach-loop"></a>Paso 5: Creación de un bucle ForEach
Cree una actividad ForEach para recorrer en bucle cada grupo de SQL dedicado. 
1. Seleccione y arrastre la actividad **ForEach** en **Iteración y condicionales** al lienzo de canalización.
1. En la pestaña **General**, asigne un nombre a la actividad; nosotros hemos usado "ForEach_pool". 
1. En la pestaña Configuración, seleccione la entrada **Elementos** y **Agregar contenido dinámico**. Desplácese hasta **Salidas de actividad** y seleccione la salida de la actividad de filtro. Agregue `.value` a la actividad. El valor debería parecerse al siguiente: `@activity('Filter_PROD').output.value`. Seleccione **Finalizar**.    
    ![Recorrer en bucle grupos de SQL dedicados](./media/how-to-pause-resume-pipelines/loop-through-sql-pools.png)
1. Seleccione la pestaña **Actividades** y seleccione el lápiz de edición para abrir el lienzo del bucle ForEach. 

## <a name="step-5a-check-the-state-of-the-dedicated-sql-pools"></a>Paso 5a: Comprobación del estado de los grupos de SQL dedicados    
La comprobación del estado del grupo de SQL dedicado requiere una actividad web, similar al paso 1. Esta actividad llama a la [API REST de comprobación de estado del grupo de SQL dedicado para Azure Synapse](../sql-data-warehouse/sql-data-warehouse-manage-compute-rest-api.md#check-database-state). 
1. Seleccione y arrastre una actividad **web** en **General** al lienzo de canalización.    
2. En la pestaña **General**, asigne a esta fase el nombre CheckState. 
3. Seleccione la pestaña **Configuración**.     
4. Haga clic en el espacio de entrada de **dirección URL** y, a continuación, seleccione **Agregar contenido dinámico**. Copie y pegue la solicitud GET que se ha parametrizado mediante la función de cadena @concat siguiente en el cuadro de contenido dinámico. Seleccione **Finalizar**. Al comprobar el estado de nuevo se usa una solicitud GET mediante la siguiente llamada:

    ```HTTP
    GET https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Synapse/workspaces/{workspace-name}/sqlPools/{database-name}?api-version=2019-06-01-preview HTTP/1.1
    ```
    
    Solicitud GET parametrizada mediante la función de cadena @concat:
    
    ```HTTP
    @concat('https://management.azure.com/subscriptions/',pipeline().parameters.SubscriptionID,'/resourceGroups/',pipeline().parameters.ResourceGroup,'/providers/Microsoft.Synapse/workspaces/',pipeline().parameters.WorkspaceName,'/sqlPools/',item().name,'?api-version=2019-06-01-preview')
    ```
    
    En este caso, se usa item().name, que es el nombre del grupo de SQL dedicado del paso 1 que se pasó a esta actividad desde el bucle ForEach. Si usa una canalización para controlar un único grupo de SQL dedicado, puede insertar aquí el nombre del grupo de SQL dedicado o usar un parámetro de la canalización. Por ejemplo, podría usar pipeline().parameters.SQLPoolName.

    La salida es una cadena JSON que contiene detalles del grupo de SQL dedicado, incluido su estado (en properties.status). La cadena JSON se pasa a la siguiente actividad. 
1. Seleccione la lista desplegable de **Método** y seleccione **GET**. Seleccione **Opciones avanzadas** para expandir el contenido. Seleccione **MSI** como tipo de autenticación. En Recurso, escriba `https://management.azure.com/`. 

![Comprobación del estado del grupo de SQL dedicado](./media/how-to-pause-resume-pipelines/check-sql-pool-state.png)
    

    
## <a name="step-5b-evaluate-the-state-of-the-dedicated-sql-pools"></a>Paso 5b: Evaluación del estado del grupo de SQL dedicado
Evalúe el estado deseado, Pausa o Reanudación, y el estado actual, En línea o En pausa, y, a continuación, inicie Pausa o Reanudación según sea necesario.

1. Seleccione y arrastre una actividad **Switch** en **Iteración y condicionales** al lienzo de canalización.    
1. Conecte la actividad **Switch** a la actividad **CheckState**. Seleccione la pestaña verde de la actividad web y arrástrela al cuadro Switch.  
1. En la pestaña **General**, asigne a esta fase el nombre State-PauseOrResume. 

    Según el estado deseado y el estado actual, solo las dos combinaciones siguientes requerirán un cambio de estado: En pausa->Reanudación o En línea->Pausa. 

1. En la pestaña **Actividades**, copie el código siguiente en **Expresión**.

    ```HTTP
    @concat(activity('CheckState').output.properties.status,'-',pipeline().parameters.PauseOrResume)
    ```
    
    Donde Check State es el nombre de la actividad web anterior con output.properties.status que define el estado actual y pipeline().parameters.PauseOrResume indica el estado deseado.
    
    La condición de comprobación realiza una comprobación del estado deseado y del estado actual. Si el estado deseado es Reanudación y el estado actual es En pausa, se invoca una actividad de reanudación dentro del caso En pausa-Reanudación. Si el estado deseado es Pausa y el estado actual es En línea, se invoca una actividad de pausa con el caso En línea-Pausa. Cualquier otro caso, como un estado deseado de Pausa y un estado actual de En pausa, o un estado deseado de Reanudación y un estado actual de En línea, no requeriría ninguna acción y se controlaría mediante el caso predeterminado, que no tiene ninguna actividad.
1. En la pestaña Actividades, seleccione **+ Agregar caso**.  Agregue los casos `Paused-Resume` y `Online-Pause`. 
    ![Comprobación de la condición de estado del grupo de SQL dedicado](./media/how-to-pause-resume-pipelines/check-condition.png)

### <a name="step-5c-pause-or-resume-dedicated-sql-pools"></a>Paso 5c: Pausa o reanudación de los grupos de SQL dedicados     

El último y único paso pertinente para algunos requisitos es iniciar la pausa o reanudación del grupo de SQL dedicado. En este paso se usa de nuevo una actividad web, que llama a la [API REST de proceso de pausa o reanudación para Azure Synapse](../sql-data-warehouse/sql-data-warehouse-manage-compute-rest-api.md#pause-compute). 
1. Seleccione el lápiz de edición de actividad y agregue una actividad **web** al lienzo State-PauseorResume. 
1. Seleccione la pestaña **Configuración**, haga clic en el espacio de entrada de **dirección URL** y, a continuación, seleccione **Agregar contenido dinámico**. Copie y pegue la solicitud POST que se ha parametrizado mediante la función de cadena @concat siguiente en el cuadro de contenido dinámico. Seleccione **Finalizar**. 
1. Seleccione la lista desplegable de **Método** y seleccione **POST**.
1. En la sección Cuerpo, escriba "Pausar y reanudar"
1. Seleccione **Avanzado** para expandir el contenido. Seleccione **MSI** como tipo de autenticación. En Recurso, escriba `https://management.azure.com/`. 
1. Agregue una segunda actividad para la funcionalidad de reanudación mediante el código con parámetros siguiente.

    ![Reanudación del grupo de SQL dedicado](./media/how-to-pause-resume-pipelines/true-condition-resume.png)
    
    
    El ejemplo aquí es reanudar un grupo de SQL dedicado, invocando una solicitud POST mediante la siguiente llamada:
    
    ```HTTP
    POST https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Synapse/workspaces/{workspace-name}/sqlPools/{database-name}/resume?api-version=2019-06-01-preview HTTP/1.1
    ```
    
    Puede parametrizar la instrucción POST anterior mediante la función de cadena @concat:
    
    ```HTTP
    @concat('https://management.azure.com/subscriptions/',pipeline().parameters.SubscriptionID,'/resourceGroups/',pipeline().parameters.ResourceGroup,'/providers/Microsoft.Synapse/workspaces/',pipeline().parameters.WorkspaceName,'/sqlPools/',activity('CheckState').output.name,'/resume?api-version=2019-06-01-preview')
    ```
    
    En este caso, se usa la actividad 'Check State'.output.name con los nombres de los grupos de SQL dedicados del paso 3a que se pasaron a esta actividad a través de la condición Switch. Si usa una sola actividad en una base de datos única, puede insertar aquí el nombre del grupo de SQL dedicado o usar un parámetro de la canalización. Por ejemplo, podría usar pipeline().parameters.DatabaseName.
    
    La solicitud POST para pausar un grupo de SQL dedicado es:
    
    ```HTTP
    POST https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Synapse/workspaces/{workspace-name}/sqlPools/{database-name}/pause?api-version=2019-06-01-preview HTTP/1.1    
    ```    
    
    La solicitud POST se puede parametrizar mediante la función de cadena @concat como se muestra a continuación:
    
    ```HTTP
    @concat('https://management.azure.com/subscriptions/',pipeline().parameters.SubscriptionID,'/resourceGroups/',pipeline().parameters.ResourceGroup,'/providers/Microsoft.Synapse/workspaces/',pipeline().parameters.WorkspaceName,'/sqlPools/',activity('CheckState').output.name,'/pause?api-version=2019-06-01-preview')
    ```

## <a name="pipeline-run-output"></a>Salida de ejecución de la canalización

Cuando se ejecute la canalización completa, verá la salida que se muestra a continuación. Puede ejecutar la canalización seleccionando el modo de **depuración** o seleccionando **Agregar desencadenador**. Para los resultados de la canalización siguientes, el parámetro de canalización denominado "ResourceGroup" se estableció en un único grupo de recursos que tenía dos áreas de trabajo de Synapse. Una se denominó testprod y se filtró, la segunda se denominó test1. El grupo de SQL dedicado test1 se ha pausado, por lo que el trabajo inició una reanudación.

![Salida de ejecución de la canalización](./media/how-to-pause-resume-pipelines/pipeline-run-output.png)

## <a name="save-your-pipeline"></a>Guarde la canalización.

Para guardar la canalización, seleccione la opción **Publicar todo** situada encima de la canalización.

## <a name="schedule-your-pause-or-resume-pipeline-to-run"></a>Programación de la ejecución de la canalización de pausa o reanudación

Para programar la canalización, seleccione **Agregar desencadenador** en la parte superior de la canalización. Siga las pantallas para programar la ejecución de la canalización en un momento especificado.

![Selección del desencadenador para establecer la hora de ejecución de la canalización](./media/how-to-pause-resume-pipelines/trigger.png)

## <a name="next-steps"></a>Pasos siguientes

Puede encontrar más detalles sobre Identidad administrada para Azure Synapse y cómo se agrega Identidad administrada al grupo de SQL dedicado aquí:

[Identidad administrada del área de trabajo de Azure Synapse](../security/synapse-workspace-managed-identity.md)

[Concesión de permisos a una identidad administrada de área de trabajo (versión preliminar)](../security/how-to-grant-workspace-managed-identity-permissions.md)

[Control de acceso SQL para las ejecuciones de canalización de Synapse](../security/how-to-set-up-access-control.md#step-73-sql-access-control-for-synapse-pipeline-runs)

