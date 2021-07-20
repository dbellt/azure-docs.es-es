---
title: Creación de una base de datos única con el libro de contabilidad habilitado
description: Cree una base de datos única en Azure SQL Database con el libro de contabilidad habilitado mediante Azure Portal.
ms.service: sql-database
ms.subservice: security
ms.devlang: ''
ms.topic: quickstart
author: JasonMAnderson
ms.author: janders
ms.reviewer: vanto
ms.date: 05/25/2021
ms.openlocfilehash: d000ec621dcf840c9f18269a2046b682b6d11b56
ms.sourcegitcommit: 91fdedcb190c0753180be8dc7db4b1d6da9854a1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/17/2021
ms.locfileid: "112292959"
---
# <a name="quickstart-create-a-database-in-azure-sql-database-with-ledger-enabled"></a>Inicio rápido: Creación de una base de datos de Azure SQL Database con el libro de contabilidad habilitado

[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

> [!NOTE]
> Actualmente, el libro de contabilidad de Azure SQL Database está en versión preliminar pública y está disponible en el Centro-oeste de EE. UU.

En este inicio rápido, creará una [base de datos de libro de contabilidad](ledger-overview.md#ledger-database) en Azure SQL Database y configurará el [almacenamiento de resumen automático con Azure Blob Storage](ledger-digest-management-and-database-verification.md#automatic-generation-and-storage-of-database-digests) mediante Azure Portal. Para más información sobre el libro de contabilidad, consulte [Libro de contabilidad de Azure SQL Database](ledger-overview.md).

## <a name="prerequisite"></a>Requisito previo

Necesita una suscripción de Azure activa. En caso de no tener ninguna, [cree una cuenta gratuita](https://azure.microsoft.com/free/).

## <a name="create-a-ledger-database-and-configure-digest-storage"></a>Creación de una base de datos de libro de contabilidad y configuración del almacenamiento de resumen

Cree una base de datos de libro de contabilidad única en el [nivel de proceso sin servidor](serverless-tier-overview.md) y configure la carga de los resúmenes de libro de contabilidad en una cuenta de Azure Storage.

### <a name="use-the-azure-portal"></a>Uso de Azure Portal

Para crear una base de datos única en Azure Portal, este inicio rápido comienza en la página de Azure SQL.

1. Vaya a la página [Seleccione una opción de implementación de SQL](https://portal.azure.com/#create/Microsoft.AzureSQL).

1. En **Bases de datos SQL**, deje **Tipo de recurso** establecido en **Base de datos única** y seleccione **Crear**.

   ![Captura de pantalla que muestra la adición a Azure SQL.](./media/single-database-create-quickstart/select-deployment.png)

1. En la pestaña **Básico** del formulario **Crear base de datos SQL**, en **Detalles del proyecto**, seleccione la suscripción de Azure que quiere utilizar.

1. En **Grupo de recursos**, seleccione **Crear nuevo**, escriba **miGrupoDeRecursos** y seleccione **Aceptar**.

1. En **Nombre de la base de datos**, escriba **demo**.

1. En **Servidor**, seleccione **Crear nuevo**. Rellene el formulario **Nuevo servidor** con los siguientes valores:
   - **Nombre del servidor**: Escriba **miServidorSql** y agregue algunos caracteres para que el nombre sea único. No se puede proporcionar un nombre de servidor exacto para usar porque los nombres de los servidores deben ser globalmente únicos para todos los servidores en Azure, no solo únicos dentro de una suscripción. Escriba algo como **mysqlserver12345** y el portal le indicará si está disponible o no.
   - **Inicio de sesión del administrador del servidor**: escriba **usuarioazure**.
   - **Contraseña**: escriba una contraseña que cumpla los requisitos. Vuelva a escribirla en el cuadro **Confirmar contraseña**.
   - **Ubicación**: Seleccione una ubicación en la lista desplegable.
   - **Allow Azure services to access this server** (Permitir que los servicios de Azure accedan a este servidor): seleccione esta opción para habilitar el acceso al almacenamiento de resúmenes.
   
   Seleccione **Aceptar**.
   
1. Deje **¿Quiere usar un grupo elástico de SQL?** establecido en **No**.

1. En **Proceso y almacenamiento**, seleccione **Configurar base de datos**.

1. En este inicio rápido se usa una base de datos sin servidor, por lo que debe seleccionar **Sin servidor** y, después, seleccionar **Aplicar**. 

      ![Captura de pantalla que muestra la configuración de una base de datos sin servidor.](./media/single-database-create-quickstart/configure-database.png)

1. En la pestaña **Redes**, en **Método de conectividad**, seleccione **Punto de conexión público**.
1. En **Reglas de firewall**, establezca **Agregar dirección IP del cliente actual** en **Sí**. Deje la opción **Permitir que los servicios y recursos de Azure accedan a este grupo de servidores** establecida en **No**.
1. Seleccione **Siguiente: Seguridad** en la parte inferior de la página.

   :::image type="content" source="media/ledger/ledger-create-database-networking-tab.png" alt-text="Captura de pantalla que muestra la pestaña Redes de la pantalla Crear base de datos SQL en Azure Portal.":::

1. En la pestaña **Security** (Seguridad), en la sección **Ledger** (Libro de contabilidad), seleccione la opción **Configure ledger** (Configurar libro de contabilidad).

    :::image type="content" source="media/ledger/ledger-configure-ledger-security-tab.png" alt-text="Captura de pantalla que muestra la configuración de un libro de contabilidad en la pestaña Seguridad de Azure Portal.":::

1. En el panel **Configure ledger** (Configurar libro de contabilidad), en la sección **Ledger** (Libro de contabilidad), seleccione la casilla **Enable for all future tables in this database** (Habilitar para todas las tablas futuras de esta base de datos). Esta configuración garantiza que todas las futuras tablas de la base de datos sean tablas de libro de contabilidad. Por esta razón, todos los datos de la base de datos mostrarán cualquier evidencia de alteración. De manera predeterminada, las nuevas tablas se crearán como tablas de libro de contabilidad actualizables, incluso si no se especifica `LEDGER = ON` en [CREATE TABLE](/sql/t-sql/statements/create-table-transact-sql). También puede dejar esta opción sin seleccionar. A continuación, se le requiere que habilite la funcionalidad de libro de contabilidad por cada tabla al crear nuevas tablas mediante Transact-SQL.

1. En la sección **Digest Storage** (Almacenamiento de resumen), la opción **Enable automatic digest storage** (Habilitar el almacenamiento de resumen automático) está seleccionada automáticamente. A continuación, se crean una cuenta y un contenedor de Azure Storage donde se almacenan los resúmenes.

1. Seleccione **Aplicar**.

    :::image type="content" source="media/ledger/ledger-configure-ledger-pane.png" alt-text="Captura de pantalla que muestra el panel Configurar libro de contabilidad (versión preliminar) de Azure Portal.":::

1. En la parte inferior de la página, seleccione **Revisar y crear**.

    :::image type="content" source="media/ledger/ledger-review-security-tab.png" alt-text="Captura de pantalla que muestra la revisión y creación de una base de datos de libro de contabilidad en la pestaña Seguridad de Azure Portal.":::

1. En la página **Revisar y crear**, después de revisar, seleccione **Crear**.

## <a name="clean-up-resources"></a>Limpieza de recursos

Mantenga el grupo de recursos, el servidor y la base de datos única para los pasos siguientes. Aprenderá a usar la característica de libro de contabilidad de la base de datos con distintos métodos.

Cuando haya terminado con estos recursos, elimine el grupo de recursos que ha creado. Esta acción también elimina el servidor y la base de datos única que hay dentro de él.

### <a name="use-the-azure-portal"></a>Uso de Azure Portal

Para eliminar **myResourceGroup** y todos sus recursos mediante Azure Portal:

1. En el portal, busque y seleccione **Grupos de recursos**. A continuación, seleccione **myResourceGroup** en la lista.
1. En la página del grupo de recursos, seleccione **Eliminar grupo de recursos**.
1. En **Escriba el nombre del grupo de recursos** escriba **myResourceGroup** y luego seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

Conéctese y consulte la base de datos con diferentes herramientas y lenguajes:

- [Creación y uso de tablas de libro de contabilidad actualizables](ledger-how-to-updatable-ledger-tables.md)
- [Creación y uso de tablas de solo adición del libro de contabilidad](ledger-how-to-append-only-ledger-tables.md) 
