---
title: Migración de la serie 8000 de StorSimple a Azure File Sync
description: Aprenda a migrar un dispositivo de las series 8100 o 8600 de StorSimple a Azure File Sync.
author: fauhse
ms.service: storage
ms.topic: how-to
ms.date: 10/16/2020
ms.author: fauhse
ms.subservice: files
ms.openlocfilehash: 1d2de439e661ef5b1d1669187355621f25400bc4
ms.sourcegitcommit: 3ee3045f6106175e59d1bd279130f4933456d5ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/31/2021
ms.locfileid: "106075633"
---
# <a name="storsimple-8100-and-8600-migration-to-azure-file-sync"></a>Migración de las series 8100 y 8600 de StorSimple a Azure File Sync

La serie StorSimple 8000 está representada por los dispositivos físicos y locales 8100 o 8600 y sus componentes de servicios en la nube. Los datos de uno de estos dispositivos se pueden migrar a un entorno de Azure File Sync. Azure File Sync es el servicio de Azure a largo plazo, estratégico y predeterminado al que se pueden migrar los dispositivos StorSimple.

StorSimple serie 8000 alcanzará el [final de su ciclo de vida](https://support.microsoft.com/en-us/lifecycle/search?alpha=StorSimple%208000%20Series) en diciembre de 2022. Es importante empezar a planear la migración lo antes posible. En este artículo se proporcionan los conocimientos básicos y los pasos de migración necesarios para realizar una migración correcta a Azure File Sync.

## <a name="phase-1-prepare-for-migration"></a>Fase 1: Preparación para la migración

Esta sección contiene los pasos que debe seguir al principio de la migración de volúmenes de StorSimple a recursos compartidos de archivos de Azure.

### <a name="inventory"></a>Tema de

Cuando empiece a planear la migración, identifique primero todos los dispositivos y volúmenes de StorSimple que necesita migrar. Una vez hecho esto, puede decidir cuál es la mejor ruta de migración.

* Los dispositivos físicos de StorSimple (serie 8000) usan esta guía de migración.
* Los dispositivos virtuales, [serie StorSimple 1200, usan una guía de migración diferente](storage-files-migration-storsimple-1200.md).

### <a name="migration-cost-summary"></a>Resumen de costos de migración

Las migraciones a recursos compartidos de archivos de Azure desde volúmenes de StorSimple a través de trabajos de migración en un recurso de StorSimple Data Manager son gratuitas. Pueden producirse otros costos durante y después de la migración:

* **Salida de red:** los archivos de StorSimple residen en una cuenta de almacenamiento dentro de una región de Azure específica. Si aprovisiona los recursos compartidos de archivos de Azure que migra a una cuenta de almacenamiento que se encuentra en la misma región de Azure, no se incurrirá en ningún costo de salida. Puede trasladar los archivos a una cuenta de almacenamiento en una región diferente como parte de esta migración. En ese caso, se le aplicarán costos de salida.
* **Transacciones de recursos compartido de archivos de Azure:** cuando los archivos se copian en un recurso compartido de archivos de Azure (como parte de una migración o fuera de una), los costos de transacción se aplican a medida que se escriben archivos y metadatos. Como procedimiento recomendado, inicie el recurso compartido de archivos de Azure del nivel optimizado para transacciones durante la migración. Cambie al nivel deseado después de finalizar la migración. Las fases siguientes recordarán este paso en el momento apropiado.
* **Cambio del nivel de un recurso compartido de archivos de Azure:** cambiar el nivel de un recurso compartido de archivos de Azure conlleva transacciones. En la mayoría de los casos, será más rentable seguir los consejos del punto anterior.
* **Costo de almacenamiento**: cuando esta migración comienza a copiar archivos en un recurso compartido de archivos de Azure, se consume almacenamiento de Azure Files y se factura. Las copias de seguridad migradas se convertirán en [instantáneas de recurso compartido de archivos de Azure](storage-snapshots-files.md). Las instantáneas de recurso compartido de archivos solo consumen la capacidad de almacenamiento para las diferencias que contienen.
* **StorSimple:** hasta que tenga la oportunidad de desaprovisionar las cuentas de almacenamiento y los dispositivos de StorSimple, se seguirá incurriendo el costo de StorSimple por almacenamiento, copias de seguridad y dispositivos.

### <a name="direct-share-access-vs-azure-file-sync"></a>Acceso directo de recursos compartidos frente a Azure File Sync

Los recursos compartidos de archivos de Azure proporcionan un sinfín de oportunidades para estructurar la implementación de servicios de archivo. Un recurso compartido de archivos de Azure es simplemente un recurso compartido de SMB en la nube, que puede configurar para que los usuarios tengan acceso directo a través del protocolo SMB con la autenticación Kerberos conocida y los permisos NTFS existentes (ACL de archivos y carpetas) funcionen de forma nativa. Obtenga más información sobre el [acceso basado en identidad a recursos compartidos de archivos de Azure](storage-files-active-directory-overview.md).

Una alternativa al acceso directo es [Azure File Sync](./storage-sync-files-planning.md). Azure File Sync es un análogo directo de la capacidad de StorSimple de almacenar en caché los archivos que se usan con frecuencia de forma local.

Azure File Sync es un servicio en la nube de Microsoft, que se basa en dos componentes principales:

* Sincronización de archivos y nube por niveles para crear una caché de acceso de rendimiento en cualquier servidor de Windows.
* Recursos compartidos de archivos como almacenamiento nativo en Azure, a los que se puede acceder a través de varios protocolos, como SMB y REST de archivos.

Los recursos compartidos de archivos de Azure conservan aspectos importantes de la fidelidad de archivos en archivos almacenados como atributos, permisos y marcas de tiempo. Con los recursos compartidos de Azure, ya no es necesario una aplicación o servicio que interprete los archivos y carpetas que están almacenados en la nube. Puede acceder a ellos de forma nativa mediante protocolos y clientes conocidos como el Explorador de archivos de Windows. Los recursos compartidos de archivos de Azure permiten almacenar datos de aplicaciones y de servidores de archivos de uso general en la nube. La copia de seguridad de un recurso compartido de archivos de Azure es una funcionalidad integrada que se puede mejorar gracias a Azure Backup.

Este artículo se centra en los pasos de migración. Si desea obtener más información sobre Azure File Sync antes de la migración, consulte los siguientes artículos:

* [Introducción a Azure File Sync](./storage-sync-files-planning.md "Introducción")
* [Guía de implementación de Azure File Sync](storage-sync-files-deployment-guide.md)

### <a name="storsimple-service-data-encryption-key"></a>Clave de cifrado de datos del servicio de StorSimple

La primera vez que se configura el dispositivo de StorSimple, se genera una "clave de cifrado de datos del servicio" y se le indica que almacene la clave de forma segura. Esta clave se usa para cifrar todos los datos de la cuenta de almacenamiento de Azure asociada, en la que el dispositivo de StorSimple almacena los archivos.

La "clave de cifrado de datos del servicio" es necesaria para llevar a cabo una migración correcta. Ahora es un buen momento para recuperar esta clave de los registros, una para cada dispositivo del inventario.

Si no puede encontrar las claves en los registros, puede recuperar una nueva clave desde el dispositivo. Cada dispositivo tiene una clave de cifrado única.

#### <a name="change-the-service-data-encryption-key"></a>Cambiar la clave de cifrado de datos de servicio

[!INCLUDE [storage-files-migration-generate-key](../../../includes/storage-files-migration-generate-key.md)]

> [!CAUTION]
> Cuando decida cómo conectarse a su dispositivo de StorSimple, tenga en cuenta lo siguiente:
>
> * La conexión a través de una sesión HTTPS es la opción más segura y la recomendada.
> * Es seguro conectarse directamente a la consola en serie del dispositivo, pero la conexión a la consola en serie a través conmutadores de red no lo es.
> * Las conexiones de sesión HTTP son una opción, pero *no están cifradas*. No se recomiendan a menos que se usen en una red cerrada y de confianza.

### <a name="storsimple-volume-backups"></a>Copias de seguridad de volúmenes de StorSimple

StorSimple ofrece copias de seguridad diferenciales en el nivel de los volúmenes. Los recursos compartidos de archivos de Azure también tienen esta capacidad, denominada "instantáneas de recurso compartido".
Los trabajos de migración solo pueden migrar copias de seguridad, no datos del volumen activo. Por lo tanto, la copia de seguridad más reciente siempre debe estar en la lista de copias de seguridad que se han trasladado en una migración.

Decida si necesita trasladar alguna copia de seguridad anterior durante la migración.
El procedimiento recomendado consiste en mantener esta lista lo más breve posible, de modo que los trabajos de migración se completen más rápido.

Para identificar las copias de seguridad críticas que se deben migrar, realice una lista de comprobación de las directivas de copia de seguridad. Por ejemplo:
* La copia de seguridad más reciente. (Nota: la copia de seguridad más reciente siempre debe formar parte de esta lista).
* Una copia de seguridad al mes durante 12 meses.
* Una copia de seguridad al año durante tres años. 

Más adelante, al crear los trabajos de migración, puede usar esta lista para identificar las copias de seguridad del volumen de StorSimple exactas que se deben migrar para cumplir los requisitos de la lista.

> [!CAUTION]
> No se admite la selección de más de **50** copias de seguridad de volumen de StorSimple.
> Los trabajos de migración solo pueden migrar copias de seguridad, nunca datos del volumen activo. Por lo tanto, la copia de seguridad más reciente es más cercana a los datos activos y, por lo tanto, debe formar parte de la lista de copias de seguridad que se trasladarán en una migración.

### <a name="map-your-existing-storsimple-volumes-to-azure-file-shares"></a>Asignación de volúmenes de StorSimple a recursos compartidos de archivos de Azure

[!INCLUDE [storage-files-migration-namespace-mapping](../../../includes/storage-files-migration-namespace-mapping.md)]

### <a name="number-of-storage-accounts"></a>Número de cuentas de almacenamiento

Es probable que la migración se beneficie de una implementación de varias cuentas de almacenamiento, cada una con un número de recursos compartidos de archivos de Azure más reducido.

Si tiene recursos compartidos muy activos (utilizados por muchos usuarios o aplicaciones), dos recursos compartidos de archivos de Azure podrían alcanzar el límite de rendimiento de una cuenta de almacenamiento. Por este motivo, el procedimiento recomendado es migrar a varias cuentas de almacenamiento, cada una con sus propios recursos compartidos de archivos y, por lo general, no más de dos o tres recursos compartidos por cuenta de almacenamiento.

Un procedimiento recomendado es implementar cuentas de almacenamiento con un recurso compartido de archivos cada una. Puede agrupar varios recursos compartidos de archivos de Azure en la misma cuenta de almacenamiento, en caso de que tenga recursos compartidos de archivo.

Estas consideraciones se aplican más al [acceso directo a la nube](#direct-share-access-vs-azure-file-sync) (a través de una VM de Azure) que a Azure File Sync. Si tiene pensado usar exclusivamente Azure File Sync en estos recursos compartidos, es correcta la agrupación de varios en una sola cuenta de almacenamiento de Azure. En el futuro, se recomienda migrar mediante lift-and-shift una aplicación a la nube que, a continuación, tendrá acceso directo a un recurso compartido de archivos, ya que este escenario se beneficiará de tener más IOPS y rendimiento. También puede empezar a usar un servicio de Azure que también se beneficiará de tener un mayor IOPS y rendimiento.

Si ha creado una lista de recursos compartidos, asigne cada recurso compartido a la cuenta de almacenamiento en la que residirá.

> [!IMPORTANT]
> Seleccione una región de Azure y asegúrese de que cada cuenta de almacenamiento y recurso de Azure File Sync coinciden con la región que ha seleccionado.
> No configure ahora las opciones de la red y del firewall para las cuentas de almacenamiento. Realizar estas configuraciones en este momento haría imposible la migración. Configure estas opciones de Azure Storage una vez completada la migración.

### <a name="phase-1-summary"></a>Resumen de la fase 1

Al final de la fase 1:

* Conocerá bien sus volúmenes y dispositivos de StorSimple.
* El servicio de Data Manager estará listo para acceder a los volúmenes de StorSimple en la nube, ya que ha recuperado la "clave de cifrado de datos del servicio" de cada dispositivo de StorSimple.
* Tendrá un plan para el que se deben migrar los volúmenes y las copias de seguridad (si hay alguno después de los más recientes).
* Sabrá cómo asignar los volúmenes al número adecuado de recursos compartidos de archivo y cuentas de almacenamiento de Azure.

## <a name="phase-2-deploy-azure-storage-and-migration-resources"></a>Fase 2: Implementación de recursos de migración y almacenamiento de Azure

En esta sección se explican las consideraciones sobre la implementación de los diferentes tipos de recursos que se necesitan en Azure. Algunos contendrán los datos después de la migración y otros solo se necesitan para la migración. No empiece a implementar recursos hasta que haya finalizado el plan de implementación. Es difícil, y a veces imposible, cambiar determinados aspectos de los recursos de Azure una vez que se han implementado.

### <a name="deploy-storage-accounts"></a>Implementación de cuentas de almacenamiento

Probablemente, tendrá que implementar varias cuentas de Azure Storage. Cada una de ellas contendrá un número menor de recursos compartidos de archivos de Azure, según el plan de implementación, completado en la sección anterior de este artículo. Vaya a Azure Portal para [implementar las cuentas de almacenamiento planeadas](../common/storage-account-create.md#create-a-storage-account). Considere la posibilidad de seguir la configuración básica de a continuación en cualquier cuenta de almacenamiento nueva.

> [!IMPORTANT]
> No configure ahora las opciones de la red y del firewall para sus cuentas de almacenamiento. Realizar esas configuraciones en este momento haría imposible la migración. Configure estas opciones de Azure Storage una vez completada la migración.

#### <a name="subscription"></a>Subscription

Puede usar la misma suscripción que usó para la implementación de StorSimple o una diferente. La única limitación es que la suscripción debe estar en el mismo inquilino de Azure Active Directory que la suscripción de StorSimple. Considere la posibilidad de mover la suscripción a StorSimple al inquilino adecuado antes de iniciar una migración. Solo puede trasladar la suscripción en su conjunto; los recursos individuales de StorSimple no se pueden trasladar a otro inquilino o suscripción.

#### <a name="resource-group"></a>Resource group

Los grupos de recursos ayudan con la organización de recursos y permisos de administración de administradores. Obtenga más información sobre los [grupos de recursos de Azure](../../azure-resource-manager/management/manage-resource-groups-portal.md#what-is-a-resource-group).

#### <a name="storage-account-name"></a>Nombre de la cuenta de almacenamiento

El nombre de la cuenta de almacenamiento pasará a formar parte de una dirección URL y tendrá ciertas limitaciones de caracteres. En su convención de nomenclatura, tenga en cuenta que los nombres de las cuentas de almacenamiento deben ser únicos, tener solo letras en minúsculas y números, contener entre 3 y 24 caracteres y no usar caracteres especiales como guiones o caracteres de subrayado. Para obtener más información, consulte [Reglas de nomenclatura de recursos de almacenamiento de Azure](../../azure-resource-manager/management/resource-name-rules.md#microsoftstorage).

#### <a name="location"></a>Location

La ubicación o la región de Azure de una cuenta de almacenamiento son muy importantes. Si usa Azure File Sync, todas las cuentas de almacenamiento deben estar en la misma región que el recurso del servicio de sincronización de almacenamiento. La región de Azure que elija debe ser cercana a los servidores y usuarios locales o ubicarse en un punto céntrico. Una vez implementado el recurso, no se puede cambiar su región.

Puede elegir una región distinta de la ubicación en la que residen los datos de StorSimple (cuenta de almacenamiento).

> [!IMPORTANT]
> Si elige una región diferente de la ubicación actual de la cuenta de almacenamiento de StorSimple, [se aplicarán cargos de salida](https://azure.microsoft.com/pricing/details/bandwidth) durante la migración. Los datos dejarán la región de StorSimple y entrarán en la nueva región de la cuenta de almacenamiento. No se aplican cargos de ancho de banda si permanece dentro de la misma región de Azure.

#### <a name="performance"></a>Rendimiento

Tiene la opción de elegir Premium Storage (SSD) para recursos compartidos de archivos de Azure o almacenamiento estándar. El almacenamiento estándar incluye [varios niveles para un recurso compartido de archivos](storage-how-to-create-file-share.md#changing-the-tier-of-an-azure-file-share). Standard Storage es la opción adecuada para la mayoría de los clientes que migran desde StorSimple.

¿Todavía no está seguro?

* Elija Premium Storage si necesita el [rendimiento de un recurso compartido de archivos Premium de Azure](understanding-billing.md#provisioned-model).
* Elija almacenamiento estándar para cargas de trabajo de servidor de archivos de uso general, incluidos los datos de acceso frecuente y los datos de archivo. Elija también almacenamiento estándar si la única carga de trabajo en el recurso compartido en la nube será Azure File Sync.

#### <a name="account-kind"></a>Tipo de cuenta

* En el caso de Standard Storage, elija *StorageV2 (uso general V2)* .
* Para recursos compartidos de archivos prémium, elija *FileStorage*.

#### <a name="replication"></a>Replicación

Hay varias configuraciones de replicación disponibles. Obtenga más información sobre los diferentes tipos de replicación.

Solo tiene que elegir una de las dos opciones siguientes:

* *Almacenamiento con redundancia local (LRS)* .
* *Almacenamiento con redundancia de zona (ZRS)* , que no está disponible en todas las regiones de Azure.

> [!NOTE]
> Solo los tipos de redundancia LRS y ZRS son compatibles con los recursos compartidos de archivos de Azure de gran tamaño, de 100 TiB de capacidad.

El almacenamiento con redundancia geográfica (GRS) en todas las variaciones no se admite actualmente. Puede cambiar el tipo de redundancia más tarde y a GRS cuando la compatibilidad con este tipo llegue a Azure.

#### <a name="enable-100-tib-capacity-file-shares"></a>Habilitación de recursos compartidos de archivos de 100 TiB de capacidad

:::row:::
    :::column:::
        :::image type="content" source="media/storage-files-how-to-create-large-file-share/large-file-shares-advanced-enable.png" alt-text="Imagen que muestra la pestaña Opciones avanzadas en Azure Portal para la creación de una cuenta de almacenamiento.":::
    :::column-end:::
    :::column:::
        En la sección **Opciones avanzadas** del Asistente para nuevas cuentas de almacenamiento de Azure Portal, puede habilitar la compatibilidad con **recursos compartidos de archivos grandes** en esta cuenta de almacenamiento. Si esta opción no está disponible, lo más probable es que haya seleccionado el tipo de redundancia equivocado. Asegúrese de seleccionar solo LRS o ZRS para que esta opción esté disponible.
    :::column-end:::
:::row-end:::

Optar por los recursos compartidos de archivos de gran tamaño de 100 TiB tiene varias ventajas:

* El rendimiento aumenta en gran medida en comparación con los recursos compartidos de archivos de menor capacidad, de 5 TiB (por ejemplo, 10 veces respecto a IOPS).
* La migración finalizará mucho más rápido.
* Asegúrese de que un recurso compartido de archivos tenga suficiente capacidad para contener todos los datos que va a migrar a él, incluida la capacidad de almacenamiento que requieren las copias de seguridad diferenciales.
* El crecimiento futuro está cubierto con esta opción.

### <a name="azure-file-shares"></a>Recursos compartidos de archivos de Azure

Una vez creadas las cuentas de almacenamiento, puede ir a la sección **Recurso compartido de archivos** de la cuenta de almacenamiento e implementar el número adecuado de recursos compartidos de archivos de Azure según el plan de migración de la fase 1. Considere la posibilidad de usar la siguiente configuración básica para los nuevos recursos compartidos de archivos de Azure.

:::row:::
    :::column:::
        :::image type="content" source="media/storage-files-migration-storsimple-8000/storage-files-migration-storsimple-8000-new-share.png" alt-text="Captura de pantalla de Azure Portal que muestra la nueva interfaz de usuario del recurso compartido de archivos.":::
    :::column-end:::
    :::column:::
        </br>**Nombre**</br>Se admiten minúsculas, números y guiones.</br></br>**Cuota**</br>La cuota aquí es comparable a la cuota máxima de SMB en una instancia de Windows Server. El procedimiento recomendado es no establecer una cuota aquí, ya que la migración y otros servicios producirán un error cuando se alcance la cuota.</br></br>**Niveles**</br>Seleccione **Optimizado para transacciones** en el nuevo recurso compartido de archivos. Durante la migración, se producirán muchas transacciones. Es más rentable cambiar el nivel más adelante al nivel más adecuado para la carga de trabajo.
    :::column-end:::
:::row-end:::

### <a name="storsimple-data-manager"></a>StorSimple Data Manager

El recurso de Azure que contendrá los trabajos de migración se llama **StorSimple Data Manager**. Seleccione **Nuevo recurso** y búsquelo. Seleccione **Crear**.

Este recurso temporal se usa para la orquestación. Se desaprovisiona una vez finalizada la migración. Debe implementarse en la misma suscripción, grupo de recursos y región que la cuenta de almacenamiento de StorSimple.

### <a name="azure-file-sync"></a>Azure File Sync

Con Azure File Sync, puede agregar almacenamiento en caché local de los archivos a los que se accede con más frecuencia. De forma similar a las funcionalidades de almacenamiento en caché de StorSimple, la característica de nube por niveles de Azure File Sync ofrece latencia de acceso local en combinación con el control mejorado de la funcionalidad de caché disponible en la instancia de Windows Server y en la sincronización de varios sitios. Si el objetivo es tener una caché local, prepare una máquina virtual de Windows Server (también se admiten servidores físicos y clústeres de conmutación por error) en su red local con capacidad de almacenamiento conectado directamente suficiente.

> [!IMPORTANT]
> No configure Azure File Sync todavía. Es mejor hacerlo después de completar la migración del recurso compartido. La implementación de Azure File Sync no debe iniciarse antes de la fase 4 de una migración.

### <a name="phase-2-summary"></a>Resumen de la fase 2

Al final de la fase 2, habrá implementado sus cuentas de almacenamiento y todos los recursos compartidos de archivos de Azure en ellas. También tendrá un recurso StorSimple Data Manager. Usará este último en la fase 3, cuando configure los trabajos de migración.

## <a name="phase-3-create-and-run-a-migration-job"></a>Fase 3: Creación y ejecución de un trabajo de migración

En esta sección se describe cómo configurar un trabajo de migración y asignar cuidadosamente los directorios de un volumen de StorSimple que debe copiarse en el recurso compartido de archivos de Azure de destino que seleccione. Para empezar, vaya a StorSimple Data Manager, busque **Definiciones de trabajo** en el menú y seleccione **+ Definición de trabajo**. El tipo de almacenamiento de destino correcto es el valor predeterminado: **Recurso compartido de archivos de Azure**.

![Tipos de trabajo de migración de StorSimple serie 8000](media/storage-files-migration-storsimple-8000/storage-files-migration-storsimple-8000-new-job-type.png "Captura de pantalla de las definiciones de trabajo en Azure Portal con un nuevo cuadro de diálogo Definición de trabajos abierto que solicita el tipo de trabajo: Cópielo en un recurso compartido de archivos o en un contenedor de blobs.").

:::row:::
    :::column:::
        ![Guía de migración de StorSimple serie 8000.](media/storage-files-migration-storsimple-8000/storage-files-migration-storsimple-8000-new-job.png "Captura de pantalla del nuevo formulario de creación de trabajos para un trabajo de migración.")
    :::column-end:::
    :::column:::
        **Job definition name (Nombre de la definición de trabajo)**</br>Este nombre debe ser indicativo del conjunto de archivos que se van a mover. Le recomendamos que le asigne un nombre similar al recurso compartido de archivos de Azure. </br></br>**Ubicación en la que se ejecuta el trabajo**</br>Al seleccionar una región, debe seleccionar la misma región que la cuenta de almacenamiento de StorSimple o, si no está disponible, una región cercana a ella. </br></br><h3>Source</h3>**Suscripción de origen**</br>Seleccione la suscripción en la que almacena el recurso de StorSimple Device Manager. </br></br>**Recurso de StorSimple**</br>Seleccione la instancia de StorSimple Device Manager en la que el dispositivo esté registrado. </br></br>**Clave de cifrado de datos de servicio**</br>Consulte esta [sección anterior de este artículo](#storsimple-service-data-encryption-key), en caso de que no pueda localizar la clave en los registros. </br></br>**Dispositivo**</br>Seleccione el dispositivo de StorSimple que contiene el volumen que desea migrar. </br></br>**Volumen**</br>Seleccione el volumen de origen. Más adelante decidirá si desea migrar el volumen o los subdirectorios completos al recurso compartido de archivos de Azure de destino.</br></br> **Copias de seguridad de volumen**</br>Puede elegir *Seleccionar copias de seguridad de volumen* para elegir las copias de seguridad específicas que se moverán como parte de este trabajo. Una [sección dedicada de este artículo más adelante](#selecting-volume-backups-to-migrate) describe el proceso de manera detallada.</br></br><h3>Destino</h3>Seleccione la suscripción, la cuenta de almacenamiento y el recurso compartido de archivos de Azure como destino de este trabajo de migración.</br></br><h3>Asignación de directorios</h3>[En una sección dedicada de este artículo](#directory-mapping), se describen todos los detalles pertinentes.
    :::column-end:::
:::row-end:::

### <a name="selecting-volume-backups-to-migrate"></a>Selección de copias de seguridad de volumen para migrar

Hay algunos aspectos importantes relacionados con la elección de las copias de seguridad que se deben migrar:

- Los trabajos de migración solo pueden migrar copias de seguridad, no datos de un volumen activo. Por lo tanto, la copia de seguridad más reciente es más cercana a los datos activos y siempre debe incluirse en la lista de copias de seguridad que se han trasladado en una migración.
- Asegúrese de que la última copia de seguridad sea reciente para mantener el tamaño del delta en el recurso compartido activo lo más pequeño posible. Podría ser conveniente desencadenar y completar manualmente otra copia de seguridad del volumen antes de crear un trabajo de migración. Un pequeño delta en el recurso compartido activo mejorará la experiencia de migración. Si este delta puede ser cero, significa que no se realizaron más cambios en el volumen de StorSimple después de que se tomó la copia de seguridad más reciente de la lista. Por lo tanto, la fase 5: migración total de los usuarios será significativamente más sencilla y rápida.
- Las copias de seguridad se deben reproducir en el recurso compartido de archivos de Azure **de la más antigua a la más reciente**. Una copia de seguridad antigua no se puede "ordenar" en la lista de copias de seguridad del recurso compartido de archivos de Azure una vez que se ha ejecutado un trabajo de migración. Por lo tanto, debe asegurarse de que la lista de copias de seguridad se ha completado *antes* de crear un trabajo. 
- Esta lista de copias de seguridad de un trabajo no se puede modificar cuando se ha creado el trabajo, si este nunca se ejecutó. 

:::row:::
    :::column:::        
        :::image type="content" source="media/storage-files-migration-storsimple-8000/storage-files-migration-storsimple-8000-job-select-backups.png" alt-text="Captura de pantalla del nuevo formulario de creación de trabajos en el que se detalla la parte donde se seleccionan las copias de seguridad de StorSimple para la migración." lightbox="media/storage-files-migration-storsimple-8000/storage-files-migration-storsimple-8000-job-select-backups-expanded.png":::
    :::column-end:::
    :::column:::
        Para seleccionar copias de seguridad del volumen de StorSimple para el trabajo de migración, seleccione la opción *Select volume backups* (seleccionar copias de seguridad del volumen) en el formulario de creación de trabajos.
    :::column-end:::
:::row-end:::
:::row:::
    :::column:::
        :::image type="content" source="media/storage-files-migration-storsimple-8000/storage-files-migration-storsimple-8000-job-select-backups-annotated.png" alt-text="Imagen que muestra que la parte superior de la hoja para seleccionar copias de seguridad enumera todas las copias de seguridad disponibles. Una copia de seguridad seleccionada estará deshabilitada en esta lista y se agregará a una segunda lista en la parte inferior de la hoja. Desde allí se puede eliminar de nuevo." lightbox="media/storage-files-migration-storsimple-8000/storage-files-migration-storsimple-8000-job-select-backups-annotated.png":::
    :::column-end:::
    :::column:::
        Cuando se abre la hoja de selección de copias de seguridad, está dividida en dos listas. En la primera lista, se muestran todas las copias de seguridad disponibles. Puede expandir y reducir el conjunto de resultados al usar un intervalo de tiempo específico como filtro. (Consulte la sección siguiente) </br></br>Una copia de seguridad seleccionada se mostrará como deshabilitada y se agregará a una segunda lista en la parte inferior de la hoja. La segunda lista muestra todas las copias de seguridad seleccionadas para la migración. Una copia de seguridad seleccionada accidentalmente también se puede volver a quitar.
        > [!CAUTION]
        > Debe seleccionar **todas** las copias de seguridad que quiera migrar. No se pueden agregar copias de seguridad más antiguas después. No se puede modificar el trabajo para cambiar la selección una vez que se ha creado el trabajo.
    :::column-end:::
:::row-end:::
:::row:::
    :::column:::
        :::image type="content" source="media/storage-files-migration-storsimple-8000/storage-files-migration-storsimple-8000-job-select-backups-time.png" alt-text="Captura de pantalla que muestra la selección de un intervalo de tiempo de la hoja Selección de copia de seguridad." lightbox="media/storage-files-migration-storsimple-8000/storage-files-migration-storsimple-8000-job-select-backups-time-expanded.png":::
    :::column-end:::
    :::column:::
        De manera predeterminada, la lista se filtra para mostrar las copias de seguridad del volumen de StorSimple en los últimos siete días para facilitar la selección de la copia de seguridad más reciente. Para las copias de seguridad que son más antiguas, use el filtro de intervalo de tiempo en la parte superior de la hoja. Puede seleccionar uno de los filtros existentes o establecer un intervalo de tiempo personalizado para filtrar solo las copias de seguridad realizadas durante ese período.
    :::column-end:::
:::row-end:::

> [!CAUTION]
> No se admite la selección de más de 50 copias de seguridad de volumen de StorSimple. Se puede producir un error en los trabajos con un gran número de copias de seguridad.

### <a name="directory-mapping"></a>Asignación de directorios

La asignación de directorios es opcional para el trabajo de migración. Si deja la sección vacía, *todos* los archivos y las carpetas de la raíz del volumen de StorSimple se moverán a la raíz del recurso compartido de archivos de Azure de destino. En la mayoría de los casos, almacenar el contenido de un volumen completo en un recurso compartido de archivos de Azure no es el mejor enfoque. A menudo, es mejor dividir el contenido de un volumen en varios recursos compartidos de archivos de Azure. Si aún no ha hecho un plan, consulte antes [Asignación del volumen de StorSimple a recursos compartidos de archivos de Azure](#map-your-existing-storsimple-volumes-to-azure-file-shares).

Como parte del plan de migración, es posible que haya decidido que las carpetas de un volumen de StorSimple deben dividirse en varios recursos compartidos de archivos de Azure. En ese caso, puede realizar esa división de la siguiente manera:

1. Definiendo varios trabajos para migrar las carpetas de un volumen, cada uno tendrá el mismo origen de volumen de StorSimple, pero un recurso compartido de archivos de Azure diferente como destino.
1. Especificando con precisión qué carpetas del volumen de StorSimple se deben migrar al recurso compartido de archivos especificado, mediante la sección de **asignación de directorios** del formulario de creación del trabajo y siguiendo la [semántica de asignación](#semantic-elements) específica.

> [!IMPORTANT]
> Las rutas de acceso y las expresiones de asignación de este formulario no se pueden validar cuando se envía el formulario. Si se especifican las asignaciones incorrectamente, puede producirse un error en el trabajo o un resultado no deseado. En ese caso, suele ser mejor eliminar el recurso compartido de archivos de Azure, volver a crearlo y, a continuación, corregir las instrucciones de asignación en un nuevo trabajo de migración del recurso compartido. La ejecución de un nuevo trabajo con instrucciones de asignación fija puede corregir carpetas omitidas y colocarlas en el recurso compartido existente. Sin embargo, solo las carpetas que se omitieron debido a errores de escritura en la ruta de acceso se pueden solucionar de esta manera.

#### <a name="semantic-elements"></a>Elementos semánticos

Una asignación se expresa de izquierda a derecha: [\ruta de acceso de origen] \> [\ruta de acceso de destino].

|Carácter semántico          | Significado  |
|:---------------------------|:---------|
| **\\**                     | Indicador de nivel raíz.       |
| **\>**                     | Operador de asignación de [origen] y [destino].     |
|**\|** o RETURN (nueva línea) | Separador de dos instrucciones de asignación de carpetas. </br>Como alternativa, puede omitir este carácter y seleccionar **Entrar** para obtener la siguiente expresión de asignación en su propia línea.        |

### <a name="examples"></a>Ejemplos
Mueve el contenido de la carpeta *Datos de usuario* a la raíz del recurso compartido de archivos de destino:
``` console
\User data > \
```
Mueve todo el contenido del volumen a una nueva ruta de acceso en el recurso compartido de archivos de destino:
``` console
\ > \Apps\HR tracker
```
Mueve el contenido de la carpeta de origen a una nueva ruta de acceso en el recurso compartido de archivos de destino:
``` console
\HR resumes-Backup > \Backups\HR\resumes
```
Ordena varias ubicaciones de origen en una nueva estructura de directorios:
``` console
\HR\Candidate Tracker\v1.0 > \Apps\Candidate tracker
\HR\Candidates\Resumes > \HR\Candidates\New
\Archive\HR\Old Resumes > \HR\Candidates\Archived
```

### <a name="semantic-rules"></a>Reglas semánticas

* Especifique siempre las rutas de acceso de la carpeta en relación con el nivel raíz.
* Comience cada ruta de acceso de carpeta con un indicador de nivel de raíz "\\".
* No incluya letras de unidad.
* Al especificar varias rutas de acceso, las rutas de acceso de origen o de destino no se pueden superponer:</br>
   Ejemplo de superposición de ruta de origen no válida:</br>
    *\\carpeta\1 > \\carpeta*</br>
    *\\carpeta\\1\\2 > \\carpeta2*</br>
   Ejemplo de superposición de ruta de destino no válida:</br>
   *\\carpeta > \\*</br>
   *\\carpeta2 > \\*</br>
* Se omitirán las carpetas de origen que no existen.
* Se crearán las estructuras de carpetas que no existen en el destino.
* Al igual que Windows, los nombres de carpeta no distinguen mayúsculas de minúsculas, pero sí se conservan.

> [!NOTE]
> El trabajo de migración no copiará el contenido de la carpeta *\System Volume Information* y *$Recycle.Bin* en el volumen de StorSimple.

### <a name="run-a-migration-job"></a>Ejecución de un trabajo de migración

Los trabajos de migración aparecen debajo de *Definiciones de trabajos* en el recurso de Data Manager que ha implementado en un grupo de recursos.
En la lista de definiciones de trabajos, seleccione el trabajo que quiere ejecutar.

En la hoja del trabajo que aparece, puede ver que el trabajo se ejecuta en la lista inferior. Inicialmente, esta lista estará vacía. En la parte superior de la hoja, hay un comando denominado *Ejecutar trabajo*. Este comando no ejecutará el trabajo de inmediato, sino que abrirá la hoja **Ejecución de trabajo**:

:::row:::
    :::column:::
        :::image type="content" source="media/storage-files-migration-storsimple-8000/storage-files-migration-storsimple-8000-run-job.png" alt-text="Una imagen que muestra la hoja de ejecución de trabajo con un control de lista desplegable abierto, que muestra las copias de seguridad seleccionadas para la migración. La copia de seguridad más antigua está resaltada y debe seleccionarse primero." lightbox="media/storage-files-migration-storsimple-8000/storage-files-migration-storsimple-8000-run-job-expanded.png":::
    :::column-end:::
    :::column:::
        En esta versión, cada trabajo se debe ejecutar varias veces. </br></br>**Debe comenzar con la copia de seguridad más antigua de la lista de copias de seguridad que quiere migrar.** (Resaltada en la imagen)</br></br>Vuelva a ejecutar el trabajo, tantas veces como haya copias de seguridad seleccionadas, cada vez con una copia de seguridad más reciente.
        </br></br>
        > [!CAUTION]
        > Es importante que ejecute primero el trabajo de migración con la copia de seguridad más antigua seleccionada y, a continuación, lo ejecute de nuevo cada vez con una copia de seguridad más reciente. Siempre debe mantener el orden de las copias de seguridad manualmente, de la más antigua a la más reciente.
    :::column-end:::
:::row-end:::

### <a name="phase-3-summary"></a>Resumen de la fase 3

Al final de la fase 3, habrá ejecutado al menos uno de los trabajos de migración desde los volúmenes de StorSimple en recursos compartidos de archivos de Azure. Habrá ejecutado el mismo trabajo de migración varias veces, de las copias de seguridad más antiguas hasta las más recientes que se deben migrar. Ahora puede centrarse en la configuración de Azure File Sync del recurso compartido (cuando se hayan completado los trabajos de migración de un recurso compartido) o el direccionamiento del acceso compartido para las aplicaciones y los trabajadores de la información al recurso compartido de archivos de Azure.

## <a name="phase-4-access-your-azure-file-shares"></a>Fase 4: Acceso a los recursos compartidos de archivos de Azure

Hay dos estrategias principales para acceder a los recursos compartidos de archivos de Azure:

* **Azure File Sync**: [implementar Azure File Sync](#deploy-azure-file-sync) en una instancia de Windows Server local. Azure File Sync tiene todas las ventajas de una memoria caché local, al igual que StorSimple.
* **Acceso directo a recursos compartidos**: [Implementación de acceso directo a recursos compartidos](#deploy-direct-share-access). Use esta estrategia si su escenario de acceso para un recurso compartido de archivos de Azure determinado no se beneficiará del almacenamiento en caché local o ya no es posible hospedar una instancia de Windows Server. Aquí, los usuarios y las aplicaciones seguirán teniendo acceso a los recursos compartidos de SMB a través del protocolo SMB. Estos recursos compartidos ya no están en un servidor local, sino directamente en la nube.

Ya debe haber decidido qué opción es la más adecuada en la [fase 1](#phase-1-prepare-for-migration) de esta guía.

El resto de esta sección se centra en las instrucciones de implementación.

### <a name="deploy-azure-file-sync"></a>Implementación de Azure File Sync

Es el momento de implementar una parte de Azure File Sync.

1. Cree el recurso de nube de Azure File Sync.
1. Implemente el agente de Azure File Sync en el servidor local.
1. Registre el servidor con el recurso de nube.

No cree todavía ningún grupo de sincronización. La configuración de la sincronización con un recurso compartido de archivos de Azure solo debe realizarse una vez que se hayan completado los trabajos de migración a un recurso compartido de archivos de Azure. Si empezó a usar Azure File Sync antes de que se complete la migración, hará que la migración sea innecesariamente difícil, ya que no es fácil saber cuándo es el momento de iniciar un recorte.

#### <a name="deploy-the-azure-file-sync-cloud-resource"></a>Implementación del recurso de nube de Azure File Sync

[!INCLUDE [storage-files-migration-deploy-afs-sss](../../../includes/storage-files-migration-deploy-azure-file-sync-storage-sync-service.md)]

> [!TIP]
> Si desea cambiar la región de Azure en la que residen los datos cuando se realiza la migración, implemente el servicio de sincronización de almacenamiento en la misma región que las cuentas de almacenamiento de destino para esta migración.

#### <a name="deploy-an-on-premises-windows-server-instance"></a>Implementación de una instancia de Windows Server local

* Cree un servidor de Windows Server 2019 (como mínimo de la edición 2012 R2), como una máquina virtual o un servidor físico. También se admite un clúster de conmutación por error de Windows Server. No reutilice el servidor que tenga delante de StorSimple 8100 u 8600.
* Aprovisione o agregue almacenamiento de conexión directa. No se admite el almacenamiento conectado a la red.

Es un procedimiento recomendado otorgar a la nueva instancia de Windows Server una cantidad igual o superior de almacenamiento de la que dispone la aplicación StorSimple 8100 o 8600 localmente para el almacenamiento en caché. Usará la instancia de Windows Server del mismo modo que usó el dispositivo StorSimple. Si tiene la misma cantidad de almacenamiento que el dispositivo, la experiencia de almacenamiento en caché debe ser similar, si no es la misma. Puede agregar o eliminar almacenamiento de la instancia de Windows Server según lo desee. Esto le permitirá escalar el tamaño del volumen local y la cantidad de almacenamiento local disponible para el almacenamiento en caché.

#### <a name="prepare-the-windows-server-instance-for-file-sync"></a>Preparación de la instancia de Windows Server para la sincronización de archivos

[!INCLUDE [storage-files-migration-deploy-afs-agent](../../../includes/storage-files-migration-deploy-azure-file-sync-agent.md)]

#### <a name="configure-azure-file-sync-on-the-windows-server-instance"></a>Configuración de Azure File Sync en la instancia de Windows Server

La instancia registrada de Windows Server local debe estar preparada y conectada a Internet para este proceso.

> [!IMPORTANT]
> Antes de continuar, debe completar la migración de archivos y carpetas de StorSimple en el recurso compartido de archivos de Azure. Asegúrese de que no haya más cambios en el recurso compartido de archivos.

[!INCLUDE [storage-files-migration-configure-sync](../../../includes/storage-files-migration-configure-sync.md)]

> [!IMPORTANT]
> Asegúrese de activar la nube por niveles. La nube por niveles es la característica de Azure File Sync que permite al servidor local tener menos capacidad de almacenamiento de la que está almacenada en la nube y, a pesar de ello, disponer de todo el espacio de nombres. Los datos localmente interesantes también se almacenan en caché para conseguir un rendimiento rápido y de acceso local. Otro motivo para activar la nube por niveles en este paso es que no queremos sincronizar el contenido del archivo en esta fase. En este momento solo se debe mover el espacio de nombres.

### <a name="deploy-direct-share-access"></a>Implementación de acceso directo a recursos compartidos

:::row:::
    :::column:::
        <iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/jd49W33DxkQ" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
    :::column-end:::
    :::column:::
        Este vídeo es una guía y demostración sobre cómo exponer de forma segura recursos compartidos de archivos de Azure directamente para las aplicaciones y trabajadores de la información en cinco sencillos pasos.</br>
        En el vídeo se hace referencia a documentación dedicada para algunos temas:

* [Introducción a las identidades](storage-files-active-directory-overview.md)
* [Unión a un dominio de una cuenta de almacenamiento](storage-files-identity-auth-active-directory-enable.md)
* [Información general sobre redes para recursos compartidos de archivos de Azure](storage-files-networking-overview.md)
* [Configuración de puntos de conexión públicos y privados](storage-files-networking-endpoints.md)
* [Configuración de una VPN S2S](storage-files-configure-s2s-vpn.md)
* [Configuración de una VPN de Windows P2S](storage-files-configure-p2s-vpn-windows.md)
* [Configuración de una VPN P2S de Linux](storage-files-configure-p2s-vpn-linux.md)
* [Configuración del reenvío de DNS](storage-files-networking-dns.md)
* [Configuración de DFS-N](/windows-server/storage/dfs-namespaces/dfs-overview)
   :::column-end:::
:::row-end:::

### <a name="phase-4-summary"></a>Resumen de la fase 4

En esta fase ha creado y ejecutado varios trabajos de migración en StorSimple Data Manager. Esos trabajos han migrado los archivos y las carpetas a recursos compartidos de archivos de Azure. También ha implementado Azure File Sync o preparado las cuentas de red y almacenamiento para el acceso directo a recursos compartidos.

## <a name="phase-5-user-cut-over"></a>Fase 5: Migración total de los usuarios

Esta fase se centra simplemente en resumir la migración:

* Planee el tiempo de inactividad.
* Póngase al día con cualquier cambio que los usuarios y las aplicaciones hayan generado en StorSimple mientras se han estado ejecutando los trabajos de migración en la fase 3.
* Conmute por error a los usuarios a la nueva instancia de Windows Server con Azure File Sync o los recursos compartidos de archivos de Azure a través del acceso directo a recursos compartidos.

### <a name="plan-your-downtime"></a>Planeación del tiempo de inactividad

Este enfoque de migración requiere cierto tiempo de inactividad para los usuarios y las aplicaciones. El objetivo es mantener un tiempo de inactividad mínimo. Las siguientes consideraciones pueden ayudar:

* Mantenga los volúmenes de StorSimple disponibles mientras ejecuta los trabajos de migración.
* Cuando haya terminado de ejecutar los trabajos de migración de datos de un recurso compartido, podrá quitar el acceso de usuario (al menos el acceso de escritura) de los volúmenes o recursos compartidos de StorSimple. Un comando RoboCopy final pondrá en marcha el recurso compartido de archivos de Azure. Después, podrá migrar a los usuarios. El lugar en el que se ejecute RoboCopy dependerá de si ha elegido usar Azure File Sync o el acceso directo a recursos compartidos. En la próxima sección sobre RoboCopy se explica este asunto.
* Una vez completada la puesta al día de RoboCopy, está listo para exponer la nueva ubicación a los usuarios mediante el recurso compartido de archivos de Azure directamente o un recurso compartido de SMB en una instancia de Windows Server con Azure File Sync. A menudo, una implementación de DFS-N lo ayudará a realizar una migración rápida y eficaz. Mantendrá coherentes las direcciones de los recursos compartidos actuales y llevará a cabo la redirección a una nueva ubicación que contenga los archivos y carpetas migrados.

### <a name="determine-when-your-namespace-has-fully-synced-to-your-server"></a>Determinación de si el espacio de nombres se ha sincronizado por completo en el servidor

Cuando se usa Azure File Sync para un recurso compartido de archivos de Azure, es importante que se determine que todo el espacio de nombres ha terminado de descargarse en el servidor *antes* de iniciar cualquier comando RoboCopy local. El tiempo necesario para descargar el espacio de nombres depende del número de elementos del recurso compartido de archivos de Azure. Existen dos métodos para determinar si el espacio de nombres se ha trasladado por completo al servidor.

#### <a name="azure-portal"></a>Azure portal

Puede usar Azure Portal para ver cuándo ha llegado por completo el espacio de nombres.

* Inicie sesión en Azure Portal y vaya al grupo de sincronización. Compruebe el estado de sincronización del grupo de sincronización y el punto de conexión del servidor.
* Se descarga la dirección de interés. Si el punto de conexión del servidor se ha aprovisionado recientemente, se mostrará la **sincronización inicial**, que indica que el espacio de nombres sigue estando disponible.
Una vez que se cambia todo, menos la **sincronización inicial**, el espacio de nombres se rellenará por completo en el servidor. Está listo para continuar con un comando RoboCopy local.

#### <a name="windows-server-event-viewer"></a>Visor de eventos de Windows Server

También puede usar el Visor de eventos en la instancia de Windows Server para saber cuándo se ha trasladado por completo el espacio de nombres.

1. Abra el **Visor de eventos** y vaya a **Aplicaciones y servicios**.
1. Vaya a **Microsoft\FileSync\Agent\Telemetry** y ábralo.
1. Busque el **evento 9102** más reciente, el cual se corresponde con una sesión de sincronización completada.
1. Seleccione **Detalles** y confirme que está examinando un evento en el que el valor **SyncDirection** es **Descargar**.
1. En el momento en que el espacio de nombres ha finalizado su descarga en el servidor, habrá un único evento con **Escenario**, valor **FullGhostedSync** y **HResult** = **0**.
1. Si no está ese evento, también puede buscar otros **eventos 9102** con **SyncDirection** = **Descargar** y **Escenario** =  **"RegularSync"** . La búsqueda de uno de estos eventos también indica que el espacio de nombres se ha terminado de descargar y la sincronización ha progresado a sesiones regulares de sincronización, haya o no algo que sincronizar, en este momento.

### <a name="a-final-robocopy"></a>Un comando RoboCopy final.

En este momento, hay diferencias entre el servidor de la instancia de Windows Server local y la aplicación StorSimple 8100 u 8600.

1. Debe ponerse al día con los cambios que los usuarios o las aplicaciones generan en StorSimple mientras la migración estaba en curso.
1. En los casos en los que se usa Azure File Sync: La aplicación StorSimple tiene una memoria caché llena en comparación con la instancia de Windows Server que, en este momento, solo tiene un espacio de nombres sin ningún contenido de archivos almacenado localmente. Por lo tanto, el comando RoboCopy final puede ayudar a iniciar la memoria caché de Azure File Sync local al extraer tanto contenido del archivo almacenado en memoria caché local como esté disponible y puede ajustarse al servidor de Azure File Sync.
1. Es posible que el trabajo de migración haya omitido algunos archivos debido a caracteres no válidos. Si es así, cópielos en la instancia de Windows Server habilitada para Azure File Sync. Más adelante, puede ajustarlos para que se sincronicen. Si no usa Azure File Sync para un recurso compartido determinado, es mejor cambiar el nombre de los archivos por caracteres no válidos en el volumen de StorSimple. Después, ejecute RoboCopy directamente en el recurso compartido de archivos de Azure.

> [!WARNING]
> En la actualidad, Robocopy en Windows Server 2019 experimenta un problema que provocará que los archivos organizados por niveles con Azure File Sync en el servidor de destino se copien nuevamente desde el origen y se vuelvan a cargar en Azure cuando se usa la función /MIR de Robocopy. Es indispensable que use Robocopy en una versión de Windows Server que no sea 2019. La mejor opción es Windows Server 2016. Esta nota se actualizará en caso de que el problema se resuelva a través de Windows Update.

> [!WARNING]
> *No debe* iniciar RoboCopy antes de que el servidor tenga el espacio de nombres de un recurso compartido de archivos de Azure descargado completamente. Para obtener más información, consulte [Determinación de si el espacio de nombres se ha sincronizado por completo en el servidor](#determine-when-your-namespace-has-fully-synced-to-your-server).

 Solo desea copiar los archivos que se cambiaron después de la última vez que se ejecutó el trabajo de migración y los archivos que no se trasladaron antes a través de estos trabajos. Puede solucionar el problema por el que no se trasladaron al servidor, una vez que haya finalizado la migración. Para más información, vea [Solución de problemas de Azure File Sync](storage-sync-files-troubleshoot.md#how-do-i-see-if-there-are-specific-files-or-folders-that-are-not-syncing).

RoboCopy tiene varios parámetros. El siguiente ejemplo muestra un comando terminado y una lista de razones para elegir estos parámetros.

```console
Robocopy /MT:16 /UNILOG:<file name> /TEE /NP /B /MIR /IT /COPYALL /DCOPY:DAT <SourcePath> <Dest.Path>
```

Fondo:

:::row:::
   :::column span="1":::
      /MT
   :::column-end:::
   :::column span="1":::
      Permite que RoboCopy se ejecute en modo multiproceso. El máximo es 128 y el valor predeterminado es 8.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
      /UNILOG:<file name>
   :::column-end:::
   :::column span="1":::
      Envía el estado al archivo de registro como UNICODE (sobrescribe el registro existente).
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
      /TEE
   :::column-end:::
   :::column span="1":::
      Envía la salida a la ventana de la consola. Se usa junto con el envío a un archivo de registro.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
      /NP
   :::column-end:::
   :::column span="1":::
      Omite el registro del progreso para mantener el registro legible.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
      /B
   :::column-end:::
   :::column span="1":::
      Ejecuta RoboCopy en el mismo modo que usaría una aplicación de copia de seguridad. Permite que Robocopy mueva los archivos para los que el usuario actual no tiene permisos.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
      /MIR
   :::column-end:::
   :::column span="1":::
      Permite que RoboCopy solo tenga en cuenta los diferenciales entre el origen (aplicación StorSimple) y el destino (directorio de Windows Server).
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
      /IT
   :::column-end:::
   :::column span="1":::
      Garantiza que se conserve la fidelidad en ciertos escenarios de reflejo.</br>Ejemplo: Entre dos ejecuciones de Robocopy, un archivo experimenta un cambio de ACL y una actualización de atributo, por ejemplo, también está marcado como *oculto*. Sin /IT, Robocopy podría omitir el cambio de ACL y, por tanto, no se transferiría a la ubicación de destino.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
      /COPY:copyflag[s]
   :::column-end:::
   :::column span="1":::
      Fidelidad de la copia de archivos (el valor predeterminado es /COPY:DAT), marcas de copia: D = datos, A = atributos, T = marcas de tiempo, S = seguridad = ACL de NTFS, O = información del propietario, U = información de auditoría.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
      /COPYALL
   :::column-end:::
   :::column span="1":::
      Copia de toda la información del archivo (equivalente a /COPY:DATSOU).
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
      /DCOPY:copyflag[s]
   :::column-end:::
   :::column span="1":::
      Fidelidad de la copia de directorios (el valor predeterminado es /DCOPY:DA), marcas de copia: D = datos, A = atributos, T = marcas de tiempo.
   :::column-end:::
:::row-end:::

Cuando configure las ubicaciones de origen y de destino del comando RoboCopy, asegúrese de revisar la estructura de origen y de destino para asegurarse de que coinciden. Si usó la característica de asignación de directorios del trabajo de migración, la estructura del directorio raíz puede ser diferente de la estructura del volumen de StorSimple. En ese caso, puede que necesite varios trabajos de RoboCopy, uno para cada subdirectorio. Si no está seguro de si el comando se ejecutará según lo previsto, puede usar el parámetro */L*, que simulará el comando sin realizar realmente ningún cambio.

Este comando RoboCopy usa /MIR, por lo que no moverá archivos que sean iguales (archivos en capas, por ejemplo). Pero si obtiene incorrectamente la ruta de acceso de origen y destino, /MIR también purga las estructuras de directorio en su instancia de Windows Server o en el recurso compartido de archivos de Azure que no están presentes en la ruta de acceso de origen de StorSimple. Deben coincidir exactamente para que el trabajo de RoboCopy alcance su objetivo de actualizar el contenido migrado con los últimos cambios realizados durante la migración.

Consulte los archivos de registro de RoboCopy para ver si se han omitido archivos. Si existen problemas, corríjalos y vuelva a ejecutar el comando RoboCopy. No desaprovisione los recursos de StorSimple antes de corregir los problemas pendientes de los archivos o carpetas que le interesan.

Si no usa Azure File Sync para almacenar en caché el recurso compartido de archivos de Azure en cuestión, sino que ha optado por el acceso directo a recursos compartidos:

1. [Monte el recurso compartido de archivos de Azure](storage-how-to-use-files-windows.md#mount-the-azure-file-share) como una unidad de red en una máquina local de Windows.
1. Ejecute el comando de RoboCopy entre StorSimple y el recurso compartido de archivos de Azure montado. Si los archivos no se copian, corrija sus nombres en StorSimple para quitar los caracteres no válidos. A continuación, vuelva a intentar la ejecución de RoboCopy. El comando RoboCopy de la lista anterior se puede ejecutar varias veces sin provocar una recuperación innecesaria en StorSimple.

### <a name="user-cut-over"></a>Migración total de los usuarios

Si usa Azure File Sync, es probable que tenga que crear los recursos compartidos de SMB en esa instancia de Windows Server habilitada para Azure File Sync que coincida con los recursos compartidos que tenía en los volúmenes de StorSimple. Puede adelantar este paso y hacerlo antes para no perder tiempo aquí. Pero debe asegurarse de que antes de este punto nadie tenga acceso para realizar cambios en la instancia de Windows Server.

Si tiene una implementación de DFS-N, puede apuntar los espacios de nombres de DFN a las nuevas ubicaciones de carpetas del servidor. Si no tiene una implementación de DFS-N y ha adelantado el dispositivo 8100/8600 localmente con una instancia de Windows Server, puede desconectar el servidor del dominio. Después, únase al dominio nuevo en la instancia de Windows Server habilitada para Azure File Sync. Durante ese proceso, asigne al servidor el mismo nombre de servidor y los nombres de los recursos compartidos que en el servidor anterior, de modo que el acceso directo sea transparente para los usuarios, la directiva de grupo y los scripts.

Más información sobre [DFS-N](/windows-server/storage/dfs-namespaces/dfs-overview).

## <a name="deprovision"></a>Desaprovisionamiento

Cuando se desaprovisiona un recurso, se pierde el acceso a la configuración de ese recurso, así como a sus datos. El aprovisionamiento no se puede deshacer. No continúe hasta que haya confirmado que:

* La migración se ha completado.
* No hay ninguna dependencia en los archivos, carpetas o copias de seguridad del volumen de StorSimple que está a punto de desaprovisionar.

Antes de empezar, es un procedimiento recomendado observar la nueva implementación de Azure File Sync en producción durante un tiempo. Esto le ofrece la oportunidad de corregir los problemas que puedan surgir. Una vez que haya observado la implementación de Azure File Sync durante al menos unos días, puede empezar a desaprovisionar los recursos en este orden:

1. Desaprovisione el recurso de StorSimple Data Manager a través de Azure Portal. Todos los trabajos de DTS se eliminarán con él. No podrá recuperar fácilmente los registros de copia. Si es importante para los registros, puede recuperarlos antes de desaprovisionarlos.
1. Asegúrese de que se han migrado las aplicaciones físicas de StorSimple y, a continuación, anule su registro. Si no está totalmente seguro de que se han migrado, no continúe. Si quita el aprovisionamiento de estos recursos mientras siguen siendo necesarios, no podrá recuperar los datos ni su configuración.<br>Opcionalmente, puede desaprovisionar primero el recurso de volumen de StorSimple, lo cual limpiará los datos del dispositivo. Esto puede tardar varios días y **no** pondrá a cero los datos del dispositivo de manera forense. Si esto es importante para usted, administre la puesta a cero del disco por separado del desaprovisionamiento de recursos y de acuerdo con sus directivas.
1. Si no quedan más dispositivos registrados en StorSimple Device Manager, puede continuar para quitar ese recurso de Device Manager.
1. Ahora es el momento de eliminar la cuenta de almacenamiento de StorSimple en Azure. Una vez más, deténgase en este punto y confirme que la migración se ha completado y no hay nada que dependa de estos datos antes de continuar.
1. Desconecte el dispositivo físico de StorSimple del centro de datos.
1. Si es el propietario del dispositivo de StorSimple, puede reciclar el equipo. Si el dispositivo está concedido, informe al arrendador y devuélvalo según corresponda.

La migración se ha completado.

> [!NOTE]
> ¿Todavía tiene preguntas o ha detectado algún problema?</br>
> Estaremos encantados de ayudarle en AzureFilesMigration@microsoft.com.

## <a name="next-steps"></a>Pasos siguientes

* Familiarícese con [Azure File Sync: aka.ms/AFS](./storage-sync-files-planning.md).
* Obtenga información sobre la flexibilidad de las [directivas de nube por niveles](storage-sync-cloud-tiering-overview.md).
* [Habilite Azure Backup](../../backup/backup-afs.md#configure-backup-from-the-file-share-pane) en los recursos compartidos de archivos de Azure para programar instantáneas y definir programaciones de retención de copia de seguridad.
* Si ve en Azure Portal que algunos archivos no se sincronizan de forma permanente, revise la [guía de solución de problemas](storage-sync-files-troubleshoot.md) para conocer los pasos necesarios para resolverlos.
