---
title: Edición de la configuración de entorno y entorno de ejecución en Azure Logic Apps de inquilino único
description: Cambie la configuración de entorno y entorno de ejecución de las aplicaciones lógicas en Azure Logic Apps de inquilino único.
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, azla
ms.topic: how-to
ms.date: 05/25/2021
ms.openlocfilehash: fcc7ac002c9d1024abc17dc26ba0b231213ceb53
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110386551"
---
# <a name="edit-host-and-app-settings-for-logic-apps-in-single-tenant-azure-logic-apps"></a>Edición de la configuración de host y aplicación para aplicaciones lógicas en Azure Logic Apps de inquilino único

En Azure Logic Apps de *inquilino único*, la *configuración de aplicación* de una aplicación lógica especifica las opciones de configuración global que afectan a *todos los flujos de trabajo* de esa aplicación lógica. Sin embargo, esta configuración *solo* se aplica cuando estos flujos de trabajo se ejecutan en el *entorno de desarrollo local*. Mientras se ejecutan localmente, los flujos de trabajo pueden acceder a esta configuración de aplicación como *variables de entorno local*, que usan las herramientas de desarrollo local para los valores que a menudo pueden cambiar entre entornos. Por ejemplo, estos valores pueden incluir cadenas de conexión. Al implementar en Azure, la configuración de aplicación se omite y no se incluye con la implementación.

La aplicación lógica también tiene la *configuración de host*, que especifica los valores y la configuración del entorno de ejecución que se aplican a *todos los flujos de trabajo* de esa aplicación lógica, por ejemplo, los valores predeterminados para el rendimiento, la capacidad, el tamaño de los datos, entre otros, *tanto si se ejecutan localmente como en Azure*.

<a name="app-settings-parameters-deployment"></a>

## <a name="app-settings-parameters-and-deployment"></a>Configuración de aplicación, parámetros e implementación

En Azure Logic Apps *multiinquilino*, la implementación depende de las plantillas de Azure Resource Manager (plantillas de ARM), que combinan y controlan el aprovisionamiento de recursos para las aplicaciones lógicas y la infraestructura. Este diseño supone un desafío cuando tiene que mantener variables de entorno para aplicaciones lógicas en varios entornos de desarrollo, pruebas y producción. Todo lo que hay en una plantilla de ARM se define en la implementación. Si necesita cambiar solo una variable, tendrá que volver a implementarlo todo.

En Azure Logic Apps de *inquilino único*, la implementación es más fácil porque puede separar el aprovisionamiento de recursos entre las aplicaciones y la infraestructura. Puede usar *parámetros* para abstraer los valores que pueden cambiar entre entornos. Al definir los parámetros que se usarán en los flujos de trabajo, primero puede centrarse en el diseño de los flujos de trabajo y, a continuación, insertar las variables específicas del entorno más adelante. Puede llamar a las variables de entorno de ejecución y hacer referencia a estas mediante la configuración y los parámetros de la aplicación. De este modo, no tiene que volver a implementar con tanta frecuencia.

La configuración de aplicación se integra con Azure Key Vault. Puede [hacer referencia directamente a cadenas seguras](../app-service/app-service-key-vault-references.md), como claves y cadenas de conexión. De forma similar a las plantillas de Azure Resource Manager (plantillas de ARM), donde puede definir variables de entorno en el momento de la implementación, puede definir la configuración de aplicación dentro de la [definición de flujo de trabajo de la aplicación lógica](/azure/templates/microsoft.logic/workflows). A continuación, puede capturar valores de infraestructura generados dinámicamente, como puntos de conexión, cadenas de almacenamiento, etc. Sin embargo, la configuración de aplicación tiene limitaciones de tamaño y no se puede hacer referencia a ella desde determinadas áreas de Azure Logic Apps.

Para obtener más información sobre cómo configurar las aplicaciones lógicas para la implementación, consulte la siguiente documentación:

- [Creación de parámetros para los valores que cambian en los flujos de trabajo entre entornos de Azure Logic Apps de inquilino único](parameterize-workflow-app.md)
- [Introducción a la implementación de DevOps para aplicaciones lógicas basadas en inquilino único](devops-deployment-single-tenant-azure-logic-apps.md)
- [Configuración de la implementación de DevOps para aplicaciones lógicas basadas en inquilino único](set-up-devops-deployment-single-tenant-azure-logic-apps.md)

## <a name="visual-studio-code-project-structure"></a>Estructura del proyecto de Visual Studio Code

[!INCLUDE [Visual Studio Code - logic app project structure](../../includes/logic-apps-single-tenant-project-structure-visual-studio-code.md)]

<a name="reference-local-settings-json"></a>

## <a name="reference-for-app-settings---localsettingsjson"></a>Referencia de la configuración de aplicación: local.settings.json

En Visual Studio Code, en el nivel de raíz del proyecto de aplicación lógica, el archivo **local.settings.json** contiene opciones de configuración global que afectan a *todos los flujos de trabajo* de esa aplicación lógica mientras se ejecuta en el entorno de desarrollo local. Cuando los flujos de trabajo se ejecutan localmente, se accede a esta configuración como variables de entorno local, y sus valores a menudo pueden cambiar entre los distintos entornos donde se ejecutan los flujos de trabajo. Para ver y administrar esta configuración, consulte [Administración de la configuración de aplicación: local.settings.json](#manage-app-settings).

La configuración de aplicación de Azure Logic Apps funciona de forma similar a la configuración de aplicación de Azure Functions o Azure Web Apps. Si ha usado estos otros servicios antes, es posible que ya esté familiarizado con la configuración de aplicación. Para obtener más información, consulte [Referencia de configuración de aplicación de Azure Functions](../azure-functions/functions-app-settings.md) y [Uso de Azure Functions Core Tools: archivo de configuración local](../azure-functions/functions-run-local.md#local-settings-file).

| Configuración | Valor predeterminado | Descripción |
|---------|---------------|-------------|
| `AzureWebJobsStorage` | Ninguno | Establece la cadena de conexión de una cuenta de Azure Storage. |
| `Workflows.<workflowName>.FlowState` | Ninguno | Establece el estado para <*workflowName*>. |
| `Workflows.<workflowName>.RuntimeConfiguration.RetentionInDays` | Ninguno | Establece las opciones de una operación de <*workflowName*>. |
| `Workflows.Connection.AuthenticationAudience` | Ninguno | Establece la audiencia para autenticar una conexión hospedada en Azure. |
| `Workflows.WebhookRedirectHostUri` | Ninguno | Establece el nombre de host que se usará para las direcciones URL de devolución de llamada de webhook. |
| `WEBSITE_LOAD_ROOT_CERTIFICATES` | Ninguno | Establece las huellas digitales para que los certificados raíz sean de confianza. |
||||

<a name="manage-app-settings"></a>

## <a name="manage-app-settings---localsettingsjson"></a>Administración de la configuración de aplicación: local.settings.json

Para agregar, actualizar o eliminar la configuración de aplicación, seleccione y revise las secciones siguientes para Visual Studio Code, Azure Portal, la CLI de Azure o la plantilla de ARM (Bicep). Para ver la configuración de aplicación específica de las aplicaciones lógicas, revise la [guía de referencia de la configuración de aplicación disponible: local.settings.json](#reference-local-settings-json).

### <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Para revisar la configuración de aplicación de la aplicación lógica en Visual Studio Code, siga estos pasos:

1. En el proyecto de aplicación lógica, en el nivel de raíz del proyecto, busque y abra el archivo **local.settings.json**.

1. En el objeto `Values`, revise la configuración de aplicación de la aplicación lógica.

   Para obtener más información sobre esta configuración, revise la [guía de referencia de la configuración de aplicación disponible: local.settings.json](#reference-local-settings-json).

Para agregar una configuración de aplicación, siga estos pasos:

1. En el archivo **local.settings.json**, busque el objeto `Values`.

1. En el objeto `Values`, agregue la configuración de aplicación que desea aplicar al ejecutar localmente en Visual Studio Code. Algunas opciones le permiten agregar una configuración para un flujo de trabajo específico, por ejemplo:

   ```json
   {
      "IsEncrypted": false,
      "Values": {
         "AzureWebJobsStorage": "UseDevelopmentStorage=true",
         "Workflows.WorkflowName1.FlowState" : "Disabled",
         <...>
     }
   }
   ```

### <a name="azure-portal"></a>[Azure Portal](#tab/azure-portal)

Para revisar la configuración de aplicación de la aplicación lógica basada en inquilino único en Azure Portal, siga estos pasos:

1. En el cuadro de búsqueda de [Azure Portal](https://portal.azure.com/), busque y abra la aplicación lógica.

1. En el menú de la aplicación lógica, en **Configuración**, seleccione **Configuración**.

1. En la página **Configuración**, en la pestaña **Configuración de aplicación,** revise la configuración de aplicación de la aplicación lógica.

   Para obtener más información sobre esta configuración, revise la [guía de referencia de la configuración de aplicación disponible: local.settings.json](#reference-local-settings-json).

1. Para ver todos los valores, seleccione **Mostrar valores**. O bien, para ver un valor único, seleccione ese valor.

Para agregar una configuración, siga estos pasos:

1. En la pestaña **Configuración de aplicación**, en **Configuración de aplicación**, seleccione **Nueva configuración de aplicación**.

1. En **Nombre**, escriba la *clave* o el nombre de la nueva configuración.

1. En **Valor**, escriba el valor de la nueva configuración.

1. Cuando esté listo para crear el nuevo par *clave-valor*, seleccione **Aceptar**.

:::image type="content" source="./media/edit-app-settings-host-settings/portal-app-settings-values.png" alt-text="Captura de pantalla que muestra Azure Portal y el panel de configuración con los valores y la configuración de la aplicación de una aplicación lógica basada en un solo inquilino" lightbox="./media/edit-app-settings-host-settings/portal-app-settings-values.png":::.

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para revisar la configuración actual de la aplicación mediante la CLI de Azure, ejecute el comando `az logicapp config appsettings list`. Asegúrese de que el comando incluye los parámetros `--name -n` y `--resource-group -g`, por ejemplo:

```azurecli
az logicapp config appsettings list --name MyLogicApp --resource-group MyResourceGroup
```

Para obtener más información sobre esta configuración, revise la [guía de referencia de la configuración de aplicación disponible: local.settings.json](#reference-local-settings-json).

Para agregar o actualizar una configuración de aplicación mediante la CLI de Azure, ejecute el comando `az logicapp config appsettings set`. Asegúrese de que el comando incluye los parámetros `--name n` y `--resource-group -g`. Por ejemplo, el comando siguiente crea una configuración con una clave denominada `CUSTOM_LOGIC_APP_SETTING` con un valor de `12345`:

```azurecli
az logicapp config appsettings set --name MyLogicApp --resource-group MyResourceGroup --settings CUSTOM_LOGIC_APP_SETTING=12345 
```

---

<a name="reference-host-json"></a>

## <a name="reference-for-host-settings---hostjson"></a>Referencia de la configuración de host: host.json

En Visual Studio Code, en el nivel de raíz del proyecto de aplicación lógica, el archivo de metadatos **host.json** contiene la configuración del entorno de ejecución y los valores predeterminados que se aplican a *todos los flujos de trabajo* de un recurso de aplicación lógica, ya sea que se ejecuten localmente o en Azure. Para ver y administrar esta configuración, consulte [Administración de la configuración de host: host.json](#manage-host-settings). También puede encontrar información relacionada sobre los límites en la documentación [Límites y configuración de Azure Logic Apps](logic-apps-limits-and-config.md#definition-limits).

<a name="job-orchestration"></a>

### <a name="job-orchestration-throughput"></a>Rendimiento de la orquestación de trabajos

Esta configuración afecta al rendimiento y la capacidad de Azure Logic Apps de inquilino único para ejecutar operaciones de flujo de trabajo.

| Configuración | Valor predeterminado | Descripción |
|---------|---------------|-------------|
| `Jobs.BackgroundJobs.DispatchingWorkersPulseInterval` | `00:00:01` <br>(1 segundo) | Establece el intervalo para que los distribuidores de trabajos sondeen la cola de trabajos cuando el sondeo anterior no devuelve ningún trabajo. Los distribuidores de trabajos sondean la cola inmediatamente cuando el sondeo anterior devuelve un trabajo. |
| `Jobs.BackgroundJobs.NumWorkersPerProcessorCount` | Instancias de trabajo del distribuidor de `192` | Establece el número de *instancias de trabajo de distribuidor* o *distribuidores de trabajos* que debe tener por núcleo de procesador. Este valor afecta al número de ejecuciones de flujo de trabajo por núcleo. |
| `Jobs.BackgroundJobs.NumPartitionsInJobTriggersQueue` | Cola de trabajos de `1` | Establece el número de colas de trabajos supervisadas por los distribuidores de trabajos para que los procese. Este valor también afecta al número de particiones de almacenamiento donde existen colas de trabajo. |
| `Jobs.BackgroundJobs.NumPartitionsInJobDefinitionsTable` | Particiones de trabajo de `4` | Establece el número de particiones de trabajo en la tabla de definición de trabajo. Este valor controla cuánto rendimiento de ejecución se ve afectado por los límites de almacenamiento de particiones. |
||||

<a name="run-duration-history"></a>

### <a name="run-duration-and-history"></a>Historial y duración de ejecución

| Configuración | Valor predeterminado | Descripción |
|---------|---------------|-------------|
| `Runtime.FlowRetentionThreshold` | `90.00:00:00` <br>(90 días) | Establece la cantidad de tiempo para mantener el historial de ejecución de los flujos de trabajo después de que se inicie una ejecución. |
| `Runtime.Backend.FlowRunTimeout` | `90.00:00:00` <br>(90 días) | Establece la cantidad de tiempo que un flujo de trabajo puede seguir ejecutándose antes de forzar un tiempo de espera. <p><p>**Importante**: Asegúrese de que este valor es menor o igual que el valor de `Runtime.FlowRetentionThreshold`. De lo contrario, los historiales de ejecución podrían eliminarse antes de que se completen los trabajos asociados. |
||||

<a name="inputs-outputs"></a>

### <a name="inputs-and-outputs"></a>Entradas y salidas

| Configuración | Valor predeterminado | Descripción |
|---------|---------------|-------------|
| `Runtime.FlowRunActionJob.MaximumActionResultSize` | `209715200` bytes | Establece el tamaño máximo en bytes que pueden tener las entradas y salidas combinadas en una acción. |
| `Runtime.ContentLink.MaximumContentSizeInBytes` | `104857600` caracteres | Establece el tamaño máximo en caracteres que puede tener una entrada o salida en un desencadenador o una acción. |
||||

<a name="pagination"></a>

### <a name="pagination"></a>Paginación

| Configuración | Valor predeterminado | Descripción |
|---------|---------------|-------------|
| `Runtime.FlowRunRetryableActionJobCallback.MaximumPageCount` | `1000` páginas | Cuando se admite y habilita la paginación en una operación, establece el número máximo de páginas que se devolverán o procesarán en tiempo de ejecución. |
||||

<a name="chunking"></a>

### <a name="chunking"></a>Fragmentación

| Configuración | Valor predeterminado | Descripción |
|---------|---------------|-------------|
| `Runtime.FlowRunRetryableActionJobCallback.MaximumContentLengthInBytesForPartialContent` | `1073741824` bytes | Cuando se admite y habilita la fragmentación en una operación, establece el tamaño máximo en bytes para el contenido descargado o cargado. |
| `Runtime.FlowRunRetryableActionJobCallback.MaxChunkSizeInBytes` | `52428800` bytes | Cuando se admite y habilita la fragmentación en una operación, establece el tamaño máximo en bytes para cada fragmento de contenido. |
| `Runtime.FlowRunRetryableActionJobCallback.MaximumRequestCountForPartialContent` | `1000` solicitudes | Cuando se admite y habilita la fragmentación en una operación, establece el número máximo de solicitudes que puede realizar una ejecución de acción para descargar contenido. |
||||

<a name="trigger-concurrency"></a>

### <a name="trigger-concurrency"></a>Simultaneidad de desencadenadores

| Configuración | Valor predeterminado | Descripción |
|---------|---------------|-------------|
| `Runtime.Trigger.MaximumRunConcurrency` | `100` ejecuciones | Establece el número máximo de ejecuciones simultáneas que puede iniciar un desencadenador. Este valor aparece en la definición de simultaneidad del desencadenador. |
| `Runtime.Trigger.MaximumWaitingRuns` | `200` ejecuciones | Establece el número máximo de ejecuciones que pueden esperar después de que las ejecuciones simultáneas cumplan el máximo. Este valor aparece en la definición de simultaneidad del desencadenador. |
||||

<a name="for-each-loop"></a>

### <a name="for-each-loops"></a>Bucles Para cada uno

| Configuración | Valor predeterminado | Descripción |
|---------|---------------|-------------|
| `Runtime.Backend.FlowDefaultForeachItemsLimit` | `100000` <br>(100 000 elementos de matriz) | Para un *flujo de trabajo con estado*, establece el número máximo de elementos de matriz que se procesarán en un bucle `For each`. |
| `Runtime.Backend.Stateless.FlowDefaultForeachItemsLimit` | `100` elementos | Para un *flujo de trabajo sin estado*, establece el número máximo de elementos de matriz que se procesarán en un bucle `For each`. |
| `Runtime.Backend.ForeachDefaultDegreeOfParallelism` | `20` iteraciones | Establece el número predeterminado de iteraciones simultáneas, o grado de paralelismo, en un bucle `For each`. Para que se ejecuten secuencialmente, establezca el valor en `1`. |
| `Runtime.Backend.FlowDefaultSplitOnItemsLimit` | `100000` <br>(100 000 elementos de matriz) | Establece el número máximo de elementos de matriz que se quitarán del lote o dividirán en varias instancias de flujo de trabajo según la configuración de `SplitOn`. |
||||

<a name="until-loop"></a>

### <a name="until-loops"></a>Bucles Until

| Configuración | Valor predeterminado | Descripción |
|---------|---------------|-------------|
| `Runtime.Backend.MaximumUntilLimitCount` | `5000` iteraciones | Para un *flujo de trabajo con estado*, establece el número máximo posible para la propiedad `Count` en una acción `Until`. |
| `Runtime.Backend.Stateless.MaximumUntilLimitCount` | `100` iteraciones | Para un *flujo de trabajo sin estado*, establece el número máximo posible para la propiedad `Count` en una acción `Until`. |
| `Runtime.Backend.Stateless.FlowRunTimeout` | `00:05:00` <br>(5 minutos) | Establece el tiempo de espera máximo para un bucle `Until` en un flujo de trabajo sin estado. |
||||

<a name="variables"></a>

### <a name="variables"></a>variables

| Configuración | Valor predeterminado | Descripción |
|---------|---------------|-------------|
| `Runtime.Backend.DefaultAppendArrayItemsLimit` | `100000` <br>(100 000 elementos de matriz) | Establece el número máximo de elementos de una variable con el tipo Array. |
| `Runtime.Backend.VariableOperation.MaximumVariableSize` | Flujo de trabajo con estado: `104857600` caracteres <p><p>Flujo de trabajo sin estado: `1024` caracteres | Establece el tamaño máximo en caracteres para el contenido que puede almacenar una variable. |
||||

<a name="http-webhook"></a>

### <a name="http-operations"></a>Operaciones HTTP

| Configuración | Valor predeterminado | Descripción |
|---------|---------------|-------------|
| `Runtime.Backend.HttpOperation.RequestTimeout` | `00:03:45` <br>(3 minutos y 45 segundos) | Establece el valor de tiempo de espera de la solicitud para desencadenadores y acciones HTTP. |
| `Runtime.Backend.HttpOperation.MaxContentSize` | `104857600` bytes | Establece el tamaño máximo de la solicitud en bytes para desencadenadores y acciones HTTP. |
| `Runtime.Backend.HttpOperation.DefaultRetryCount` | `4` reintentos | Establece el recuento de reintentos predeterminado para desencadenadores y acciones HTTP. |
| `Runtime.Backend.HttpOperation.DefaultRetryInterval` | `00:00:07` <br>(7 segundos) | Establece el recuento de reintentos predeterminado para desencadenadores y acciones HTTP. |
| `Runtime.Backend.HttpOperation.DefaultRetryMaximumInterval` | `01:00:00` <br>(1 hora) | Establece el recuento de reintentos predeterminado para desencadenadores y acciones HTTP. |
| `Runtime.Backend.HttpOperation.DefaultRetryMinimumInterval` | `00:00:05` <br>(5 segundos) | Establece el recuento de reintentos predeterminado para desencadenadores y acciones HTTP. |
||||

<a name="http-webhook"></a>

### <a name="http-webhook-operations"></a>Operaciones de webhook

| Configuración | Valor predeterminado | Descripción |
|---------|---------------|-------------|
| `Runtime.Backend.HttpWebhookOperation.RequestTimeout` | `00:02:00` <br>(2 minutos) | Establece el valor de tiempo de espera de la solicitud para acciones y desencadenadores de webhook HTTP. |
| `Runtime.Backend.HttpWebhookOperation.MaxContentSize` | `104857600` bytes | Establece el tamaño máximo de la solicitud en bytes para acciones y desencadenadores de webhook HTTP. |
| `Runtime.Backend.HttpWebhookOperation.DefaultRetryCount` | `4` reintentos | Establece el recuento de reintentos predeterminado para acciones y desencadenadores de webhook HTTP. |
| `Runtime.Backend.HttpWebhookOperation.DefaultRetryInterval` | `00:00:07` <br>(7 segundos) | Establece el intervalo de reintentos predeterminado para acciones y desencadenadores de webhook HTTP. |
| `Runtime.Backend.HttpWebhookOperation.DefaultRetryMaximumInterval` | `01:00:00` <br>(1 hora) | Establece el intervalo máximo de reintentos para acciones y desencadenadores de webhook HTTP. |
| `Runtime.Backend.HttpWebhookOperation.DefaultRetryMinimumInterval` | `00:00:05` <br>(5 segundos) | Establece el intervalo mínimo de reintentos para acciones y desencadenadores de webhook HTTP. |
| `Runtime.Backend.HttpWebhookOperation.DefaultWakeUpInterval` | `01:00:00` <br>(1 hora) | Establece el intervalo de reactivación predeterminado para los trabajos de acción y desencadenador de webhook HTTP. |
||||

<a name="built-in-azure-functions"></a>

### <a name="built-in-azure-functions-operations"></a>Operaciones de Azure Functions integradas

| Configuración | Valor predeterminado | Descripción |
|---------|---------------|-------------|
| `Runtime.Backend.FunctionOperation.RequestTimeout` | `00:03:45` <br>(3 minutos y 45 segundos) | Establece el valor de tiempo de espera de la solicitud para acciones de Azure Functions. |
| `Runtime.Backend.FunctionOperation.MaxContentSize` | `104857600` bytes | Establece el tamaño máximo de solicitud en bytes para acciones de Azure Functions. |
| `Runtime.Backend.FunctionOperation.DefaultRetryCount` | `4` reintentos | Establece el número de reintentos predeterminado para acciones de Azure Functions. |
| `Runtime.Backend.FunctionOperation.DefaultRetryInterval` | `00:00:07` <br>(7 segundos) | Establece el intervalo de reintentos predeterminado para acciones de Azure Functions. |
| `Runtime.Backend.FunctionOperation.DefaultRetryMaximumInterval` | `01:00:00` <br>(1 hora) | Establece el intervalo máximo de reintentos para acciones de Azure Functions. |
| `Runtime.Backend.FunctionOperation.DefaultRetryMinimumInterval` | `00:00:05` <br>(5 segundos) | Establece el intervalo mínimo de reintentos para acciones de Azure Functions. |
||||

<a name="built-in-sql"></a>

### <a name="built-in-sql-operations"></a>Operaciones de SQL integradas

| Configuración | Valor predeterminado | Descripción |
|---------|---------------|-------------|
| `Runtime.ServiceProviders.Sql.QueryExecutionTimeout` | `00:00:30` <br>(30 segundos) | Establece el valor de tiempo de espera de solicitud para las operaciones del proveedor de servicios de SQL. |
||||

<a name="built-in-service-bus"></a>

### <a name="built-in-azure-service-bus-operations"></a>Operaciones de Azure Service Bus integradas

| Configuración | Valor predeterminado | Descripción |
|---------|---------------|-------------|
| `Runtime.ServiceProviders.ServiceBus.MessageSenderPoolSizePerProcessorCount` | `64` remitentes de mensajes | Establece el número de remitentes de mensajes de Azure Service Bus por núcleo de procesador que se usarán en el grupo de remitentes de mensajes. |
||||

<a name="managed-api-connector"></a>

### <a name="managed-api-connector-operations"></a>Operaciones del conector de API administradas

| Configuración | Valor predeterminado | Descripción |
|---------|---------------|-------------|
| `Runtime.Backend.ApiConnectionOperation.RequestTimeout` | `00:02:00` <br>(2 minutos) | Establece el valor de tiempo de espera de la solicitud para los desencadenadores y las acciones del conector de API administradas. |
| `Runtime.Backend.ApiConnectionOperation.MaxContentSize` | `104857600` bytes | Establece el tamaño máximo de solicitud en bytes para los desencadenadores y las acciones del conector de API administradas. |
| `Runtime.Backend.ApiConnectionOperation.DefaultRetryCount` | `4` reintentos | Establece el recuento de reintentos predeterminado para los desencadenadores y las acciones del conector de API administradas. |
| `Runtime.Backend.ApiConnectionOperation.DefaultRetryInterval` | `00:00:07` <br>(7 segundos) | Establece el intervalo de reintentos predeterminado para los desencadenadores y las acciones del conector de API administradas. |
| `Runtime.Backend.ApiWebhookOperation.DefaultRetryMaximumInterval` | `01:00:00` <br>(1 día) | Establece el intervalo máximo de reintentos para los desencadenadores y las acciones del conector de API administradas. |
| `Runtime.Backend.ApiConnectionOperation.DefaultRetryMinimumInterval` | `00:00:05` <br>(5 segundos) | Establece el intervalo mínimo de reintentos para los desencadenadores y las acciones del conector de API administradas. |
| `Runtime.Backend.ApiWebhookOperation.DefaultWakeUpInterval` | `01:00:00` <br>(1 día) | Establece el intervalo de reactivación predeterminado para los trabajos de acción y desencadenador de webhook del conector de API administradas. |
||||

<a name="blob-storage"></a>

### <a name="blob-storage"></a>Blob Storage

| Configuración | Valor predeterminado | Descripción |
|---------|---------------|-------------|
| `Runtime.ContentStorage.RequestOptionsServerTimeout` | `00:00:30` <br>(30 segundos) | Establece el valor de tiempo de espera para las solicitudes de blob del entorno de ejecución de Azure Logic Apps. |
| `Runtime.DataStorage.RequestOptionsMaximumExecutionTime` | `00:02:00` <br>(2 minutos) | Establece el valor de tiempo de espera de la operación, incluidos los reintentos, para las solicitudes de Table Storage y Queue Storage del entorno de ejecución de Azure Logic Apps. |
| `Runtime.ContentStorage.RequestOptionsDeltaBackoff` | `00:00:02` <br>(2 segundos) | Establece el intervalo de retroceso entre los reintentos enviados a Blob Storage. |
| `Runtime.ContentStorage.RequestOptionsMaximumAttempts` | `4` reintentos | Establece el número máximo de reintentos enviados a Table Storage y Queue Storage. |
||||

<a name="store-inline-or-blob"></a>

### <a name="store-content-inline-or-use-blobs"></a>Almacenamiento de contenido en línea o uso de blobs

| Configuración | Valor predeterminado | Descripción |
|---------|---------------|-------------|
| `Runtime.FlowRunEngine.ForeachMaximumItemsForContentInlining` | `20` elementos | Cuando se ejecuta un bucle `For each`, el valor de cada elemento se almacena en línea con otros metadatos en Table Storage o por separado en Blob Storage. Establece el número de elementos que se almacenarán en línea con otros metadatos. |
| `Runtime.FlowRunRetryableActionJobCallback.MaximumPagesForContentInlining` | `20` páginas | Establece el número máximo de páginas que se almacenarán como contenido en línea en Table Storage antes de almacenarlas en Blob Storage. |
| `Runtime.FlowTriggerSplitOnJob.MaximumItemsForContentInlining` | `40` elementos | Cuando la configuración `SplitOn` quita del lote los elementos de matriz en varias instancias de flujo de trabajo, el valor de cada elemento se almacena en línea con otros metadatos en Table Storage o por separado en Blob Storage. Establece el número de elementos que se almacenarán en línea. |
| `Runtime.ScaleUnit.MaximumCharactersForContentInlining` | `8192` caracteres | Establece el número máximo de caracteres de entrada y salida de la operación que se almacenarán en línea en Table Storage antes de almacenarlos en Blob Storage. |
||||

<a name="table-queue-storage"></a>

### <a name="table-and-queue-storage"></a>Table Storage y Queue Storage

| Configuración | Valor predeterminado | Descripción |
|---------|---------------|-------------|
| `Runtime.DataStorage.RequestOptionsServerTimeout` | `00:00:16` <br>(16 segundos) | Establece el valor de tiempo de espera para las solicitudes de Table Storage y Queue Storage del entorno de ejecución de Azure Logic Apps. |
| `Runtime.DataStorage.RequestOptionsMaximumExecutionTime` | `00:00:45` <br>(45 segundos) | Establece el valor de tiempo de espera de la operación, incluidos los reintentos, para las solicitudes de Table Storage y Queue Storage del entorno de ejecución de Azure Logic Apps. |
| `Runtime.DataStorage.RequestOptionsDeltaBackoff` | `00:00:02` <br>(2 segundos) | Establece el intervalo de retroceso entre los reintentos enviados a Table Storage y Queue Storage. |
| `Runtime.DataStorage.RequestOptionsMaximumAttempts` | `4` reintentos | Establece el número máximo de reintentos enviados a Table Storage y Queue Storage. |
||||

<a name="retry-policy"></a>

### <a name="retry-policy-for-all-other-operations"></a>Directiva de reintentos para todas las demás operaciones

| Configuración | Valor predeterminado | Descripción |
|---------|---------------|-------------|
| `Runtime.ScaleMonitor.MaxPollingLatency` | `00:00:30` <br>(30 segundos) | Establece la latencia máxima de sondeo para el escalado en tiempo de ejecución. |
| `Runtime.Backend.Operation.MaximumRetryCount` | `90` reintentos | Establece el número máximo de reintentos en la definición de directiva de reintentos para una operación de flujo de trabajo. |
| `Runtime.Backend.Operation.MaximumRetryInterval` | `01:00:00:01` <br>(1 día y 1 segundo) | Establece el intervalo máximo en la definición de directiva de reintentos para una operación de flujo de trabajo. |
| `Runtime.Backend.Operation.MinimumRetryInterval` | `00:00:05` <br>(5 segundos) | Establece el intervalo mínimo en la definición de directiva de reintentos para una operación de flujo de trabajo. |
||||

<a name="manage-host-settings"></a>

## <a name="manage-host-settings---hostjson"></a>Administración de la configuración de host: host.json

Puede agregar, actualizar o eliminar la configuración de host, que especifica los valores y la configuración del entorno de ejecución que se aplican a *todos los flujos de trabajo* de esa aplicación lógica, por ejemplo, los valores predeterminados para el rendimiento, la capacidad, el tamaño de los datos, entre otros, *tanto si se ejecutan localmente como en Azure*. Para la configuración de host específica de las aplicaciones lógicas, revise la [guía de referencia de la configuración de implementación y de entorno de ejecución disponible: host.json](#reference-host-json).

### <a name="visual-studio-code---hostjson"></a>Visual Studio Code: host.json

Para revisar la configuración de host de la aplicación lógica en Visual Studio Code, siga estos pasos:

1. En el proyecto de aplicación lógica, en el nivel de raíz del proyecto, busque y abra el archivo **host.json**.

1. En el objeto `extensions`, en `workflows` y `settings`, revise los valores de configuración de host que se hayan agregado anteriormente para la aplicación lógica. De lo contrario, el objeto `extensions` no aparecerá en el archivo.

   Para obtener más información sobre la configuración de host, revise la [guía de referencia de la configuración de host disponible: host.json](#reference-host-json).

Para agregar una configuración de host, siga estos pasos:

1. En el archivo **host.json**, en el objeto `extensionBundle`, agregue el objeto `extensions`, que incluye los objetos `workflow` y `settings`, por ejemplo:

   ```json
   {
      "version": "2.0",
      "extensionBundle": {
         "id": "Microsoft.Azure.Functions.ExtensionBundle",
         "version": "[1.*, 2.0.0)"
      },
      "extensions": {
         "workflow": {
            "settings": {
            }
         }
      }
   }
   ```

1. En el objeto `settings`, agregue una lista plana con la configuración de host que desea usar para todos los flujos de trabajo de la aplicación lógica, tanto si esos flujos de trabajo se ejecutan localmente como en Azure, por ejemplo:

   ```json
   {
      "version": "2.0",
      "extensionBundle": {
         "id": "Microsoft.Azure.Functions.ExtensionBundle",
         "version": "[1.*, 2.0.0)"
      },
      "extensions": {
         "workflow": {
            "settings": {
               "Runtime.Trigger.MaximumWaitingRuns": "100"
            }
         }
      }
   }
   ```

### <a name="azure-portal---hostjson"></a>Azure Portal: host.json

Para revisar la configuración de host de la aplicación lógica basada en inquilino único en Azure Portal, siga estos pasos:

1. En el cuadro de búsqueda de [Azure Portal](https://portal.azure.com/), busque y abra la aplicación lógica.

1. En el menú de la aplicación lógica, en **Herramientas de desarrollo**, seleccione **Herramientas avanzadas**.

1. En la página **Herramientas avanzadas**, seleccione **Ir**, que abre el entorno de **Kudu** para la aplicación lógica.

1. En la barra de herramientas de Kudu, abra el menú **Consola de depuración** y seleccione **CMD**.

1. Detenga la aplicación lógica en Azure Portal.

   1. En el menú de la aplicación lógica, seleccione **Introducción**.

   1. En la barra de herramientas del panel **Información general**, seleccione **Detener**.

1. En el menú de la aplicación lógica, en **Herramientas de desarrollo**, seleccione **Herramientas avanzadas**.

1. En el panel **Herramientas avanzadas**, seleccione **Ir**, que abre el entorno de Kudu para la aplicación lógica.

1. En la barra de herramientas de Kudu, abra el menú **Consola de depuración** y seleccione **CMD**.

   Se abre una ventana de consola, por lo que puede desplazarse a la carpeta **wwwroot** mediante el símbolo del sistema. O bien, puede examinar la estructura de directorios que aparece encima de la ventana de la consola.

1. Examine la siguiente ruta de acceso a la carpeta **wwwroot**: `...\home\site\wwwroot`.

1. Encima de la ventana de la consola, en la tabla de directorios, junto al archivo **host.json**, seleccione **Editar**.

1. Una vez abierto el archivo **host.json**, revise los valores de configuración de host que se hayan agregado anteriormente para la aplicación lógica.

   Para obtener más información sobre la configuración de host, revise la [guía de referencia de la configuración de host disponible: host.json](#reference-host-json).

Para agregar una configuración, siga estos pasos:

1. Antes de agregar o editar la configuración, detenga la aplicación lógica en Azure Portal.

   1. En el menú de la aplicación lógica, seleccione **Introducción**.
   1. En la barra de herramientas del panel **Información general**, seleccione **Detener**.

1. Vuelva al archivo **host.json**. En el objeto `extensionBundle`, agregue el objeto `extensions`, que incluye los objetos `workflow` y `settings`, por ejemplo:

   ```json
   {
      "version": "2.0",
      "extensionBundle": {
         "id": "Microsoft.Azure.Functions.ExtensionBundle",
         "version": "[1.*, 2.0.0)"
      },
      "extensions": {
         "workflow": {
            "settings": {
            }
         }
      }
   }
   ```

1. En el objeto `settings`, agregue una lista plana con la configuración de host que desea usar para todos los flujos de trabajo de la aplicación lógica, tanto si esos flujos de trabajo se ejecutan localmente como en Azure, por ejemplo:

   ```json
   {
      "version": "2.0",
      "extensionBundle": {
         "id": "Microsoft.Azure.Functions.ExtensionBundle",
         "version": "[1.*, 2.0.0)"
      },
      "extensions": {
         "workflow": {
            "settings": {
               "Runtime.Trigger.MaximumWaitingRuns": "100"
            }
         }
      }
   }
   ```

1. Cuando haya terminado, recuerde seleccionar la opción **Guardar**.

1. Ahora, reinicie la aplicación lógica. Vuelva a la página **Información general** de la aplicación lógica y seleccione **Reiniciar**.

---

## <a name="next-steps"></a>Pasos siguientes

- [Creación de parámetros para los valores que cambian en los flujos de trabajo entre entornos de Azure Logic Apps de inquilino único](parameterize-workflow-app.md)
- [Configuración de la implementación de DevOps para Azure Logic Apps de inquilino único](set-up-devops-deployment-single-tenant-azure-logic-apps.md)
