---
title: Uso de dbForge Studio for MySQL para migrar una base de datos mySQL a Azure Database for MySQL
description: En este artículo se muestra cómo migrar a Azure Database for MySQL a través de dbForge Studio for MySQL.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: conceptual
ms.date: 03/03/2021
ms.openlocfilehash: f29d64da678ca65c91a90857cfa5a0e533cc72d6
ms.sourcegitcommit: aba63ab15a1a10f6456c16cd382952df4fd7c3ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107989169"
---
# <a name="migrate-data-to-azure-database-for-mysql-with-dbforge-studio-for-mysql"></a>Migración de datos a Azure Database for MySQL con dbForge Studio for MySQL

¿Desea mover las bases de datos de MySQL a Azure Database for MySQL? Considere la posibilidad de usar las herramientas de migración de dbForge Studio for MySQL. Con esta aplicación, la transferencia de bases de datos se puede configurar, guardar, editar, automatizar y programar.

Para completar los ejemplos de este artículo, deberá descargar e instalar [dbForge Studio for MySQL](https://www.devart.com/dbforge/mysql/studio/).

## <a name="connect-to-azure-database-for-mysql"></a>Conexión a Azure Database for MySQL

1. En dbForge Studio for MySQL, seleccione **New Connection** (Nueva conexión) en el menú **Database** (Base de datos).

1. Escriba un nombre de host y las credenciales de inicio de sesión.

1. Seleccione **Test Connection** (Probar conexión) para comprobar la configuración.

:::image type="content" source="media/concepts-migrate-dbforge-studio-for-mysql/azure-connection-1.png" alt-text="Captura de pantalla que muestra una prueba de conexión correcta Azure Database for MySQL." lightbox="media/concepts-migrate-dbforge-studio-for-mysql/azure-connection-1.png":::

## <a name="migrate-with-the-backup-and-restore-functionality"></a>Migración con la funcionalidad Backup and Restore (Copia de seguridad y restauración)

Puede elegir entre muchas opciones al usar dbForge Studio for MySQL para migrar bases de datos a Azure. Si tiene que mover toda la base de datos, es mejor usar la funcionalidad **Backup and Restore** (Copia de seguridad y restauración).

En este ejemplo, vamos a migrar la base de datos *sakila* de un servidor MySQL a Azure Database for MySQL. La lógica que subyace a la funcionalidad **Backup and Restore** (Copia de seguridad y restauración) consiste en crear una copia de seguridad de la base de datos MySQL y, a continuación, restaurarla en Azure Database for MySQL.

### <a name="back-up-the-database"></a>Copia de seguridad de la base de datos

1. En dbForge Studio for MySQL, seleccione **Backup Database** (Copia de seguridad de base de datos) en el menú **Backup and Restore** (Copia de seguridad y restauración). Se abre la ventana **Database Backup Wizard** (Asistente para hacer copias de seguridad de bases de datos).

1. En la pestaña **Backup content** (Contenido de la copia de seguridad) del **Database Backup Wizard** (Asistente para hacer copias de seguridad de bases de datos), seleccione los objetos de base de datos de los que quiera hacer una copia de seguridad.

1. En la pestaña **Options** (Opciones), configure el proceso de copia de seguridad para que se ajuste a sus necesidades.

    :::image type="content" source="media/concepts-migrate-dbforge-studio-for-mysql/back-up-wizard-options.png" alt-text="Captura de pantalla que muestra el panel de opciones de Database Backup Wizard (Asistente para hacer copias de seguridad de bases de datos)." lightbox="media/concepts-migrate-dbforge-studio-for-mysql/back-up-wizard-options.png":::

1. Seleccione **Next** (Siguiente) y, a continuación, especifique las opciones de registro y el comportamiento del procesamiento de errores.

1. Seleccione **Backup** (Hacer copia de seguridad).

### <a name="restore-the-database"></a>Restaurar la base de datos

1.  En dbForge Studio for MySQL, conéctese a Azure Database for MySQL. [Consulte las instrucciones](#connect-to-azure-database-for-mysql).

1. Seleccione **Restore Database** (Restaurar base de datos) en el menú **Backup and Restore** (Copia de seguridad y restauración). Aparece la ventana de **Database Restore Wizard** (Asistente para la restauración de bases de datos).

1. En **Database Restore Wizard** (Asistente para la restauración de bases de datos), seleccione un archivo con una copia de seguridad de base de datos.

    :::image type="content" source="media/concepts-migrate-dbforge-studio-for-mysql/restore-step-1.png" alt-text="Captura de pantalla que muestra el paso de restauración de Database Restore Wizard (Asistente para la restauración de bases de datos)." lightbox="media/concepts-migrate-dbforge-studio-for-mysql/restore-step-1.png":::

1. Seleccione **Restaurar**.

1. Consulte los resultados.

## <a name="migrate-with-the-copy-databases-functionality"></a>Migración con la funcionalidad Copy Databases (Copiar bases de datos)

La funcionalidad **Copy Databases** (Copiar bases de datos) de dbForge Studio for MySQL es similar a **Backup and Restore** (Copia de seguridad y restauración), salvo que no requiere dos pasos para migrar una base de datos. También permite transferir dos o más bases de datos a la vez.

>[!NOTE]
> La funcionalidad **Copy Databases** (Copiar bases de datos) solo está disponible en la edición Enterprise de dbForge Studio for MySQL.

En este ejemplo, vamos a migrar la base de datos *world_x* de un servidor MySQL a Azure Database for MySQL.

Para migrar una base de datos mediante la funcionalidad Copy Databases (Copiar bases de datos):

1. En dbForge Studio for MySQL, seleccione **Copy Databases** (Copiar bases de datos) en el menú **Database** (Base de datos). 

1. En la pestaña **Copy Databases (Copiar bases de datos)** , especifique la conexión de origen y de destino. Seleccione también las bases de datos que se van a migrar. 

   Escribimos la conexión de Azure MySQL y seleccionamos la base de datos *world_x*. Seleccione la flecha verde para iniciar el proceso.

1. Consulte los resultados.

Verá que la base de *datos world_x* ha aparecido correctamente en Azure MySQL.

:::image type="content" source="media/concepts-migrate-dbforge-studio-for-mysql/copy-databases-result.png" alt-text="Captura de pantalla que muestra los resultados de la función Copy Databases (Copiar bases de datos)." lightbox="media/concepts-migrate-dbforge-studio-for-mysql/copy-databases-result.png":::

## <a name="migrate-a-database-with-schema-and-data-comparison"></a>Migración de una base de datos con comparación de esquemas y datos

Puede elegir entre muchas opciones al usar dbForge Studio for MySQL para migrar bases de datos, esquemas o datos a Azure. Si necesita mover tablas selectivas de una base de datos MySQL a Azure, es mejor usar la funcionalidad **Schema Comparison** (Comparación de esquemas) y **Data Comparison** (Comparación de datos).

En este ejemplo, vamos a migrar la base de datos *world* de un servidor MySQL a Azure Database for MySQL. 

La lógica que subyace a la funcionalidad **Backup and Restore** (Copia de seguridad y restauración) consiste en crear una copia de seguridad de la base de datos MySQL y, a continuación, restaurarla en Azure Database for MySQL.

La lógica que subyace a este enfoque es crear una base de datos vacía en Azure Database for MySQL y sincronizarla con la base de datos MySQL de origen. Primero usamos la herramienta **Schema Comparison** (Comparación de esquemas) y, a continuación, usamos la funcionalidad **Data Comparison** (Comparación de datos). Estos pasos garantizan que los esquemas y los datos de MySQL se mueven con precisión a Azure.

Para completar este ejercicio, primero deberá [conectarse a Azure Database for MySQL](#connect-to-azure-database-for-mysql) y crear una base de datos vacía.

### <a name="schema-synchronization"></a>Sincronización de esquemas

1. En el menú **Comparison** (Comparación), seleccione **New Schema Comparison** (Nueva comparación de esquemas). Aparece **New Schema Comparison Wizard** (Asistente de comparación de nuevos esquemas).

1. Elija el origen y el destino y, a continuación, especifique las opciones de comparación de esquemas. Seleccione **Compare** (Comparar).

1. En la cuadrícula de resultados de comparación que aparece, seleccione los objetos para la sincronización. Seleccione el botón de flecha verde para abrir **Schema Synchronization Wizard** (Asistente para la sincronización de esquemas).

1. Siga los pasos del asistente para configurar la sincronización. Seleccione **Synchronize** (Sincronizar) para implementar los cambios.

    :::image type="content" source="media/concepts-migrate-dbforge-studio-for-mysql/schema-sync-wizard.png" alt-text="Captura de pantalla que muestra el Asistente para Schema Synchronization Wizard (Asistente para la sincronización de esquemas)." lightbox="media/concepts-migrate-dbforge-studio-for-mysql/schema-sync-wizard.png":::

### <a name="data-comparison"></a>Comparación de datos

1. En el menú **Comparison** (Comparación), seleccione **New Data Comparison** (Nueva comparación de datos). Aparece **New Data Comparison Wizard** (Asistente de comparación de nuevos datos).

1. Elija el origen y el destino y, a continuación, especifique las opciones de comparación de datos. Cambie las asignaciones si es necesario y, a continuación, seleccione **Compare** (Comparar).

1. En la cuadrícula de resultados de comparación que aparece, seleccione los objetos para la sincronización. Seleccione el botón de flecha verde para abrir **Data Synchronization Wizard** (Asistente para la sincronización de datos).

    :::image type="content" source="media/concepts-migrate-dbforge-studio-for-mysql/data-comp-result.png" alt-text="Captura de pantalla en la que se muestran los resultados de la consulta." lightbox="media/concepts-migrate-dbforge-studio-for-mysql/data-comp-result.png":::

1. Siga los pasos del asistente para configurar la sincronización. Seleccione **Synchronize** (Sincronizar) para implementar los cambios.

1. Consulte los resultados.

    :::image type="content" source="media/concepts-migrate-dbforge-studio-for-mysql/data-sync-result.png" alt-text="Captura de pantalla que muestra los resultados deData Synchronization wizard (Asistente para la sincronización de datos)." lightbox="media/concepts-migrate-dbforge-studio-for-mysql/data-sync-result.png":::

## <a name="next-steps"></a>Pasos siguientes
- [Información general de MySQL](overview.md)
