---
title: Envío de configuraciones a App Configuration con Azure Pipelines
description: Aprenda a usar Azure Pipelines para enviar pares clave-valor a un almacén de App Configuration
services: azure-app-configuration
author: AlexandraKemperMS
ms.service: azure-app-configuration
ms.topic: how-to
ms.date: 02/23/2021
ms.author: alkemper
ms.openlocfilehash: a3b3c8760c3bf7d6bf4bee444bef7ed77134fb5a
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108748311"
---
# <a name="push-settings-to-app-configuration-with-azure-pipelines"></a>Envío de configuraciones a App Configuration con Azure Pipelines

La tarea [Azure App Configuration Push](https://marketplace.visualstudio.com/items?itemName=AzureAppConfiguration.azure-app-configuration-task-push) envía pares clave-valor de un archivo de configuración en el almacén de App Configuration. Esta tarea habilita la funcionalidad completa dentro de la canalización, ya que ahora es posible extraer la configuración del almacén de App Configuration, así como enviar la configuración al mismo destino.

## <a name="prerequisites"></a>Requisitos previos

- Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/)
- Recurso de App Configuration: cree uno de forma gratuita en [Azure Portal](https://portal.azure.com).
- Proyecto de Azure DevOps: [cree uno gratis](https://go.microsoft.com/fwlink/?LinkId=2014881).
- Tarea Azure App Configuration Push: descárguela de manera gratuita de [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=AzureAppConfiguration.azure-app-configuration-task-push).

## <a name="create-a-service-connection"></a>Creación de una conexión de servicio

[!INCLUDE [azure-app-configuration-service-connection](../../includes/azure-app-configuration-service-connection.md)]

## <a name="add-role-assignment"></a>Agregar asignación de roles

[!INCLUDE [azure-app-configuration-role-assignment](../../includes/azure-app-configuration-role-assignment.md)]

## <a name="use-in-builds"></a>Uso en compilaciones

En esta sección se explicará cómo usar la tarea Azure App Configuration Push en una canalización de compilación de Azure DevOps.

1. Vaya a la página de canalización de compilación al hacer clic en **Canalizaciones** > **Canalizaciones**. La documentación para las canalizaciones de compilación se puede encontrar [aquí](/azure/devops/pipelines/create-first-pipeline?tabs=tfs-2018-2).
      - Si va a crear una nueva canalización de compilación, en el último paso del proceso, en la pestaña **Revisar**, seleccione **Mostrar asistente** en el lado derecho de la canalización.
      ![Captura de pantalla que muestra el botón Mostrar asistente de una nueva canalización.](./media/new-pipeline-show-assistant.png)
      - Si usa una canalización de compilación existente, haga clic en el botón **Editar** en la parte superior derecha.
      ![Captura de pantalla que muestra el botón Editar para una canalización existente.](./media/existing-pipeline-show-assistant.png)
1. Busque la tarea de **Insertar de Azure App Configuration**.
![Captura de pantalla que muestra el cuadro de diálogo Agregar tarea con Insertar de Azure App Configuration en el cuadro de búsqueda.](./media/add-azure-app-configuration-push-task.png)
1. Configure los parámetros necesarios para que la tarea envíe los pares clave-valor del archivo de configuración al almacén de App Configuration. Las explicaciones de los parámetros están disponibles en la sección **Parámetros** siguiente y en la información sobre herramientas junto a cada parámetro.
![Captura de pantalla que muestra los parámetros de la tarea de inserciones de configuración de la aplicación.](./media/azure-app-configuration-push-parameters.png)
1. Guarde y ponga en cola una compilación. El registro de compilación mostrará los errores que se produjeron durante la ejecución de la tarea.

## <a name="use-in-releases"></a>Uso en versiones

En esta sección se explicará cómo usar la tarea Azure App Configuration Push en las canalizaciones de versión de Azure DevOps.

1. Vaya a la página de canalización de versión; para ello, seleccione **Canalizaciones** > **Versiones**. La documentación para las canalizaciones de versión se puede encontrar [aquí](/azure/devops/pipelines/release).
1. Elija una canalización de versión existente. Si no tiene una, seleccione **+ Nueva** para crear una nueva.
1. Seleccione el botón **Editar** en la esquina superior derecha para editar la canalización de versión.
1. En el menú desplegable **Tareas**, seleccione la **Fase** en la que quiere agregar la tarea. Puede encontrar más información sobre las fases [aquí](/azure/devops/pipelines/release/environments).
![Captura de pantalla que muestra la fase seleccionada en el menú desplegable Tareas.](./media/pipeline-stage-tasks.png)
1. Haga clic en **+** junto al trabajo al que quiere agregar una nueva tarea.
![Captura de pantalla que muestra el botón de signo más junto al trabajo.](./media/add-task-to-job.png)
1. En el cuadro de diálogo **agregar tareas**, escriba **Inserciones de Azure App Configuration** en el cuadro de búsqueda y selecciónela.
1. Configure los parámetros necesarios dentro de la tarea para enviar los pares clave-valor del archivo de configuración al almacén de App Configuration. Las explicaciones de los parámetros están disponibles en la sección **Parámetros** siguiente y en la información sobre herramientas junto a cada parámetro.
1. Guarde y ponga en cola una versión. El registro de versión mostrará los errores que se hayan encontrado durante la ejecución de la tarea.

## <a name="parameters"></a>Parámetros

La tarea App Configuration Push usa los siguientes parámetros:

- **Suscripción de Azure**: una lista desplegable que contiene las conexiones de servicio de Azure disponibles. Para actualizar la lista de conexiones de servicio de Azure disponibles, presione el botón **Refresh Azure subscription** (Actualizar suscripción a Azure) a la derecha del cuadro de texto.
- **Nombre de App Configuration**: una lista desplegable que carga los almacenes de configuración disponibles para la suscripción seleccionada. Para actualizar la lista de almacenes de configuración disponibles, presione el botón **Refresh App Configuration Name** (Actualizar nombre de App Configuration) a la derecha del cuadro de texto.
- **Ruta del archivo de configuración**: la ruta de acceso a su archivo de configuración. El parámetro **Ruta de archivo de configuración** comienza en la raíz del repositorio de archivos. Puede examinar el artefacto de compilación para seleccionar un archivo de configuración. (Botón `...` a la derecha del cuadro de texto). Los formatos de archivo admitidos son: YAML, JSON y de propiedades. El siguiente es un archivo de configuración de ejemplo en formato JSON.
    ```json
    {
        "TestApp:Settings:BackgroundColor":"#FFF",
        "TestApp:Settings:FontColor":"#000",
        "TestApp:Settings:FontSize":"24",
        "TestApp:Settings:Message": "Message data"
    }
    ```
- **Separador**: el separador que se usa para aplanar los archivos .yml y .json.
- **Profundidad**: la profundidad con la que se aplanarán los archivos .yml y .json.
- **Prefijo**: una cadena que se anexa al principio de cada clave enviada al almacén de App Configuration.
- **Etiqueta**: una cadena que se agrega a cada par clave-valor como etiqueta en el almacén de App Configuration.
- **Tipo de contenido**: una cadena que se agrega a cada par clave-valor como tipo de contenido en el almacén de App Configuration.
- **Etiquetas**: un objeto JSON en el formato `{"tag1":"val1", "tag2":"val2"}`, que define las etiquetas que se agregaron a cada par clave-valor enviado al almacén de App Configuration.
- **Elimine todos los demás pares clave-valor en el almacén con el prefijo y la etiqueta especificados**: El valor predeterminado es **Sin activar**.
  - **Activado**: quita todos los pares clave-valor en el almacén de App Configuration que coinciden con el prefijo y la etiqueta especificados antes de enviar nuevos pares clave-valor del archivo de configuración.
  - **Sin activar**: envía todos los pares clave-valor del archivo de configuración al almacén de App Configuration y deja intacto el resto del contenido del almacén.



## <a name="troubleshooting"></a>Solución de problemas

Si se produce un error inesperado, los registros de depuración se pueden habilitar; para ello, establezca la variable de canalización `system.debug` en `true`.

## <a name="faq"></a>Preguntas más frecuentes

**¿Cómo puedo cargar varios archivos de configuración?**

Cree varias instancias de la tarea Azure App Configuration Push dentro de la misma canalización para enviar varios archivos de configuración al almacén de App Configuration.

**¿Por qué recibo un error 409 al intentar enviar pares clave-valor a mi almacén de configuración?**

Se producirá un mensaje de error de conflicto 409 si la tarea intenta quitar o sobrescribir un par clave-valor que está bloqueado en el almacén de App Configuration.
