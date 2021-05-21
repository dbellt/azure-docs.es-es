---
title: 'Tutorial: Creación y administración de datos exportados desde Azure Cost Management'
description: En este artículo se muestra cómo crear y administrar datos de Azure Cost Management para utilizarlos en sistemas externos.
author: bandersmsft
ms.author: banders
ms.date: 05/06/2021
ms.topic: tutorial
ms.service: cost-management-billing
ms.subservice: cost-management
ms.reviewer: adwise
ms.custom: seodec18, devx-track-azurepowershell, devx-track-azurecli
ms.openlocfilehash: 9f9afe0b7c27fb2199f9bdcd1fa5edbe4da01602
ms.sourcegitcommit: 38d81c4afd3fec0c56cc9c032ae5169e500f345d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109517647"
---
# <a name="tutorial-create-and-manage-exported-data"></a>Tutorial: Creación y administración de datos exportados

Si lee el tutorial de análisis de costos, estará familiarizado con la descarga manual de los datos de Cost Management. Sin embargo, puede crear una tarea periódica para exportar automáticamente los datos de Cost Management en el almacenamiento de Azure con una periodicidad diaria, semanal o mensual. Los datos exportados están en formato CSV y contienen toda la información recopilada por Cost Management. A continuación, puede usar los datos exportados en el almacenamiento de Azure con sistemas externos y combinarlos con sus propios datos personalizados. También puede usar los datos exportados en un sistema externo, como un panel u otro sistema financiero.

Vea el vídeo sobre [cómo programar exportaciones a almacenamiento con Azure Cost Management](https://www.youtube.com/watch?v=rWa_xI1aRzo) para conocer más sobre la creación de una exportación programada de los datos de sus costos de Azure a Azure Storage. Para ver otros vídeos, visite el [canal de YouTube de Cost Management](https://www.youtube.com/c/AzureCostManagement).

>[!VIDEO https://www.youtube.com/embed/rWa_xI1aRzo]

Los ejemplos de este tutorial le guiarán durante la exportación de los datos de administración de costos y la comprobación de que los datos se exporten correctamente.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Creación de una exportación diaria
> * Verificación de la recopilación de los datos

## <a name="prerequisites"></a>Requisitos previos

La exportación de datos está disponible para varios tipos de cuenta de Azure, entre las que se incluyen [Contrato Enterprise (EA)](https://azure.microsoft.com/pricing/enterprise-agreement/) y [Contrato de cliente de Microsoft](get-started-partners.md). Para ver la lista completa de tipos de cuenta compatibles, consulte [Understand Cost Management data](understand-cost-mgt-data.md) (Información sobre los datos de Cost Management). Para la exportación de datos por usuario y grupo, se admiten los siguientes permisos o ámbitos de Azure por suscripción. Para más información sobre los ámbitos, consulte [Descripción y uso de ámbitos](understand-work-scopes.md).

- Propietario: puede crear, modificar o eliminar exportaciones programadas para una suscripción.
- Colaborador: puede crear, modificar o eliminar sus propias exportaciones programadas. Puede modificar el nombre de las exportaciones programadas creadas por otros usuarios.
- Lector: puede programar las exportaciones para las que tenga permiso.

**Para más información sobre los ámbitos, incluido el acceso necesario para configurar las exportaciones de Contrato Enterprise y los ámbitos del Contrato de cliente de Microsoft, consulte [Descripción y uso de ámbitos](understand-work-scopes.md)** .

Para las cuentas de Azure Storage:
- Se necesitan permisos de escritura para cambiar la cuenta de Storage configurada, independientemente de los permisos sobre la exportación.
- La cuenta de Azure Storage debe configurarse para el almacenamiento de blobs o archivos.

Si su suscripción es nueva, no podrá usar inmediatamente las características de Cost Management. Para poder hacerlo deberán transcurrir un máximo de 48 horas.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure
Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com/).

## <a name="create-a-daily-export"></a>Creación de una exportación diaria

### <a name="portal"></a>[Portal](#tab/azure-portal)

Para crear o ver una exportación de datos o programar una exportación, elija un ámbito en Azure Portal y seleccione **Análisis de costos** en el menú. Por ejemplo, vaya a **Suscripciones**, seleccione una suscripción de la lista y **Análisis de costos**, en el menú. En la parte superior de la página Análisis de costos, seleccione **Configuración** y, después, **Exportaciones**.

> [!NOTE]
> - Además de las suscripciones, puede crear exportaciones en grupo de recursos, grupos de administración, departamentos e inscripciones. Para más información sobre los ámbitos, consulte [Descripción y uso de ámbitos](understand-work-scopes.md).
> - Cuando haya iniciado sesión como asociado en el ámbito de la cuenta de facturación o en el inquilino de un cliente, podrá exportar los datos a una cuenta de Azure Storage vinculada a su cuenta de almacenamiento de asociado. Sin embargo, debe tener una suscripción activa en el inquilino de CSP.

1. Seleccione **Agregar** y escriba el nombre de la exportación.
1. En **Métrica**, seleccione una de estas opciones:
    - **Actual cost (Usage and Purchases)** (Costo real [uso y compras]): seleccione esta opción para exportar el uso y las compras estándar.
    - **Amortized cost (Usage and Purchases)** (Costo amortizado [uso y compras]): seleccione esta opción para exportar los costos amortizados para compras como las reservas de Azure.
1. En **Tipo de exportación**, seleccione una de estas opciones:
    - **Exportación diaria de costos desde ese mes hasta la fecha actual**: proporciona un nuevo archivo de exportación diaria de los costos desde un mes hasta la fecha. Se agregan los datos más recientes de las exportaciones diarias anteriores.
    - **Weekly export of cost for the last seven days** (Exportación semanal de los costos en los últimos siete días): crea una exportación semanal de los costos de los últimos siete días, a partir de la fecha de inicio seleccionada.
    - **Monthly export of last month's costs** (Exportación mensual de los costos del mes pasado): proporciona una exportación de los costos del último mes, en comparación con el mes actual en el que se crea la exportación. Luego, la programación ejecuta una exportación el quinto día de cada mes con los costos de los meses anteriores.
    - **One-time export** (Exportación una sola vez): permite elegir un intervalo de fechas para la exportación de los datos históricos a Azure Blob Storage. Puede exportar un máximo de 90 días del historial de costos a partir del día que elija. Esta exportación se ejecuta de inmediato y está disponible en la cuenta de almacenamiento en un plazo de dos horas.
        En función del tipo de exportación, elija una fecha de inicio o elija una fecha en los campos **Desde** y **Hasta**.
1. Especifique la suscripción de su cuenta de almacenamiento de Azure y, luego, seleccione un grupo de recursos o créelo.
1. Seleccione el nombre de la cuenta de almacenamiento o cree una.
1. Seleccione la ubicación (región de Azure).
1. Especifique el contenedor de almacenamiento y la ruta de acceso del directorio al que quiere que vaya el archivo de exportación.
    :::image type="content" source="./media/tutorial-export-acm-data/basics_exports.png" alt-text="Ejemplo de exportación nueva" lightbox="./media/tutorial-export-acm-data/basics_exports.png":::
1. Revise los detalles de la exportación y seleccione **Crear**.

La nueva exportación aparece en la lista de exportaciones. De forma predeterminada, se habilitan las nuevas exportaciones. Si quiere deshabilitar o eliminar una exportación programada, seleccione cualquier elemento de la lista y, después, **Deshabilitar** o **Eliminar**.

Al principio, la exportación puede tardar entre 12 y 24 horas en ejecutarse. Sin embargo, pueden pasar aún más tiempo para que se muestren los datos en los archivos exportados.

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Al crear una exportación mediante programación, debe registrar manualmente el proveedor de recursos `Microsoft.CostManagementExports` con la suscripción donde reside la cuenta de almacenamiento. El registro se produce automáticamente cuando se crea la exportación mediante Azure Portal. Para más información sobre cómo registrar un proveedor de recursos, consulte [Registro del proveedor de recursos](../../azure-resource-manager/management/resource-providers-and-types.md#register-resource-provider).

Empiece por preparar el entorno para la CLI de Azure:

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../../includes/azure-cli-prepare-your-environment-no-header.md)]

1. Después de iniciar sesión, para ver las exportaciones actuales, use el comando [az costmanagement export list](/cli/azure/costmanagement/export#az_costmanagement_export_list):

   ```azurecli
   az costmanagement export list --scope "subscriptions/00000000-0000-0000-0000-000000000000"
   ```

   >[!NOTE]
   >
   >* Además de las suscripciones, puede crear exportaciones en grupo de recursos y grupos de administración. Para más información sobre los ámbitos, consulte [Descripción y uso de ámbitos](understand-work-scopes.md).
   >* Cuando haya iniciado sesión como asociado en el ámbito de la cuenta de facturación o en el inquilino de un cliente, podrá exportar los datos a una cuenta de Azure Storage vinculada a su cuenta de almacenamiento de asociado. Sin embargo, debe tener una suscripción activa en el inquilino de CSP.

1. Cree un grupo de recursos o use uno existente. Para crear un grupo de recursos, use el comando [az group create](/cli/azure/group#az_group_create):

   ```azurecli
   az group create --name TreyNetwork --location "East US"
   ```

1. Cree una cuenta de almacenamiento para recibir las exportaciones o use una existente. Para crear una cuenta de almacenamiento, use el comando [az storage account create](/cli/azure/storage/account#az_storage_account_create):

   ```azurecli
   az storage account create --resource-group TreyNetwork --name cmdemo
   ```

1. Ejecute el comando [az costmanagement export create](/cli/azure/costmanagement/export#az_costmanagement_export_create) para crear la exportación:

   ```azurecli
   az costmanagement export create --name DemoExport --type ActualCost \
   --scope "subscriptions/00000000-0000-0000-0000-000000000000" --storage-account-id cmdemo \
   --storage-container democontainer --timeframe MonthToDate --recurrence Daily \
   --recurrence-period from="2020-06-01T00:00:00Z" to="2020-10-31T00:00:00Z" \
   --schedule-status Active --storage-directory demodirectory
   ```

   Para el parámetro **--type**, puede elegir `ActualCost`, `AmortizedCost` o `Usage`.

   En este ejemplo se usa `MonthToDate`. La exportación crea un archivo de exportación diariamente para los costos mensuales hasta la fecha. Se agregan los datos más recientes de las exportaciones diarias anteriores de este mes.

1. Para ver los detalles de la operación de exportación, use el comando [az costmanagement export show](/cli/azure/costmanagement/export#az_costmanagement_export_show):

   ```azurecli
   az costmanagement export show --name DemoExport \
      --scope "subscriptions/00000000-0000-0000-0000-000000000000"
   ```

1. Actualice una exportación mediante el comando [az costmanagement export update](/cli/azure/costmanagement/export#az_costmanagement_export_update):

   ```azurecli
   az costmanagement export update --name DemoExport
      --scope "subscriptions/00000000-0000-0000-0000-000000000000" --storage-directory demodirectory02
   ```

   En este ejemplo se cambia el directorio de salida.

>[!NOTE]
>Al principio, la exportación puede tardar entre 12 y 24 horas en ejecutarse. Sin embargo, los datos pueden tardar más en mostrarse en los archivos exportados.

Puede eliminar una exportación mediante el comando [az costmanagement export delete](/cli/azure/costmanagement/export#az_costmanagement_export_delete):

```azurecli
az costmanagement export delete --name DemoExport --scope "subscriptions/00000000-0000-0000-0000-000000000000"
```

### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

Al crear una exportación mediante programación, debe registrar manualmente el proveedor de recursos `Microsoft.CostManagementExports` con la suscripción donde reside la cuenta de almacenamiento. El registro se produce automáticamente cuando se crea la exportación mediante Azure Portal. Para más información sobre cómo registrar un proveedor de recursos, consulte [Registro del proveedor de recursos](../../azure-resource-manager/management/resource-providers-and-types.md#register-resource-provider).

Empiece por preparar el entorno para Azure PowerShell:

[!INCLUDE [azure-powershell-requirements-no-header.md](../../../includes/azure-powershell-requirements-no-header.md)]

  > [!IMPORTANT]
  > Aunque el módulo **Az.CostManagement** de PowerShell está en versión preliminar, se debe instalar por separado mediante el cmdlet `Install-Module`. Una vez que este módulo de PowerShell esté disponible con carácter general, formará parte de las futuras versiones del módulo Az de PowerShell y estará disponible de forma predeterminada en Azure Cloud Shell.

  ```azurepowershell-interactive
  Install-Module -Name Az.CostManagement
  ```

1. Después de iniciar sesión, para ver las exportaciones actuales, use el cmdlet [Get-AzCostManagementExport](/powershell/module/Az.CostManagement/get-azcostmanagementexport):

   ```azurepowershell-interactive
   Get-AzCostManagementExport -Scope 'subscriptions/00000000-0000-0000-0000-000000000000'
   ```

   >[!NOTE]
   >
   >* Además de las suscripciones, puede crear exportaciones en grupo de recursos y grupos de administración. Para más información sobre los ámbitos, consulte [Descripción y uso de ámbitos](understand-work-scopes.md).
   >* Cuando haya iniciado sesión como asociado en el ámbito de la cuenta de facturación o en el inquilino de un cliente, podrá exportar los datos a una cuenta de Azure Storage vinculada a su cuenta de almacenamiento de asociado. Sin embargo, debe tener una suscripción activa en el inquilino de CSP.

1. Cree un grupo de recursos o use uno existente. Para crear un grupo de recursos, use el cmdlet [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup):

   ```azurepowershell-interactive
   New-AzResourceGroup -Name TreyNetwork -Location eastus
   ```

1. Cree una cuenta de almacenamiento para recibir las exportaciones o use una existente. Para crear una cuenta de almacenamiento, use el cmdlet [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount):

   ```azurepowershell-interactive
   New-AzStorageAccount -ResourceGroupName TreyNetwork -AccountName cmdemo -SkuName Standard_RAGRS -Location eastus
   ```

1. Ejecute el cmdlet [New-AzCostManagementExport](/powershell/module/Az.CostManagement/new-azcostmanagementexport) para crear la exportación:

   ```azurepowershell-interactive
   $Params = @{
     Name = 'DemoExport'
     DefinitionType = 'ActualCost'
     Scope = 'subscriptions/00000000-0000-0000-0000-000000000000'
     DestinationResourceId = '/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/treynetwork/providers/Microsoft.Storage/storageAccounts/cmdemo'
     DestinationContainer = 'democontainer'
     DefinitionTimeframe = 'MonthToDate'
     ScheduleRecurrence = 'Daily'
     RecurrencePeriodFrom = '2020-06-01T00:00:00Z'
     RecurrencePeriodTo = '2020-10-31T00:00:00Z'
     ScheduleStatus = 'Active'
     DestinationRootFolderPath = 'demodirectory'
     Format = 'Csv'
   }
   New-AzCostManagementExport @Params
   ```

   Para el parámetro **DefinitionType**, puede elegir `ActualCost`, `AmortizedCost` o `Usage`.

   En este ejemplo se usa `MonthToDate`. La exportación crea un archivo de exportación diariamente para los costos mensuales hasta la fecha. Se agregan los datos más recientes de las exportaciones diarias anteriores de este mes.

1. Para ver los detalles de la operación de exportación, use el cmdlet `Get-AzCostManagementExport`:

   ```azurepowershell-interactive
   Get-AzCostManagementExport -Scope 'subscriptions/00000000-0000-0000-0000-000000000000'
   ```

1. Actualice una exportación mediante el cmdlet [Update-AzCostManagementExport](/powershell/module/Az.CostManagement/update-azcostmanagementexport):

   ```azurepowershell-interactive
   Update-AzCostManagementExport -Name DemoExport -Scope 'subscriptions/00000000-0000-0000-0000-000000000000' -DestinationRootFolderPath demodirectory02
   ```

   En este ejemplo se cambia el directorio de salida.

>[!NOTE]
>Al principio, la exportación puede tardar entre 12 y 24 horas en ejecutarse. Sin embargo, los datos pueden tardar más en mostrarse en los archivos exportados.

Las exportaciones se pueden eliminar mediante el cmdlet [Remove-AzCostManagementExport](/powershell/module/Az.CostManagement/remove-azcostmanagementexport):

```azurepowershell-interactive
Remove-AzCostManagementExport -Name DemoExport -Scope 'subscriptions/00000000-0000-0000-0000-000000000000'
```

---

### <a name="export-schedule"></a>Programación de las exportaciones

Las exportaciones programadas se ven afectadas por la hora y el día de la semana en que se crearon inicialmente. Al crear una exportación programada, esta se ejecuta con la misma frecuencia las veces siguientes. Por ejemplo, en el caso de una exportación diaria de los costos de un mes hasta la fecha con una frecuencia diaria, la exportación se ejecuta todos los días. Pasa lo mismo con una exportación semanal, se ejecuta cada semana, el mismo día en el que fue programada. No se garantiza el tiempo de entrega exacto de la exportación y los datos exportados están disponibles en un plazo de cuatro horas a partir del tiempo de ejecución.

Cada exportación crea un nuevo archivo, por lo que las exportaciones anteriores no se sobrescriben.

#### <a name="create-an-export-for-multiple-subscriptions"></a>Creación de una exportación para varias suscripciones

Si tiene un Contrato Enterprise, puede usar un grupo de administración para agregar información de costos de suscripción en un solo contenedor. Después, puede exportar los datos de administración de costos para el grupo de administración.

No se admiten las exportaciones de grupos de administración de otros tipos de suscripciones.

1. Si aún no ha creado un grupo de administración, hágalo y asígnele suscripciones.
1. En análisis de costos, establezca el ámbito en el grupo de administración y seleccione **Select this management group** (Seleccionar este grupo de administración).
    :::image type="content" source="./media/tutorial-export-acm-data/management-group-scope.png" alt-text="Ejemplo en el que se muestra la opción Select this management group (Seleccionar este grupo de administración)" lightbox="./media/tutorial-export-acm-data/management-group-scope.png":::
1. Cree una exportación en el ámbito para obtener datos de administración de costos para las suscripciones en el grupo de administración.
    :::image type="content" source="./media/tutorial-export-acm-data/new-export-management-group-scope.png" alt-text="Ejemplo que muestra la opción Crear nueva exportación con un ámbito de grupo de administración.":::

### <a name="file-partitioning-for-large-datasets"></a>Creación de particiones de archivos para grandes conjuntos de datos

Si tiene un Contrato de cliente de Microsoft o un contrato Microsoft Partner Agreement, puede habilitar Exportaciones para fragmentar el archivo en varias particiones de archivo más pequeñas a fin de ayudar con la ingesta de datos. Al configurar inicialmente la exportación, establezca la opción **File Partitioning** (Creación de particiones de archivos) en **Activado**. El valor es **Desactivado** de forma predeterminada.

:::image type="content" source="./media/tutorial-export-acm-data/file-partition.png" alt-text="Captura de pantalla que muestra la opción File Partitioning (Creación de particiones de archivos)." lightbox="./media/tutorial-export-acm-data/file-partition.png" :::

Si no tiene un Contrato de cliente de Microsoft o un contrato de Microsoft Partner Agreement, no verá la opción **File Partitioning** (Creación de particiones de archivos).

#### <a name="update-existing-exports-to-use-file-partitioning"></a>Actualización de las exportaciones existentes para usar la creación de particiones de archivos

Si tiene exportaciones existentes y quiere configurar la creación de particiones de archivos, cree una exportación. La creación de particiones de archivos solo está disponible con la versión más reciente de Exportaciones. Puede haber cambios menores en algunos de los campos de los archivos de uso que se crean.

Si habilita la creación de particiones de archivos en una exportación existente, es posible que vea cambios menores en los campos en la salida del archivo. Los cambios se deben a las actualizaciones realizadas en Exportaciones después de configurar inicialmente las suyas.

#### <a name="partitioning-output"></a>Salida de creación de particiones

Cuando se habilita la creación de particiones de archivos, se obtiene un archivo para cada partición de datos de la exportación junto con un archivo _manifest.json. El manifiesto contiene un resumen del conjunto de datos completo e información de cada partición de archivo que contiene. Cada partición de archivo tiene encabezados y solo contiene un subconjunto del conjunto de datos completo. Para controlar el conjunto de datos completo, debe ingerir cada partición de la exportación.

Este es un ejemplo del archivo de manifiesto _manifest.json.

```json
{
  "manifestVersion": "2021-01-01",
  "dataFormat": "csv",
  "blobCount": 1,
  "byteCount": 160769,
  "dataRowCount": 136,
  "blobs": [
    {
      "blobName": "blobName.csv",
      "byteCount": 160769,
      "dataRowCount": 136,
      "headerRowCount": 1,
      "contentMD5": "md5Hash"
    }
  ]
}
```
## <a name="verify-that-data-is-collected"></a>Verificación de la recopilación de los datos

Puede comprobar fácilmente que los datos de Cost Management se están recopilando y consultar el archivo CSV exportado mediante el Explorador de Azure Storage.

En la lista de exportaciones, seleccione el nombre de la cuenta de almacenamiento. En la página de la cuenta de almacenamiento, seleccione Abrir en el explorador. Si ve un cuadro de confirmación, seleccione **Sí** para abrir el archivo en el Explorador de Azure Storage.

![Página de la cuenta de almacenamiento que muestra información de ejemplo y un vínculo a Abrir en el Explorador](./media/tutorial-export-acm-data/storage-account-page.png)

En el Explorador de Storage, vaya al contenedor que quiera abrir y seleccione la carpeta correspondiente al mes actual. Se muestra una lista de archivos CSV. Seleccione uno y **Open** (Abrir).

![Información de ejemplo que se muestra en el Explorador de almacenamiento](./media/tutorial-export-acm-data/storage-explorer.png)

El archivo se abre con el programa o la aplicación que se ha establecido para abrir las extensiones de archivos CSV. Este es un ejemplo en Excel.

![Datos CSV exportados de ejemplo que se muestran en Excel](./media/tutorial-export-acm-data/example-export-data.png)

### <a name="download-an-exported-csv-data-file"></a>Descarga de un archivo de datos CSV exportado

También puede descargar el archivo CSV exportado en Azure Portal. En los pasos siguientes se explica cómo encontrarlo desde el análisis de costos.

1. En el análisis de costos, seleccione **Configuración** y, después, **Exportaciones**.
1. En la lista de exportaciones, seleccione la cuenta de almacenamiento para una exportación.
1. En la cuenta de almacenamiento, seleccione **Contenedores**.
1. En la lista de contenedores, seleccione uno de ellos.
1. Desplácese por los directorios y los blobs de almacenamiento hasta la fecha que desee.
1. Seleccione el archivo CSV y, después, seleccione **Descargar**.

[![Descarga de exportación de ejemplo](./media/tutorial-export-acm-data/download-export.png)](./media/tutorial-export-acm-data/download-export.png#lightbox)

## <a name="view-export-run-history"></a>Visualización del historial de ejecución de una exportación

Para ver el historial de ejecución de una exportación programada, seleccione cualquier exportación individual en la página de la lista de exportaciones. Esta página también proporciona acceso rápido para ver el tiempo de ejecución de las exportaciones anteriores y la próxima vez que se va a ejecutar la exportación. En este ejemplo se muestra el historial de ejecución.

:::image type="content" source="./media/tutorial-export-acm-data/run-history.png" alt-text="Captura de pantalla que muestra el panel de exportaciones.":::

Seleccione una exportación para ver su historial de ejecución.

:::image type="content" source="./media/tutorial-export-acm-data/single-export-run-history.png" alt-text="Captura de pantalla que muestra el historial de ejecución de una exportación.":::

## <a name="access-exported-data-from-other-systems"></a>Acceso a los datos exportados desde otros sistemas

Uno de los propósitos de exportar los datos de Cost Management es tener acceso a los datos desde sistemas externos. Puede usar un sistema de paneles u otro sistema financiero. Estos sistemas pueden variar considerablemente, por lo que no es práctico mostrar un ejemplo.  Sin embargo, puede comenzar accediendo a sus datos desde aplicaciones en [Introducción a Azure Storage](../../storage/common/storage-introduction.md).

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Creación de una exportación diaria
> * Verificación de la recopilación de los datos

Vaya al siguiente tutorial para optimizar y mejorar la eficiencia mediante la identificación de recursos inactivos e infrautilizados.

> [!div class="nextstepaction"]
> [Consulta de las recomendaciones sobre optimización y medidas](tutorial-acm-opt-recommendations.md)
