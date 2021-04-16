---
title: Conexión a Azure Database for MySQL mediante dbForge Studio for MySQL
description: En este artículo se muestra cómo conectarse a Azure Database for MySQL Server a través de dbForge Studio for MySQL.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: conceptual
ms.date: 03/03/2021
ms.openlocfilehash: 942651aadf4113c1aca32e4e1d2c558b0d764421
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107377232"
---
# <a name="connect-to-azure-database-for-mysql-using-dbforge-studio-for-mysql"></a>Conexión a Azure Database for MySQL mediante dbForge Studio for MySQL

Para conectarse a Azure Database for MySQL mediante [dbForge Studio for MySQL](https://www.devart.com/dbforge/mysql/studio/):

1. En el menú Database (Base de datos), seleccione New Connection (Nueva conexión).

2. Escriba un nombre de host y las credenciales de inicio de sesión.

3. Seleccione el botón Test Connection (Probar conexión) para comprobar la configuración.

:::image type="content" source="media/concepts-migrate-dbforge-studio-for-mysql/azure-connection-1.png" alt-text="Conexión de Azure":::

## <a name="migrate-a-database-using-the-backup-and-restore-functionality"></a>Migración de una base de datos mediante la funcionalidad de copia de seguridad y restauración

Studio permite migrar bases de datos a Azure de muchas maneras, la opción depende exclusivamente de sus necesidades. Si tiene que mover toda la base de datos, es mejor usar la funcionalidad Backup and Restore (Copia de seguridad y restauración). En este ejemplo, migramos a Azure Database for MySQL la base de datos *sakila*, que reside en un servidor MySQL. La lógica que subyace al proceso de migración con la funcionalidad Backup and Restore (Copia de seguridad y restauración) de dbForge Studio for MySQL consiste en crear una copia de seguridad de la base de datos MySQL y, a continuación, restaurarla en Azure Database for MySQL.

### <a name="back-up-the-database"></a>Copia de seguridad de la base de datos

1. En el menú Database (Base de datos), seleccione Backup and Restore (Copia de seguridad y restauración) y, a continuación, seleccione Backup Database (Hacer una copia de seguridad de la base de datos). Se abre la ventana Database Backup Wizard (Asistente para hacer copias de seguridad de bases de datos).

2. En la pestaña Backup content (Contenido de la copia de seguridad) del asistente para hacer copias de seguridad de bases de datos, seleccione los objetos de base de datos de los que quiera hacer una copia de seguridad.

3. En la pestaña Options (Opciones), configure el proceso de copia de seguridad para que se ajuste a sus necesidades.

    :::image type="content" source="media/concepts-migrate-dbforge-studio-for-mysql/back-up-wizard-options.png" alt-text="Opciones del asistente para hacer copias de seguridad":::

4. A continuación, especifique las opciones de registro y comportamiento de procesamiento de errores.

5. Seleccione Backup (Hacer copia de seguridad).

### <a name="restore-the-database"></a>Restaurar la base de datos

1. Conéctese a Azure Database for MySQL como se ha descrito anteriormente.

2. Haga clic con el botón derecho en el cuerpo de Database Explorer (Explorador de bases de datos), seleccione Backup and Restore (Copia de seguridad y restauración) y, a continuación, seleccione Restore Database (Restaurar base de datos).

3. Se abre Database Restore Wizard (Asistente para restauración de bases de datos), seleccione un archivo con una copia de seguridad de base de datos.

    :::image type="content" source="media/concepts-migrate-dbforge-studio-for-mysql/restore-step-1.png" alt-text="Paso de restauración":::

4. Selecciones Restore (Restaurar).

5. Consulte los resultados.

## <a name="migrate-a-database-using-the-copy-databases-functionality"></a>Migración de una base de datos mediante la funcionalidad de copia de bases de datos

La funcionalidad Copy Databases (Copiar bases de datos) es similar a Backup and Restore (Copia de seguridad y restauración), con la salvedad de que no se necesitan dos pasos para migrar una base de datos. Y lo que es más, la característica permite transferir dos bases de datos o más de una sola vez. La funcionalidad Copy Databases (Copiar bases de datos) solo está disponible en la edición Enterprise de dbForge Studio for MySQL.
En este ejemplo, vamos a migrar la base de datos *world_x* de un servidor MySQL a Azure Database for MySQL.
Para migrar una base de datos mediante la funcionalidad Copy Databases (Copiar bases de datos):

1. En el menú Database (Base de datos), seleccione Copy Databases (Copiar bases de datos). 

2. En la pestaña Copy Databases (Copiar bases de datos) que aparece, especifique la conexión de origen y de destino, y seleccione las bases de datos que se van a migrar. Escribimos la conexión de Azure MySQL y seleccionamos la base de datos *world_x*. Seleccione la flecha verde para iniciar el proceso.

3. Consulte los resultados.

Como resultado de los esfuerzos por migrar la base de datos, *world_x* ha aparecido correctamente en Azure MySQL.

:::image type="content" source="media/concepts-migrate-dbforge-studio-for-mysql/copy-databases-result.png" alt-text="Resultado de la copia de bases de datos":::

## <a name="migrate-a-database-using-schema-and-data-compare-tools"></a>Migración de una base de datos mediante herramientas de comparación de esquemas y datos

dbForge Studio for MySQL incorpora algunas herramientas que permiten migrar bases de datos MySQL, esquemas de MySQL o datos a Azure. La elección de la funcionalidad depende de sus necesidades y de los requisitos del proyecto. Si tiene que mover una base de datos de forma selectiva, es decir, migrar determinadas tablas de MySQL a Azure, lo mejor es usar la funcionalidad Schema and Data Compare (Comparación de esquemas y datos).
En este ejemplo, migramos a Azure Database for MySQL la base de datos *world*, que reside en un servidor MySQL. La lógica que subyace al proceso de migración mediante la funcionalidad Schema and Data Compare (Comparación de esquemas y datos) de dbForge Studio for MySQL consiste en crear una base de datos vacía en Azure Database for MySQL, sincronizarla con la base de datos MySQL necesaria primero mediante la herramienta Schema Compare (Comparación de esquemas) y, a continuación, usar la herramienta Data Compare (Comparación de datos). De este modo, los esquemas y datos de MySQL se mueven correctamente a Azure.

### <a name="step-1-connect-to-azure-database-for-mysql-and-create-an-empty-database"></a>Paso 1. Conexión a Azure Database for MySQL y creación de una base de datos vacía

### <a name="step-2-schema-synchronization"></a>Paso 2. Sincronización de esquemas

1. En el menú Comparison (Comparación), seleccione New Schema Comparison (Nueva comparación de esquemas).
Aparece New Schema Comparison Wizard (Asistente de comparación de nuevos esquemas).

2. Seleccione Source (Origen) y Target (Destino) y, a continuación, especifique las opciones de comparación de esquemas. Seleccione Compare (Comparar).

3. En la cuadrícula de resultados de comparación que aparece, seleccione los objetos para la sincronización. Seleccione el botón de flecha verde para abrir Schema Synchronization Wizard (Asistente para la sincronización de esquemas).

4. Siga los pasos del asistente para configurar la sincronización. Seleccione Synchronize (Sincronizar) para implementar los cambios.

    :::image type="content" source="media/concepts-migrate-dbforge-studio-for-mysql/schema-sync-wizard.png" alt-text="Asistente para la sincronización de esquemas":::

### <a name="step-3-data-comparison"></a>Paso 3. Comparación de datos

1. En el menú Comparison (Comparación), seleccione New Data Comparison (Nueva comparación de datos). Aparece New Data Comparison Wizard (Asistente de comparación de nuevos datos).

2. Seleccione Source (Origen) y Target (Destino) y, a continuación, especifique las opciones de comparación de datos y cambie las asignaciones, si es necesario. Seleccione Compare (Comparar).

3. En la cuadrícula de resultados de comparación que aparece, seleccione los objetos para la sincronización. Seleccione el botón de flecha verde para abrir Data Synchronization Wizard (Asistente para la sincronización de datos).

    :::image type="content" source="media/concepts-migrate-dbforge-studio-for-mysql/data-comp-result.png" alt-text="Resultado de la comparación de datos":::

4. Siga los pasos del asistente para configurar la sincronización. Seleccione Synchronize (Sincronizar) para implementar los cambios.

5. Consulte los resultados.

    :::image type="content" source="media/concepts-migrate-dbforge-studio-for-mysql/data-sync-result.png" alt-text="Resultado de la sincronización de datos":::

## <a name="summary"></a>Resumen

En la actualidad, más empresas están moviendo sus bases de datos a Azure Database for MySQL, ya que este servicio de bases de datos es fácil de configurar, administrar y escalar. No hay necesidad de que la migración sea complicada. dbForge Studio for MySQL dispone de excelentes herramientas de migración que pueden facilitar enormemente el proceso. Studio permite que la transferencia de bases de datos se configure, guarde, edite, automatice y programe de manera sencilla.

## <a name="next-steps"></a>Pasos siguientes
- [Información general de MySQL](overview.md)
