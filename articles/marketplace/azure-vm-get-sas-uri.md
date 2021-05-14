---
title: 'Generación de un URI de SAS para una imagen de máquina virtual: Azure Marketplace'
description: Genere un URI de Firma de acceso compartido (SAS) para discos duros virtuales (VHD) en Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: iqshahmicrosoft
ms.author: krsh
ms.date: 04/21/2021
ms.openlocfilehash: 0f7d63a1213b6fe6625f16f33f18680b7a9e6bcb
ms.sourcegitcommit: b4032c9266effb0bf7eb87379f011c36d7340c2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107906843"
---
# <a name="how-to-generate-a-sas-uri-for-a-vm-image"></a>Generación de un URI de SAS para una imagen de máquina virtual

> [!NOTE]
> Para publicar la máquina virtual, no necesita un URI de SAS. Simplemente puede compartir una imagen en el Centro de partners. Consulte [Procedimiento para crear una máquina virtual con una base aprobada](./azure-vm-create-using-approved-base.md) o [Procedimiento para crear una máquina virtual mediante su propia imagen](./azure-vm-create-using-own-image.md).

La generación de los identificadores URI de SAS para los discos duros virtuales debe cumplir los requisitos siguientes:

- Solo se requieren permisos de lectura y enumeración. No proporcione acceso de escritura o eliminación.
- La duración del acceso (fecha de expiración) debe ser un mínimo de tres semanas a partir de la creación del identificador URI de SAS.
- Como protección frente a los cambios de hora UTC, establezca la fecha de inicio en un día antes de la fecha actual. Por ejemplo, si la fecha actual es el 16 de junio de 2020, seleccione 15/6/2020.

## <a name="extract-vhd-from-a-vm"></a>Extracción de un disco duro virtual de una máquina virtual

> [!NOTE]
> Este paso se puede omitir si ya tiene un disco duro virtual cargado en una cuenta de almacenamiento.

Para extraer el disco duro virtual de la máquina virtual, debe tomar una instantánea del disco de la máquina virtual y extraer el disco duro virtual de la instantánea.

Tome una instantánea del disco de la máquina virtual para comenzar:

1. Inicie sesión en Azure Portal.
2. En el menú superior izquierdo, seleccione Crear un recurso y, a continuación, busque y seleccione Instantánea.
3. En la hoja Instantánea, seleccione Crear.
4. Escriba un nombre para la instantánea.
5. Seleccione un grupo de recursos existente o escriba el nombre de uno nuevo.
6. Como disco de origen, seleccione el disco administrado para la instantánea.
7. Seleccione el tipo de cuenta que se usará para almacenar la instantánea. Use HDD estándar, a menos que necesite almacenarla en un disco SSD de alto rendimiento.
8. Seleccione Crear.

### <a name="extract-the-vhd"></a>Extracción del disco duro

Use el siguiente script para exportar la instantánea a un disco duro virtual de la cuenta de almacenamiento.

```azurecli
#Provide the subscription Id where the snapshot is created
$subscriptionId=yourSubscriptionId

#Provide the name of your resource group where the snapshot is created
$resourceGroupName=myResourceGroupName

#Provide the snapshot name
$snapshotName=mySnapshot

#Provide Shared Access Signature (SAS) expiry duration in seconds (such as 3600)
#Know more about SAS here: https://docs.microsoft.com/en-us/azure/storage/storage-dotnet-shared-access-signature-part-1
$sasExpiryDuration=3600

#Provide storage account name where you want to copy the underlying VHD file. Currently, only general purpose v1 storage is supported.
$storageAccountName=mystorageaccountname

#Name of the storage container where the downloaded VHD will be stored.
$storageContainerName=mystoragecontainername

#Provide the key of the storage account where you want to copy the VHD 
$storageAccountKey=mystorageaccountkey

#Give a name to the destination VHD file to which the VHD will be copied.
$destinationVHDFileName=myvhdfilename.vhd

az account set --subscription $subscriptionId

$sas=$(az snapshot grant-access --resource-group $resourceGroupName --name $snapshotName --duration-in-seconds $sasExpiryDuration --query [accessSas] -o tsv)

az storage blob copy start --destination-blob $destinationVHDFileName --destination-container $storageContainerName --account-name $storageAccountName --account-key $storageAccountKey --source-uri $sas
```

### <a name="script-explanation"></a>Explicación del script
En este script se usan los siguientes comandos para generar el URI de SAS de una instantánea y se copia el disco duro virtual (VHD) subyacente en una cuenta de almacenamiento mediante dicho URI. Cada comando de la tabla crea un vínculo a documentación específica del comando.


|Get-Help  |Notas  |
|---------|---------|
| az disk grant-access    |     Genera la SAS de solo lectura que se usa para copiar el archivo de VHD subyacente en una cuenta de almacenamiento o descargarlo localmente.    |
|  az storage blob copy start   |    Copia un blob de forma asincrónica desde una cuenta de almacenamiento a otra. Use az storage blob show para comprobar el estado del nuevo blob.     |
|

## <a name="generate-the-sas-address"></a>Generación de la dirección de SAS

Hay dos herramientas comunes que se usan para crear una dirección de SAS (URL):

1. **Explorador de Azure Storage**: disponible en Azure Portal.
2. **CLI de Azure**: recomendada para los sistemas operativos que no sean Windows y los entornos automatizados o de integración continua.

### <a name="using-tool-1-azure-storage-explorer"></a>Uso de la herramienta 1: Explorador de Azure Storage

1. Vaya a la **cuenta de almacenamiento**.
1. Abra el **Explorador de Storage**.

    :::image type="content" source="media/create-vm/storge-account-explorer.png" alt-text="Ventana de la cuenta de almacenamiento.":::

3. En el **contenedor**, haga clic con el botón derecho en el archivo VHD y seleccione **Obtener firma de acceso compartido**.
4. En el cuadro de diálogo **Firma de acceso compartido**, complete los campos siguientes:

    1. Fecha de inicio: fecha de inicio del permiso para acceder al disco duro virtual. Proporcione una fecha que sea un día antes de la fecha actual.
    2. Fecha de expiración: fecha de expiración del permiso para acceder al disco duro virtual. Proporcione una fecha al menos tres semanas después de la fecha actual.
    3. Permisos: seleccione los permisos Lista y Lectura.
    4. Nivel de contenedor: active la casilla Generar identificador URI de firma de acceso compartido de nivel de contenedor.

    ![Cuadro de diálogo Firma de acceso compartido.](media/vm/create-sas-uri-storage-explorer.png)

5. Seleccione **Crear** para crear el identificador URI de SAS asociado a este disco duro virtual.
6. Copie el identificador URI y guárdelo en un archivo de texto en una ubicación segura. Este identificador URI de SAS generado es para el acceso de nivel de contenedor. Para que sea específico, edite el archivo de texto para agregar el nombre del disco duro virtual.
7. Inserte el nombre del disco duro virtual después de la cadena vhds del identificador URI de SAS (incluya una barra diagonal inicial). El identificador URI de SAS final debe ser similar a este:

    `<blob-service-endpoint-url> + /vhds/ + <vhd-name>? + <sas-connection-string>`

8. Repita estos pasos con todos los discos duros virtuales de los planes que va a publicar.

### <a name="using-tool-2-azure-cli"></a>Uso de la herramienta 2: Azure CLI

1. Descargue e instale la [CLI de Microsoft Azure](/cli/azure/install-azure-cli). Hay versiones disponibles para Windows, macOS y varias distribuciones de Linux.
2. Cree un archivo de PowerShell (con la extensión de archivo .ps1), copie el código siguiente y luego guárdelo localmente.

    ```azurecli-interactive
    az storage container generate-sas --connection-string ‘DefaultEndpointsProtocol=https;AccountName=<account-name>;AccountKey=<account-key>;EndpointSuffix=core.windows.net’ --name <container-name> --permissions rl --start ‘<start-date>’ --expiry ‘<expiry-date>’
    ```

3. Edite el archivo para proporcionar los valores de parámetro siguientes. Proporcione las fechas en formato de fecha y hora UTC, como 2020-04-01T00:00:00Z.

    - account-name: nombre de su cuenta de Azure Storage.
    - account-key: clave de su cuenta de Azure Storage.
    - start-date: fecha de inicio del permiso para acceder al disco duro virtual. Proporcione una fecha un día antes de la fecha actual.
    - expiry-date: fecha de expiración del permiso para acceder al disco duro virtual. Proporcione una fecha al menos tres semanas después de la fecha actual.

    Este es un ejemplo de valores de parámetro adecuados (en el momento de redactar este artículo):

    ```azurecli-interactive
    az storage container generate-sas --connection-string ‘DefaultEndpointsProtocol=https;AccountName=st00009;AccountKey=6L7OWFrlabs7Jn23OaR3rvY5RykpLCNHJhxsbn9ON c+bkCq9z/VNUPNYZRKoEV1FXSrvhqq3aMIDI7N3bSSvPg==;EndpointSuffix=core.windows.net’ --name <container-name> -- permissions rl --start ‘2020-04-01T00:00:00Z’ --expiry ‘2021-04-01T00:00:00Z’
    ```

1. Guarde los cambios.
2. Use uno de los siguientes métodos para ejecutar este script con privilegios administrativos para crear una cadena de conexión de SAS para el acceso de nivel de contenedor:

    - Ejecutar el script desde la consola. En Windows, haga clic con el botón derecho en el script y seleccione **Ejecutar como administrador**.
    - Ejecutar el script desde un editor de scripts de PowerShell, como [Windows PowerShell ISE](/powershell/scripting/components/ise/introducing-the-windows-powershell-ise). En esta pantalla se muestra la creación de una cadena de conexión de SAS en este editor:

    [![creación de una cadena de conexión de SAS en el editor de PowerShell](media/vm/create-sas-uri-power-shell-ise.png)](media/vm/create-sas-uri-power-shell-ise.png#lightbox)

6. Copie la cadena de conexión de SAS y guárdela en un archivo de texto en una ubicación segura. Edite esta cadena para agregar la información de ubicación del disco duro virtual para crear el identificador URI de SAS final.
7. En Azure Portal, vaya al almacenamiento de blobs que contiene el disco duro virtual asociado con el nuevo identificador URI.
8. Copie la dirección URL del punto de conexión de Blob service:

    ![Copia de la dirección URL del punto de conexión de Blob service.](media/vm/create-sas-uri-blob-endpoint.png)

9. Modifique el archivo de texto con la cadena de conexión de SAS del paso 6. Cree el identificador URI de SAS completo con este formato:

    `<blob-service-endpoint-url> + /vhds/ + <vhd-name>? + <sas-connection-string>`

## <a name="verify-the-sas-uri"></a>Comprobación del URI de SAS

Compruebe el URI de SAS antes de publicarlo en el Centro de partners para evitar problemas relacionados con el URI de SAS después de enviar la solicitud. Este proceso es opcional, pero se recomienda.

- El identificador URI incluye el nombre de archivo de la imagen de disco duro virtual, incluida la extensión `.vhd`.
- Aparece `Sp=rl` en la parte central del identificador URI. Esta cadena muestra que se ha especificado el acceso de lectura y enumeración.
- Cuando aparece `sr=c`, significa que se especifica el acceso de nivel de contenedor.
- Copie y pegue el identificador URI en un explorador para probar y descargar el blob (puede cancelar la operación antes de que se complete la descarga).

## <a name="next-steps"></a>Pasos siguientes

- Si tiene problemas, consulte [Mensajes de error de SAS de VM](azure-vm-sas-failure-messages.md).
- [Inicio de sesión en el Centro de partners](https://partner.microsoft.com/dashboard/account/v3/enrollment/introduction/partnership)
- [Creación de una oferta de máquina virtual en Azure Marketplace](azure-vm-create.md)