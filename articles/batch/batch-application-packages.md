---
title: Implementación de paquetes de aplicación en nodos de proceso
description: Utilice la característica paquetes de aplicación de Azure Batch para administrar fácilmente varias aplicaciones y versiones para la instalación en nodos de proceso de Batch.
ms.topic: how-to
ms.date: 03/24/2021
ms.custom:
- H1Hack27Feb2017
- devx-track-csharp
- contperf-fy21q1
ms.openlocfilehash: 17729b2b6a2c4faba0554339a382cad559e9894a
ms.sourcegitcommit: c3739cb161a6f39a9c3d1666ba5ee946e62a7ac3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107210506"
---
# <a name="deploy-applications-to-compute-nodes-with-batch-application-packages"></a>Implementación de aplicaciones en nodos de proceso con paquetes de aplicaciones de Batch

Los paquetes de aplicación pueden simplificar el código de la solución de Azure Batch y facilitar la administración de las aplicaciones que las tareas ejecutan. Con paquetes de aplicación, puede cargar y administrar varias versiones de las aplicaciones que las tareas ejecutan, incluidos los archivos auxiliares. A continuación, se pueden implementar automáticamente una o varias de estas aplicaciones en los nodos de proceso del grupo.

Las API para crear y administrar paquetes de aplicaciones forman parte de la biblioteca [Batch Management .NET](/dotnet/api/overview/azure/batch/management). Las API para la instalación de paquetes de aplicaciones en un nodo de proceso forman parte de la biblioteca de [Batch para .NET](/dotnet/api/overview/azure/batch/client). Las características comparables están en las API de Batch disponibles para otros idiomas.

En este artículo, se explica cómo cargar y administrar paquetes de aplicación en Azure Portal. También se muestra cómo instalarlos en los nodos de proceso de un grupo mediante la biblioteca de [.NET para Batch](/dotnet/api/overview/azure/batch/client).

## <a name="application-package-requirements"></a>Requisitos de los paquetes de aplicación

Para utilizar paquetes de aplicación, primero se debe [vincular una cuenta de Azure Storage](#link-a-storage-account) a su cuenta de Batch.

Hay restricciones en el número de aplicaciones y paquetes de aplicación que puede haber en una cuenta de Batch, así como en el tamaño máximo del paquete de aplicación. Para más información, consulte [Límites y cuotas del servicio Azure Batch](batch-quota-limit.md).

> [!NOTE]
> Los grupos de Batch creados antes del 5 de julio de 2017 no admiten paquetes de aplicación (a menos que se hayan creado después del 10 de marzo de 2016 mediante la configuración de Cloud Services). La característica de paquetes de aplicaciones que se describe aquí reemplaza a la característica de aplicaciones de Batch disponible en versiones anteriores del servicio.

## <a name="understand-applications-and-application-packages"></a>Aplicaciones y paquetes de aplicación

En Azure Batch, una *aplicación* hace referencia a un conjunto de archivos binarios con versiones que se pueden descargar automáticamente en los nodos de proceso del grupo. Una aplicación contiene uno o varios *paquetes de aplicación*, que representan diferentes versiones de la aplicación.

Cada *paquete de aplicación* es un archivo .zip que contiene los archivos binarios de la aplicación y los archivos auxiliares. Solo se admite el formato. zip.

:::image type="content" source="media/batch-application-packages/app_pkg_01.png" alt-text="Diagrama que muestra una vista de alto nivel de aplicaciones y paquetes de aplicación.":::

Los paquetes de aplicación se pueden especificar en el nivel de grupo o de tarea.

- **paquetes de aplicación del grupo** se implementan en cada nodo del grupo. Las aplicaciones se implementan cuando un nodo se une a un grupo y cuando se reinicia o se restablece la imagen inicial.
  
    Los paquetes de aplicación de grupo son adecuados cuando todos los nodos de un grupo van a ejecutar las tareas de un trabajo. Al crear un grupo, puede especificar uno o varios paquetes de aplicación para implementarlos. También puede agregar o actualizar los paquetes de un grupo existente. Para instalar un nuevo paquete en un grupo existente, debe reiniciar sus nodos.

- **paquetes de aplicación de tareas** solo se implementan en un nodo de proceso programado para ejecutar una tarea, justo antes de ejecutar la línea de comandos de la tarea. Si el paquete de aplicación y la versión especificados ya están en el nodo, este no se volverá a implementar y se utilizará el paquete existente.
  
    Los paquetes de aplicaciones de tareas son útiles en entornos de grupo compartido, donde distintos trabajos se ejecutan en un grupo que no se elimina cuando se completa un trabajo. Si el trabajo tiene menos tareas que nodos en el grupo, los paquetes de aplicación de las tareas pueden minimizar la transferencia de datos, ya que la aplicación se implementa solo en los nodos que ejecutan tareas.
  
    Otros escenarios que pueden beneficiarse de los paquetes de aplicación de tareas son los trabajos que ejecutan una aplicación grande, pero solo para unas pocas tareas. Por ejemplo, las aplicaciones de tareas pueden ser útiles para una fase de procesamiento previo pesada o una tarea de combinación.

Con los paquetes de aplicación, la tarea de inicio del grupo no tiene que especificar una larga lista de archivos de recursos individuales que se deben instalar en los nodos. No es preciso administrar manualmente varias versiones de los archivos de la aplicación en Azure Storage ni en los nodos. Tampoco es preciso preocuparse de generar [direcciones URL de SAS](../storage/common/storage-sas-overview.md) para proporcionar acceso a los archivos de su cuenta de almacenamiento. Batch funciona en segundo plano con Azure Storage para almacenar paquetes de aplicación e implementarlos en los nodos de proceso.

> [!NOTE]
> El tamaño total de una tarea de inicio debe ser menor o igual a 32 768 caracteres, incluidos los archivos de recursos y las variables de entorno. Si la tarea de inicio supera este límite, otra opción es usar paquetes de aplicación. Puede crear también un archivo .zip que contenga los archivos de recursos, cargarlo como un blob en Azure Storage y luego descomprimirlo desde la línea de comandos de la tarea de inicio.

## <a name="upload-and-manage-applications"></a>Carga y administración de aplicaciones

Puede usar [Azure Portal](https://portal.azure.com) o las API de Batch Management para administrar los paquetes de aplicaciones en la cuenta de Batch. En las secciones siguientes se explica cómo vincular una cuenta de almacenamiento y cómo agregar y administrar aplicaciones y paquetes de aplicación en Azure Portal.

> [!NOTE]
> Aunque puede definir los valores de la aplicación en el recurso [Microsoft.BatCH/batchAccounts](/azure/templates/microsoft.batch/batchaccounts) de una [plantilla de ARM](quick-create-template.md), actualmente no es posible usar una de estas plantillas para cargar paquetes de aplicación que se usarán en la cuenta de Batch. Deberá cargarlos en la cuenta de almacenamiento vinculada, como se describe [a continuación](#add-a-new-application).

### <a name="link-a-storage-account"></a>Vínculo a una cuenta de almacenamiento

Para utilizar paquetes de aplicación, primero debe vincular una [cuenta de Azure Storage](accounts.md#azure-storage-accounts) a su cuenta de Batch. El servicio Batch utilizará la cuenta de almacenamiento asociada para almacenar los paquetes de aplicación. Se recomienda crear una cuenta de almacenamiento específicamente para su uso con la cuenta de Batch.

Si aún no ha configurado ninguna cuenta de almacenamiento, Azure Portal muestra una advertencia la primera vez que seleccione **Aplicaciones** en la cuenta de Batch. Para vincular una cuenta de almacenamiento a su cuenta de Batch, seleccione **Cuenta de almacenamiento** en la ventana **Advertencia** y, después, **Cuenta de almacenamiento** de nuevo.

Una vez que haya vinculado las dos cuentas, Batch puede implementar automáticamente los paquetes almacenados en la cuenta de Almacenamiento vinculada en los nodos de proceso.

> [!IMPORTANT]
> No puede usar paquetes de aplicación con cuentas de Azure Storage configuradas con [reglas de firewall](../storage/common/storage-network-security.md) o con **Espacio de nombres jerárquico** establecido en **Habilitado**.

El servicio Batch utiliza Azure Storage para almacenar los paquetes de aplicación como blobs en bloques. Los datos de blob en bloques se [cobran al precio habitual](https://azure.microsoft.com/pricing/details/storage/) y el tamaño de cada paquete no puede superar el tamaño máximo del blob en bloques. Para más información, consulte [Objetivos de escalabilidad y rendimiento de Azure Storage para las cuentas de almacenamiento](../storage/blobs/scalability-targets.md). Para minimizar los costos, asegúrese de tener en cuenta el tamaño y número de los paquetes de aplicación y elimine periódicamente los paquetes en desuso.

### <a name="view-current-applications"></a>Visualización de las aplicaciones actuales

Para ver las aplicaciones de la cuenta de Batch, seleccione **Aplicaciones** en el menú de navegación izquierdo.

:::image type="content" source="media/batch-application-packages/app_pkg_02.png" alt-text="Captura de pantalla del elemento de menú Aplicaciones en Azure Portal.":::

Al seleccionar esta opción de menú se abre la ventana **Aplicaciones**. Esta ventana muestra el identificador de cada aplicación en su cuenta y las propiedades siguientes:

- **Paquetes**: el número de versiones asociadas a la aplicación.
- **Versión predeterminada**: si procede, versión de la aplicación que se instalará si no se especifica ninguna versión al implementar la aplicación.
- **Permitir actualizaciones**: especifica si se permiten las actualizaciones y eliminaciones de paquetes.

Si quiere ver la [estructura de archivos](files-and-directories.md) del paquete de aplicación en un nodo de proceso, vaya a su cuenta de Batch en Azure Portal. Seleccione **Grupos**. Seleccione el grupo que contiene los nodos de proceso. Seleccione el nodo de proceso en el que está instalado el paquete de aplicación y abra la carpeta **applications**.

### <a name="view-application-details"></a>Visualización de los detalles de una aplicación

Para consultar los detalles de una aplicación, selecciónela en la ventana **Aplicaciones**. Puede configurar los siguientes valores para la aplicación.

- **Permitir actualizaciones**: indica si sus paquetes de aplicación se pueden [actualizar o eliminar](#update-or-delete-an-application-package). El valor predeterminado es **Sí**. Si se establece en **No**, los paquetes de aplicación existentes no se pueden actualizar ni eliminar, pero todavía se pueden agregar nuevas versiones de paquetes de aplicación.
- **Versión predeterminada**: paquete de aplicación predeterminado que se utilizará cuando se implemente la aplicación si no se especifica ninguna versión.
- **Nombre para mostrar**: nombre descriptivo que la solución de Batch puede usar cuando muestra información sobre la aplicación. Por ejemplo, este nombre se puede usar en la interfaz de usuario de un servicio que se proporciona a los clientes a través de Batch.

### <a name="add-a-new-application"></a>Adición de una nueva aplicación

Para crear una aplicación, agregue un paquete de aplicación y especifique un identificador de aplicación único.

En la cuenta de Batch, seleccione **Aplicaciones** y luego elija **Agregar**.

:::image type="content" source="media/batch-application-packages/app_pkg_05.png" alt-text="Captura de pantalla del proceso de creación Nueva aplicación en Azure Portal.":::

Escriba la siguiente información:

- **Identificador de aplicación**: identificador de la nueva aplicación.
- **Versión**": versión del paquete de aplicación que se carga.
- **Paquete de aplicación**: archivo .zip que contiene los archivos binarios y los archivos auxiliares necesarios para ejecutar la aplicación.

Los valores que escriba en **Id. de la aplicación** y **Versión** deben seguir estos requisitos:

- En los nodos de Windows, el identificador puede contener cualquier combinación de caracteres alfanuméricos, guiones y caracteres de subrayado. En los nodos de Linux, solo se permiten caracteres alfanuméricos y caracteres de subrayado.
- No pueden contener más de 64 caracteres.
- Deben ser único en la cuenta de Batch.
- Los identificadores conservan las mayúsculas y minúsculas, aunque no las distinguen.

Cuando esté listo, seleccione **Enviar**. Después de cargar el archivo. zip en la cuenta de Azure Storage, el portal muestra una notificación. En función del tamaño del archivo que se va a cargar y de la velocidad de la conexión de red, esta operación puede tardar cierto tiempo.

### <a name="add-a-new-application-package"></a>Adición de un nuevo paquete de aplicación

Para agregar una versión del paquete de aplicación para una aplicación existente, seleccione la aplicación en la sección **Aplicaciones** de la cuenta de Batch y luego elija **Agregar**.

Como hizo con la nueva aplicación, especifique la **versión** del nuevo paquete, cargue el archivo. zip en el campo **Paquete de aplicación** y seleccione **Enviar**.

### <a name="update-or-delete-an-application-package"></a>Actualización o eliminación de un paquete de aplicación

Para actualizar o eliminar un paquete de aplicación existente, seleccione la aplicación en la sección **Aplicaciones** de la cuenta de Batch. Seleccione el botón de puntos suspensivos de la fila del paquete de aplicación que quiere modificar y luego elija la acción que quiere realizar.

:::image type="content" source="media/batch-application-packages/app_pkg_07.png" alt-text="Captura de pantalla que muestra las opciones Actualizar y Eliminar para los paquetes de aplicación en Azure Portal.":::

Si selecciona **Actualizar**, podrá cargar un nuevo archivo .zip. Con ello reemplazará el archivo .zip anterior que cargó para esa versión.

Si selecciona **Eliminar**, se le pedirá que confirme la eliminación de esa versión. Cuando haya seleccionado **Aceptar**, Batch eliminará el archivo .zip de la cuenta de Azure Storage. Si elimina la versión predeterminada de una aplicación, se quita el valor **Versión predeterminada** para esa aplicación.

## <a name="install-applications-on-compute-nodes"></a>Instalación de aplicaciones en nodos de proceso

Ahora que ha aprendido cómo administrar paquetes de aplicación en Azure Portal, podemos analizar cómo implementarlos en nodos de ejecución y ejecutarlos con tareas de Batch.

### <a name="install-pool-application-packages"></a>Instalación de paquetes de aplicación de grupos

Para instalar un paquete de aplicación en todos los nodos de proceso de un grupo, especifique una o varias referencias de paquetes de aplicación para el grupo. Los paquetes de aplicación que especifique para un grupo se instalan en cada nodo de proceso que se une al grupo, y en cualquier nodo que se reinicie o del que se restablezca la imagen inicial.

En el entorno de .NET para Batch, especifique uno o varios valores [CloudPool.ApplicationPackageReferences](/dotnet/api/microsoft.azure.batch.cloudpool.applicationpackagereferences) al crear un grupo o utilizar un grupo existente. La clase [ApplicationPackageReference](/dotnet/api/microsoft.azure.batch.applicationpackagereference) especifica una versión y un identificador de la aplicación que se va a instalar en los nodos de proceso de un grupo.

```csharp
// Create the unbound CloudPool
CloudPool myCloudPool =
    batchClient.PoolOperations.CreatePool(
        poolId: "myPool",
        targetDedicatedComputeNodes: 1,
        virtualMachineSize: "standard_d1_v2",
        cloudServiceConfiguration: new CloudServiceConfiguration(osFamily: "5"));

// Specify the application and version to install on the compute nodes
myCloudPool.ApplicationPackageReferences = new List<ApplicationPackageReference>
{
    new ApplicationPackageReference {
        ApplicationId = "litware",
        Version = "1.1001.2b" }
};

// Commit the pool so that it's created in the Batch service. As the nodes join
// the pool, the specified application package is installed on each.
await myCloudPool.CommitAsync();
```

> [!IMPORTANT]
> Si la implementación de un paquete de aplicación produce un error, el servicio Batch marca el nodo como [inutilizable](/dotnet/api/microsoft.azure.batch.computenode.state) y no se programarán tareas de ejecución en ese nodo. Si esto sucede, reinicie el nodo para reiniciar la implementación del paquete. Al reiniciar el nodo, también se vuelve a habilitar la programación de tareas en el nodo.

### <a name="install-task-application-packages"></a>Instalación de paquetes de aplicación de tareas

De forma similar a un grupo, puede especificar las referencias de un paquete de aplicación para una tarea. Cuando una tarea está programada para ejecutarse en un nodo, el paquete se descarga y extrae justo antes de ejecutar la línea de comandos de la tarea. Si el paquete y versión especificados ya están instalados en el nodo, el paquete no se descarga y se utiliza el paquete existente.

Para instalar un paquete de aplicación de tareas, configure la propiedad [CloudTask.ApplicationPackageReferences](/dotnet/api/microsoft.azure.batch.cloudtask.applicationpackagereferences) de la tarea:

```csharp
CloudTask task =
    new CloudTask(
        "litwaretask001",
        "cmd /c %AZ_BATCH_APP_PACKAGE_LITWARE%\\litware.exe -args -here");

task.ApplicationPackageReferences = new List<ApplicationPackageReference>
{
    new ApplicationPackageReference
    {
        ApplicationId = "litware",
        Version = "1.1001.2b"
    }
};
```

## <a name="execute-the-installed-applications"></a>Ejecución de las aplicaciones instaladas

Los paquetes que ha especificado para un grupo o tarea se descargan y extraen en un directorio con nombre dentro del nodo `AZ_BATCH_ROOT_DIR` . Batch también permite crear una variable de entorno que contiene la ruta de acceso al directorio con nombre. Las líneas de comando de la tarea usan esta variable de entorno al hacer referencia a la aplicación en el nodo.

En los nodos de Windows, la variable está en el formato siguiente:

```
Windows:
AZ_BATCH_APP_PACKAGE_APPLICATIONID#version
```

En los nodos de Linux, el formato es ligeramente diferente. Los puntos (.), guiones (-) y signos de número (##) se convierten en caracteres de subrayado en la variable de entorno. Además, tenga en cuenta que se conserva el caso del identificador de la aplicación. Por ejemplo:

```
Linux:
AZ_BATCH_APP_PACKAGE_applicationid_version
```

`APPLICATIONID` y `version` son los valores que corresponden a la versión de la aplicación y del paquete que ha especificado para la implementación. Por ejemplo, si especificó que se debía instalar la versión 2.7 de la aplicación *blender* en los nodos de Windows, las líneas de comando de la tarea usarán esta variable de entorno para tener acceso a sus archivos:

```
Windows:
AZ_BATCH_APP_PACKAGE_BLENDER#2.7
```

En los nodos de Linux, especifique la variable de entorno con este formato. Acople los puntos (.), guiones (-) y almohadillas (#) a los caracteres de subrayado y conserve el caso del identificador de la aplicación:

```
Linux:
AZ_BATCH_APP_PACKAGE_blender_2_7
```

Cuando carga un paquete de aplicación, puede especificar una versión predeterminada que implementar en los nodos de proceso. Si ha especificado una versión predeterminada para una aplicación, puede omitir el sufijo de versión al hacer referencia a ella. Puede especificar la versión predeterminada de la aplicación en Azure Portal, en la ventana **Aplicaciones**, como se muestra en [Carga y administración de aplicaciones](#upload-and-manage-applications).

Por ejemplo, si especifica la versión "2.7" como la versión predeterminada de la aplicación *blender*, las tareas pueden hacer referencia a la siguiente variable de entorno y, después, los nodos de Windows ejecutarán la versión 2.7:

`AZ_BATCH_APP_PACKAGE_BLENDER`

El siguiente fragmento de código muestra una línea de comandos de una tarea de ejemplo que inicia la versión predeterminada de la aplicación *blender* :

```csharp
string taskId = "blendertask01";
string commandLine =
    @"cmd /c %AZ_BATCH_APP_PACKAGE_BLENDER%\blender.exe -args -here";
CloudTask blenderTask = new CloudTask(taskId, commandLine);
```

> [!TIP]
> Para más información sobre la configuración del entorno del nodo de proceso, consulte [Configuración del entorno para las tareas](jobs-and-tasks.md#environment-settings-for-tasks).

## <a name="update-a-pools-application-packages"></a>Actualización de los paquetes de aplicación de un grupo

Si un grupo existente ya se ha configurado con un paquete de aplicación, se puede especificar un paquete nuevo para el grupo. Esto significa lo siguiente:

- El servicio Batch instala el paquete recién especificado en todos los nodos nuevos que se unen al grupo y en cualquier nodo existente que se reinicie o cuya imagen inicial se restablezca.
- Los nodos de proceso que ya estén en el grupo cuando se actualicen las referencias del paquete no instalan automáticamente el paquete de aplicación nuevo. Estos nodos de proceso deben reiniciarse o se debe restablecer su imagen inicial para recibir el nuevo paquete.
- Cuando se implementa un nuevo paquete, las variables de entorno creadas reflejan las nuevas referencias del paquete de aplicación.

En este ejemplo, el grupo existente tiene la versión 2.7 de la aplicación *Blender* configurada como uno de sus valores [CloudPool.ApplicationPackageReferences](/dotnet/api/microsoft.azure.batch.cloudpool.applicationpackagereferences). Para actualizar los nodos del grupo con la versión 2.76b, especifique una nueva clase [ApplicationPackageReference](/dotnet/api/microsoft.azure.batch.applicationpackagereference) con la nueva versión y confirme el cambio.

```csharp
string newVersion = "2.76b";
CloudPool boundPool = await batchClient.PoolOperations.GetPoolAsync("myPool");
boundPool.ApplicationPackageReferences = new List<ApplicationPackageReference>
{
    new ApplicationPackageReference {
        ApplicationId = "blender",
        Version = newVersion }
};
await boundPool.CommitAsync();
```

Ahora que se ha configurado la nueva versión, el servicio Batch instala la versión 2.76b en todos los nodos nuevos que se unan al grupo. Para instalar la versión 2.76b en los nodos que ya están en el grupo, reinícielos o restablezca su imagen inicial. Tenga en cuenta que los nodos reiniciados conservan los archivos de las anteriores implementaciones del paquete.

## <a name="list-the-applications-in-a-batch-account"></a>Enumeración de las aplicaciones en una cuenta de Batch

Puede mostrar las aplicaciones y sus paquetes en una cuenta de Batch mediante el método [ApplicationOperations.ListApplicationSummaries](/dotnet/api/microsoft.azure.batch.applicationoperations.listapplicationsummaries).

```csharp
// List the applications and their application packages in the Batch account.
List<ApplicationSummary> applications = await batchClient.ApplicationOperations.ListApplicationSummaries().ToListAsync();
foreach (ApplicationSummary app in applications)
{
    Console.WriteLine("ID: {0} | Display Name: {1}", app.Id, app.DisplayName);

    foreach (string version in app.Versions)
    {
        Console.WriteLine("  {0}", version);
    }
}
```

## <a name="next-steps"></a>Pasos siguientes

- La [API de REST de Batch](/rest/api/batchservice) también proporciona compatibilidad para trabajar con paquetes de aplicación. Por ejemplo, consulte el elemento [applicationPackageReferences](/rest/api/batchservice/pool/add#applicationpackagereference) para saber cómo especificar los paquetes que se van instalar, y [Applications](/rest/api/batchservice/application) para saber cómo obtener información de la aplicación.
- Aprenda a [administrar mediante programación cuentas y cuotas de Azure Batch con Batch Management .NET](batch-management-dotnet.md). La biblioteca de [Batch Management .NET](/dotnet/api/overview/azure/batch/management) puede habilitar las características de creación y eliminación de cuentas de una aplicación o servicio de Batch.
