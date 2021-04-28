---
title: Uso de Azure Import/Export para exportar datos de los blobs de Azure | Microsoft Docs
description: Aprenda a crear trabajos de exportación en Azure Portal para transferir datos de los blobs de Azure.
author: alkohli
services: storage
ms.service: storage
ms.topic: how-to
ms.date: 03/03/2021
ms.author: alkohli
ms.subservice: common
ms.custom: devx-track-azurepowershell, devx-track-azurecli, contperf-fy21q3
ms.openlocfilehash: 2d4885f23e775f84a412d176568d992ebe01166b
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107875708"
---
# <a name="use-the-azure-importexport-service-to-export-data-from-azure-blob-storage"></a>Uso del servicio Azure Import/Export para exportar datos de Azure Blob Storage

Este artículo proporciona instrucciones paso a paso sobre cómo usar el servicio Azure Import/Export para exportar de forma segura grandes cantidades de datos desde Azure Blob Storage. El servicio requiere que envíe unidades de disco vacías al centro de datos de Azure. El servicio exporta datos de la cuenta de almacenamiento a las unidades de disco y luego envía de vuelta las unidades de disco.

## <a name="prerequisites"></a>Requisitos previos

Antes de crear un trabajo de exportación para transferir datos fuera de Azure Blob Storage, revise con cuidado y complete la siguiente lista de requisitos previos para este servicio.
Debe:

- Tener una suscripción activa de Azure que pueda usarse para el servicio Import/Export.
- Tener al menos una cuenta de Azure Storage. Consulte la lista de [las cuenta de almacenamiento y los tipos de almacenamiento admitidos para el servicio Import/Export](storage-import-export-requirements.md). Para obtener información acerca de la creación de una nueva cuenta de almacenamiento, consulte [Creación de una cuenta de almacenamiento](../storage/common/storage-account-create.md).
- Tener un número suficiente de discos de los [tipos admitidos](storage-import-export-requirements.md#supported-disks).
- Tener una cuenta de FedEx o DHL. Si quiere usar alguna empresa de mensajería que no sea FedEx o DHL, póngase en contacto con el equipo de operaciones de Azure Data Box en `adbops@microsoft.com`.
  - La cuenta debe ser válida, debe tener saldo positivo y debe tener capacidades de devolución de envíos.
  - Generar un número de seguimiento del trabajo de exportación.
  - Cada trabajo debe tener un número de seguimiento independiente. No se admiten varios trabajos con el mismo número de seguimiento.
  - Si no tiene una cuenta de transportista, vaya a:
    - [Crear una cuenta de FedEx](https://www.fedex.com/en-us/create-account.html), o
    - [Crear una cuenta de DHL](http://www.dhl-usa.com/en/express/shipping/open_account.html).

## <a name="step-1-create-an-export-job"></a>Paso 1: Crear un trabajo de exportación

### <a name="portal"></a>[Portal](#tab/azure-portal)

Siga estos pasos para crear un trabajo de exportación en Azure Portal.

1. Inicie sesión en <https://portal.azure.com/>.
2. Busque los **trabajos de importación y exportación**.

    ![Búsqueda de los trabajos de importación y exportación](./media/storage-import-export-data-to-blobs/import-to-blob-1.png)

3. Seleccione **+ Nuevo**.

    ![Selección de + Nuevo para crear uno ](./media/storage-import-export-data-to-blobs/import-to-blob-2.png)

4. En **Aspectos básicos**:

   1. Seleccione una suscripción.
   1. Seleccione un grupo de recursos o elija **Crear nuevo** y cree uno.
   1. Escriba un nombre descriptivo para el trabajo de importación. Utilice el nombre para realizar un seguimiento del progreso de los trabajos.
       * El nombre solo pueden contener letras minúsculas, números y guiones.
       * El nombre tiene que empezar por una letra y no puede contener espacios.

   1. Seleccione **Export from Azure** (Exportar desde Azure).

    ![Opciones básicas de un pedido de exportación](./media/storage-import-export-data-from-blobs/export-from-blob-3.png)

    Seleccione **Siguiente: Detalles del trabajo >** para continuar.

5. En **Detalles del trabajo**:

   1. Seleccione la región de Azure en la que estén los datos en este momento.
   1. Seleccione la cuenta de Azure Storage desde la que quiere exportar los datos. Use una cuenta de almacenamiento cercana a su ubicación.

      La ubicación de la entrega se rellena automáticamente según la región de la cuenta de almacenamiento seleccionada.

   1. Especifique los datos de blobs que quiere exportar desde su cuenta de almacenamiento a una o varias unidades vacías. Elija uno de los tres métodos siguientes:

      - Elija **Exportar todos** los datos de blob de la cuenta de almacenamiento.

        ![Exportar todo](./media/storage-import-export-data-from-blobs/export-from-blob-4.png)

      - Elijas **Seleccionar contenedores y blobs** y especifique los contenedores y los blobs que se exportarán. Puede usar más de uno de los métodos de selección. Al seleccionar una opción **Agregar**, se abre un panel a la derecha donde puede agregar las cadenas de selección.

        |Opción|Descripción|
        |------|-----------|      
        |**Agregar contenedores**|Exporte todos los blobs de un contenedor.<br>Seleccione **Agregar contenedores** y escriba cada nombre de contenedor.|
        |**Agregar blobs**|Especifique cada uno de los blobs que se van a exportar.<br>Seleccione **Agregar blobs**. Luego, especifique la ruta de acceso relativa al blob, empezando con el nombre del contenedor. Utilice *$root* para especificar el contenedor raíz.<br>Tiene que proporcionar las rutas de acceso del blob en formato válido para evitar errores durante el procesamiento, tal y como se muestra en esta captura de pantalla. Para más información, consulte [Ejemplos de rutas de acceso del blob válidas](#examples-of-valid-blob-paths).|
        |**Agregar prefijos**|Use un prefijo para seleccionar un conjunto de contenedores con nombres parecidos o blobs con nombres parecidos en un contenedor. El prefijo puede ser el prefijo del nombre del contenedor, el nombre del contenedor completo o el nombre del contendor completo seguido del prefijo del nombre del blob. |

        ![Exportación de blobs y contenedores seleccionados](./media/storage-import-export-data-from-blobs/export-from-blob-5.png)

    - Elija **Exportar desde archivo de lista de blobs (formato XML)** y seleccione un archivo XML que contenga una lista de rutas de acceso y prefijos para los blobs que se van a exportar desde la cuenta de almacenamiento. Debe crear el archivo XML y almacenarlo en un contenedor de la cuenta de almacenamiento. El archivo no puede estar vacío.

      > [!IMPORTANT]
      > Si utiliza un archivo XML para seleccionar los blobs que se van a exportar, asegúrese de que el XML contenga rutas de acceso o prefijos válidos. Si el archivo no es válido o no hay ningún dato que coincida con las rutas de acceso especificadas, el pedido finaliza con una exportación de datos parcial o sin datos exportados.

       Para ver cómo agregar un archivo XML a un contenedor, consulte [Pedido de exportación mediante un archivo XML](../databox/data-box-deploy-export-ordered.md#export-order-using-xml-file).

      ![Exportación desde el archivo de lista de blobs](./media/storage-import-export-data-from-blobs/export-from-blob-6.png)

   > [!NOTE]
   > Si el blob que se va a exportar está en uso en el transcurso de la copia de datos, el servicio Azure Import/Export tomará una instantánea del blob y la copiará.

   Seleccione **Siguiente: Envío >** para continuar.

6. En **Envío**:

    - Seleccione el transportista en la lista desplegable. Si desea usar una empresa de mensajería que no sea FedEx o DHL, elija una de las opciones de la lista desplegable. Póngase en contacto con el equipo de operaciones de Azure Data Box en `adbops@microsoft.com` con la información relacionada con el transportista que quiere usar.
    - Escriba un número válido de cuenta de transportista que haya creado con ese transportista. Microsoft usa esta cuenta para devolverle las unidades una vez que haya finalizado el trabajo de exportación.
    - Proporcione la siguiente información completa y válida: nombre de contacto, teléfono, correo electrónico, dirección postal, ciudad, código postal, estado o provincia y país o región.

        > [!TIP]
        > En lugar de especificar una dirección de correo electrónico para un solo usuario, proporcione un correo electrónico de grupo. Esto garantiza que recibirá notificaciones incluso si sale un administrador.

    Seleccione **Revisar y crear** para continuar.

7. En **Revisar y crear**:

   1. Revise los detalles del trabajo.
   1. Anote el nombre del trabajo y proporcione la dirección de envío del centro de datos de Azure para el envío de discos a Azure.

      > [!NOTE]
      > Envíe siempre los discos al centro de datos que se ha indicado en Azure Portal. Si los discos se envían al centro de datos incorrecto, no se procesará el trabajo.

   1. Revise los **Términos** del pedido relativos a la privacidad y la eliminación de los datos de origen. Si acepta los términos, active la casilla que aparece debajo. Comienza la validación del pedido.

   ![Revisión y creación del pedido de exportación](./media/storage-import-export-data-from-blobs/export-from-blob-6-a.png)

 1. Una vez pasada la validación, seleccione **Crear**.

<!--Replaced text: Steps 4 - end of "Create an export job." Wizard design changes required both screen and text updates.

4. In **Basics**:

    - Select **Export from Azure**.
    - Enter a descriptive name for the export job. Use the name you choose to track the progress of your jobs.
        - The name may contain only lowercase letters, numbers, hyphens, and underscores.
        - The name must start with a letter, and may not contain spaces.
    - Select a subscription.
    - Enter or select a resource group.

        ![Basics](./media/storage-import-export-data-from-blobs/export-from-blob-3.png)

5. In **Job details**:

    - Select the storage account where the data to be exported resides. Use a storage account close to where you are located.
    - The dropoff location is automatically populated based on the region of the storage account selected.
    - Specify the blob data you wish to export from your storage account to your blank drive or drives.
    - Choose to **Export all** blob data in the storage account.

         ![Export all](./media/storage-import-export-data-from-blobs/export-from-blob-4.png)

    - You can specify which containers and blobs to export.
        - **To specify a blob to export**: Use the **Equal To** selector. Specify the relative path to the blob, beginning with the container name. Use *$root* to specify the root container.
        - **To specify all blobs starting with a prefix**: Use the **Starts With** selector. Specify the prefix, beginning with a forward slash '/'. The prefix may be the prefix of the container name, the complete container name, or the complete container name followed by the prefix of the blob name. You must provide the blob paths in valid format to avoid errors during processing, as shown in this screenshot. For more information, see [Examples of valid blob paths](#examples-of-valid-blob-paths).

           ![Export selected containers and blobs](./media/storage-import-export-data-from-blobs/export-from-blob-5.png)

    - You can export from  the blob list file.

        ![Export from blob list file](./media/storage-import-export-data-from-blobs/export-from-blob-6.png)

   > [!NOTE]
   > If the blob to be exported is in use during data copy, Azure Import/Export service takes a snapshot of the blob and copies the snapshot.

6. In **Return shipping info**:

    - Select the carrier from the dropdown list. If you want to use a carrier other than FedEx/DHL, choose an existing option from the dropdown. Contact Azure Data Box Operations team at `adbops@microsoft.com`  with the information regarding the carrier you plan to use.
    - Enter a valid carrier account number that you have created with that carrier. Microsoft uses this account to ship the drives back to you once your export job is complete.
    - Provide a complete and valid contact name, phone, email, street address, city, zip, state/province, and country/region.

        > [!TIP]
        > Instead of specifying an email address for a single user, provide a group email. This ensures that you receive notifications even if an admin leaves.

7. In **Summary**:

    - Review the details of the job.
    - Make a note of the job name and provided Azure datacenter shipping address for shipping disks to Azure.

        > [!NOTE]
        > Always send the disks to the datacenter noted in the Azure portal. If the disks are shipped to the wrong datacenter, the job will not be processed.

    - Click **OK** to complete export job creation.
-->

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Siga los pasos a continuación para crear un trabajo de exportación en Azure Portal.

[!INCLUDE [azure-cli-prepare-your-environment-h3.md](../../includes/azure-cli-prepare-your-environment-h3.md)]

### <a name="create-a-job"></a>Creación de un trabajo

1. Use el comando [az extension add](/cli/azure/extension#az_extension_add) para agregar la extensión [az import-export](/cli/azure/import-export):

    ```azurecli
    az extension add --name import-export
    ```

1. Para obtener una lista de las ubicaciones desde las que puede recibir discos, use el comando [az import-export location list](/cli/azure/import-export/location#az_import_export_location_list):

    ```azurecli
    az import-export location list
    ```

1. Ejecute el comando [az import-export create](/cli/azure/import-export#az_import_export_create) siguiente para crear un trabajo de exportación que use su cuenta de almacenamiento existente:

    ```azurecli
    az import-export create \
        --resource-group myierg \
        --name Myexportjob1 \
        --location "West US" \
        --backup-drive-manifest true \
        --diagnostics-path waimportexport \
        --export blob-path=/ \
        --type Export \
        --log-level Verbose \
        --shipping-information recipient-name="Microsoft Azure Import/Export Service" \
            street-address1="3020 Coronado" city="Santa Clara" state-or-province=CA postal-code=98054 \
            country-or-region=USA phone=4083527600 \
        --return-address recipient-name="Gus Poland" street-address1="1020 Enterprise way" \
            city=Sunnyvale country-or-region=USA state-or-province=CA postal-code=94089 \
            email=gus@contoso.com phone=4085555555" \
        --storage-account myssdocsstorage
    ```

    > [!TIP]
    > En lugar de especificar una dirección de correo electrónico para un solo usuario, proporcione un correo electrónico de grupo. Esto garantiza que recibirá notificaciones incluso si sale un administrador.

   Este trabajo exporta todos los blobs de la cuenta de almacenamiento. Puede especificar un blob para la exportación si reemplaza este valor por **--export**:

    ```azurecli
    --export blob-path=$root/logo.bmp
    ```

   Este valor de parámetro exporta el blob denominado *logo.bmp* en el contenedor raíz.

   También tiene la opción de seleccionar todos los blobs de un contenedor mediante un prefijo. Reemplace este valor por **--export**:

    ```azurecli
    blob-path-prefix=/myiecontainer
    ```

   Para más información, consulte [Ejemplos de rutas de acceso del blob válidas](#examples-of-valid-blob-paths).

   > [!NOTE]
   > Si el blob que se va a exportar está en uso en el transcurso de la copia de datos, el servicio Azure Import/Export tomará una instantánea del blob y la copiará.

1. Use el comando [az import-export list](/cli/azure/import-export#az_import_export_list) para ver todos los trabajos del grupo de recursos myierg:

    ```azurecli
    az import-export list --resource-group myierg
    ```

1. Para actualizar o cancelar el trabajo, ejecute el comando [az import-export update](/cli/azure/import-export#az_import_export_update):

    ```azurecli
    az import-export update --resource-group myierg --name MyIEjob1 --cancel-requested true
    ```

### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

Siga los pasos a continuación para crear un trabajo de exportación en Azure PowerShell.

[!INCLUDE [azure-powershell-requirements-h3.md](../../includes/azure-powershell-requirements-h3.md)]

> [!IMPORTANT]
> Aunque el módulo **Az.ImportExport** de PowerShell está en versión preliminar, se debe instalar por separado mediante el cmdlet `Install-Module`. Una vez que este módulo de PowerShell esté disponible con carácter general, formará parte de las futuras versiones del módulo Az de PowerShell y estará disponible de forma predeterminada en Azure Cloud Shell.

```azurepowershell-interactive
Install-Module -Name Az.ImportExport
```

### <a name="create-a-job"></a>Creación de un trabajo

1. Para obtener una lista de las ubicaciones desde las que puede recibir discos, use el cmdlet [Get-AzImportExportLocation](/powershell/module/az.importexport/get-azimportexportlocation):

   ```azurepowershell-interactive
   Get-AzImportExportLocation
   ```

1. Ejecute el ejemplo [New-AzImportExport](/powershell/module/az.importexport/new-azimportexport) siguiente para crear un trabajo de exportación que use su cuenta de almacenamiento existente:

   ```azurepowershell-interactive
   $Params = @{
      ResourceGroupName = 'myierg'
      Name = 'Myexportjob1'
      Location = 'westus'
      BackupDriveManifest = $true
      DiagnosticsPath = 'waimportexport'
      ExportBlobListblobPath = '\'
      JobType = 'Export'
      LogLevel = 'Verbose'
      ShippingInformationRecipientName = 'Microsoft Azure Import/Export Service'
      ShippingInformationStreetAddress1 = '3020 Coronado'
      ShippingInformationCity = 'Santa Clara'
      ShippingInformationStateOrProvince = 'CA'
      ShippingInformationPostalCode = '98054'
      ShippingInformationCountryOrRegion = 'USA'
      ShippingInformationPhone = '4083527600'
      ReturnAddressRecipientName = 'Gus Poland'
      ReturnAddressStreetAddress1 = '1020 Enterprise way'
      ReturnAddressCity = 'Sunnyvale'
      ReturnAddressStateOrProvince = 'CA'
      ReturnAddressPostalCode = '94089'
      ReturnAddressCountryOrRegion = 'USA'
      ReturnAddressPhone = '4085555555'
      ReturnAddressEmail = 'gus@contoso.com'
      StorageAccountId = '/subscriptions/<SubscriptionId>/resourceGroups/myierg/providers/Microsoft.Storage/storageAccounts/myssdocsstorage'
   }
   New-AzImportExport @Params
   ```

    > [!TIP]
    > En lugar de especificar una dirección de correo electrónico para un solo usuario, proporcione un correo electrónico de grupo. Esto garantiza que recibirá notificaciones incluso si sale un administrador.

   Este trabajo exporta todos los blobs de la cuenta de almacenamiento. Puede especificar un blob para la exportación si reemplaza este valor por **-ExportBlobListblobPath**:

   ```azurepowershell-interactive
   -ExportBlobListblobPath $root\logo.bmp
   ```

   Este valor de parámetro exporta el blob denominado *logo.bmp* en el contenedor raíz.

   También tiene la opción de seleccionar todos los blobs de un contenedor mediante un prefijo. Reemplace este valor por **-ExportBlobListblobPath**:

   ```azurepowershell-interactive
   -ExportBlobListblobPath '/myiecontainer'
   ```

   Para más información, consulte [Ejemplos de rutas de acceso del blob válidas](#examples-of-valid-blob-paths).

   > [!NOTE]
   > Si el blob que se va a exportar está en uso en el transcurso de la copia de datos, el servicio Azure Import/Export tomará una instantánea del blob y la copiará.

1. Use el cmdlet [Get-AzImportExport](/powershell/module/az.importexport/get-azimportexport) para ver todos los trabajos del grupo de recursos de myierg:

   ```azurepowershell-interactive
   Get-AzImportExport -ResourceGroupName myierg
   ```

1. Para actualizar el trabajo o cancelarlo, ejecute el cmdlet [Update-AzImportExport](/powershell/module/az.importexport/update-azimportexport):

   ```azurepowershell-interactive
   Update-AzImportExport -Name MyIEjob1 -ResourceGroupName myierg -CancelRequested
   ```

---

<!--## (Optional) Step 2: -->

## <a name="step-2-ship-the-drives"></a>Paso 2: Envío de las unidades de disco

Si no conoce el número de unidades de disco que necesita, vaya a [Comprobación del número de unidades de disco](#check-the-number-of-drives). Si conoce el número de unidades, continúe con el envío de las unidades.

[!INCLUDE [storage-import-export-ship-drives](../../includes/storage-import-export-ship-drives.md)]

## <a name="step-3-update-the-job-with-tracking-information"></a>Paso 3: Actualización del trabajo con información de seguimiento

[!INCLUDE [storage-import-export-update-job-tracking](../../includes/storage-import-export-update-job-tracking.md)]

## <a name="step-4-receive-the-disks"></a>Paso 4: Recepción de los discos

Cuando el panel notifica que el trabajo está completo, se le envían los discos y el número de seguimiento para el envío está disponible en el portal.

1. Después de que haya recibido las unidades de disco con los datos exportados, tiene que obtener las claves de BitLocker para desbloquear las unidades de disco. Vaya al trabajo de exportación en Azure Portal. Haga clic en la pestaña **Import/Export**.
2. Seleccione el trabajo de exportación de la lista y haga clic en él. Vaya a **Cifrado** y copie las claves.

   ![Visualización de claves de BitLocker de un trabajo de exportación](./media/storage-import-export-data-from-blobs/export-from-blob-7.png)

3. Use las claves de BitLocker para desbloquear los discos.

La exportación se ha terminado.

## <a name="step-5-unlock-the-disks"></a>Paso 5: Desbloqueo de los discos

Escriba el siguiente comando para desbloquear la unidad:

   `WAImportExport Unlock /bk:<BitLocker key (base 64 string) copied from Encryption blade in Azure portal> /driveLetter:<Drive letter>`

Este es un ejemplo de la entrada.

   `WAImportExport.exe Unlock /bk:CAAcwBoAG8AdQBsAGQAIABiAGUAIABoAGkAZABkAGUAbgA= /driveLetter:e`

En este momento, puede eliminar el trabajo o dejarlo. Los trabajos se eliminan automáticamente después de 90 días.

## <a name="check-the-number-of-drives"></a>Comprobación del número de unidades de disco

Este paso *opcional* le ayuda a determinar el número de unidades necesarias para el trabajo de exportación. Realice este paso en un sistema de Windows que ejecute una [versión admitida del sistema operativo](storage-import-export-requirements.md#supported-operating-systems).

1. [Descargar la versión 1 de WAImportExport](https://www.microsoft.com/download/details.aspx?id=42659) en el sistema de Windows.
2. Descomprima en la carpeta predeterminada `waimportexportv1`. Por ejemplo, `C:\WaImportExportV1`.
3. Abra una ventana de PowerShell o de línea de comandos con privilegios administrativos. Para cambiar el directorio a la carpeta descomprimida, ejecute el siguiente comando:

   `cd C:\WaImportExportV1`

4. Para comprobar el número de discos necesarios para los blobs seleccionados, ejecute el siguiente comando:

   `WAImportExport.exe PreviewExport /sn:<Storage account name> /sk:<Storage account key> /ExportBlobListFile:<Path to XML blob list file> /DriveSize:<Size of drives used>`

    Los parámetros se describen en la tabla siguiente:

    |Parámetro de línea de comandos|Descripción|
    |--------------------------|-----------------|
    |**/logdir:**|Opcional. Directorio de registro. Los archivos de registro detallados se escriben en este directorio. Si no se especifica, se usa el directorio actual como directorio de registro.|
    |**/sn:**|Necesario. El nombre de la cuenta de almacenamiento para el trabajo de exportación.|
    |**/sk:**|Necesario únicamente si no se especifica un SAS del contenedor. La clave de cuenta para la cuenta de almacenamiento correspondiente al trabajo de exportación.|
    |**/csas:**|Necesario únicamente si no se especifica una clave de cuenta de almacenamiento. El contenedor SAS para enumerar los blobs que se van a exportar en el trabajo de exportación.|
    |**/ExportBlobListFile:**|Necesario. Ruta de acceso al archivo XML que contiene una lista de rutas de acceso de blob o prefijos de ruta de acceso para los blobs que se van a exportar. El formato de archivo usado en el elemento `BlobListBlobPath` de la operación [Put Job](/rest/api/storageimportexport/jobs) de la API de REST del servicio Import/Export.|
    |**/DriveSize:**|Necesario. El tamaño de las unidades que se van a usar para un trabajo de exportación, *por ejemplo*, 500 GB o 1,5 TB.|

    Vea el [Ejemplo del comando PreviewExport](#example-of-previewexport-command).

5. Compruebe que puede leer o escribir en las unidades de disco que se van a enviar para el trabajo de exportación.

### <a name="example-of-previewexport-command"></a>Ejemplo del comando PreviewExport

En el siguiente ejemplo se muestra el comando `PreviewExport`:

```powershell
    WAImportExport.exe PreviewExport /sn:bobmediaaccount /sk:VkGbrUqBWLYJ6zg1m29VOTrxpBgdNOlp+kp0C9MEdx3GELxmBw4hK94f7KysbbeKLDksg7VoN1W/a5UuM2zNgQ== /ExportBlobListFile:C:\WAImportExport\mybloblist.xml /DriveSize:500GB
```

El archivo de lista de blobs de exportación puede contener nombres de blob y prefijos de blob, como se muestra aquí:

```xml
<?xml version="1.0" encoding="utf-8"?>
<BlobList>
<BlobPath>pictures/animals/koala.jpg</BlobPath>
<BlobPathPrefix>/vhds/</BlobPathPrefix>
<BlobPathPrefix>/movies/</BlobPathPrefix>
</BlobList>
```

La herramienta Azure Import/Export enumera todos los blobs que se van a exportar y calcula cómo empaquetarlos en unidades del tamaño especificado, teniendo en cuenta cualquier sobrecarga necesaria; después calcula el número de unidades necesarias para albergar los blobs y la información de uso de la unidad.

Este es un ejemplo de la salida, con los registros informativos omitidos:

```powershell
Number of unique blob paths/prefixes:   3
Number of duplicate blob paths/prefixes:        0
Number of nonexistent blob paths/prefixes:      1

Drive size:     500.00 GB
Number of blobs that can be exported:   6
Number of blobs that cannot be exported:        2
Number of drives needed:        3
        Drive #1:       blobs = 1, occupied space = 454.74 GB
        Drive #2:       blobs = 3, occupied space = 441.37 GB
        Drive #3:       blobs = 2, occupied space = 131.28 GB
```

## <a name="examples-of-valid-blob-paths"></a>Ejemplos de rutas de acceso del blob válidas

En la siguiente tabla se muestran ejemplos de rutas de acceso del blob válidas:

   | Selector | Ruta del blob | Descripción |
   | --- | --- | --- |
   | Empieza por |/ |Exporta todos los blobs de la cuenta de almacenamiento. |
   | Empieza por |/$root/ |Exporta todos los blobs del contenedor raíz. |
   | Empieza por |/book |Exporta todos los blobs de cualquier contenedor que empiecen por el prefijo **book** |
   | Empieza por |/music/ |Exporta todos los blobs del contenedor **music** |
   | Empieza por |/music/love |Exporta todos los blobs del contenedor **music** que empiecen por el prefijo **love**. |
   | Igual a |$root/logo.bmp |Exporta el blob **logo.bmp** del contenedor raíz. |
   | Igual a |videos/story.mp4 |Exporta el blob **story.mp4** del contenedor **videos** |

## <a name="next-steps"></a>Pasos siguientes

- [Visualización del estado del trabajo y de la unidad de disco](storage-import-export-view-drive-status.md)
- [Revisión de los requisitos de Import/Export](storage-import-export-requirements.md)
