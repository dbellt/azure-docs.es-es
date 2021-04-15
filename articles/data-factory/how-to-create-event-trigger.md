---
title: Creación de desencadenadores basados en eventos en Azure Data Factory
description: Aprenda a crear un desencadenador en Azure Data Factory que ejecute una canalización en respuesta a un evento.
ms.service: data-factory
author: chez-charlie
ms.author: chez
ms.reviewer: jburchel
ms.topic: conceptual
ms.date: 03/11/2021
ms.openlocfilehash: 3021d049a38f1d883518fc7c45aa8ca0a906c2f7
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/02/2021
ms.locfileid: "106221592"
---
# <a name="create-a-trigger-that-runs-a-pipeline-in-response-to-a-storage-event"></a>Creación de un desencadenador que ejecuta una canalización en respuesta a un evento de almacenamiento

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se describen los desencadenadores de eventos de almacenamiento que se pueden crear en las canalizaciones de Data Factory.

La arquitectura dirigida por eventos (EDA) es un patrón de integración de datos común que implica la producción, detección, consumo y reacción a los eventos. Los escenarios de integración de datos a menudo requieren que los clientes de Data Factory desencadenen canalizaciones basadas en los eventos que se producen en la cuenta de almacenamiento, como la llegada o la eliminación de un archivo en la cuenta de Azure Blob Storage. Data Factory se integra de forma nativa con [Azure Event Grid](https://azure.microsoft.com/services/event-grid/), que permite desencadenar canalizaciones en estos eventos.

Si desea ver una demostración y una introducción de diez minutos de esta característica, vea el siguiente vídeo de nueve minutos de duración:

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Event-based-data-integration-with-Azure-Data-Factory/player]

> [!NOTE]
> La integración descrita en este artículo depende de [Azure Event Grid](https://azure.microsoft.com/services/event-grid/). Asegúrese de que el proveedor de la suscripción se registra con el proveedor de recursos de Event Grid. Para más información, consulte [Tipos y proveedores de recursos](../azure-resource-manager/management/resource-providers-and-types.md#azure-portal). Debe poder realizar la acción *Microsoft.EventGrid/eventSubscriptions/* *. Esta acción forma parte del rol integrado Colaborador de EventSubscription EventGrid.

## <a name="data-factory-ui"></a>Interfaz de usuario de Data Factory

En esta sección se muestra cómo crear un desencadenador de eventos de almacenamiento dentro de la interfaz de usuario de Azure Data Factory.

1. Cambie a la pestaña **Edit** (Editar), que se muestra con un símbolo de lápiz.

1. Seleccione **Trigger** (Desencadenador) en el menú y, después, seleccione **New/Edit** (Nuevo/Editar).

1. En la página **Add Triggers** (Agregar desencadenadores), seleccione **Choose trigger...** (Elegir desencadenador) y, después, seleccione **+New** (+Nuevo).

1. Seleccione el tipo de desencadenador **Evento de almacenamiento**.

    :::image type="content" source="media/how-to-create-event-trigger/event-based-trigger-image1.png" alt-text="Captura de pantalla de la página de autor para crear un nuevo desencadenador de eventos de almacenamiento en la interfaz de usuario de Data Factory.":::

1. Seleccione la cuenta de almacenamiento en la lista desplegable de suscripción de Azure o manualmente con el identificador de recurso de la cuenta de almacenamiento. Elija en qué contenedor quiere que se produzcan los eventos. La selección de contenedores es obligatoria, pero tenga en cuenta que si se seleccionan todos los contenedores el número de eventos puede resultar grande.

   > [!NOTE]
   > Actualmente, el desencadenador de eventos de almacenamiento solo es compatible con las cuentas de almacenamiento de Azure Data Lake Storage Gen2 y de la versión 2 de uso general. Debido a una limitación de Azure Event Grid, Azure Data Factory solo admite un máximo de 500 desencadenadores de eventos de almacenamiento por cuenta de almacenamiento. Si alcanza el límite, póngase en contacto con el servicio de soporte técnico para recomendaciones y aumentar el límite tras la evaluación del equipo de Event Grid. 

   > [!NOTE]
   > Para crear un desencadenador de eventos de almacenamiento o modificar uno existente, la cuenta de Azure usada para iniciar sesión en Data Factory y publicar el desencadenador de evento de almacenamiento debe tener el permiso control de acceso basado en rol (Azure RBAC) adecuado en la cuenta de almacenamiento. No se requiere ningún permiso adicional: la entidad de servicio para Azure Data Factory _no_ necesita permisos especiales para la cuenta de Storage o Event Grid. Para obtener más información sobre el control de acceso, consulte [Control de acceso basado en roles](#role-based-access-control).

1. Las propiedades **Blob path begins with** y **Blob path ends with** permiten especificar los contenedores, las carpetas y los nombres de blob en los que quiere recibir eventos. El desencadenador de eventos de almacenamiento requiere que se defina al menos una de estas propiedades. Como se muestra en ejemplos que podrá encontrar en este mismo artículo, se pueden usar varios patrones para las propiedades **Blob path begins with** y **Blob path ends with**.

    * **Blob path begins with:** la ruta de acceso del blob debe comenzar con una ruta de acceso de carpeta. Entre los valores válidos se incluyen `2018/` y `2018/april/shoes.csv`. No se puede seleccionar este campo si no se ha seleccionado un contenedor.
    * **Blob path ends with:** la ruta de acceso del blob debe terminar con un nombre de archivo o una extensión. Entre los valores válidos se incluyen `shoes.csv` y `.csv`. Cuando se especifiquen, los nombres del contenedor y de la carpeta deben estar separados por un segmento `/blobs/`. Por ejemplo, un contenedor denominado "Orders" puede tener un valor de `/orders/blobs/2018/april/shoes.csv`. Para especificar una carpeta en cualquier contenedor, omita el carácter "/" inicial. Por ejemplo, `april/shoes.csv` desencadenará un evento en cualquier archivo denominado `shoes.csv` en la carpeta a denominada "April" en cualquier contenedor.
    * Tenga en cuenta que la ruta de acceso de blob que **comienza con** y **termina con son** las únicas coincidencias de patrones permitidas en el desencadenador de eventos de almacenamiento. No se admiten otros tipos de coincidencia de caracteres comodín para el tipo de desencadenador.

1. Seleccione si el desencadenador responderá a un evento **Blob creado**, **Blob eliminado** o a ambos. En la ubicación de almacenamiento especificada, cada evento desencadenará las canalizaciones Data Factory asociadas al desencadenador.

    :::image type="content" source="media/how-to-create-event-trigger/event-based-trigger-image2.png" alt-text="Captura de pantalla de la página de creación del desencadenador de eventos de almacenamiento.":::

1. Seleccione si el desencadenador omitirá o no los blobs con cero bytes.

1. Después de configurar el desencadenador, haga clic en **Siguiente: versión preliminar de datos**. Esta pantalla muestra los blobs existentes que coinciden con la configuración del desencadenador de eventos de almacenamiento. Asegúrese de que tiene filtros específicos. La configuración de filtros demasiado amplios puede coincidir con un gran número de archivos creados o eliminados y puede afectar significativamente al costo. Una vez comprobadas las condiciones de filtro, haga clic en **Finalizar**.

    :::image type="content" source="media/how-to-create-event-trigger/event-based-trigger-image3.png" alt-text="Captura de pantalla de la página de la versión preliminar del desencadenador de eventos de almacenamiento.":::

1. Para adjuntar una canalización a este desencadenador, vaya al lienzo de canalización, haga clic en **Trigger** (Desencadenador) y seleccione **New/Edit** (Nuevo/editar). Cuando aparezca la barra de navegación lateral, haga clic en el desplegable **Choose trigger** (Elegir desencadenador...) y seleccione el desencadenador que creó. Haga clic en **Siguiente: Vista previa de los datos** para confirmar que la configuración es correcta y, después, **Siguiente** para validar que la vista previa de los datos es correcta.

1. Si la canalización tiene parámetros, puede especificarlos en el parámetro de ejecución del desencadenador. El desencadenador de eventos de almacenamiento captura el nombre de archivo y la ruta de acceso del blob en las propiedades `@triggerBody().folderPath` y `@triggerBody().fileName`. Para usar los valores de estas propiedades en una canalización, debe asignar las propiedades a los parámetros de la canalización. Después de asignar las propiedades a los parámetros, puede tener acceso a los valores capturados por el desencadenador mediante la expresión `@pipeline().parameters.parameterName` en toda la canalización. Para obtener una explicación detallada, consulte [Referencia de metadatos de desencadenador en Pipelines](how-to-use-trigger-parameterization.md)

   :::image type="content" source="media/how-to-create-event-trigger/event-based-trigger-image4.png" alt-text="Captura de pantalla de la asignación de propiedades del desencadenador de eventos de almacenamiento a parámetros de canalización.":::

   En el ejemplo anterior, el desencadenador está configurado para activarse cuando se crea una ruta de acceso de blob que termina en .csv en la carpeta _prueba-de-eventos_ en el contenedor _datos-de-ejemplo_. Las propiedades **folderPath** y **fileName** capturan la ubicación del nuevo blob. Por ejemplo, cuando se agrega MoviesDB.csv a la ruta de acceso datos-de-ejemplo/prueba-de-eventos, `@triggerBody().folderPath` tiene un valor de `sample-data/event-testing` y `@triggerBody().fileName` tiene un valor de `moviesDB.csv`. Estos valores se asignan en el ejemplo a los parámetros de canalización `sourceFolder` y `sourceFile`, que se pueden utilizar en toda la canalización como `@pipeline().parameters.sourceFolder` y `@pipeline().parameters.sourceFile` respectivamente.

   > [!NOTE]
   > Si va a crear la canalización y el desencadenador en [Azure Synapse Analytics](../synapse-analytics/overview-what-is.md), debe usar `@trigger().outputs.body.fileName` y `@trigger().outputs.body.folderPath` como parámetros. Esas dos propiedades capturan información del blob. Utilice esas propiedades en lugar de usar `@triggerBody().fileName` y `@triggerBody().folderPath`.

1. Cuando haya terminado, haga clic en **Finalizar**.

## <a name="json-schema"></a>Esquema JSON

En la tabla siguiente se proporciona información general acerca de los elementos de esquema que están relacionados con los desencadenadores de eventos de almacenamiento:

| **Elemento de JSON** | **Descripción** | **Tipo** | **Valores permitidos** | **Obligatorio** |
| ---------------- | --------------- | -------- | ------------------ | ------------ |
| **scope** | El identificador de recursos de Azure Resource Manager de la cuenta de almacenamiento. | String | Identificador de Azure Resource Manager | Sí |
| **eventos** | El tipo de eventos que provocan la activación de este desencadenador. | Array    | Microsoft.Storage.BlobCreated, Microsoft.Storage.BlobDeleted | Sí, cualquier combinación de estos valores. |
| **blobPathBeginsWith** | La ruta de acceso del blob debe comenzar con el patrón proporcionado para que se active el desencadenador. Por ejemplo, `/records/blobs/december/` solo activa el desencadenador de blobs en la carpeta `december` bajo el contenedor `records`. | String   | | Proporcione un valor para al menos una de estas propiedades: `blobPathBeginsWith` o `blobPathEndsWith`. |
| **blobPathEndsWith** | La ruta de acceso del blob debe finalizar con el patrón proporcionado para que se active el desencadenador. Por ejemplo, `december/boxes.csv` solo activa el desencadenador de blobs denominado "`boxes`" en una carpeta `december`. | String   | | Tendrá que proporcionar un valor para al menos una de estas propiedades: `blobPathBeginsWith` o `blobPathEndsWith`. |
| **ignoreEmptyBlobs** | Indica si los blobs de cero bytes desencadenarán o no la ejecución de una canalización. De manera predeterminada, se establece en true. | Boolean | true o false | No |

## <a name="examples-of-storage-event-triggers"></a>Ejemplos de desencadenadores de eventos de almacenamiento

En esta sección encontrará ejemplos de configuración de desencadenadores de eventos de almacenamiento.

> [!IMPORTANT]
> Debe incluir el segmento `/blobs/` de la ruta de acceso, tal como se muestra en los siguientes ejemplos, siempre que especifique el contenedor y la carpeta, el contenedor y el archivo, o el contenedor, la carpeta y el archivo. En el caso de **blobPathBeginsWith**, la interfaz de usuario de Data Factory agregará automáticamente `/blobs/` entre la carpeta y el nombre del contenedor en el JSON del desencadenador.

| Propiedad | Ejemplo | Descripción |
|---|---|---|
| **Ruta de acceso de blobs que empieza con** | `/containername/` | Recibe eventos de cualquier blob del contenedor. |
| **Ruta de acceso de blobs que empieza con** | `/containername/blobs/foldername/` | Recibe eventos de los blobs en el contenedor `containername` y la carpeta `foldername`. |
| **Ruta de acceso de blobs que empieza con** | `/containername/blobs/foldername/subfoldername/` | También puede hacer referencia a una subcarpeta. |
| **Ruta de acceso de blobs que empieza con** | `/containername/blobs/foldername/file.txt` | Recibe eventos de un blob denominado `file.txt` en la carpeta `foldername`, en el contenedor `containername`. |
| **Ruta de acceso de blobs que termina con** | `file.txt` | Recibe eventos para un blob denominado `file.txt` en cualquier ruta de acceso. |
| **Ruta de acceso de blobs que termina con** | `/containername/blobs/file.txt` | Recibe eventos de un blob denominado `file.txt` en el contenedor `containername`. |
| **Ruta de acceso de blobs que termina con** | `foldername/file.txt` | Recibe eventos de un blob denominado `file.txt` en la carpeta `foldername` en cualquier contenedor. |

## <a name="role-based-access-control"></a>Control de acceso basado en rol

Azure Data Factory usa el control de acceso basado en roles de Azure (RBAC de Azure) para asegurarse de que el acceso no autorizado a la escucha, la suscripción a las actualizaciones de y el desencadenamiento de canalizaciones vinculadas a eventos de blob, están estrictamente prohibidos.

* Para crear correctamente un desencadenador de evento de almacenamiento existente o actualizarlo, la cuenta de Azure con la sesión iniciada en Data Factory debe tener el acceso adecuado a la cuenta de almacenamiento pertinente. De lo contrario, se _Deniega el acceso_ a la operación con error.
* Data Factory no necesita ningún permiso especial en el Event Grid y _no_ es necesario asignar un permiso de RBAC especial a Data Factory entidad de servicio para la operación.

Cualquiera de las siguientes configuraciones de RBAC funciona para el desencadenador de eventos de almacenamiento:

* Rol de propietario en la cuenta de almacenamiento
* Rol de colaborador de la cuenta de almacenamiento
* _Microsoft.EventGrid/EventSubscriptions/Write_ Permiso para la cuenta de almacenamiento _/subscriptions/####/resourceGroups/####/providers/Microsoft.Storage/storageAccounts/storageAccountName_

Para entender cómo Azure Data Factory ofrece las dos promesas, volvamos un paso y echemos un vistazo a la escena. Estos son los flujos de trabajo de alto nivel para la integración entre Data Factory, Almacenamiento y Event Grid.

### <a name="create-a-new-storage-event-trigger"></a>Cree un desencadenador de eventos de almacenamiento

Este flujo de trabajo de alto nivel describe la forma en que Azure Data Factory interactúa con Event Grid para crear un desencadenador de evento de almacenamiento

:::image type="content" source="media/how-to-create-event-trigger/storage-event-trigger-5-create-subscription.png" alt-text="Flujo de trabajo de creación del desencadenador de eventos de almacenamiento.":::

Dos llamadas perceptibles desde los flujos de trabajo:

* Azure Data Factory _no_ realiza contacto directo con la cuenta de almacenamiento. En su lugar, Event Grid transmite y procesa la solicitud para crear una suscripción. Por lo tanto, Data Factory no necesita permisos para la cuenta de almacenamiento para este paso.

* El control de acceso y la comprobación de permisos se producen en el lado de Azure Data Factory. Antes de que ADF envíe una solicitud para suscribirse al evento de almacenamiento, comprueba el permiso para el usuario. Más específicamente, comprueba si la cuenta de Azure que inició sesión e intentó crear el desencadenador de evento de almacenamiento tiene el acceso adecuado a la cuenta de almacenamiento relevante. Si se produce un error en la comprobación de permisos, la creación del desencadenador también produce un error.

### <a name="storage-event-trigger-data-factory-pipeline-run"></a>Ejecución de canalización Data Factory del desencadenador de eventos de almacenamiento

Estos flujos de trabajo de alto nivel describen cómo se ejecutan las canalizaciones de los desencadenadores de eventos de almacenamiento a través de Event Grid

:::image type="content" source="media/how-to-create-event-trigger/storage-event-trigger-6-trigger-pipeline.png" alt-text="Flujo de trabajo de ejecuciones de canalización de desencadenamiento de eventos de almacenamiento.":::

Cuando se trata de una canalización de desencadenamiento de eventos en Data Factory, tres llamadas perceptibles en el flujo de trabajo:

* Event Grid usa un modelo de inserción que retransmite el mensaje lo antes posible cuando el almacenamiento lo coloca en el sistema. Esto es diferente del sistema de mensajería, como Kafka, donde se usa un sistema de extracción.
* El desencadenador de eventos en Azure Data Factory actúa como cliente de escucha activo para el mensaje entrante y desencadena correctamente la canalización asociada.
* El desencadenador de evento de almacenamiento no realiza contacto directo con la cuenta de almacenamiento

  * Dicho esto, si tiene una copia u otra actividad dentro de la canalización para procesar los datos de la cuenta de almacenamiento, Data Factory hará contacto directo con el almacenamiento mediante las credenciales almacenadas en el servicio vinculado. Asegúrese de que el servicio vinculado esté configurado correctamente
  * Sin embargo, si no hace referencia a la cuenta de almacenamiento en la canalización, no es necesario conceder permiso a Data Factory para acceder a la cuenta de almacenamiento

## <a name="next-steps"></a>Pasos siguientes

* Para obtener información detallada acerca de los desencadenadores, consulte el artículo [Pipeline execution and triggers](concepts-pipeline-execution-triggers.md#trigger-execution) (Ejecución de canalizaciones y desencadenadores).
* Aprenda a hacer referencia a los metadatos de desencadenador en la canalización; para ello, consulte [Referencia a los metadatos de desencadenador en las ejecuciones de canalización](how-to-use-trigger-parameterization.md).
