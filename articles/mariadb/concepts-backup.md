---
title: 'Copia de seguridad y restauración: Azure Database for MariaDB'
description: Obtenga información acerca de cómo realizar copias de seguridad y restaurar automáticamente su servidor de Azure Database for MariaDB.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: conceptual
ms.date: 8/13/2020
ms.openlocfilehash: 08e75f9eb5ea111cc977d02f66b945de4eae5126
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107306183"
---
# <a name="backup-and-restore-in-azure-database-for-mariadb"></a>Copia de seguridad y restauración en Azure Database for MariaDB

Azure Database for MariaDB crea automáticamente copias de seguridad del servidor y las almacena en el almacenamiento con redundancia local o con redundancia geográfica configurado por el usuario. Las copias de seguridad pueden utilizarse para restaurar el servidor a un momento dado. Las copias de seguridad y las restauraciones son una parte esencial de cualquier estrategia de continuidad del negocio, ya que protegen los datos frente a daños o eliminaciones accidentales.

## <a name="backups"></a>Copias de seguridad

Azure Database for MariaDB hace copias de seguridad de los archivos de datos y del registro de transacciones. Estas copias de seguridad permiten restaurar un servidor a un momento dado dentro del período de retención de copias de seguridad configurado. El período de retención predeterminado es siete días. [Opcionalmente, puede configurarlo](howto-restore-server-portal.md#set-backup-configuration) hasta 35 días. Todas las copias de seguridad se cifran mediante cifrado AES de 256 bits.

Estos archivos de copia de seguridad no están expuestos al usuario y no se pueden exportar. Estas copias de seguridad solo se pueden usar en operaciones de restauración de Azure Database for MySQL. Puede usar [mysqldump](howto-migrate-dump-restore.md) para copiar una base de datos.

El tipo y la frecuencia de la copia de seguridad dependen del almacenamiento de back-end de los servidores.

### <a name="backup-type-and-frequency"></a>Tipo y frecuencia de copia de seguridad

#### <a name="basic-storage-servers"></a>Servidores de almacenamiento básico

El almacenamiento básico es el almacenamiento de back-end que admite los [servidores de nivel Básico](concepts-pricing-tiers.md). Las copias de seguridad de los servidores de almacenamiento básico se basan en instantáneas. Cada día se realiza una instantánea de base de datos completa. No se hacen copias de seguridad diferenciales para los servidores de almacenamiento básico y todas las copias de seguridad de instantánea solo son copias de seguridad de bases de datos completas.

Las copias de seguridad del registro de transacciones tienen lugar cada cinco minutos.

#### <a name="general-purpose-storage-servers-with-up-to-4-tb-storage"></a>Servidores de almacenamiento de uso general con hasta 4 TB

El almacenamiento de uso general es el almacenamiento de back-end que admite el servidor [De uso general](concepts-pricing-tiers.md) y el servidor de [nivel Optimizado para memoria](concepts-pricing-tiers.md). En el caso de los servidores con almacenamiento de uso general de hasta 4 TB, las copias de seguridad completas se realizan cada semana. Las copias de seguridad diferenciales se realizan dos veces al día. Las copias de seguridad del registro de transacciones tienen lugar cada cinco minutos. Las copias de seguridad del almacenamiento de uso general de hasta 4 TB no se basan en instantáneas y consumen ancho de banda de E/S en el momento de la copia de seguridad. En el caso de las bases de datos de gran tamaño (> 1 TB) en el almacenamiento de 4 TB, se recomienda considerar lo siguiente:

- Aprovisionamiento de más IOPS para asegurar E/S de copia de seguridad, O
- También puede migrar a un almacenamiento de uso general que admite hasta 16 TB si la infraestructura de almacenamiento subyacente está disponible en las [regiones de Azure](./concepts-pricing-tiers.md#storage) de su preferencia. No hay ningún costo adicional para el almacenamiento de uso general que admite hasta 16 TB. Si necesita ayuda para migrar a un almacenamiento de 16 TB, abra una incidencia de soporte técnico en Azure Portal.

#### <a name="general-purpose-storage-servers-with-up-to-16-tb-storage"></a>Servidores de almacenamiento de uso general con hasta 16 TB

En un subconjunto de [regiones de Azure](./concepts-pricing-tiers.md#storage), todos los servidores recién aprovisionados pueden admitir un almacenamiento de uso general de hasta 16 TB. En otras palabras, el almacenamiento de hasta 16 TB es el almacenamiento de uso general predeterminado para todas las [regiones](concepts-pricing-tiers.md#storage) donde se admite. Las copias de seguridad de estos servidores con 16 TB almacenamiento se basan en instantáneas. La primera copia de seguridad de instantáneas completa, se programa inmediatamente después de la creación del servidor. Esa primera copia se conserva como la copia de seguridad base del servidor. Las copias de seguridad de instantáneas posteriores son solo copias de seguridad diferenciales.

Las copias de seguridad de instantáneas diferenciales se producen al menos una vez al día. Las copias de seguridad de instantáneas diferenciales no se realizan según una programación fija. Las copias de seguridad de instantáneas diferenciales se producen cada 24 horas, a menos que el registro de transacciones (binlog en MariaDB) supere los 50 GB desde la última copia de seguridad diferencial. En un día, se permite un máximo de seis instantáneas diferenciales.

Las copias de seguridad del registro de transacciones tienen lugar cada cinco minutos.
 

### <a name="backup-retention"></a>Retención de copias de seguridad

Las copias de seguridad se conservan según el valor del período de retención de copia de seguridad en el servidor. Puede seleccionar un período de retención de 7 a 35 días. El período de retención predeterminado es de 7 días. Puede establecer el período de retención durante la creación del servidor o en otro momento actualizando la configuración de copia de seguridad con [Azure Portal](howto-restore-server-portal.md#set-backup-configuration) o la [CLI de Azure](howto-restore-server-cli.md#set-backup-configuration). 

El período de retención de copia de seguridad rige durante cuánto tiempo se puede realizar una restauración a un momento dado, porque se basa en las copias de seguridad disponibles. El período de retención de copia de seguridad también se puede tratar como un período de recuperación desde una perspectiva de restauración. Todas las copias de seguridad que se necesitan para realizar una restauración a un momento dado dentro del período de retención de copia de seguridad, se conservan en el almacenamiento de copia de seguridad. Por ejemplo, si el período de retención de la copia de seguridad se establece en 7 días, el período de recuperación comprendería los últimos 7 días. En este escenario, se conservan todas las copias de seguridad necesarias para restaurar el servidor en los últimos 7 días. Con un período de retención de copia de seguridad de siete días:

- Los servidores con un almacenamiento de hasta 4 TB conservarán hasta 2 copias de seguridad completas de la base de datos, todas las copias de seguridad diferenciales y las copias de seguridad del registro de transacciones realizadas desde la primera copia de seguridad completa de la base de datos.
-   Los servidores con un almacenamiento de hasta 16 TB conservarán la instantánea de base de datos completa, todas las instantáneas diferenciales y las copias de seguridad del registro de transacciones de los últimos 8 días.

#### <a name="long-term-retention-of-backups"></a>Retención a largo plazo de copias de seguridad
Actualmente, el servicio todavía no admite de forma nativa la retención a largo plazo de las copias de seguridad de más de 35 días. Tiene la opción de usar mysqldump para realizar copias de seguridad y almacenarlas para la retención a largo plazo. El equipo de soporte técnico ha escrito un [artículo paso a paso](https://techcommunity.microsoft.com/t5/azure-database-for-mysql/automate-backups-of-your-azure-database-for-mysql-server-to/ba-p/1791157) para compartir cómo puede lograrlo. 

### <a name="backup-redundancy-options"></a>Opciones de redundancia de copia de seguridad

Azure Database for MariaDB permite elegir entre almacenamiento de copia de seguridad con redundancia local o con redundancia geográfica en los niveles Uso General y Memoria optimizada. Cuando las copias de seguridad se almacenan en un almacenamiento de copia de seguridad con redundancia geográfica, no solo se almacenan en la región en la que se hospeda el servidor, también se replican en un [centro de datos emparejado](../best-practices-availability-paired-regions.md). Esto proporciona una mejor protección y capacidad de restaurar el servidor en una región diferente en caso de desastre. El nivel Básico solo ofrece almacenamiento de copia de seguridad con redundancia local.

#### <a name="moving-from-locally-redundant-to-geo-redundant-backup-storage"></a>Cambio de redundancia local a almacenamiento de copia de seguridad con redundancia geográfica
La configuración de un almacenamiento con redundancia local o con redundancia geográfica para copia de seguridad solo se puede realizar durante la creación del servidor. Una vez que se ha aprovisionado el servidor, no se puede cambiar la opción de redundancia del almacenamiento de copia de seguridad. Para trasladar el almacenamiento de copia de seguridad del almacenamiento con redundancia local a otro con redundancia geográfica, la única opción que se admite es crear un servidor y migrar los datos mediante [volcado y restauración](howto-migrate-dump-restore.md).

### <a name="backup-storage-cost"></a>Costo del almacenamiento de copia de seguridad

Azure Database for MariaDB proporciona hasta un 100 % del almacenamiento del servidor aprovisionado como almacenamiento de copia de seguridad, sin costos adicionales. El cargo de cualquier almacenamiento de copia de seguridad adicional que se use se realizará por GB/mes. Por ejemplo, si ha aprovisionado un servidor con 250 GB de almacenamiento, tiene 250 GB de almacenamiento adicional disponible para las copias de seguridad del servidor sin ningún cargo adicional. El almacenamiento consumido para copias de seguridad que supere los 250 GB se cobra según el [modelo de precios](https://azure.microsoft.com/pricing/details/mariadb/). 

Puede usar la métrica [Almacenamiento de copia de seguridad utilizado](concepts-monitoring.md) en Azure Monitor disponible en Azure Portal para supervisar el almacenamiento de copia de seguridad que usa un servidor. La métrica Almacenamiento de copia de seguridad utilizado representa la suma del almacenamiento consumido por todas las copias de seguridad de base de datos completas, copias de seguridad diferenciales y copias de seguridad de registros, que se conservan durante el período de retención de copia de seguridad establecido para el servidor. El servicio administra la frecuencia de las copias de seguridad como se ha explicado anteriormente. Una gran actividad transaccional en el servidor puede hacer que el uso del almacenamiento de copia de seguridad aumente, independientemente del tamaño total de la base de datos. En el caso del almacenamiento con redundancia geográfica, el uso del almacenamiento de copia de seguridad es dos veces el del almacenamiento con redundancia local. 

El medio principal para controlar el costo de almacenamiento de copia de seguridad es establecer el período de retención apropiado y elegir las opciones adecuadas de redundancia de copia de seguridad para satisfacer los objetivos de recuperación deseados. Puede seleccionar un período de retención de entre 7 y 35 días. Los servidores de uso general y optimizados para memoria pueden tener almacenamiento con redundancia geográfica para copias de seguridad.

## <a name="restore"></a>Restauración

En Azure Database for MariaDB, al realizar una restauración se crea un nuevo servidor a partir de las copias de seguridad del servidor original, y se restauran todas las bases de datos incluidas en el servidor.

Hay dos tipos de restauración disponibles:

- La **restauración a un momento dado** está disponible con cualquier opción de redundancia de copia de seguridad y crea un nuevo servidor en la misma región que el servidor original por medio de la combinación de copias de seguridad completas y de registros.
- La **restauración geográfica** solo está disponible si ha configurado el servidor para almacenamiento con redundancia geográfica y permite restaurar el servidor en una región diferente usando la copia de seguridad más reciente creada.

El tiempo estimado de recuperación depende de varios factores, como el tamaño de la bases de datos, el tamaño del registro de transacciones, el ancho de banda de red y el número total de bases de datos que se están recuperando en la misma región al mismo tiempo. Normalmente, el tiempo de recuperación es inferior a 12 horas.

> [!IMPORTANT]
> Los servidores eliminados se pueden restaurar solo dentro del plazo de **cinco días** a partir de la eliminación, después de lo cual se eliminan las copias de seguridad. La copia de seguridad de datos solo se puede acceder y restaurar desde la suscripción de Azure que hospeda al servidor. Para restaurar un servidor eliminado, consulte los [pasos documentados](howto-restore-dropped-server.md). Para proteger los recursos del servidor, después de la implementación, de eliminaciones accidentales o cambios inesperados, los administradores pueden aprovechar los [bloqueos de administración](../azure-resource-manager/management/lock-resources.md).

### <a name="point-in-time-restore"></a>Restauración a un momento dado

Independientemente de la opción de redundancia de copia de seguridad, puede realizar una restauración a cualquier momento del tiempo dentro de su período de retención de copias de seguridad. Se crea un nuevo servidor en la misma región de Azure que el servidor original. Se crea con la configuración del servidor original para el plan de tarifa, generación de procesos, número de núcleos virtuales, tamaño de almacenamiento, período de retención de copia de seguridad y opción de redundancia de copia de seguridad.

La restauración a un momento dado es útil en diversos escenarios. Por ejemplo, cuando un usuario elimina accidentalmente los datos, elimina una tabla importante o la base de datos, o si una aplicación sobrescribe accidentalmente los datos correctos con datos incorrectos debido a un defecto de la aplicación.

Quizás deba esperar a que se realice la siguiente copia de seguridad del registro de transacciones antes de poder restaurar a un momento dado de los últimos cinco minutos.

### <a name="geo-restore"></a>Geo-restore

Puede restaurar un servidor en otra región de Azure donde el servicio esté disponible, si ha configurado el servidor para copias de seguridad con redundancia geográfica. Los servidores que admiten hasta 4 TB de almacenamiento se pueden restaurar en la región emparejada geográficamente o en cualquier región que admita hasta 16 TB de almacenamiento. En el caso de los servidores que admiten hasta 16 TB de almacenamiento, las copias de seguridad geográficas se pueden restaurar en cualquier región que admita también servidores de 16 TB. Revise los [planes de tarifa de Azure Database for MariaDB](concepts-pricing-tiers.md) para ver la lista de regiones admitidas.

La restauración geográfica es la opción de recuperación predeterminada cuando el servidor no está disponible debido a una incidencia en la región en la que se hospeda el servidor. Si un incidente a gran escala en una región provoca la falta de disponibilidad de una aplicación de base de datos, puede restaurar un servidor a partir de las copias de seguridad con redundancia geográfica en un servidor de cualquier otra región. La restauración geográfica usa la copia de seguridad más reciente del servidor. Hay un retraso entre momento en que se realiza una copia de seguridad y el momento en que se replica en una región diferente. Este retraso puede ser de hasta una hora; por lo tanto, si se produce un desastre, puede haber una pérdida de datos de hasta una hora.

> [!IMPORTANT]
>Si se realiza una restauración geográfica de un servidor recién creado, la sincronización de la copia de seguridad inicial puede tardar más de 24 horas, en función del tamaño de los datos, ya que el tiempo de archivo de copia de seguridad de la instantánea completa inicial es mucho mayor. Las copias de seguridad de instantáneas posteriores son copias incrementales y, por lo tanto, las restauraciones son más rápidas después de las 24 horas de creación del servidor. Si va a evaluar las restauraciones geográficas para definir el RTO, le recomendamos que espere y evalúe la restauración geográfica **una vez transcurridas 24 horas** después de crear el servidor para obtener mejores estimaciones.

Durante la restauración geográfica, las configuraciones de servidor que se pueden cambiar incluyen la generación de procesos, núcleos virtuales, período de retención de copia de seguridad y opciones de redundancia de copia de seguridad. No se permite cambiar el Plan de tarifa (Básico, Uso general o Memoria optimizada) ni el tamaño de Almacenamiento durante la restauración geográfica.

El tiempo estimado de recuperación depende de varios factores, como el tamaño de la bases de datos, el tamaño del registro de transacciones, el ancho de banda de red y el número total de bases de datos que se están recuperando en la misma región al mismo tiempo. Normalmente, el tiempo de recuperación es inferior a 12 horas.

### <a name="perform-post-restore-tasks"></a>Tareas posteriores a la restauración

Cuando efectúe la restauración con cualquiera de los mecanismos de recuperación, deberá realizar las siguientes tareas para que los usuarios y las aplicaciones vuelvan a conectarse:

- Si el nuevo servidor está destinado a reemplazar al servidor original, redirija los clientes y las aplicaciones de cliente al nuevo servidor.
- Asegúrese de que haya vigentes reglas de red virtual adecuadas para que los usuarios se conecten. Estas reglas no se copian desde el servidor original.
- No se olvide de emplear los permisos de nivel de base de datos y los inicios de sesión apropiados.
- Configure las alertas según corresponda.

## <a name="next-steps"></a>Pasos siguientes

- Para más información acerca de la continuidad del negocio, consulte la  [introducción a la continuidad de negocio](concepts-business-continuity.md).
- Para restaurar a un momento dado con Azure Portal, vea la  [restauración de un servidor a un momento dado con Azure Portal](howto-restore-server-portal.md).
- Para restaurar a un momento dado con la CLI de Azure, vea la  [restauración de un servidor a un momento dado con la CLI de Azure](howto-restore-server-cli.md).
