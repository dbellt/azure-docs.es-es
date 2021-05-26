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
ms.openlocfilehash: d03d104a5b0d9ad9d83cbecdeeab8614cf0ee728
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388781"
---
# <a name="quickstart-create-an-azure-sql-database-with-ledger-enabled"></a>Inicio rápido: Creación de una instancia de Azure SQL Database con el libro de contabilidad habilitado

[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

> [!NOTE]
> El libro de contabilidad de Azure SQL Database se encuentra actualmente en **versión preliminar pública**.

En este inicio rápido, creará una [base de datos de libro de contabilidad](ledger-overview.md#ledger-database) en Azure SQL Database y configurará el [almacenamiento de resumen automático](ledger-digest-management-and-database-verification.md#automatic-generation-and-storage-of-database-digests) con Azure Blob Storage mediante Azure Portal. Para más información sobre el libro de contabilidad, consulte [Libro de contabilidad de Azure SQL Database](ledger-overview.md).

## <a name="prerequisite"></a>Requisito previo

- Una suscripción de Azure activa. En caso de no tener ninguna, [cree una cuenta gratuita](https://azure.microsoft.com/free/).

## <a name="create-a-ledger-database-and-configure-digest-storage"></a>Creación de una base de datos de libro de contabilidad y configuración del almacenamiento de resumen

Cree una base de datos de libro de contabilidad única en el [nivel de proceso sin servidor](serverless-tier-overview.md) y configure la carga de los resúmenes de libro de contabilidad en una cuenta de Azure Storage.

### <a name="using-the-azure-portal"></a>Uso de Azure Portal

Para crear una base de datos única en Azure Portal, este inicio rápido comienza en la página de Azure SQL.

1. Vaya a la página [Seleccione una opción de implementación de SQL](https://portal.azure.com/#create/Microsoft.AzureSQL).

1. En **Bases de datos SQL**, deje **Tipo de recurso** establecido en **Base de datos única** y seleccione **Crear**.

   ![Incorporación a Azure SQL](./media/single-database-create-quickstart/select-deployment.png)

1. En la pestaña **Básico** del formulario **Create SQL Database**, en **Detalles del proyecto**, seleccione la **suscripción** de Azure correcta.

1. En **Grupo de recursos**, seleccione **Crear nuevo**, escriba *miGrupoDeRecursos* y seleccione **Aceptar**.

1. En **Nombre de la base de datos**, escriba *demo*.

1. En **Servidor**, seleccione **Crear nuevo** y rellene el formulario **Nuevo servidor** con los valores siguientes:
   - **Nombre del servidor**: Escriba *miServidorSql* y agregue algunos caracteres para que el nombre sea único. No se puede proporcionar un nombre de servidor exacto para usar porque los nombres de los servidores deben ser globalmente únicos para todos los servidores en Azure, no solo únicos dentro de una suscripción. Por lo tanto, escriba algo como mysqlserver12345 y el portal le indicará si está disponible o no.
   - **Inicio de sesión del administrador del servidor**: escriba *usuarioazure*.
   - **Contraseña**: escriba una contraseña que cumpla los requisitos y escríbala de nuevo en el campo **Confirmar contraseña**.
   - **Ubicación**: Seleccione una ubicación en la lista desplegable.
   - Seleccione la opción **Allow Azure services to access this server** (Permitir que los servicios de Azure accedan a este servidor) para habilitar el acceso al almacenamiento de resumen.
   
   Seleccione **Aceptar**.
   
1. Deje **¿Quiere usar un grupo elástico de SQL?** establecido en **No**.

1. En **Proceso y almacenamiento**, seleccione **Configurar base de datos**.

1. En este inicio rápido se usa una base de datos sin servidor, por lo que debe seleccionar **Sin servidor** y, después, seleccionar **Aplicar**. 

      ![configuración de una base de datos sin servidor](./media/single-database-create-quickstart/configure-database.png)

1. En la pestaña **Redes**, en **Método de conectividad**, seleccione **Punto de conexión público**.
1. En **Reglas de firewall**, establezca **Agregar dirección IP del cliente actual** en **Sí**. Deje la opción **Permitir que los servicios y recursos de Azure accedan a este grupo de servidores** establecida en **No**.
1. Seleccione **Siguiente: Seguridad** en la parte inferior de la página.

   :::image type="content" source="media/ledger/ledger-create-database-networking-tab.png" alt-text="Pestaña Redes de Crear base de datos en Azure Portal":::

1. En la pestaña **Security** (Seguridad), en la sección **Ledger** (Libro de contabilidad), seleccione la opción **Configure ledger** (Configurar libro de contabilidad).

    :::image type="content" source="media/ledger/ledger-configure-ledger-security-tab.png" alt-text="Configurar libro de contabilidad en la pestaña Seguridad de Azure Portal":::

1. En el panel **Configure ledger** (Configurar libro de contabilidad), en la sección **Ledger** (Libro de contabilidad), seleccione la casilla **Enable for all future tables in this database** (Habilitar para todas las tablas futuras de esta base de datos). Esta configuración garantiza que todas las tablas futuras de la base de datos serán tablas de libro de contabilidad, lo que significa que todos los datos de la base de datos serán a prueba de manipulación. De manera predeterminada, las nuevas tablas se crearán como tablas de libro de contabilidad actualizables, incluso si no se especifica `LEDGER = ON` en [CREATE TABLE](/sql/t-sql/statements/create-table-transact-sql). Como alternativa, puede dejar esta opción sin seleccionar, lo que requiere que habilite la funcionalidad de libro de contabilidad por cada tabla al crear nuevas tablas mediante Transact-SQL.

1. En la sección **Digest storage** (Almacenamiento de resumen), la opción **Enable automatic digest storage** (Habilitar el almacenamiento de resumen automático) se seleccionará automáticamente y, posteriormente, se crea una nueva cuenta de Azure Storage y un contenedor donde se almacenarán los resúmenes.

1. Haga clic en el botón **Aplicar**.

    :::image type="content" source="media/ledger/ledger-configure-ledger-pane.png" alt-text="Panel Configurar libro de contabilidad en Azure Portal":::

1. En la parte inferior de la página, seleccione **Revisar y crear**.

    :::image type="content" source="media/ledger/ledger-review-security-tab.png" alt-text="Revisar y crear una base de datos de libro de contabilidad en la pestaña Seguridad de Azure Portal":::

1. En la página **Revisar y crear**, después de revisar, seleccione **Crear**.

## <a name="clean-up-resources"></a>Limpieza de recursos

Mantenga el grupo de recursos, el servidor y la base de datos única para los pasos siguientes y aprenda cómo usar la característica de libro de contabilidad de la base de datos con distintos métodos.

Cuando haya terminado de usar estos recursos, puede eliminar el grupo de recursos que creó, lo que también eliminará el servidor y la base de datos única que se encuentran del grupo.

### <a name="using-the-azure-portal"></a>Uso de Azure Portal

Para eliminar **myResourceGroup** y todos sus recursos con Azure Portal:

1. En el portal, busque y seleccione **Grupos de recursos** y, después, seleccione **myResourceGroup** en la lista.
1. En la página del grupo de recursos, seleccione **Eliminar grupo de recursos**.
1. En **Escriba el nombre del grupo de recursos** escriba *myResourceGroup* y luego seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

Conéctese y consulte la base de datos con diferentes herramientas e idiomas:

- [Creación y uso de tablas de libro de contabilidad actualizables](ledger-how-to-updatable-ledger-tables.md)
- [Creación y uso de tablas de solo adición del libro de contabilidad](ledger-how-to-append-only-ledger-tables.md) 
