---
title: Uso de Azure Import/Export para transferir datos a Azure Files | Microsoft Docs
description: Aprenda a crear trabajos de importación en Azure Portal para transferir datos a Azure Files.
author: alkohli
services: storage
ms.service: storage
ms.topic: how-to
ms.date: 03/03/2021
ms.author: alkohli
ms.subservice: common
ms.custom: devx-track-azurepowershell, devx-track-azurecli, contperf-fy21q3
ms.openlocfilehash: 9c13ffc597349cdd2b304889d142ca7c2f89c713
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107861542"
---
# <a name="use-azure-importexport-service-to-import-data-to-azure-files"></a>Uso del servicio Azure Import/Export para importar datos a Azure Files

Este artículo proporciona instrucciones paso a paso sobre cómo usar el servicio Azure Import/Export para importar de forma segura grandes cantidades de datos en Azure Files. Para importar datos, el servicio necesita que envíe las unidades de disco admitidas que contiene los datos a un centro de datos de Azure.

El servicio Import/Export admite solo la importación de Azure Files en Azure Storage. No se admite la exportación de archivos desde Azure Files.

## <a name="prerequisites"></a>Prerrequisitos

Antes de crear un trabajo de importación para transferir datos en Azure Files, revise con cuidado y complete la siguiente lista de requisitos previos. Debe:

- Tener una suscripción activa de Azure para usar con el servicio Import/Export.
- Tener al menos una cuenta de Azure Storage. Consulte la lista de [las cuenta de almacenamiento y los tipos de almacenamiento admitidos para el servicio Import/Export](storage-import-export-requirements.md). Para obtener información acerca de la creación de una nueva cuenta de almacenamiento, consulte [Creación de una cuenta de almacenamiento](../storage/common/storage-account-create.md).
- Tener un número suficiente de discos de los [tipos admitidos](storage-import-export-requirements.md#supported-disks).
- Tener un sistema de Windows que ejecute una [versión admitida del sistema operativo](storage-import-export-requirements.md#supported-operating-systems).
- [Descargar la versión 2 de WAImportExport](https://aka.ms/waiev2) en el sistema de Windows. Descomprima en la carpeta predeterminada `waimportexport`. Por ejemplo, `C:\WaImportExport`.
- Tener una cuenta de FedEx o DHL. Si quiere usar alguna empresa de mensajería que no sea FedEx o DHL, póngase en contacto con el equipo de operaciones de Azure Data Box en `adbops@microsoft.com`.
    - La cuenta debe ser válida, debe tener saldo positivo y debe tener capacidades de devolución de envíos.
    - Generar un número de seguimiento del trabajo de exportación.
    - Cada trabajo debe tener un número de seguimiento independiente. No se admiten varios trabajos con el mismo número de seguimiento.
    - Si no tiene una cuenta de transportista, vaya a:
        - [Crear una cuenta de FedEx](https://www.fedex.com/en-us/create-account.html), o
        - [Crear una cuenta de DHL](http://www.dhl-usa.com/en/express/shipping/open_account.html).


## <a name="step-1-prepare-the-drives"></a>Paso 1: Preparación de las unidades de disco

Este paso genera un archivo de diario. El archivo de diario almacena información básica como el número de serie de unidad, la clave de cifrado y los detalles de la cuenta de almacenamiento.

Para preparar las unidades, realice los pasos siguientes:

1. Conecte las unidades de disco al sistema de Windows a través de los conectores SATA.
2. Cree un volumen NTFS único en cada unidad. Asigne una letra de unidad al volumen. No utilice puntos de montaje.
3. Modifique el archivo *dataset.csv* en la carpeta raíz donde reside la herramienta. Dependiendo de si desea importar un archivo o carpeta o ambos, agregue las entradas en el archivo *dataset.csv* de forma similar a los siguientes ejemplos.

   - **Para importar un archivo**: en el ejemplo siguiente, los datos que se van a copiar residen en la unidad F: El archivo *MyFile1.txt* se copia en la raíz de *MyAzureFileshare1*. Si *MyAzureFileshare1* no existe, se crea en la cuenta de Azure Storage. Se mantiene la estructura de carpetas.

       ```
           BasePath,DstItemPathOrPrefix,ItemType,Disposition,MetadataFile,PropertiesFile
           "F:\MyFolder1\MyFile1.txt","MyAzureFileshare1/MyFile1.txt",file,rename,"None",None

       ```
   - **Para importar una carpeta**: todos los archivos y carpetas bajo *MyFolder2* se copiarán en el recurso compartido de archivos de forma recursiva. Se mantiene la estructura de carpetas.

       ```
           "F:\MyFolder2\","MyAzureFileshare1/",file,rename,"None",None

       ```
     Se pueden realizar varias entradas en el mismo archivo correspondiente a las carpetas o archivos que se importan.

       ```
           "F:\MyFolder1\MyFile1.txt","MyAzureFileshare1/MyFile1.txt",file,rename,"None",None
           "F:\MyFolder2\","MyAzureFileshare1/",file,rename,"None",None

       ```
     Obtenga más información sobre cómo [preparar el archivo CSV de conjunto de datos](/previous-versions/azure/storage/common/storage-import-export-tool-preparing-hard-drives-import).


4. Modifique el archivo *driveset.csv* en la carpeta raíz donde reside la herramienta. Agregar entradas en el archivo *driveset.csv* similares a las de los siguientes ejemplos. El archivo driveset tiene la lista de discos y las letras de unidad de disco correspondiente para que la herramienta pueda elegir correctamente la lista de discos que se van a preparar.

    En este ejemplo se da por supuesto que están conectados dos discos y que se crean volúmenes NTFS básicos G:\ y H:\. H:\no está cifrado mientras G: ya está cifrado. La herramienta da formato y cifra el disco que hospeda solo H:\ (y no G:\).

   - **Para un disco que no está cifrado**: especificar *Encrypt* para habilitar el cifrado de BitLocker en el disco.

       ```
       DriveLetter,FormatOption,SilentOrPromptOnFormat,Encryption,ExistingBitLockerKey
       H,Format,SilentMode,Encrypt,
       ```

   - **Para un disco que ya está cifrado**: especificar *AlreadyEncrypted* y proporcionar la clave de BitLocker.

       ```
       DriveLetter,FormatOption,SilentOrPromptOnFormat,Encryption,ExistingBitLockerKey
       G,AlreadyFormatted,SilentMode,AlreadyEncrypted,060456-014509-132033-080300-252615-584177-672089-411631
       ```

     Se pueden realizar varias entradas en el mismo archivo que correspondan a varias unidades de disco. Obtenga más información sobre cómo [preparar el archivo CSV de conjunto de unidades](/previous-versions/azure/storage/common/storage-import-export-tool-preparing-hard-drives-import).

5. Use la opción `PrepImport` para copiar y preparar los datos para la unidad de disco. Para la primera sesión de copia para copiar directorios o archivos con una nueva sesión de copia, ejecute el siguiente comando:

    ```cmd
    .\WAImportExport.exe PrepImport /j:<JournalFile> /id:<SessionId> [/logdir:<LogDirectory>] [/sk:<StorageAccountKey>] [/silentmode] [/InitialDriveSet:<driveset.csv>]/DataSet:<dataset.csv>
    ```

   A continuación se muestra un ejemplo de importación.

    ```cmd
    .\WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#1  /sk:************* /InitialDriveSet:driveset.csv /DataSet:dataset.csv /logdir:C:\logs
    ```

6. Se crea un archivo de diario con el nombre proporcionado con el parámetro `/j:` para cada ejecución de la línea de comandos. Cada unidad que prepare tiene un archivo de diario que se tiene que cargar cuando se crea el trabajo de importación. Las unidades sin archivos de diario no se procesan.

    > [!IMPORTANT]
    > - No modifique los datos de las unidades de disco o el archivo de diario después de completar la preparación del disco.

Para ejemplos adicionales, vaya a [Ejemplos para archivos de diario](#samples-for-journal-files).

## <a name="step-2-create-an-import-job"></a>Paso 2: Crear un trabajo de importación

### <a name="portal"></a>[Portal](#tab/azure-portal)

Siga estos pasos para crear un trabajo de importación en Azure Portal.
1. Inicie sesión en https://portal.azure.com/.
2. Busque los **trabajos de importación y exportación**.

    ![Buscar en trabajos de importación y exportación](./media/storage-import-export-data-to-blobs/import-to-blob-1.png)

3. Seleccione **+ Nuevo**.

    ![Seleccione Nuevo para crear uno nuevo. ](./media/storage-import-export-data-to-blobs/import-to-blob-2.png)

4. En **Aspectos básicos**:

   1. Seleccione una suscripción.
   1. Seleccione un grupo de recursos o elija **Crear nuevo** y cree uno.
   1. Escriba un nombre descriptivo para el trabajo de importación. Utilice el nombre para realizar un seguimiento del progreso de los trabajos.
       * El nombre solo pueden contener letras minúsculas, números y guiones.
       * El nombre tiene que empezar por una letra y no puede contener espacios.
   1. Seleccione **Import into Azure** (Importar en Azure).

    ![Creación del trabajo de importación: Paso 1](./media/storage-import-export-data-to-blobs/import-to-blob-3.png)

   Seleccione **Siguiente: Detalles del trabajo >** para continuar.

5. En **Detalles del trabajo**:

   1. Cargar los archivos de diario que creó con anterioridad en [Paso 1: Preparación de las unidades de disco](#step-1-prepare-the-drives).
   1. Seleccione la región de Azure de destino del pedido.
   1. Seleccione la cuenta de almacenamiento de la importación.

      La ubicación de la entrega se rellena automáticamente según la región de la cuenta de almacenamiento seleccionada.

   1. Si no quiere guardar un registro detallado, desactive la opción **Guardar registro detallado en el contenedor de blobs 'waimportexport'** .


   ![Creación del trabajo de importación: Paso 2](./media/storage-import-export-data-to-blobs/import-to-blob-4.png)

   Seleccione **Siguiente: Envío >** para continuar.

4. En **Envío**:

    1. Seleccione el transportista en la lista desplegable. Si desea usar una empresa de mensajería que no sea FedEx o DHL, elija una de las opciones de la lista desplegable. Póngase en contacto con el equipo de operaciones de Azure Data Box en `adbops@microsoft.com` e indíqueles el transportista que quiere usar.
    1. Escriba un número válido de cuenta de transportista que haya creado con ese transportista. Microsoft usa esta cuenta para devolverle las unidades una vez que haya finalizado el trabajo de importación.
    1. Proporcione información completa y válida del contacto: nombre, teléfono, correo electrónico, dirección postal, ciudad, código postal, estado o provincia y país o región.

        > [!TIP]
        > En lugar de especificar una dirección de correo electrónico para un solo usuario, proporcione un correo electrónico de grupo. Esto garantiza que recibirá notificaciones incluso si sale un administrador.

    ![Creación del trabajo de importación - Paso 3](./media/storage-import-export-data-to-blobs/import-to-blob-5.png)

   Seleccione **Revisar y crear** para continuar.

5. En el resumen del pedido:

   1. Revise los **Términos** y, luego, seleccione "Reconozco que toda la información proporcionada es correcta y estoy de acuerdo con los términos y las condiciones anteriores". A continuación, se realiza la validación.
   1. Revise la información de trabajo proporcionada en el resumen. Anote el nombre del trabajo y la dirección de envío del centro de datos Azure para enviar discos a Azure. Esta información se utiliza posteriormente en la etiqueta de envío.
   1. Seleccione **Crear**.

        ![Creación del trabajo de importación: Paso 4](./media/storage-import-export-data-to-blobs/import-to-blob-6.png)

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Use los pasos siguientes para crear un trabajo de importación en la CLI de Azure.

[!INCLUDE [azure-cli-prepare-your-environment-h3.md](../../includes/azure-cli-prepare-your-environment-h3.md)]

### <a name="create-a-job"></a>Creación de un trabajo

1. Use el comando [az extension add](/cli/azure/extension#az_extension_add) para agregar la extensión [az import-export](/cli/azure/import-export):

    ```azurecli
    az extension add --name import-export
    ```

1. Puede usar un grupo de recursos existente o crear uno. Para crear un grupo de recursos, ejecute el comando [az group create](/cli/azure/group#az_group_create):

    ```azurecli
    az group create --name myierg --location "West US"
    ```

1. Puede usar una cuenta de almacenamiento existente o crear una. Para crear una cuenta de almacenamiento, ejecute el comando [az storage account create](/cli/azure/storage/account#az_storage_account_create):

    ```azurecli
    az storage account create -resource-group myierg -name myssdocsstorage --https-only
    ```

1. Para obtener una lista de las ubicaciones a las que puede enviar discos, use el comando [az import-export location list](/cli/azure/import-export/location#az_import_export_location_list):

    ```azurecli
    az import-export location list
    ```

1. Use el comando [az import-export location show](/cli/azure/import-export/location#az_import_export_location_show) para obtener las ubicaciones de su región:

    ```azurecli
    az import-export location show --location "West US"
    ```

1. Ejecute el siguiente comando [az import-export create](/cli/azure/import-export#az_import_export_create) para crear un trabajo de importación:

    ```azurecli
    az import-export create \
        --resource-group myierg \
        --name MyIEjob1 \
        --location "West US" \
        --backup-drive-manifest true \
        --diagnostics-path waimportexport \
        --drive-list bit-locker-key=439675-460165-128202-905124-487224-524332-851649-442187 \
            drive-header-hash= drive-id=AZ31BGB1 manifest-file=\\DriveManifest.xml \
            manifest-hash=69512026C1E8D4401816A2E5B8D7420D \
        --type Import \
        --log-level Verbose \
        --shipping-information recipient-name="Microsoft Azure Import/Export Service" \
            street-address1="3020 Coronado" city="Santa Clara" state-or-province=CA postal-code=98054 \
            country-or-region=USA phone=4083527600 \
        --return-address recipient-name="Gus Poland" street-address1="1020 Enterprise way" \
            city=Sunnyvale country-or-region=USA state-or-province=CA postal-code=94089 \
            email=gus@contoso.com phone=4085555555" \
        --return-shipping carrier-name=FedEx carrier-account-number=123456789 \
        --storage-account myssdocsstorage
    ```

   > [!TIP]
   > En lugar de especificar una dirección de correo electrónico para un solo usuario, proporcione un correo electrónico de grupo. Esto garantiza que recibirá notificaciones incluso si sale un administrador.


1. Use el comando [az import-export list](/cli/azure/import-export#az_import_export_list) para ver todos los trabajos del grupo de recursos de myierg:

    ```azurecli
    az import-export list --resource-group myierg
    ```

1. Para actualizar o cancelar el trabajo, ejecute el comando [az import-export update](/cli/azure/import-export#az_import_export_update):

    ```azurecli
    az import-export update --resource-group myierg --name MyIEjob1 --cancel-requested true
    ```

### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

Use los pasos siguientes para crear un trabajo de importación en Azure PowerShell.

[!INCLUDE [azure-powershell-requirements-h3.md](../../includes/azure-powershell-requirements-h3.md)]

> [!IMPORTANT]
> Aunque el módulo **Az.ImportExport** de PowerShell está en versión preliminar, se debe instalar por separado mediante el cmdlet `Install-Module`. Una vez que este módulo de PowerShell esté disponible con carácter general, formará parte de las futuras versiones del módulo Az de PowerShell y estará disponible de forma predeterminada en Azure Cloud Shell.

```azurepowershell-interactive
Install-Module -Name Az.ImportExport
```

### <a name="create-a-job"></a>Creación de un trabajo

1. Puede usar un grupo de recursos existente o crear uno. Para crear un grupo de recursos, ejecute el cmdlet [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup):

   ```azurepowershell-interactive
   New-AzResourceGroup -Name myierg -Location westus
   ```

1. Puede usar una cuenta de almacenamiento existente o crear una. Para crear una cuenta de almacenamiento, ejecute el cmdlet [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount):

   ```azurepowershell-interactive
   New-AzStorageAccount -ResourceGroupName myierg -AccountName myssdocsstorage -SkuName Standard_RAGRS -Location westus -EnableHttpsTrafficOnly $true
   ```

1. Para obtener una lista de las ubicaciones a las que puede enviar discos, use el cmdlet [Get-AzImportExportLocation](/powershell/module/az.importexport/get-azimportexportlocation):

   ```azurepowershell-interactive
   Get-AzImportExportLocation
   ```

1. Use el cmdlet `Get-AzImportExportLocation` con el parámetro `Name` para obtener las ubicaciones para la región:

   ```azurepowershell-interactive
   Get-AzImportExportLocation -Name westus
   ```

1. Ejecute el siguiente ejemplo de [New-AzImportExport](/powershell/module/az.importexport/new-azimportexport) para crear un trabajo de importación:

   ```azurepowershell-interactive
   $driveList = @(@{
     DriveId = '9CA995BA'
     BitLockerKey = '439675-460165-128202-905124-487224-524332-851649-442187'
     ManifestFile = '\\DriveManifest.xml'
     ManifestHash = '69512026C1E8D4401816A2E5B8D7420D'
     DriveHeaderHash = 'AZ31BGB1'
   })

   $Params = @{
      ResourceGroupName = 'myierg'
      Name = 'MyIEjob1'
      Location = 'westus'
      BackupDriveManifest = $true
      DiagnosticsPath = 'waimportexport'
      DriveList = $driveList
      JobType = 'Import'
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
      ReturnShippingCarrierName = 'FedEx'
      ReturnShippingCarrierAccountNumber = '123456789'
      StorageAccountId = '/subscriptions/<SubscriptionId>/resourceGroups/myierg/providers/Microsoft.Storage/storageAccounts/myssdocsstorage'
   }
   New-AzImportExport @Params
   ```

   > [!TIP]
   > En lugar de especificar una dirección de correo electrónico para un solo usuario, proporcione un correo electrónico de grupo. Esto garantiza que recibirá notificaciones incluso si sale un administrador.

1. Use el cmdlet [Get-AzImportExport](/powershell/module/az.importexport/get-azimportexport) para ver todos los trabajos del grupo de recursos de myierg:

   ```azurepowershell-interactive
   Get-AzImportExport -ResourceGroupName myierg
   ```

1. Para actualizar o cancelar el trabajo, ejecute el cmdlet [Update-AzImportExport](/powershell/module/az.importexport/update-azimportexport):

   ```azurepowershell-interactive
   Update-AzImportExport -Name MyIEjob1 -ResourceGroupName myierg -CancelRequested
   ```

---

## <a name="step-3-ship-the-drives-to-the-azure-datacenter"></a>Paso 3: Envío de las unidades de disco al centro de datos de Azure

[!INCLUDE [storage-import-export-ship-drives](../../includes/storage-import-export-ship-drives.md)]

## <a name="step-4-update-the-job-with-tracking-information"></a>Paso 4: Actualización del trabajo con información de seguimiento

[!INCLUDE [storage-import-export-update-job-tracking](../../includes/storage-import-export-update-job-tracking.md)]

## <a name="step-5-verify-data-upload-to-azure"></a>Paso 5: Verificación de la carga de datos en Azure

Siga el trabajo hasta su finalización. Una vez que se haya finalizado el trabajo, compruebe que los datos se han cargado en Azure. Elimine los datos locales solo después de comprobar que la carga fue correcta.

## <a name="samples-for-journal-files"></a>Ejemplos para archivos de diario

Para **agregar más unidades**, cree un nuevo archivo de conjunto de unidades y ejecute el comando como se indica a continuación.

En las sesiones de copia posteriores en unidades de disco distintas a las especificadas en el archivo *InitialDriveset.csv*, indique un nuevo archivo *.csv* de conjunto de unidades y proporciónelo como valor del parámetro `AdditionalDriveSet`. Use el **mismo nombre de archivo de diario** y proporcione un **nuevo identificador de sesión**. El formato del archivo CSV AdditionalDriveset es el mismo que el formato de InitialDriveSet.

```cmd
WAImportExport.exe PrepImport /j:<JournalFile> /id:<SessionId> /AdditionalDriveSet:<driveset.csv>
```

A continuación se muestra un ejemplo de importación.

```cmd
WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#3  /AdditionalDriveSet:driveset-2.csv
```


Para agregar datos adicionales al mismo conjunto de unidades, use el comando PrepImport para sesiones de copia posteriores para copiar archivos o directorios adicionales.

En las sucesivas sesiones de copia en las mismas unidades de disco duro especificadas en el archivo *InitialDriveset.csv*, especifique el mismo nombre **de archivo de diario** e indique un **nuevo identificador de sesión**. No hace falta proporcionar la clave de la cuenta de almacenamiento.

```cmd
WAImportExport PrepImport /j:<JournalFile> /id:<SessionId> /j:<JournalFile> /id:<SessionId> [/logdir:<LogDirectory>] DataSet:<dataset.csv>
```

A continuación se muestra un ejemplo de importación.

```cmd
WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#2  /DataSet:dataset-2.csv
```

## <a name="next-steps"></a>Pasos siguientes

* [Visualización del estado del trabajo y de la unidad de disco](storage-import-export-view-drive-status.md)
* [Revisión de los requisitos de Import/Export](storage-import-export-requirements.md)