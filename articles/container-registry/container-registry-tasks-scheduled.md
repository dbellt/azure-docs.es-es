---
title: 'Tutorial: Programación de una tarea de ACR'
description: En este tutorial, aprenderá a ejecutar una tarea de Azure Container Registry según una programación definida mediante la configuración de uno o más desencadenadores de temporizador.
ms.topic: article
ms.date: 11/24/2020
ms.openlocfilehash: fa80bcbd318266a86c5bec08c9ee60fc0d22a10d
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107780881"
---
# <a name="tutorial-run-an-acr-task-on-a-defined-schedule"></a>Tutorial: Ejecución de una tarea de ACR según una programación definida

En este tutorial se muestra cómo ejecutar una [Tarea de ACR](container-registry-tasks-overview.md) según una programación. Para programar una tarea, configure uno o varios *desencadenadores de temporizador*. Los desencadenadores de temporizador se pueden usar solos o en combinación con otros desencadenadores de tareas.

En este tutorial, obtendrá información sobre la programación de tareas y:

> [!div class="checklist"]
> * Creación de una tarea con un desencadenador de temporizador
> * Administración de desencadenadores de temporizador

La programación de tareas es útil para escenarios como los siguientes:

* Ejecutar una carga de trabajo de contenedor para operaciones de mantenimiento programadas. Por ejemplo, ejecute una aplicación en contenedores para que elimine las imágenes innecesarias del registro.
* Ejecutar un conjunto de pruebas en una imagen de producción durante el día de trabajo como parte de la supervisión del sitio activo.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

## <a name="about-scheduling-a-task"></a>Acerca de la programación de tareas

* **Desencadenador con una expresión cron**: el desencadenador de temporizador de una tarea utiliza una *expresión cron*. La expresión es una cadena con cinco campos que especifican el minuto, la hora, el día, el mes y el día de la semana en que se va a desencadenar la tarea. Se admiten frecuencias de hasta una vez por minuto.

  Por ejemplo, la expresión `"0 12 * * Mon-Fri"` desencadena una tarea al mediodía hora UTC todos los días de la semana. Consulte los [detalles](#cron-expressions) más adelante en este artículo.
* **Varios desencadenadores de temporizador**: se permite la adición de varios temporizadores a una tarea, siempre que las programaciones sean distintas.
    * Puede especificar varios desencadenadores de temporizador al crear la tarea o agregarlos más tarde.
    * Opcionalmente, dé un nombre a los desencadenadores para facilitar la administración; de lo contrario, ACR Tasks proporcionará nombres de desencadenador predeterminados.
    * Si las programaciones del temporizador se superponen en algún momento, ACR Tasks desencadena la tarea a la hora programada para cada temporizador.
* **Otros desencadenadores de tarea**: en una tarea desencadenada por temporizadores, también puede habilitar desencadenadores en función de la [confirmación del código fuente](container-registry-tutorial-build-task.md) o las [actualizaciones de la imagen de base](container-registry-tutorial-base-image-update.md). Al igual que otras tareas de ACR, también puede [ejecutar manualmente][az-acr-task-run] una tarea programada.

## <a name="create-a-task-with-a-timer-trigger"></a>Creación de una tarea con un desencadenador de temporizador

### <a name="task-command"></a>Comando de tarea

Primero, rellene la variables de entorno de shell siguiente con un valor adecuado para el entorno. Este paso no es estrictamente necesario, pero hace que la ejecución de los comandos de varias líneas de la CLI de Azure en este tutorial sea un poco más fácil. Si no rellena esta variable, debe reemplazar manualmente cada valor siempre que aparezca en los comandos de ejemplo.

[![Insertar inicio](https://shell.azure.com/images/launchcloudshell.png "Inicio de Azure Cloud Shell")](https://shell.azure.com)

```console
ACR_NAME=<registry-name>        # The name of your Azure container registry
```

Cuando crea una tarea con el comando [az acr task create][az-acr-task-create], opcionalmente puede agregar un desencadenador de temporizador. Agregue el parámetro `--schedule` y pase una expresión cron para el temporizador.

A modo de ejemplo sencillo, la siguiente tarea desencadena la ejecución de la imagen `hello-world` de Microsoft Container Registry todos los días a las 21:00 UTC. La tarea se ejecuta sin un contexto de código fuente.

```azurecli
az acr task create \
  --name timertask \
  --registry $ACR_NAME \
  --cmd mcr.microsoft.com/hello-world \
  --schedule "0 21 * * *" \
  --context /dev/null
```

Ejecute el comando [az acr task show][az-acr-task-show] para ver que el desencadenador de temporizador está configurado. De forma predeterminada, el desencadenador de actualización de la imagen de base también está habilitado.

```azurecli
az acr task show --name timertask --registry $ACR_NAME --output table
```

```output
NAME      PLATFORM    STATUS    SOURCE REPOSITORY       TRIGGERS
--------  ----------  --------  -------------------     -----------------
timertask linux       Enabled                           BASE_IMAGE, TIMER
```

## <a name="trigger-the-task"></a>Desencadenar la tarea

Desencadene la tarea manualmente con [az acr task run][az-acr-task-run] para asegurarse de que está configurado correctamente:

```azurecli
az acr task run --name timertask --registry $ACR_NAME
```

Si el contenedor se ejecuta correctamente, el resultado es similar al siguiente. La salida está condensada para mostrar los pasos principales.

```output
Queued a run with ID: cf2a
Waiting for an agent...
2020/11/20 21:03:36 Using acb_vol_2ca23c46-a9ac-4224-b0c6-9fde44eb42d2 as the home volume
2020/11/20 21:03:36 Creating Docker network: acb_default_network, driver: 'bridge'
[...]
2020/11/20 21:03:38 Launching container with name: acb_step_0

Hello from Docker!
This message shows that your installation appears to be working correctly.
[...]
```

Después de la hora programada, ejecute el comando [az acr task list-runs][az-acr-task-list-runs] para comprobar que el temporizador desencadenó la tarea según lo esperado:

```azurecli
az acr task list-runs --name timertask --registry $ACR_NAME --output table
```

Cuando el temporizador desencadena la tarea correctamente, el resultado es similar al siguiente:

```output
RUN ID    TASK       PLATFORM    STATUS     TRIGGER    STARTED               DURATION
--------  ---------  ----------  ---------  ---------  --------------------  ----------
ca15      timertask  linux       Succeeded  Timer      2020-11-20T21:00:23Z  00:00:06
ca14      timertask  linux       Succeeded  Manual     2020-11-20T20:53:35Z  00:00:06
```

## <a name="manage-timer-triggers"></a>Administración de desencadenadores de temporizador

Use los comandos [az acr task timer][az-acr-task-timer] para administrar los desencadenadores de temporizador de una tarea de ACR.

### <a name="add-or-update-a-timer-trigger"></a>Adición o actualización de un desencadenador de temporizador

Una vez que se ha creado una tarea, tiene la opción de agregar un desencadenador de temporizador mediante el comando [az acr task timer add][az-acr-task-timer-add]. En el ejemplo siguiente se agrega un desencadenador de temporizador denominado *timer2* a la tarea *timertask* creada anteriormente. Este temporizador desencadena la tarea todos los días a las 10:30 UTC.

```azurecli
az acr task timer add \
  --name timertask \
  --registry $ACR_NAME \
  --timer-name timer2 \
  --schedule "30 10 * * *"
```

Actualice la programación de un desencadenador existente o cambie su estado mediante el comando [az acr task timer update][az-acr-task-timer-update]. Por ejemplo, actualice el desencadenador denominado *timer2* para que desencadene la tarea a las 11:30 UTC:

```azurecli
az acr task timer update \
  --name timertask \
  --registry $ACR_NAME \
  --timer-name timer2 \
  --schedule "30 11 * * *"
```

### <a name="list-timer-triggers"></a>Enumeración de los desencadenadores de temporizador

El comando [az acr task timer list][az-acr-task-timer-list] muestra los desencadenadores de temporizador configurados para una tarea:

```azurecli
az acr task timer list --name timertask --registry $ACR_NAME
```

Salida de ejemplo:

```JSON
[
  {
    "name": "timer2",
    "schedule": "30 11 * * *",
    "status": "Enabled"
  },
  {
    "name": "t1",
    "schedule": "0 21 * * *",
    "status": "Enabled"
  }
]
```

### <a name="remove-a-timer-trigger"></a>Eliminación de un desencadenador de temporizador

Use el comando [az acr task timer remove][az-acr-task-timer-remove] para eliminar un desencadenador de temporizador de una tarea. En el ejemplo siguiente, se elimina el desencadenador *timer2* de *timertask*:

```azurecli
az acr task timer remove \
  --name timertask \
  --registry $ACR_NAME \
  --timer-name timer2
```

## <a name="cron-expressions"></a>Expresiones Cron

ACR Tasks usa la biblioteca [NCronTab](https://github.com/atifaziz/NCrontab) para interpretar expresiones cron. Las expresiones admitidas en ACR Tasks tienen cinco campos obligatorios, separados por espacios en blanco:

`{minute} {hour} {day} {month} {day-of-week}`

La zona horaria que se usa con las expresiones cron es la Hora universal coordinada (UTC). Las horas están en formato de 24 horas.

> [!NOTE]
> ACR Tasks no admite el campo `{second}` o `{year}` en las expresiones cron. Si copia una expresión cron que se utiliza en otro sistema, asegúrese de quitar esos campos, si se usan.

Cada campo puede tener uno de los siguientes tipos de valores:

|Tipo  |Ejemplo  |Cuándo se desencadena  |
|---------|---------|---------|
|Un valor específico |<nobr>`"5 * * * *"`</nobr>|cada hora a "y cinco"|
|Todos los valores (`*`)|<nobr>`"* 5 * * *"`</nobr>|cada minuto de la hora empezando a las 5:00 UTC (60 veces al día)|
|Un intervalo (operador `-`)|<nobr>`"0 1-3 * * *"`</nobr>|3 veces al día, a la 1:00, a las 2:00 y a las 3:00 UTC|
|Un conjunto de valores (operador `,`)|<nobr>`"20,30,40 * * * *"`</nobr>|3 veces por hora, a "y veinte", a la media y a "menos veinte"|
|Un valor de intervalo (operador `/`)|<nobr>`"*/10 * * * *"`</nobr>|6 veces por hora, a "y diez", a "y veinte", etc.

[!INCLUDE [functions-cron-expressions-months-days](../../includes/functions-cron-expressions-months-days.md)]

### <a name="cron-examples"></a>Ejemplos Cron

|Ejemplo|Cuándo se desencadena  |
|---------|---------|
|`"*/5 * * * *"`|Una vez cada cinco minutos|
|`"0 * * * *"`|Una vez al principio de cada hora|
|`"0 */2 * * *"`|Una vez cada dos horas|
|`"0 9-17 * * *"`|una vez cada hora de las 9:00 a las 17:00 UTC|
|`"30 9 * * *"`|a las 9:30 UTC todos los días|
|`"30 9 * * 1-5"`|a las 9:30 UTC todos los días laborables|
|`"30 9 * Jan Mon"`|a las 9:30 UTC todos los lunes de enero|

## <a name="clean-up-resources"></a>Limpieza de recursos

Para quitar todos los recursos que ha creado en esta serie de tutoriales, incluido el registro o los registros de contenedor, la instancia de contenedor, el almacén de claves y la entidad de servicio, ejecute los comandos siguientes:

```azurecli
az group delete --resource-group $RES_GROUP
az ad sp delete --id http://$ACR_NAME-pull
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a crear tareas de Azure Container Registry que se desencadenan automáticamente con un temporizador. 

Para ver un ejemplo del uso de una tarea programada para limpiar los repositorios de un registro, consulte[Purga automática de imágenes a partir de una instancia de Azure Container Registry](container-registry-auto-purge.md).

Para ver ejemplos de tareas desencadenadas por confirmaciones de código fuente o actualizaciones de imágenes base, consulte otros artículos de la [serie de tutoriales de ACR Tasks](container-registry-tutorial-quick-task.md).



<!-- LINKS - External -->
[task-examples]: https://github.com/Azure-Samples/acr-tasks


<!-- LINKS - Internal -->
[az-acr-task-create]: /cli/azure/acr/task#az_acr_task_create
[az-acr-task-show]: /cli/azure/acr/task#az_acr_task_show
[az-acr-task-list-runs]: /cli/azure/acr/task#az_acr_task_list_runs
[az-acr-task-timer]: /cli/azure/acr/task/timer
[az-acr-task-timer-add]: /cli/azure/acr/task/timer#az_acr_task_timer_add
[az-acr-task-timer-remove]: /cli/azure/acr/task/timer#az_acr_task_timer_remove
[az-acr-task-timer-list]: /cli/azure/acr/task/timer#az_acr_task_timer_list
[az-acr-task-timer-update]: /cli/azure/acr/task/timer#az_acr_task_timer_update
[az-acr-task-run]: /cli/azure/acr/task#az_acr_task_run
[az-acr-task]: /cli/azure/acr/task
[azure-cli-install]: /cli/azure/install-azure-cli
