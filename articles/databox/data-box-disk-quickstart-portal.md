---
title: Inicio rápido de Microsoft Azure Data Box Disk | Microsoft Docs
description: Use esta guía de inicio rápido para implementar rápidamente Azure Data Box Disk en Azure Portal
services: databox
author: alkohli
ms.service: databox
ms.subservice: disk
ms.topic: quickstart
ms.date: 11/04/2020
ms.author: alkohli
ms.localizationpriority: high
ms.openlocfilehash: 241b7c0c07d1fbaa6a43c6be4b264424612f538a
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107869048"
---
::: zone target="docs"

# <a name="quickstart-deploy-azure-data-box-disk-using-the-azure-portal"></a>Inicio rápido: Implementación de Azure Data Box Disk mediante Azure Portal

En esta guía de inicio rápido se describe cómo implementar Azure Data Box Disk mediante Azure Portal. Los pasos incluyen cómo crear un pedido, recibir los discos, desempaquetarlos, conectarlos y copiar datos en los discos para que se carguen en Azure rápidamente.

Para obtener instrucciones detalladas para realizar una implementación y el seguimiento del proceso, vaya a [Tutorial: Pedido de Azure Data Box](data-box-disk-deploy-ordered.md). 

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F&preserve-view=true).

::: zone-end

::: zone target="chromeless"

En esta guía se explican los pasos que deben darse para usar Azure Data Box Disk en Azure Portal. En esta guía le ayuda a responder las siguientes preguntas.

::: zone-end

::: zone target="docs"

## <a name="prerequisites"></a>Prerrequisitos

Antes de empezar:

- Asegúrese de que su suscripción está habilitada para el servicio Azure Data Box. Para habilitar la suscripción para este servicio, [regístrese en el servicio](https://aka.ms/azuredataboxfromdiskdocs).

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en [https://aka.ms/azuredataboxfromdiskdocs](https://aka.ms/azuredataboxfromdiskdocs).

::: zone-end

::: zone target="chromeless"

> [!div class="checklist"]
>
> - **Revisión de los requisitos previos**: compruebe el número de discos y cables, el sistema operativo y otros programas de software.
> - **Conexión y desbloqueo**: conecte el dispositivo y desbloquee el disco para copiar los datos.
> - **Copia de los datos en el disco y validación**: copie los datos en los discos en las carpetas creadas.
> - **Devolución de los discos**: devuelva los discos al centro de datos de Azure en el se cargan los datos en la cuenta de almacenamiento.
> - **Comprobación de los datos en Azure**: compruebe que los datos se han cargado en la cuenta de almacenamiento antes de eliminarlos del servidor de datos de origen.

::: zone-end


::: zone target="docs"

## <a name="order"></a>Pedido de

### <a name="portal"></a>[Portal](#tab/azure-portal)

Este paso tarda aproximadamente 5 minutos.

1. Cree un nuevo recurso de Azure Data Box en Azure Portal. 
2. Seleccione una suscripción habilitada para este servicio y elija el tipo de transferencia como de **importación**. Proporcione el **país de origen** donde residen los datos y la **región de destino de Azure** para la transferencia de datos.
3. Seleccione **Data Box Disk**. La capacidad máxima de la solución es de 35 TB y puede crear varios pedidos de disco para los tamaños de datos mayores.  
4. Escriba los detalles del pedido y la información de envío. Si el servicio está disponible en su región, proporcione las direcciones de correo electrónico de notificación, revise el resumen y, a continuación, cree el pedido.

Una vez que se creó el pedido, los discos están preparados para su envío.

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Utilice estos comandos de la CLI de Azure para crear un trabajo de Data Box Disk.

[!INCLUDE [azure-cli-prepare-your-environment-h3.md](../../includes/azure-cli-prepare-your-environment-h3.md)]

1. Ejecute el comando [az group create](/cli/azure/group#az_group_create) para crear un grupo de recursos o use un grupo de recursos existente:

   ```azurecli
   az group create --name databox-rg --location westus
   ```

1. Use el comando [az storage account create](/cli/azure/storage/account#az_storage_account_create) para crear una cuenta de almacenamiento o utilice una cuenta de almacenamiento existente:

   ```azurecli
   az storage account create --resource-group databox-rg --name databoxtestsa
   ```

1. Ejecute el comando [az databox job create](/cli/azure/databox/job#az_databox_job_create) para crear un trabajo de Data Box con el SKU DataBoxDisk:

   ```azurecli
   az databox job create --resource-group databox-rg --name databoxdisk-job \
       --location westus --sku DataBoxDisk --contact-name "Jim Gan" --phone=4085555555 \
       –-city Sunnyvale --email-list JimGan@contoso.com --street-address1 "1020 Enterprise Way" \
       --postal-code 94089 --country US --state-or-province CA \
       --storage-account databoxtestsa --expected-data-size 1
   ```

1. Ejecute el comando [az databox job update](/cli/azure/databox/job#az_databox_job_update) para actualizar un trabajo, como en este ejemplo, donde puede cambiar el nombre y la dirección de correo electrónico del contacto:

   ```azurecli
   az databox job update -g databox-rg --name databox-job --contact-name "Robert Anic" --email-list RobertAnic@contoso.com
   ```

   Ejecute el comando [az databox job show](/cli/azure/databox/job#az_databox_job_show) para obtener información sobre el trabajo:

   ```azurecli
   az databox job show --resource-group databox-rg --name databox-job
   ```

   Use el comando [az databox job list]( /cli/azure/databox/job#az_databox_job_list) para ver todos los trabajos de Data Box de un grupo de recursos:

   ```azurecli
   az databox job list --resource-group databox-rg
   ```

   Ejecute el comando [az databox job cancel](/cli/azure/databox/job#az_databox_job_cancel) para cancelar un trabajo:

   ```azurecli
   az databox job cancel –resource-group databox-rg --name databox-job --reason "Cancel job."
   ```

   Ejecute el comando [az databox job delete](/cli/azure/databox/job#az_databox_job_delete) para eliminar un trabajo:

   ```azurecli
   az databox job delete –resource-group databox-rg --name databox-job
   ```

1. Use el comando [az databox job list-credentials]( /cli/azure/databox/job#az_databox_job_list_credentials) para enumerar las credenciales de un trabajo de Data Box:

   ```azurecli
   az databox job list-credentials --resource-group "databox-rg" --name "databoxdisk-job"
   ```

Una vez que se creó el pedido, el dispositivo está preparado para su envío.

---

## <a name="unpack"></a>Desempaquetado

Este paso tarda aproximadamente 5 minutos.

Data Box Disk se envía por correo en una caja de UPS Express. Abra la caja y compruebe que contiene:

- De 1 a 5 discos USB en envoltorio de burbujas.
- Un cable de conexión por disco.
- Una etiqueta para el envío de devolución.

## <a name="connect-and-unlock"></a>Conexión y desbloqueo

Este paso tarda aproximadamente 5 minutos.

1. Utilice el cable incluido para conectar el disco a una máquina Windows/Linux que ejecute un sistema operativo compatible. Para más información sobre las versiones de sistemas operativos compatibles, vaya a [Requisitos del sistema de Azure Data Box Disk](data-box-disk-system-requirements.md). 
2. Para desbloquear el disco:

    1. En Azure Portal, vaya a **General > Detalles del dispositivo** y obtenga la clave de paso.
    2. Descargue y extraiga la herramienta de desbloqueo de Data Box Disk específica del sistema operativo en el equipo que se usa para copiar los datos en discos. 
    3. Ejecute la herramienta de desbloqueo de Data Box Disk y proporcione la clave de paso. Si desea reinsertar algún disco, vuelva a ejecutar la herramienta de desbloqueo y escriba la clave de paso. **No utilice el cuadro de diálogo de BitLocker ni la clave de BitLocker para desbloquear el disco.** Para más información sobre cómo desbloquear los discos, vaya a [Desbloqueo de discos en el cliente de Windows](data-box-disk-deploy-set-up.md#unlock-disks-on-windows-client) o [Desbloqueo de discos en el cliente de Linux](data-box-disk-deploy-set-up.md#unlock-disks-on-linux-client).
    4. La herramienta muestra la letra de unidad asignada al disco. Tome nota de la letra de unidad del disco. Se usa en los pasos siguientes.

## <a name="copy-data-and-validate"></a>Copia de datos y validación

El tiempo en completar esta operación depende del tamaño de los datos.

1. La unidad contiene las carpetas *PageBlob*, *BlockBlob*, *AzureFile*, *ManagedDisk* y *DataBoxDiskImport*. Arrastre y coloque los datos que deben importarse como blobs en bloques en la carpeta *BlockBlob* para copiarlos. De forma similar, puede arrastrar y colocar los datos como VHD/VHDX en la carpeta *PageBlob* y los datos apropiados en *AzureFile*. Copie los discos duros virtuales que desea cargar como discos administrados en una carpeta en *ManagedDisk*.

    Se crea un contenedor en la cuenta de almacenamiento de Azure para cada subcarpeta bajo las carpetas *BlockBlob* y *PageBlob*. Se crea un recurso compartido para una subcarpeta en *AzureFile*.

    Todos los archivos bajo las carpetas *BlockBlob* y *PageBlob* se copian en un contenedor predeterminado `$root` bajo la cuenta de Azure Storage. Copie los archivos en una carpeta dentro de *AzureFile*. Los archivos que se copian directamente en la carpeta *AzureFile* presentan errores y se cargan como blobs en bloques.

    > [!NOTE]
    > - Todos los contenedores, blobs y archivos deben cumplir las [convenciones de nomenclatura de Azure](data-box-disk-limits.md#azure-block-blob-page-blob-and-file-naming-conventions). Si no se siguen estas reglas, se producirá un error en la carga de datos en Azure.
    > - Asegúrese de que los archivos no superen ~4,75 TiB para blobs en bloques, ~ 8 TiB para blobs en páginas y ~ 1 TiB para Azure Files.

2. **(Opcional pero recomendado)** Una vez completada la copia, se recomienda encarecidamente que, como mínimo, se ejecute el `DataBoxDiskValidation.cmd` proporcionado en la carpeta *DataBoxDiskImport* y se seleccione la opción 1 para validar los archivos. También se recomienda que, si el tiempo lo permite, use la opción 2 para generar también sumas de comprobación para la validación (puede tardar tiempo en función del tamaño de los datos). Estos pasos minimizan las posibilidades de errores al cargar los datos en Azure.
3. Quite la unidad de forma segura.

## <a name="ship-to-azure"></a>Envío a Azure

Este paso tarda entre 5 y 7 minutos en completarse.

1. Coloque todos los discos juntos en el paquete original. Utilice la etiqueta de envío incluida. Si la etiqueta se daña o se pierde, descárguela desde el portal. Vaya a **Información general** y haga clic en **Download shipping label** (Descargar etiqueta de envío) desde la barra de comandos.
2. Entregue el paquete precintado en la ubicación para realizar el envío.  

El servicio Data Box Disk envía una notificación por correo electrónico y actualiza el estado del pedido en Azure Portal.

## <a name="verify-your-data"></a>Confirmación de los datos

El tiempo en completar esta operación depende del tamaño de los datos.

1. Cuando Data Box Disk se conecta a la red del centro de datos de Azure, se inicia automáticamente la carga de datos en Azure.
2. El servicio Azure Data Box le notifica a través de Azure Portal que la copia de datos se ha completado.
    
    1. Compruebe en los registros de errores si hay errores y tome las medidas adecuadas.
    2. Compruebe que los datos estén en las cuentas de almacenamiento antes de eliminarlos del origen.

## <a name="clean-up-resources"></a>Limpieza de recursos

Este paso tarda de 2 a 3 minutos en completarse.

Para limpiarlos, puede cancelar el pedido de Data Box y, a continuación, eliminarlo.

- Puede cancelar el pedido de Data Box en Azure Portal antes de procesar el pedido. Una vez que el pedido se procesa, no se puede cancelar. El pedido sigue su curso hasta que alcanza la fase de finalización.

    Para cancelar el pedido, vaya a **Información general** y haga clic en **Cancelar** desde la barra de comandos.  

- Podrá eliminar el pedido una vez que el estado se muestre como **Completed** (Completado) o **Canceled** (Cancelado) en Azure Portal.

    Para eliminar el pedido, vaya a **Información general** y haga clic en **Eliminar** desde la barra de comandos.

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha implementado Azure Data Box Disk para ayudar a importar los datos en Azure. Para más información sobre la administración de Azure Data Box Disk, pase al tutorial siguiente:

> [!div class="nextstepaction"]
> [Uso de Azure Portal para administrar Azure Data Box Disk](data-box-portal-ui-admin.md)

::: zone-end
