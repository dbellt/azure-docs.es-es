---
title: Cambio en la forma en que se replican las cuentas de almacenamiento
titleSuffix: Azure Storage
description: Aprenda a cambiar la forma en que se replican los datos de una cuenta de almacenamiento existente.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 06/09/2021
ms.author: tamram
ms.subservice: common
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: d060a066c80f10fb9d887db90bde434cc89922a5
ms.sourcegitcommit: f9e368733d7fca2877d9013ae73a8a63911cb88f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111901637"
---
# <a name="change-how-a-storage-account-is-replicated"></a>Cambio en la forma en que se replican las cuentas de almacenamiento

Azure Storage siempre almacena varias copias de los datos, con el fin de protegerlos de eventos planeados y no planeados, como errores transitorios del hardware, interrupciones del suministro eléctrico o cortes de la red, y desastres naturales masivos. La redundancia garantiza que la cuenta de almacenamiento cumpla el [Acuerdo de Nivel de Servicio (SLA) de Azure Storage](https://azure.microsoft.com/support/legal/sla/storage/), incluso en caso de errores.

Azure Storage ofrece los siguientes tipos de replicación:

- Almacenamiento con redundancia local (LRS)
- Almacenamiento con redundancia de zona (ZRS)
- Almacenamiento con redundancia geográfica (GRS) o almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS)
- Almacenamiento con redundancia de zona geográfica (GZRS) o almacenamiento con redundancia de zona geográfica con acceso de lectura (RA-GZRS)

Para obtener información general de cada una de estas opciones, consulte [Redundancia de Azure Storage](storage-redundancy.md).

## <a name="switch-between-types-of-replication"></a>Conmutación entre los distintos tipos de replicación

Las cuentas de almacenamiento pueden cambiar de un tipo de replicación a cualquier otro, pero algunos escenarios son más sencillos que otros. Si desea agregar o quitar la replicación geográfica o el acceso de lectura a la región secundaria, puede usar Azure Portal, PowerShell o la CLI de Azure para actualizar el valor de replicación. Sin embargo, si desea cambiar la forma en que se replican los datos en la región primaria mediante el traslado de LRS a ZRS, o viceversa, debe realizar una migración manual.

En la tabla siguiente se proporciona información general sobre cómo cambiar de cualquiera de los tipos de replicación a otro:

| Conmutación | ... a LRS | …a GRS/RA-GRS | …a ZRS | …a GZRS/RA-GZRS |
|--------------------|----------------------------------------------------|---------------------------------------------------------------------|----------------------------------------------------|---------------------------------------------------------------------|
| <b>… desde LRS</b> | N/D | Uso de Azure Portal, PowerShell o la CLI para cambiar la opción de replicación<sup>1,2</sup> | Realización de una migración manual <br /><br /> O BIEN <br /><br /> Solicitud de una migración en vivo | Realización de una migración manual <br /><br /> O BIEN <br /><br /> Cambie a GRS/RA-GRS primero y, a continuación, solicite una migración activa<sup>1</sup> |
| <b>… desde GRS/RA-GRS</b> | Uso de Azure Portal, PowerShell o la CLI para cambiar la opción de replicación | N/D | Realización de una migración manual <br /><br /> O BIEN <br /><br /> Cambio primero a LRS y luego solicitud de una migración en vivo | Realización de una migración manual <br /><br /> O BIEN <br /><br /> Solicitud de una migración en vivo |
| <b>... desde ZRS</b> | Realización de una migración manual | Realización de una migración manual | N/D | Solicitud de una migración en vivo |
| <b>… desde GZRS/RA-GZRS</b> | Realización de una migración manual | Realización de una migración manual | Uso de Azure Portal, PowerShell o la CLI para cambiar la opción de replicación | N/D |

<sup>1</sup> Incurre en un cargo de salida único.<br />
<sup>2</sup> No se admite la migración de LRS a GRS si la cuenta de almacenamiento contiene blobs en el nivel de archivo.<br />

> [!CAUTION]
> Si realizó una [conmutación por error de la cuenta](storage-disaster-recovery-guidance.md) para la cuenta de (RA-)GRS o (RA-)GZRS, la cuenta será redundante localmente (LRS) en la nueva región principal después de la conmutación por error. No se admite la migración en vivo a ZRS o GZRS para una cuenta de LRS resultante de una conmutación por error. Esto se cumple incluso en el caso de las denominadas operaciones de conmutación por recuperación. Por ejemplo, si realiza una conmutación por error de cuentas de RA-GZRS a LRS en la región secundaria y, a continuación, la configura de nuevo en RA-GRS y realiza otra conmutación por error de cuentas en la región primaria original, no puede ponerse en contacto con el soporte técnico para la migración en vivo original a RA-GZRS en la región primaria. En su lugar, tendrá que realizar una migración manual a ZRS o GZRS.

## <a name="change-the-replication-setting"></a>Cambio del valor de la replicación

Para cambiar la opción de replicación de una cuenta de almacenamiento se pueden usar Azure Portal, PowerShell o la CLI de Azure, siempre que no vaya a cambiar la forma en que se replican los datos en la región primaria. Si va a realizar la migración desde LRS en la región primaria a ZRS en la región primaria, o viceversa, debe realizar una migración manual o una migración en vivo.

El cambio de la forma en que se replica la cuenta de almacenamiento no provoca un tiempo de inactividad en las aplicaciones.

# <a name="portal"></a>[Portal](#tab/portal)

Para cambiar la opción de redundancia de la cuenta de almacenamiento en Azure Portal, siga estos pasos:

1. Vaya a la cuenta de almacenamiento en Azure Portal.
1. En **Configuración**, seleccione **Configuración**.
1. Actualice el valor de **Replicación**.

    :::image type="content" source="media/redundancy-migration/change-replication-option.png" alt-text="Captura de pantalla que muestra cómo cambiar la opción de replicación en el portal." lightbox="media/redundancy-migration/change-replication-option.png":::

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para cambiar la opción de redundancia de una cuenta de almacenamiento con PowerShell, llame al comando [Set-AzStorageAccount](/powershell/module/az.storage/set-azstorageaccount) y especifique el parámetro `-SkuName`:

```powershell
Set-AzStorageAccount -ResourceGroupName <resource_group> `
    -AccountName <storage_account> `
    -SkuName <sku>
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para cambiar la opción de redundancia de una cuenta de almacenamiento con la CLI de Azure, llame al comando [az storage account update](/cli/azure/storage/account#az_storage_account_update) y especifique el parámetro `--sku`:

```azurecli-interactive
az storage account update \
    --name <storage-account>
    --resource-group <resource_group> \
    --sku <sku>
```

---

## <a name="perform-a-manual-migration-to-zrs-gzrs-or-ra-gzrs"></a>Migración manual a ZRS, GZRS o RA-GZRS

Si desea cambiar la forma en que se replican los datos de la cuenta de almacenamiento en la región primaria mediante el traslado de LRS a ZRS o viceversa, debe elegir una migración manual. Una migración manual proporciona más flexibilidad que una migración en vivo. Puede controlar el momento en que se hará una migración manual, por lo que debe usar esta opción si necesita que la migración se complete en una fecha determinada.

Cuando realiza una migración manual desde LRS a ZRS en la región primaria, o viceversa, la cuenta de almacenamiento de destino puede tener redundancia geográfica y también puede configurarse para que tenga acceso de lectura a la región secundaria. Por ejemplo, puede migrar una cuenta LRS a una cuenta GZRS o RA-GZRS en un solo paso.

Una migración manual puede provocar tiempos de inactividad de la aplicación. Si la aplicación requiere alta disponibilidad, Microsoft también proporciona una opción de migración en vivo. Una migración en vivo es una migración en contexto sin tiempo de inactividad.

Con una migración manual, copia los datos de la cuenta de almacenamiento existente en una nueva cuenta de almacenamiento que usa ZRS en la región primaria. Para realizar una migración manual, puede usar una de las siguientes opciones:

- Copie los datos mediante una herramienta existente como AzCopy, una de las bibliotecas cliente de Azure Storage o una herramienta de terceros de confianza.
- Si conoce Hadoop o HDInsight, puede asociar tanto la cuenta de almacenamiento de origen como la de destino a su clúster. Después, realice un paralelismo del proceso de copia de datos con una herramienta como DistCp.

## <a name="request-a-live-migration-to-zrs-gzrs-or-ra-gzrs"></a>Solicitud de una migración en vivo a ZRS, GZRS o RA-GZRS

Si necesita migrar una cuenta de almacenamiento de LRS a ZRS en la región primaria sin que las aplicaciones tengan tiempo de inactividad, puede solicitar a Microsoft una migración en vivo. Para migrar de LRS a GZRS o RA-GZRS, primero cambie a GRS o RA-GRS y, luego, solicite una migración en vivo. Del mismo modo, puede solicitar una migración en vivo de GRS o RA-GRS a GZRS o RA-GZRS. Para migrar de GRS a RA-GRS o ZRS, primero cambie a LRS y, luego, solicite una migración en vivo.

En una migración en vivo, se puede acceder a los datos de la cuenta de almacenamiento sin pérdida de durabilidad ni disponibilidad. El Acuerdo de Nivel de Servicio de Azure Storage se mantiene durante el proceso de migración. No hay ningún tipo de pérdida de datos asociada con una migración en vivo. Los puntos de conexión de servicio, las claves de acceso, las firmas de acceso compartido y otras opciones de la cuenta permanecen inalteradas después de la migración.

ZRS solo admite cuentas de uso general v2, por lo que es preciso asegurarse de actualizar la cuenta de almacenamiento antes de enviar una solicitud de migración en vivo a ZRS. Para más información, consulte [Actualización a una cuenta de almacenamiento de uso general v2](storage-account-upgrade.md). Las cuentas de almacenamiento deben contener los datos que se van a migrar con la migración en vivo.

La migración en vivo se admite solo para las cuentas de almacenamiento que utilizan la replicación de LRS o GRS. Si su cuenta usa RA-GRS, debe cambiar primero el tipo de replicación de la cuenta a LRS o GRS antes de continuar. Este paso intermedio quita el punto de conexión de solo lectura secundario proporcionado por RA-GRS antes de la migración.

Si bien Microsoft controla la solicitud de migración en vivo rápidamente, no hay ninguna garantía respecto de cuándo se completará una migración en vivo. Si necesita migrar los datos a ZRS en una fecha concreta, Microsoft recomienda que, en su lugar, realice una migración manual. En general, cuantos más datos tenga en su cuenta, más tiempo se tardará en migrar esos datos.

La migración manual debe realizarse en los siguientes casos:

- Desea migrar los datos a una cuenta de almacenamiento de ZRS que se encuentra en una región distinta de la región de la cuenta de origen.
- La cuenta de almacenamiento es una cuenta de blob en páginas o de blob en bloques prémium.
- Quiere migrar datos de ZRS a LRS, GRS o RA-GRS.
- La cuenta de almacenamiento incluye datos en el nivel de archivo.

Puede solicitar la migración en vivo mediante el [Portal de soporte técnico de Azure](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview). 

> [!IMPORTANT]
> Si necesita migrar más de una cuenta de almacenamiento, cree una incidencia de soporte técnico y, en la pestaña **Detalles**, especifique los nombres de las cuentas que se van a convertir.

Siga estos pasos para solicitar una migración en vivo:

1. En Azure Portal, vaya a la cuenta de almacenamiento que quiere migrar.
1. En **Soporte técnico y solución de problemas**, elija **Nueva solicitud de soporte técnico**.
1. Rellene la pestaña **Aspectos básicos** según la información de su cuenta:
    - **Tipo de problema**: seleccione **Técnico**.
    - **Servicio**: seleccione **Mis servicios** y, luego, **Storage Account Management** (Administración de cuentas de almacenamiento).
    - **Recurso**: seleccione una cuenta de almacenamiento para migrar. Si tiene que especificar varias cuentas de almacenamiento, puede hacerlo en la sección **Detalles**.
    - **Tipo de problema**: elija **Migración de datos**.
    - **Subtipo de problema**: elija **Migrate to ZRS, GZRS, or RA-GZRS** (Migrar a ZRS, GZRS o RA-GZRS).

    :::image type="content" source="media/redundancy-migration/request-live-migration-basics-portal.png" alt-text="Captura de pantalla que muestra cómo solicitar una migración en vivo: pestaña Aspectos básicos":::

1. Seleccione **Next** (Siguiente). En la pestaña **Soluciones**, puede comprobar la idoneidad de las cuentas de almacenamiento para la migración.
1. Seleccione **Next** (Siguiente). Si tiene más de una cuenta de almacenamiento para migrar, en la pestaña **Detalles**, especifique el nombre de cada cuenta separado por punto y coma.

    :::image type="content" source="media/redundancy-migration/request-live-migration-details-portal.png" alt-text="Captura de pantalla que muestra cómo solicitar una migración en vivo: pestaña Detalles":::

1. Rellene la información adicional necesaria en la pestaña **Detalles** y, luego, seleccione **Revisar y crear** para revisar y enviar la incidencia de soporte técnico. Un responsable de soporte técnico se pondrá en contacto con usted para proporcionarle la asistencia que necesite.

> [!NOTE]
> Los recursos compartidos de archivos Premium solo están disponibles para LRS y ZRS.
>
> Las cuentas de almacenamiento de GZRS no admiten actualmente el nivel de archivo. Consulte [Azure Blob Storage: niveles de acceso frecuente, esporádico y de archivo](../blobs/storage-blob-storage-tiers.md) para más información.
>
> Los discos administrados solo están disponible para LRS y no se pueden migrar a ZRS. Puede almacenar instantáneas e imágenes de discos administrados SSD estándar en un almacenamiento HDD estándar y [elegir entre las opciones LRS y ZRS](https://azure.microsoft.com/pricing/details/managed-disks/). Para más información sobre la integración con conjuntos de disponibilidad, consulte [Introducción a los discos administrados de Azure](../../virtual-machines/managed-disks-overview.md#integration-with-availability-sets).

## <a name="switch-from-zrs-classic"></a>Conmutación desde ZRS clásico

> [!IMPORTANT]
> Microsoft dejará de usar y migrará las cuentas de ZRS clásico el 31 de marzo de 2021. Se enviarán más detalles a los clientes de ZRS clásico antes de que estas cuentas dejen de usarse.
>
> En cuanto ZRS esté disponible con carácter general en una región dada, los clientes ya no podrán crear cuentas de ZRS clásico desde Azure Portal en dicha región. El uso de Microsoft PowerShell y la CLI de Azure para crear cuentas de ZRS clásico se admite como opción hasta que ZRS clásico esté en desuso. Para más información sobre las ubicaciones en que ZRS está disponible, consulte [Redundancia de Azure Storage](storage-redundancy.md).

ZRS clásico replica asincrónicamente datos en centros de datos dentro de una o dos regiones. Puede que los datos replicados no estén disponibles hasta que Microsoft inicie la conmutación por error al elemento secundario. Una cuenta de ZRS clásico no puede convertirse a o desde LRS, GRS o RA-GRS. Las cuentas de ZRS clásico no admiten ni las métricas ni el registro.

ZRS clásico solo está disponible para **blobs en bloques** de cuentas de almacenamiento de uso general V1 (GPv1). Para más información sobre las cuentas de almacenamiento, vea [Introducción a las cuentas de Azure Storage](storage-account-overview.md).

Para migrar manualmente datos de cuentas de ZRS hacia o desde una cuenta de LRS, ZRS clásico, GRS o RA-GRS, use una de las siguientes herramientas: AzCopy, Explorador de Azure Storage, PowerShell o la CLI de Azure. También puede compilar su propia solución de migración con una de las bibliotecas cliente de Azure Storage.

Las cuentas de almacenamiento de ZRS clásico también se pueden actualizar a ZRS desde Azure Portal, PowerShell o la CLI de Azure en aquellas regiones en las que ZRS esté disponible.

# <a name="portal"></a>[Portal](#tab/portal)

Para actualizar a ZRS en Azure Portal, vaya a las opciones de **Configuración** de la cuenta y elija **Actualizar**:

![Actualización de ZRS clásico a ZRS en el portal](media/redundancy-migration/portal-zrs-classic-upgrade.png)

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para actualizar a ZRS mediante PowerShell, llame al siguiente comando:

```powershell
Set-AzStorageAccount -ResourceGroupName <resource_group> -AccountName <storage_account> -UpgradeToStorageV2
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para actualizar a ZRS mediante la CLI de Azure, llame al siguiente comando:

```cli
az storage account update -g <resource_group> -n <storage_account> --set kind=StorageV2
```

---

## <a name="costs-associated-with-changing-how-data-is-replicated"></a>Costos asociados a la modificación de la forma en que se replican los datos

Los costos asociados a la modificación de la forma en que se replican los datos dependen de la ruta de la conversión. Si se ordenan de la más económica a la más costosa, las ofertas de redundancia de Azure Storage son LRS, ZRS, GRS, RA-GRS, GZRS y RA-GZRS.

Por ejemplo, si se pasa *de* LRS a cualquier otra opción, se incurrirá en cargos adicionales porque se pasará a un nivel de redundancia más sofisticado. La migración *a* GRS o RA-GRS incurrirá en un cargo de ancho de banda de salida en el momento de la migración porque toda la cuenta de almacenamiento se replica en la región secundaria. Todas las escrituras posteriores en la región primaria también incurren en cargos de ancho de banda de salida para replicar la escritura en la región secundaria. Para obtener más información sobre los cargos de ancho de banda, consulte la [página de precios de Azure Storage](https://azure.microsoft.com/pricing/details/storage/blobs/).

Si migra la cuenta de almacenamiento de GRS a LRS no hay ningún costo adicional, pero los datos replicados se eliminarán de la ubicación secundaria.

> [!IMPORTANT]
> Si migra la cuenta de almacenamiento de RA-GRS a GRS o LRS, esa cuenta se factura como RA-GRS durante 30 días más después de la fecha de conversión.

## <a name="see-also"></a>Consulte también

- [Redundancia de Azure Storage](storage-redundancy.md)
- [Comprobación de la propiedad Hora de la última sincronización de una cuenta de almacenamiento](last-sync-time-get.md)
- [Uso de redundancia geográfica para diseñar aplicaciones de alta disponibilidad](geo-redundant-design.md)
