---
title: Uso de Azure Import/Export para transferir datos a blobs de Azure | Microsoft Docs
description: Aprenda a crear trabajos de importación y exportación en Azure Portal para transferir datos a los blobs de Azure y recibirlos de estos.
author: alkohli
services: storage
ms.service: storage
ms.topic: how-to
ms.date: 03/15/2021
ms.author: alkohli
ms.subservice: common
ms.custom: devx-track-azurepowershell, devx-track-azurecli, contperf-fy21q3
ms.openlocfilehash: 39eb6c164751ebdfa293798850a8d663fe988b82
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107875690"
---
# <a name="use-the-azure-importexport-service-to-import-data-to-azure-blob-storage"></a>Uso del servicio Azure Import/Export para importar datos de Azure Blob Storage

Este artículo proporciona instrucciones paso a paso sobre cómo usar el servicio Azure Import/Export para importar de forma segura grandes cantidades de datos a Azure Blob Storage. Para importar datos en los blobs de Azure, el servicio necesita que envíe las unidades de disco cifradas que contienen los datos a un centro de datos de Azure.

## <a name="prerequisites"></a>Prerrequisitos

Antes de crear un trabajo de importación para transferir datos a Azure Blob Storage, revise con cuidado y complete la siguiente lista de requisitos previos para este servicio.
Debe:

* Tener una suscripción activa de Azure que pueda usarse para el servicio Import/Export.
* Tener por lo menos una cuenta de Azure Storage con un contenedor de almacenamiento. Consulte la lista de [las cuenta de almacenamiento y los tipos de almacenamiento admitidos para el servicio Import/Export](storage-import-export-requirements.md).
  * Para obtener información acerca de la creación de una nueva cuenta de almacenamiento, consulte [Creación de una cuenta de almacenamiento](../storage/common/storage-account-create.md).
  * Para información sobre contenedores de almacenamiento, vaya a [Creación de un contenedor de almacenamiento](../storage/blobs/storage-quickstart-blobs-portal.md#create-a-container).
* Tener un número suficiente de discos de los [tipos admitidos](storage-import-export-requirements.md#supported-disks).
* Tener un sistema de Windows que ejecute una [versión admitida del sistema operativo](storage-import-export-requirements.md#supported-operating-systems).
* Habilitar BitLocker en el sistema de Windows. Consulte [Cómo habilitar BitLocker](https://thesolving.com/storage/how-to-enable-bitlocker-on-windows-server-2012-r2/).
* [Descargar la versión 1 más reciente de WAImportExport](https://www.microsoft.com/download/details.aspx?id=42659) en el sistema Windows. La versión más reciente de la herramienta tiene actualizaciones de seguridad para permitir un protector externo para la clave de BitLocker, así como la característica actualizada del modo de desbloqueo.

  * Descomprima en la carpeta predeterminada `waimportexportv1`. Por ejemplo, `C:\WaImportExportV1`.
* Tener una cuenta de FedEx o DHL. Si quiere usar alguna empresa de mensajería que no sea FedEx o DHL, póngase en contacto con el equipo de operaciones de Azure Data Box en `adbops@microsoft.com`.
  * La cuenta debe ser válida, debe tener saldo positivo y debe tener capacidades de devolución de envíos.
  * Generar un número de seguimiento del trabajo de exportación.
  * Cada trabajo debe tener un número de seguimiento independiente. No se admiten varios trabajos con el mismo número de seguimiento.
  * Si no tiene una cuenta de transportista, vaya a:
    * [Crear una cuenta de FedEx](https://www.fedex.com/en-us/create-account.html), o
    * [Crear una cuenta de DHL](http://www.dhl-usa.com/en/express/shipping/open_account.html).

## <a name="step-1-prepare-the-drives"></a>Paso 1: Preparación de las unidades de disco

Este paso genera un archivo de diario. El archivo de diario almacena información básica como el número de serie de unidad, la clave de cifrado y los detalles de la cuenta de almacenamiento.

Realice los pasos siguientes para preparar las unidades de disco.

1. Conecte las unidades de disco al sistema de Windows a través de los conectores SATA.
2. Cree un volumen NTFS único en cada unidad. Asigne una letra de unidad al volumen. No utilice puntos de montaje.
3. Habilite el cifrado de BitLocker en el volumen NTFS. Si usa un sistema de Windows Server, siga las instrucciones [How to enable BitLocker on Windows Server 2012 R2](https://thesolving.com/storage/how-to-enable-bitlocker-on-windows-server-2012-r2/) (Cómo habilitar BitLocker en Windows Server 2012 R2).
4. Copie los datos en el volumen cifrado. Use arrastrar y soltar o Robocopy o cualquier herramienta de copia de este tipo. Se crea un archivo de diario ( *.jrn*) en la misma carpeta en la que se ejecuta la herramienta.

   Si la unidad está bloqueada y necesita desbloquearla, los pasos para desbloquearla pueden variar en función del caso de uso.

   * Si ha agregado datos a una unidad cifrada previamente (la herramienta WAImportExport no se usó para el cifrado), use la clave de BitLocker (una contraseña numérica que usted especifique) en el menú emergente para desbloquear la unidad.

   * Si ha agregado datos a una unidad que se cifró mediante la herramienta WAImportExport, utilice el siguiente comando para desbloquear la unidad:

        `WAImportExport Unlock /bk:<BitLocker key (base 64 string) copied from journal (*.jrn*) file>`

5. Abra una ventana de PowerShell o de línea de comandos con privilegios administrativos. Para cambiar el directorio a la carpeta descomprimida, ejecute el siguiente comando:

    `cd C:\WaImportExportV1`
6. Para obtener la clave de BitLocker de la unidad, ejecute el siguiente comando:

    `manage-bde -protectors -get <DriveLetter>:`
7. Para preparar el disco, ejecute el siguiente comando. **Dependiendo del tamaño de los datos, la preparación del disco puede llevar horas o días.**

    ```powershell
    ./WAImportExport.exe PrepImport /j:<journal file name> /id:session<session number> /t:<Drive letter> /bk:<BitLocker key> /srcdir:<Drive letter>:\ /dstdir:<Container name>/ /blobtype:<BlockBlob or PageBlob> /skipwrite
    ```

    Se crea un archivo de diario en la misma carpeta en la que se ejecutó la herramienta. También se crean otros dos archivos: un archivo *.xml* (la carpeta en la que se ejecuta la herramienta) y un archivo *drive-manifest.xml* (la carpeta en la que residen los datos).

    Los parámetros que se usan se describen en la tabla siguiente:

    |Opción  |Descripción  |
    |---------|---------|
    |/j:     |nombre del archivo de diario, con la extensión .jrn. Se genera un archivo de diario por unidad. Se recomienda usar el número de serie del disco como nombre del archivo de diario.         |
    |/id:     |El identificador de sesión. Use un número de sesión único para cada instancia del comando.      |
    |/t:     |letra de unidad del disco que se va a enviar. Por ejemplo, unidad `D`.         |
    |/bk:     |clave de BitLocker de la unidad. Su contraseña numérica de salida de `manage-bde -protectors -get D:`      |
    |/srcdir:     |letra de unidad del disco que se va a enviar seguida de `:\`. Por ejemplo, `D:\`.         |
    |/dstdir:     |El nombre del contenedor de destino en Azure Storage.         |
    |/blobtype:     |Esta opción especifica el tipo de blobs en que desea importar los datos. En el caso de los blobs en bloques, el tipo de blob es `BlockBlob`, mientras que en el caso de los blobs en páginas es `PageBlob`.         |
    |/skipwrite:     | Especifica que no es necesario copiar nuevos datos y que se deben preparar los datos existentes en el disco.          |
    |/enablecontentmd5:     |Cuando esta opción está habilitada, garantiza que se calcula MD5 y se establece como propiedad `Content-md5` en todos los blobs. Esta opción solo se utiliza si se desea usar el campo `Content-md5` después de cargar los datos en Azure. <br> Esta opción no afecta a la comprobación de la integridad de datos (que se produce de forma predeterminada). El valor de aumenta el tiempo necesario para cargar datos en la nube.          |
8. Repita el paso anterior para cada disco que vaya a enviar. Se crea un archivo de diario con el nombre proporcionad para cada ejecución de la línea de comandos.

    > [!IMPORTANT]
    > * Junto con el archivo de diario, se crea también un archivo `<Journal file name>_DriveInfo_<Drive serial ID>.xml` en la misma carpeta en la que reside la herramienta. Si, al crear un trabajo, el archivo de diario es demasiado grande, se utiliza el archivo .xml en su lugar.
   > * El tamaño máximo del archivo de diario que el portal permite es de 2 MB. Si el archivo de diario supera ese límite, se devuelve un error.

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

   1. Cargar los archivos de diario que creó con anterioridad en [Paso 1: Preparación de las unidades de disco](#step-1-prepare-the-drives). Si utilizó `waimportexport.exe version1`, cargue un archivo por cada unidad que haya preparado. Si el tamaño del archivo diario supera los 2 MB, puede usar el archivo `<Journal file name>_DriveInfo_<Drive serial ID>.xml` que se creó también junto con el archivo de diario.
   1. Seleccione la región de Azure de destino del pedido.
   1. Seleccione la cuenta de almacenamiento de la importación.
      
      La ubicación de la entrega se rellena automáticamente según la región de la cuenta de almacenamiento seleccionada.
   1. Si no quiere guardar un registro detallado, desactive la opción **Guardar registro detallado en el contenedor de blobs 'waimportexport'** .

   ![Creación del trabajo de importación: Paso 2](./media/storage-import-export-data-to-blobs/import-to-blob-4.png).

   Seleccione **Siguiente: Envío >** para continuar.

6. En **Envío**:

   1. Seleccione el transportista en la lista desplegable. Si desea usar una empresa de mensajería que no sea FedEx o DHL, elija una de las opciones de la lista desplegable. Póngase en contacto con el equipo de operaciones de Azure Data Box en `adbops@microsoft.com` con la información relacionada con el transportista que quiere usar.
   1. Escriba un número válido de cuenta de transportista que haya creado con ese transportista. Microsoft usa esta cuenta para devolverle las unidades una vez que haya finalizado el trabajo de importación. Si no tiene un número de cuenta, cree una cuenta de transportista [FedEx](https://www.fedex.com/us/oadr/) o [DHL](https://www.dhl.com/).
   1.  Proporcione información completa y válida del contacto: nombre, teléfono, correo electrónico, dirección postal, ciudad, código postal, estado o provincia y país o región.

       > [!TIP]
       > En lugar de especificar una dirección de correo electrónico para un solo usuario, proporcione un correo electrónico de grupo. Esto garantiza que recibirá notificaciones incluso si sale un administrador.

   ![Creación del trabajo de importación - Paso 3](./media/storage-import-export-data-to-blobs/import-to-blob-5.png)

   Seleccione **Revisar y crear** para continuar.

7. En el resumen del pedido:

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
    az storage account create --resource-group myierg --name myssdocsstorage --https-only
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

## <a name="step-3-optional-configure-customer-managed-key"></a>Paso 3 (opcional): Configuración de la clave administrada por el cliente

Omita este paso y vaya al paso siguiente si desea usar la clave administrada por Microsoft para proteger las claves de BitLocker para las unidades. Para configurar su propia clave con el fin de proteger la clave de BitLocker, siga las instrucciones de [Configuración de claves administradas por el cliente con Azure Key Vault para Azure Import/Export en Azure Portal](storage-import-export-encryption-key-portal.md).

## <a name="step-4-ship-the-drives"></a>Paso 4: Envío de las unidades de disco

[!INCLUDE [storage-import-export-ship-drives](../../includes/storage-import-export-ship-drives.md)]

## <a name="step-5-update-the-job-with-tracking-information"></a>Paso 5: Actualización del trabajo con información de seguimiento

[!INCLUDE [storage-import-export-update-job-tracking](../../includes/storage-import-export-update-job-tracking.md)]

## <a name="step-6-verify-data-upload-to-azure"></a>Paso 6: Comprobación de la carga de datos en Azure

Siga el trabajo hasta su finalización. Una vez que se haya finalizado el trabajo, compruebe que los datos se han cargado en Azure. Elimine los datos de forma local después de comprobar que la carga fue correcta.

## <a name="next-steps"></a>Pasos siguientes

* [Visualización del estado del trabajo y de la unidad de disco](storage-import-export-view-drive-status.md)
* [Revisión de los requisitos de Import/Export](storage-import-export-requirements.md)
