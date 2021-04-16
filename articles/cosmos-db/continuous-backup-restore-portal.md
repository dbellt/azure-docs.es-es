---
title: Configuración de copia de seguridad continua y restauración a un momento dado con Azure Portal en Azure Cosmos DB
description: Aprenda a identificar el punto de restauración y a configurar la copia de seguridad continua con Azure Portal. Muestra cómo restaurar una cuenta dinámica y eliminada.
author: kanshiG
ms.service: cosmos-db
ms.topic: how-to
ms.date: 04/05/2021
ms.author: govindk
ms.reviewer: sngun
ms.openlocfilehash: 707ef9f60891c1da7c13638e233ee74e78fc20dd
ms.sourcegitcommit: b8995b7dafe6ee4b8c3c2b0c759b874dff74d96f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2021
ms.locfileid: "106283944"
---
# <a name="configure-and-manage-continuous-backup-and-point-in-time-restore-preview---using-azure-portal"></a>Configuración y administración de copia de seguridad continua y restauración a un momento dado (versión preliminar) con Azure Portal
[!INCLUDE[appliesto-sql-mongodb-api](includes/appliesto-sql-mongodb-api.md)]

> [!IMPORTANT]
> La característica de restauración a un momento dado (modo de copia de seguridad continua) de Azure Cosmos DB está actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

La característica de restauración a un momento dado (versión preliminar) de Azure Cosmos DB ayuda a recuperarse de un cambio accidental en un contenedor, a restaurar una cuenta, una base de datos o un contenedor eliminados o a realizar una restauración en cualquier región (donde existan copias de seguridad). El modo de copia de seguridad continua permite realizar la restauración a cualquier momento dado de los últimos 30 días.

En este artículo se describe cómo identificar el punto de restauración y configurar la copia de seguridad continua con Azure Portal.

## <a name="provision-an-account-with-continuous-backup"></a><a id="provision"></a>Aprovisionamiento de una cuenta con copia de seguridad continua

Al crear una cuenta de Azure Cosmos DB, en la opción **Directiva de copia de seguridad**, elija el modo **continuo** para habilitar la funcionalidad de restauración a un momento dato para la cuenta nueva. Después de habilitar esta característica para la cuenta, todas las bases de datos y contenedores estarán disponibles para la copia de seguridad continua. Con la restauración a un momento dado, los datos siempre se restauran en una cuenta nueva; actualmente, no se puede restaurar en una cuenta existente.

:::image type="content" source="./media/continuous-backup-restore-portal/configure-continuous-backup-portal.png" alt-text="Aprovisionamiento de una cuenta de Azure Cosmos DB con la configuración de copia de seguridad continua." border="true":::

## <a name="backup-storage-redundancy"></a>Redundancia del almacenamiento de copia de seguridad

De forma predeterminada, Azure Cosmos DB almacena datos de copia de seguridad en modo continuo en blobs de almacenamiento con redundancia local. En el caso de las regiones que tienen configurada la redundancia de zona, la copia de seguridad se almacena en blobs de almacenamiento con redundancia de zona. En este modo, no se puede actualizar la redundancia de almacenamiento de copia de seguridad.

## <a name="restore-a-live-account-from-accidental-modification"></a><a id="restore-live-account"></a>Restauración de una cuenta activa después de una modificación accidental

Puede usar Azure Portal para restaurar una cuenta activa o las bases de datos y contenedores seleccionados en ella. Siga estos pasos para restaurar los datos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
1. Vaya a la cuenta de Azure Cosmos DB y abra el panel **Restauración a un momento dado**.

   > [!NOTE]
   > El panel de restauración en Azure Portal solo se completa si cuenta con el permiso`Microsoft.DocumentDB/locations/restorableDatabaseAccounts/*/read`. Para más información sobre cómo establecer este permiso, consulte el artículo sobre los [permisos de copia de seguridad y restauración](continuous-backup-restore-permissions.md).

1. Complete estos detalles para realizar la restauración:

   * **Punto de restauración (UTC)** : marca de tiempo dentro de los últimos 30 días. La cuenta debe existir en esa marca de tiempo. Puede especificar el punto de restauración en hora UTC. El valor puede acercarse lo más posible al segundo que quiere restaurar. Seleccione el vínculo **Haga clic aquí** para obtener ayuda sobre cómo [identificar el punto de restauración](#event-feed).

   * **Ubicación**: región de destino en que se restaura la cuenta. La cuenta debe existir en esta región en la marca de tiempo especificada (por ejemplo, Oeste de EE. UU. o Este de EE. UU.). Una cuenta solo se puede restaurar en las regiones en las que existe la cuenta de origen.

   * **Recurso de restauración**: puede elegir restaurar **toda la cuenta** o una **base de datos o contenedor seleccionados**. Las bases de datos y los contenedores deben existir en la marca de tiempo especificada. Los recursos de restauración se rellenan en función de la ubicación y el punto de restauración seleccionados, lo que permite al usuario seleccionar bases de datos o contenedores específicos que se deben restaurar.

   * **Grupo de recursos**: grupo de recursos en el que se creará y restaurará la cuenta de destino. El grupo de recursos ya debe existir.

   * **Cuenta de destino de restauración**: nombre de la cuenta de destino. El nombre de la cuenta de destino debe seguir las mismas directrices que cuando se crea una cuenta. El proceso de restauración creará esta cuenta en la misma región en la que existe la cuenta de destino.
 
   :::image type="content" source="./media/continuous-backup-restore-portal/restore-live-account-portal.png" alt-text="Restauración de una cuenta activa después de una modificación accidental con Azure Portal." border="true":::

1. Después de seleccionar los parámetros anteriores, haga clic en el botón **Enviar** para iniciar una restauración. El costo de la restauración es un cargo único que se basa en la cantidad de datos y en los cargos del almacenamiento en la región especificada. Para más información, consulte la sección de [precios](continuous-backup-restore-introduction.md#continuous-backup-pricing).

## <a name="use-event-feed-to-identify-the-restore-time"></a><a id="event-feed"></a>Uso de una fuente de eventos para identificar la hora de restauración

Al rellenar la hora del punto de restauración en Azure Portal, si necesita ayuda para identificar el punto de restauración, seleccione el vínculo **Haga clic aquí** que lo lleva a la hoja de la fuente de eventos. La fuente de eventos proporciona una lista de fidelidad total de eventos de creación, reemplazo y eliminación en bases de datos y contenedores de la cuenta de origen. 

Por ejemplo, si quiere restaurar al punto antes de la eliminación o actualización de un contenedor determinado, compruebe esta fuente de eventos. Los eventos se muestran en orden cronológico descendente respecto del momento en que se produjeron, con los eventos recientes en la parte superior. Puede examinar los resultados y seleccionar la hora antes o después del evento para restringir aún más el tiempo.

:::image type="content" source="./media/continuous-backup-restore-portal/event-feed-portal.png" alt-text="Uso de una fuente de eventos para identificar la hora del punto de restauración." border="true":::

> [!NOTE]
> La fuente de eventos no muestra los cambios en los recursos del elemento. Siempre puede especificar manualmente cualquier marca de tiempo de los últimos 30 días (siempre que exista una cuenta en ese momento) para la restauración.

## <a name="restore-a-deleted-account"></a><a id="restore-deleted-account"></a>Restauración de una cuenta eliminada

Puede usar Azure Portal para restaurar completamente una cuenta eliminada en un plazo de 30 días a contar de su eliminación. Siga estos pasos para restaurar una cuenta eliminada:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
1. Busque recursos de *Azure Cosmos DB* en la barra de búsqueda global. Se muestran todas las cuentas existentes.
1. Luego, haga clic en el botón **Restaurar**. En el panel Restaurar se muestra una lista de las cuentas eliminadas que se pueden restaurar dentro del período de retención, que es de 30 días a contar del momento de la eliminación.
1. Elija la cuenta que quiere restaurar.

   :::image type="content" source="./media/continuous-backup-restore-portal/restore-deleted-account-portal.png" alt-text="Restauración de una cuenta eliminada desde Azure Portal." border="true":::

   > [!NOTE]
   > Nota: El panel de restauración en Azure Portal solo se completa si cuenta con el permiso`Microsoft.DocumentDB/locations/restorableDatabaseAccounts/*/read`. Para más información sobre cómo establecer este permiso, consulte el artículo sobre los [permisos de copia de seguridad y restauración](continuous-backup-restore-permissions.md).

1. Seleccione una cuenta para restaurar y escriba los detalles siguientes para restaurar una cuenta eliminada:

   * **Punto de restauración (UTC)** : marca de tiempo dentro de los últimos 30 días. La cuenta debe haber existido en esa marca de tiempo. Especifique el punto de restauración en hora UTC. El valor puede acercarse lo más posible al segundo que quiere restaurar.

   * **Ubicación**: región de destino en que se debe restaurar la cuenta. La cuenta de origen debe existir en esta región en la marca de tiempo especificada. Por ejemplo, Oeste de EE. UU., Este de EE. UU.  

   * **Grupo de recursos**: grupo de recursos en el que se creará y restaurará la cuenta de destino. El grupo de recursos ya debe existir.

   * **Cuenta de destino de restauración**: el nombre de la cuenta de destino debe seguir las mismas directrices que cuando se crea una cuenta. El proceso de restauración creará esta cuenta en la misma región en la que existe la cuenta de destino.

## <a name="track-the-status-of-restore-operation"></a><a id="track-restore-status"></a>Seguimiento del estado de una operación de restauración

Después de iniciar una operación de restauración, seleccione el icono de campana de **Notificación** que se encuentra en la esquina superior derecha del portal. Proporciona un vínculo que muestra el estado de la cuenta que se está restaurando. Mientras la restauración esté en curso, el estado de la cuenta será *En creación* y, cuando la operación de restauración se complete, cambiará a *En línea*.

:::image type="content" source="./media/continuous-backup-restore-portal/track-restore-operation-status.png" alt-text="El estado de la cuenta restaurada cambia de En creación a En línea cuando se completa la operación." border="true":::

## <a name="next-steps"></a>Pasos siguientes

* Configure y administre la copia de seguridad continua mediante la [CLI de Azure](continuous-backup-restore-command-line.md), [PowerShell](continuous-backup-restore-powershell.md) o [Azure Resource Manager](continuous-backup-restore-template.md).
* [Modelo de recursos del modo de copia de seguridad continua](continuous-backup-restore-resource-model.md)
* [Administre los permisos](continuous-backup-restore-permissions.md) necesarios para restaurar datos con el modo de copia de seguridad continua.
