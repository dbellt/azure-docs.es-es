---
title: Configuración de una cuenta de Azure Cosmos DB con copia de seguridad periódica
description: En este artículo se explica cómo configurar cuentas de Azure Cosmos DB con copia de seguridad periódica con intervalo de copias de seguridad. y retención. También cómo ponerse en contacto con soporte técnico para restaurar los datos.
author: kanshiG
ms.service: cosmos-db
ms.topic: how-to
ms.date: 04/05/2021
ms.author: govindk
ms.reviewer: sngun
ms.openlocfilehash: 69677ed419fa9bac2cbcb06c394c92f68d0b7777
ms.sourcegitcommit: bd1a4e4df613ff24e954eb3876aebff533b317ae
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2021
ms.locfileid: "107930936"
---
# <a name="configure-azure-cosmos-db-account-with-periodic-backup"></a>Configuración de una cuenta de Azure Cosmos DB con copia de seguridad periódica
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Azure Cosmos DB crea automáticamente copias de seguridad de los datos a intervalos regulares. Las copias de seguridad automáticas se crean sin afectar al rendimiento o a la disponibilidad de las operaciones de las bases de datos. Todas las copias de seguridad se almacenan por separado en un servicio de almacenamiento y se replican globalmente para lograr resistencia frente a desastres regionales. Con Azure Cosmos DB no solo los datos, sino también las copias de seguridad de los mismos, son altamente redundantes y resistentes ante los desastres regionales. Los pasos siguientes muestran cómo Azure Cosmos DB realiza la copia de seguridad de datos:

* Azure Cosmos DB toma automáticamente una copia de seguridad completa de la base de datos cada 4 horas y en cualquier momento, pero solo se almacenan las últimas 2 de manera predeterminada. Si los intervalos predeterminados no son suficientes para las cargas de trabajo, puede cambiar el intervalo y el período de retención de copia de seguridad desde Azure Portal. Puede cambiar la configuración de copia de seguridad durante la creación de la cuenta de Azure Cosmos o con posterioridad. Si se elimina el contenedor o la base de datos, Azure Cosmos DB conserva las instantáneas existentes de un contenedor o base de datos dado durante 30 días.

* Azure Cosmos DB almacena estas copias de seguridad en Azure Blob Storage, mientras que los datos reales residen localmente dentro de Azure Cosmos DB.

* Para garantizar la baja latencia, la instantánea de la copia de seguridad se guarda en Azure Blob Storage en la misma región que la región de escritura actual (o **una** de las regiones de escritura, si tiene una configuración de escritura para varias regiones). Para lograr resistencia frente a desastres regionales, cada instantánea de la copia de seguridad de los datos en Azure Blob Storage se vuelve a replicar en otra región mediante almacenamiento con redundancia geográfica (GRS). La región a la que se replica la copia de seguridad se basa en la región de origen y el par regional asociado con la región de origen. Para más información, consulte el artículo [lista de parejas regionales de Azure para redundancia geográfica](../best-practices-availability-paired-regions.md). No se puede acceder directamente a esta copia de seguridad. El equipo de Azure Cosmos DB restaurará la copia de seguridad cuando lo pida mediante una solicitud de soporte técnico.

  La imagen siguiente muestra cómo se crea una copia de seguridad de un contenedor de Azure Cosmos con las tres particiones físicas principales en el Oeste de EE. UU. en una cuenta de Azure Blob Storage remota en el Oeste de EE. UU. y, luego, se replica al Este de EE. UU.:

  :::image type="content" source="./media/configure-periodic-backup-restore/automatic-backup.png" alt-text="Copias de seguridad completas periódicas de todas las entidades de Cosmos DB en Azure Storage con GRS" lightbox="./media/configure-periodic-backup-restore/automatic-backup.png" border="false":::

* Las copias de seguridad se crean sin afectar el rendimiento ni la disponibilidad de la aplicación. Azure Cosmos DB realiza la copia de seguridad de datos en segundo plano sin consumir ningún rendimiento aprovisionado (RU) adicional ni afectar al rendimiento y la disponibilidad de la base de datos.

> [!Note]
> No se admiten las cuentas con Synapse Link habilitado.

## <a name="backup-storage-redundancy"></a><a id="backup-storage-redundancy"></a>Redundancia del almacenamiento de copia de seguridad

De forma predeterminada, Azure Cosmos DB almacena los datos de copia de seguridad de modo periódico en el [almacenamiento de blobs](../storage/common/storage-redundancy.md) con redundancia geográfica que se replica en una [región emparejada](../best-practices-availability-paired-regions.md).  

Para asegurarse de que los datos de copia de seguridad permanecen dentro de la misma región donde se aprovisiona la cuenta de Azure Cosmos DB, puede cambiar la opción predeterminada de almacenamiento de copia de seguridad con redundancia geográfica y configurar un almacenamiento con redundancia local o con redundancia de zona. Los mecanismos de redundancia de almacenamiento almacenan varias copias de seguridad, con el fin de protegerlo de eventos planeados y no planeados, como errores transitorios del hardware, interrupciones del suministro eléctrico o cortes de la red, y desastres naturales masivos.

Los datos de copia de seguridad de Azure Cosmos DB se replican tres veces en la región primaria. Puede configurar la redundancia de almacenamiento para el modo de copia de seguridad periódica en el momento de creación de la cuenta o actualizarla para una cuenta existente. Puede usar las tres opciones de redundancia de datos siguientes en el modo de copia de seguridad periódica:

* **Almacenamiento de copia de seguridad con redundancia geográfica:** esta opción copia los datos de forma asincrónica en la región emparejada.

* **Almacenamiento de copia de seguridad con redundancia de zona:** esta opción copia los datos de forma sincrónica en tres zonas de disponibilidad de Azure de la región primaria.

* **Almacenamiento de copia de seguridad con redundancia local**: esta opción copia los datos de forma asincrónica tres veces dentro de una única ubicación física en la región primaria.

> [!NOTE]
> El almacenamiento con redundancia de zona solo está disponible actualmente en [regiones específicas](high-availability.md#availability-zone-support). En función de la región seleccionada; esta opción no estará disponible para las cuentas nuevas o existentes.
>
> La actualización de la redundancia del almacenamiento de copia de seguridad no afectará a los precios del almacenamiento de copia de seguridad.

## <a name="modify-the-backup-interval-and-retention-period"></a><a id="configure-backup-interval-retention"></a>Modificación del intervalo de copias de seguridad y el período de retención

Azure Cosmos DB toma automáticamente una copia de seguridad completa de los datos cada 4 horas y en cualquier momento, pero se almacenan las últimas 2. Esta configuración es la opción predeterminada y se ofrece sin ningún costo adicional. Puede cambiar el intervalo de copias de seguridad y el período de retención predeterminados durante la creación de la cuenta de Azure Cosmos o con posterioridad. La configuración de copia de seguridad se establece en el nivel de cuenta de Azure Cosmos y debe ajustarla en cada cuenta. Después de configurar las opciones de copia de seguridad de una cuenta, se aplica a todos los contenedores dentro de esa cuenta. Actualmente, solo puede cambiar las opciones de copia de seguridad en Azure Portal.

Si eliminó o dañó accidentalmente los datos, **antes de crear una solicitud de soporte técnico para restaurar los datos, asegúrese de aumentar la retención de copias de seguridad de la cuenta al menos a siete días. Es mejor aumentar la retención en un plazo de 8 horas después de este evento.** De esta manera, el equipo de Azure Cosmos DB tiene tiempo suficiente para restaurar la cuenta.

### <a name="modify-backup-options-for-an-existing-account"></a>Modificación de las opciones de copia de seguridad de una cuenta existente

Siga estos pasos para cambiar las opciones de copia de seguridad predeterminadas para una cuenta de Azure Cosmos existente:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
1. Vaya a la cuenta de Azure Cosmos y abra el panel **Copia de seguridad y restauración**. Actualice el intervalo de copia de seguridad y el período de retención de copia de seguridad según sea necesario.

   * **Intervalo de copia de seguridad**: es el intervalo en el que Azure Cosmos DB intenta realizar una copia de seguridad de los datos. La copia de seguridad tarda una cantidad de tiempo distinta de cero y, en algunos casos, podría producir un error debido a las dependencias de nivel inferior. Azure Cosmos DB intenta realizar una copia de seguridad en el intervalo configurado; sin embargo, no garantiza que la copia de seguridad se complete dentro de ese intervalo de tiempo. Este valor se puede configurar en horas o minutos. El intervalo de copia de seguridad no puede ser menor que 1 hora ni mayor que 24 horas. Al cambiar este intervalo, el nuevo intervalo entra en vigor a partir de la hora en que se realizó la última copia de seguridad.

   * **Retención de copia de seguridad**: representa el período en el que se conserva cada copia de seguridad. Puede configurarlo en horas o días. El período de retención mínimo no puede ser inferior a dos veces el intervalo de copia de seguridad (en horas) y no puede ser superior a 720 horas.

   * **Copias de los datos conservadas**: de manera predeterminada, se ofrecen dos copias de seguridad de los datos de forma gratuita. Se aplica un cargo adicional si se necesitan más de dos copias. Vea la sección Almacenamiento consumido de la [página Precios](https://azure.microsoft.com/pricing/details/cosmos-db/) para saber el precio exacto de las copias adicionales.

   * **Redundancia de almacenamiento de Backup**: elija la opción de redundancia de almacenamiento necesaria; consulte la sección [Redundancia de almacenamiento de Backup](#backup-storage-redundancy) para ver las opciones disponibles. De manera predeterminada, las cuentas del modo de copia de seguridad periódica existente tienen almacenamiento con redundancia geográfica. Puede elegir otro almacenamiento, como con redundancia local, para asegurarse de que la copia de seguridad no se replica en otra región. Los cambios realizados en una cuenta existente solo se aplican a las copias de seguridad futuras. Una vez actualizada la redundancia del almacenamiento de copia de seguridad de una cuenta existente, el intervalo de copia de seguridad puede tardar hasta el doble para que los cambios surtan efecto y se **perderá el acceso para restaurar las copias de seguridad más antiguas inmediatamente.**

   > [!NOTE]
   > Debe tener asignado el [rol de lector de cuenta de Azure Cosmos DB](../role-based-access-control/built-in-roles.md#cosmos-db-account-reader-role) en el nivel de suscripción para configurar la redundancia del almacenamiento de copia de seguridad.

   :::image type="content" source="./media/configure-periodic-backup-restore/configure-backup-options-existing-accounts.png" alt-text="Configuración del intervalo de copia de seguridad, la retención y la redundancia de almacenamiento para una cuenta de Azure Cosmos existente." border="true":::

### <a name="modify-backup-options-for-a-new-account"></a>Modificación de las opciones de copia de seguridad de una cuenta nueva

Al aprovisionar una cuenta nueva, en la pestaña **Directiva de copia de seguridad**, seleccione la directiva de copia de seguridad **Periodic** _ (Periódica). La directiva periódica le permite configurar el intervalo y la retención de las copias de seguridad, así como la redundancia del almacenamiento de copia de seguridad. Por ejemplo, puede elegir las opciones de _ *almacenamiento de copia de seguridad con redundancia local** o de **almacenamiento de copia de seguridad con redundancia de zona** para evitar la replicación de datos de copia de seguridad fuera de la región.

:::image type="content" source="./media/configure-periodic-backup-restore/configure-backup-options-new-accounts.png" alt-text="Configuración de una directiva de copia de seguridad periódica o continua para nuevas cuentas de Azure Cosmos." border="true":::

## <a name="request-data-restore-from-a-backup"></a><a id="request-restore"></a>Solicitud de una restauración de datos a partir de una copia de seguridad

En caso de que elimine accidentalmente la base de datos o un contenedor, puede [presentar una incidencia de soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) o [llamar al servicio de Soporte técnico de Azure](https://azure.microsoft.com/support/options/) para restaurar los datos a partir de copias de seguridad en línea automáticas. El Soporte técnico de Azure está disponible solo para algunos planes, por ejemplo, **Estándar** o **Desarrollador**, y planes superiores. El soporte técnico de Azure no está disponible con el plan **Básico**. Para más información sobre los diferentes planes de soporte técnico, consulte la página de [planes de soporte técnico de Azure](https://azure.microsoft.com/support/plans/).

Para restaurar una instantánea específica de la copia de seguridad, Azure Cosmos DB necesita que los datos estén disponibles durante el ciclo de copia de seguridad de esa instantánea.
Antes de solicitar una restauración, debe tener los siguientes detalles:

* Tenga listo el identificador de su suscripción.

* En función de cómo se hayan eliminado o modificado por accidente los datos, debe prepararse para tener información adicional. Se recomienda tener disponible la información con antelación para minimizar las idas y venidas que pueden ser perjudiciales en algunos casos donde el tiempo es fundamental.

* Si se elimina toda la cuenta de Azure Cosmos DB, deberá proporcionar el nombre de la cuenta eliminada. Si crea otra cuenta con el mismo nombre que la cuenta eliminada, indique esto al equipo de soporte técnico, ya que esto ayuda a establecer la cuenta adecuada que debe elegirse. Se recomienda presentar distintas incidencias de soporte técnico para cada cuenta eliminada, ya que esto minimiza la confusión del estado de la restauración.

* Si se han eliminado una o más bases de datos, debe proporcionar la cuenta de Azure Cosmos, así como los nombres de las bases de datos de Azure Cosmos, y especificar si existe alguna base de datos nueva con el mismo nombre.

* Si se han eliminado uno o varios contenedores, debe proporcionar el nombre de cuenta de Azure Cosmos, los nombres de las bases de datos y los nombres de los contenedores. Además, especifique si existe un contenedor con el mismo nombre.

* Si accidentalmente ha eliminado o dañado los datos, debe ponerse en contacto con el [soporte técnico de Azure](https://azure.microsoft.com/support/options/) en un plazo de ocho horas para que el equipo de Azure Cosmos DB pueda ayudarle a restaurar los datos a partir de las copias de seguridad. **Antes de crear una solicitud de soporte técnico para restaurar los datos, asegúrese de [aumentar la retención de copias de seguridad](#configure-backup-interval-retention) de la cuenta al menos a siete días. Es mejor aumentar la retención en un plazo de 8 horas después de este evento.** De este modo, el equipo de soporte técnico de Azure Cosmos DB tendrá suficiente tiempo para restaurar la cuenta.

Además del nombre de la cuenta de Azure Cosmos, los nombres de las bases de datos y los nombres de los contenedores, debe especificar el momento en el que se pueden restaurar los datos. Es importante ser lo más preciso posible para ayudarnos a determinar las mejores copias de seguridad disponibles en ese momento. **También es importante especificar la hora en UTC.**

En la captura de pantalla siguiente se muestra cómo crear una solicitud de soporte técnico para un contenedor (colección/grafo/tabla) para restaurar los datos mediante Azure Portal. Proporcione otros detalles, como el tipo de datos, el propósito de la restauración o la hora a la que se eliminaron los datos, para ayudarnos a clasificar la solicitud.

:::image type="content" source="./media/configure-periodic-backup-restore/backup-support-request-portal.png" alt-text="Creación de una solicitud de soporte técnico de copia de seguridad con Azure Portal." border="true":::

## <a name="considerations-for-restoring-the-data-from-a-backup"></a>Consideraciones para restaurar los datos a partir de una copia de seguridad

Puede eliminar o modificar accidentalmente los datos en uno de los siguientes escenarios:  

* Se elimina toda la cuenta de Azure Cosmos.

* Se elimina una o varias bases de datos de Azure Cosmos.

* Se elimina uno o varios contenedores de Azure Cosmos.

* Se eliminan o modifican elementos de Azure Cosmos (por ejemplo, documentos) dentro de un contenedor. Este caso concreto normalmente se conoce como "daño de datos".

* Se elimina o se daña una base de datos de oferta compartida o los contenedores dentro de una base de datos de oferta compartida.

Azure Cosmos DB puede restaurar los datos en todos los escenarios anteriores. Al restaurar, se crea una nueva cuenta de Azure Cosmos para almacenar los datos restaurados. El nombre de la nueva cuenta, si no se especifica, tendrá el formato `<Azure_Cosmos_account_original_name>-restored1`. Si se realizan varios intentos de restauración, el último dígito se va incrementando. No se pueden restaurar los datos en una cuenta de Azure Cosmos creada previamente.

Si elimina accidentalmente una cuenta de Azure Cosmos, se pueden restaurar los datos en una cuenta nueva con el mismo nombre, siempre que el nombre de la cuenta no esté en uso. Por lo tanto, se recomienda no volver a crear la cuenta después de eliminarla, porque ello no solo impide que los datos restaurados usen el mismo nombre, sino que además dificulta la detección de la cuenta correcta que se quiere restaurar.

Cuando elimina una base de datos de Azure Cosmos, podemos restaurar la base de datos completa o un subconjunto de los contenedores dentro de esta. También es posible seleccionar contenedores específicos en las bases de datos y restaurarlos en una nueva cuenta de Azure Cosmos.

Cuando elimina o modifica accidentalmente uno o varios elementos dentro de un contenedor (el caso del "daño de datos"), tendrá que especificar el momento al que se restaurará. La hora es importante si hay daños en los datos. Puesto que el contenedor está activo, las copias de seguridad se siguen ejecutando, por lo que si espera más allá del período de retención (el valor predeterminado es de ocho horas) se sobrescribirán las copias de seguridad. Para evitar que se sobrescriba la copia de seguridad, aumente la retención de copia de seguridad de la cuenta en al menos siete días. Es mejor aumentar la retención en un plazo de 8 horas a partir del daño en los datos.

Si accidentalmente ha eliminado o dañado los datos, debe ponerse en contacto con el [soporte técnico de Azure](https://azure.microsoft.com/support/options/) en un plazo de ocho horas para que el equipo de Azure Cosmos DB pueda ayudarle a restaurar los datos a partir de las copias de seguridad. De este modo, el equipo de soporte técnico de Azure Cosmos DB tendrá suficiente tiempo para restaurar la cuenta.

> [!NOTE]
> Después de restaurar los datos, no todas las capacidades o configuraciones de origen se trasladan a la cuenta restaurada. La siguiente configuración no se lleva a cabo en la cuenta nueva:
> * listas de control de acceso de VNET
> * Procedimientos almacenados, desencadenadores y funciones definidas por el usuario
> * Configuración de varias regiones  

Si aprovisiona el rendimiento en el nivel de base de datos, el proceso de copia de seguridad y restauración en este caso se produce a nivel de base de datos completa y no a nivel de contenedores individuales. En tales casos, no puede seleccionar un subconjunto de contenedores para restaurar.

## <a name="required-permissions-to-change-retention-or-restore-from-the-portal"></a>Permisos necesarios para cambiar la retención o restaurar desde el portal
Las entidades de seguridad que forman parte del rol [CosmosdbBackupOperator](../role-based-access-control/built-in-roles.md#cosmosbackupoperator), propietario o colaborador pueden solicitar una restauración o cambiar el período de retención.

## <a name="understanding-costs-of-extra-backups"></a>Reconocimiento de los costos de las copias de seguridad adicionales
Se proporcionan dos copias de seguridad gratuitas y las adicionales se cobran según los precios de la región del almacenamiento de copia de seguridad que se describen en los [precios de almacenamiento de copia de seguridad](https://azure.microsoft.com/pricing/details/cosmos-db/). Por ejemplo, si la retención de copia de seguridad está configurada en 240 horas, es decir, 10 días, y el intervalo de copia de seguridad en 24 horas, eso significa 10 copias de los datos de copia de seguridad. Supongamos 1 TB de datos en Oeste de EE. UU. 2, el coste sería 0,12 * 1000 * 8 por el almacenamiento de copia de seguridad de un mes determinado.

## <a name="options-to-manage-your-own-backups"></a>Opciones para gestionar sus propias copias de seguridad

Con las cuentas de la API de SQL de Azure Cosmos DB, también puede mantener sus propias copias de seguridad mediante uno de los métodos siguientes:

* Con [Azure Data Factory](../data-factory/connector-azure-cosmos-db.md) para mover datos periódicamente a un almacenamiento de su elección.

* Con la [fuente de cambios](change-feed.md) de Azure Cosmos DB para leer datos periódicamente para copias de seguridad completas o para cambios incrementales, y almacenarlo todo en su propio almacenamiento.

## <a name="post-restore-actions"></a>Acciones posteriores a la restauración

El objetivo principal de la restauración de datos es recuperar los datos que ha eliminado o modificado accidentalmente. Por lo tanto, se recomienda que primero inspeccione el contenido de los datos recuperados para garantizar que contiene lo que esperaba. Si todo tiene buen aspecto, puede volver a migrar los datos a la cuenta principal. Aunque se puede usar la cuenta restaurada como nueva cuenta activa, no es una opción recomendada si tiene cargas de trabajo de producción. 

Después de restaurar los datos, recibirá una notificación sobre el nombre de la nueva cuenta (normalmente se encuentra en el formato `<original-name>-restored1`) y la hora a la que se ha restaurado la cuenta. La cuenta restaurada tendrá el mismo rendimiento aprovisionado, directivas de indexación y estará en la misma región que la cuenta original. Un usuario que sea administrador de suscripciones o administrador conjunto puede ver la cuenta restaurada.

### <a name="migrate-data-to-the-original-account"></a>Migración de datos a la cuenta original

Estas son las diferentes formas de volver a migrar datos a la cuenta original:

* Use la [herramienta de migración de datos de Azure Cosmos DB](import-data.md).
* Use [Azure Data Factory](../data-factory/connector-azure-cosmos-db.md).
* Use la [fuente de cambios](change-feed.md) en Azure Cosmos DB.
* Puede escribir su propio código personalizado.

Se recomienda eliminar el contenedor o la base de datos inmediatamente después de migrar los datos. Si no elimina las bases de datos o los contenedores restaurados, supondrá un costo para las unidades de solicitud, el almacenamiento y la salida.

## <a name="next-steps"></a>Pasos siguientes

* Para efectuar una solicitud de restauración, póngase en contacto con Soporte técnico de Azure y [rellene un vale en Azure Portal](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).
* Configure y administre la copia de seguridad continua mediante [Azure Portal](continuous-backup-restore-portal.md), [PowerShell](continuous-backup-restore-powershell.md), la [CLI](continuous-backup-restore-command-line.md) o [Azure Resource Manager](continuous-backup-restore-template.md).
* [Administre los permisos](continuous-backup-restore-permissions.md) necesarios para restaurar datos con el modo de copia de seguridad continua.
