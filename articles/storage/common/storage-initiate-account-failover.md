---
title: Inicio de una conmutación por error de la cuenta de almacenamiento
titleSuffix: Azure Storage
description: Aprenda a iniciar una conmutación por error de la cuenta de almacenamiento en caso de que el punto de conexión principal de dicha cuenta deje de estar disponible. La conmutación por error actualiza la región secundaria para convertirla en la región primaria de la cuenta de almacenamiento.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 05/07/2021
ms.author: tamram
ms.subservice: common
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 28f46ec6354f98c11ce68beeb2e3de375c7a0249
ms.sourcegitcommit: ba8f0365b192f6f708eb8ce7aadb134ef8eda326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2021
ms.locfileid: "109632338"
---
# <a name="initiate-a-storage-account-failover"></a>Inicio de una conmutación por error de la cuenta de almacenamiento

Si, por cualquier motivo, el punto de conexión principal de la cuenta de almacenamiento con redundancia geográfica deja de estar disponible, puede iniciar una conmutación por error de la cuenta. Una conmutación por error de la cuenta actualiza el punto de conexión secundario para convertirlo en el punto de conexión principal de la cuenta de almacenamiento. Una vez finalizada la conmutación por error, los clientes pueden empezar a escribir en la nueva región primaria. La conmutación por error forzada le permite mantener una alta disponibilidad para sus aplicaciones.

En este artículo se muestra cómo iniciar una conmutación por error de la cuenta de almacenamiento mediante Azure Portal, PowerShell o la CLI de Azure. Para más información sobre la conmutación por error de la cuenta, consulte [Recuperación ante desastres y conmutación por error de la cuenta de almacenamiento](storage-disaster-recovery-guidance.md).

> [!WARNING]
> Una conmutación por error de la cuenta tiene normalmente como resultado la pérdida de datos. Para entender sus implicaciones y prepararse para dicha pérdida de datos, revise [Descripción del proceso de conmutación por error de la cuenta](storage-disaster-recovery-guidance.md#understand-the-account-failover-process).

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Requisitos previos

Para poder realizar una conmutación por error de su cuenta de almacenamiento, asegúrese de que la cuenta de almacenamiento está configurada para la replicación geográfica. La cuenta de almacenamiento puede usar cualquiera de las siguientes opciones de redundancia:

- Almacenamiento con redundancia geográfica (GRS) o almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS)
- Almacenamiento con redundancia de zona geográfica (GZRS) o almacenamiento con redundancia de zona geográfica con acceso de lectura (RA-GZRS)

Para más información sobre la redundancia de Azure Storage, consulte [Redundancia de Azure Storage](storage-redundancy.md).

Tenga en cuenta que las siguientes características y servicios no son compatibles con la conmutación por error de una cuenta:

- Azure File Sync no admite la conmutación por error de una cuenta de almacenamiento. No se debe realizar la conmutación por error de las cuentas de almacenamiento que contienen recursos compartidos de archivos de Azure y que se usan como puntos de conexión de nube en Azure File Sync. Si lo hace, la sincronización dejará de funcionar y también podría provocar una pérdida inesperada de datos en el caso de archivos recién organizados en capas.
- Actualmente, no se admiten cuentas de almacenamiento que tienen habilitado el espacio de nombres jerárquico (como Data Lake Storage Gen2).
- No se puede conmutar por error una cuenta de almacenamiento que contiene blobs en bloques Premium. Las cuentas de almacenamiento que admiten los blobs en bloques Premium actualmente no admiten la redundancia geográfica.
- No se puede conmutar por error una cuenta de almacenamiento que contenga contenedores habilitados por la [Directiva de inmutabilidad de gusanos](../blobs/storage-blob-immutable-storage.md). Las directivas de retención legal o retención basada en tiempo desbloqueada o bloqueada impiden la conmutación por error para mantener el cumplimiento.

## <a name="initiate-the-failover"></a>Inicio de la conmutación por error

## <a name="portal"></a>[Portal](#tab/azure-portal)

Para iniciar una conmutación por error de la cuenta desde Azure Portal, siga estos pasos:

1. Vaya a la cuenta de almacenamiento.
1. En **Configuración**, seleccione **Replicación geográfica**. En la imagen siguiente se muestra el estado de la replicación geográfica y de la conmutación por error de una cuenta de almacenamiento.

    :::image type="content" source="media/storage-initiate-account-failover/portal-failover-prepare.png" alt-text="Captura de pantalla que muestra el estado de la replicación geográfica y de la conmutación por error":::

1. Compruebe que la cuenta de almacenamiento está configurada para el almacenamiento con redundancia geográfica (GRS) o el almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS). Si no es así, seleccione **Configuración** en **Configuración** para actualizar su cuenta a fin de que tenga redundancia geográfica.
1. La propiedad **Hora de la última sincronización** indica a qué distancia está la región secundaria de la primaria. El valor de **Hora de la última sincronización** proporciona una estimación del alcance de la pérdida de datos que experimentará una vez finalizada la conmutación por error. Para más información sobre cómo comprobar la propiedad **Hora de la última actualización**, consulte [Comprobación de la propiedad Hora de la última sincronización de una cuenta de almacenamiento](last-sync-time-get.md).
1. Seleccione **Preparar la conmutación por error**.
1. Revise el cuadro de diálogo de confirmación. Cuando esté listo, escriba **Sí** para confirmar e iniciar la conmutación por error.

    :::image type="content" source="media/storage-initiate-account-failover/portal-failover-confirm.png" alt-text="Captura de pantalla que muestra el cuadro de diálogo de confirmación de una conmutación por error de la cuenta":::

## <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

La característica de conmutación por error de la cuenta está disponible con carácter general, pero todavía depende de un módulo en versión preliminar para PowerShell. Para usar PowerShell para iniciar una conmutación por error de la cuenta, primero debe instalar el módulo[1.1.1-preview](https://www.powershellgallery.com/packages/Az.Storage/1.1.1-preview). Para instalar el módulo, siga estos pasos:

1. Desinstale las instalaciones anteriores de Azure PowerShell:

    - Quite cualquier instalación anterior de Azure PowerShell desde Windows usando el ajuste **Aplicaciones y características** en **Configuración**.
    - Quite todos los módulos de **Azure** desde `%Program Files%\WindowsPowerShell\Modules`.

1. Asegúrese de tener instalada la versión más reciente de PowerShellGet. Abra una ventana de Windows PowerShell y ejecute el siguiente comando para instalar la versión más reciente:

    ```powershell
    Install-Module PowerShellGet –Repository PSGallery –Force
    ```

1. Cierre y vuelva a abrir la ventana de PowerShell después de instalar PowerShellGet.

1. Instale la versión más reciente de Azure PowerShell:

    ```powershell
    Install-Module Az –Repository PSGallery –AllowClobber
    ```

1. Instale uno de los módulos de versión preliminar de Azure Storage que admita la conmutación por error de cuentas:

    ```powershell
    Install-Module Az.Storage –Repository PSGallery -RequiredVersion 1.1.1-preview –AllowPrerelease –AllowClobber –Force
    ```

Para iniciar una conmutación por error de la cuenta desde PowerShell, ejecute el siguiente comando:

```powershell
Invoke-AzStorageAccountFailover -ResourceGroupName <resource-group-name> -Name <account-name>
```

## <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para usar la CLI de Azure para iniciar una conmutación por error de la cuenta, ejecute los siguientes comandos:

```azurecli-interactive
az storage account show \ --name accountName \ --expand geoReplicationStats
az storage account failover \ --name accountName
```

---

## <a name="important-implications-of-account-failover"></a>Implicaciones importantes de la conmutación por error de la cuenta

Cuando se inicia una conmutación por error de la cuenta de almacenamiento, se actualizan los registros DNS del punto de conexión secundario para que pase a ser el punto de conexión principal. Asegúrese de comprender el posible efecto para la cuenta de almacenamiento antes de iniciar una conmutación por error.

Para calcular el alcance de la posible pérdida de datos antes de iniciar una conmutación por error, compruebe la propiedad **Hora de la última sincronización**. Para más información sobre cómo comprobar la propiedad **Hora de la última actualización**, consulte [Comprobación de la propiedad Hora de la última sincronización de una cuenta de almacenamiento](last-sync-time-get.md).

El tiempo que se tarda en realizar la conmutación por error después del inicio puede variar, aunque por lo general tarda menos de una hora.

Después de la conmutación por error, el tipo de cuenta de almacenamiento se convierte automáticamente en almacenamiento con redundancia local (LRS) en la nueva región primaria. Puede volver a habilitar el almacenamiento con redundancia geográfica (GRS) o el almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS) para la cuenta. Tenga en cuenta que la conversión de LRS a GRS o a RA-GRS supone un costo adicional. El costo se debe a los cargos de salida de red para volver a replicar los datos en la nueva región secundaria. Para más información, consulte [Detalles de precios de ancho de banda](https://azure.microsoft.com/pricing/details/bandwidth/).

Después de volver a habilitar GRS para la cuenta de almacenamiento, Microsoft comienza a replicar los datos de la cuenta en la nueva región secundaria. El tiempo de replicación depende de muchos factores, entre otros:

- El número y el tamaño de los objetos en la cuenta de almacenamiento. Muchos objetos pequeños pueden tardar más de menos objetos más grandes.
- Los recursos disponibles para la replicación en segundo plano, como CPU, memoria, disco y capacidad WAN. El tráfico en directo tiene prioridad sobre la replicación geográfica.
- Si usa Blob Storage, el número de instantáneas por blob.
- Si usa Table Storage, la [estrategia de creación de particiones de datos](/rest/api/storageservices/designing-a-scalable-partitioning-strategy-for-azure-table-storage). El proceso de replicación no se puede escalar más allá del número de claves de partición que se usan.
  
## <a name="next-steps"></a>Pasos siguientes

- [Recuperación ante desastres y conmutación por error de la cuenta de almacenamiento](storage-disaster-recovery-guidance.md)
- [Comprobación de la propiedad Hora de la última sincronización de una cuenta de almacenamiento](last-sync-time-get.md)
- [Uso de redundancia geográfica para diseñar aplicaciones de alta disponibilidad](geo-redundant-design.md)
- [Tutorial: Creación de una aplicación de alta disponibilidad con Blob Storage](../blobs/storage-create-geo-redundant-storage.md)
