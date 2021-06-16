---
title: 'Inicio rápido: Creación de una instancia mediante Azure Portal'
titleSuffix: Azure Database Migration Service
description: Utilice Azure Portal para crear una instancia de Azure Database Migration Service.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: quickstart
ms.date: 01/29/2021
ms.openlocfilehash: 267857df03bb85ad8455979919e3b1e267a9e127
ms.sourcegitcommit: e832f58baf0b3a69c2e2781bd8e32d4f1ae932c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110586071"
---
# <a name="quickstart-create-an-instance-of-the-azure-database-migration-service-by-using-the-azure-portal"></a>Inicio rápido: Creación de una instancia de Azure Database Migration Service mediante Azure Portal

En este inicio rápido, puede usar Azure Portal para crear una instancia de Azure Database Migration Service. Después de crear la instancia, puede usarla para migrar datos desde varios orígenes de base de datos a plataformas de datos de Azure, como desde SQL Server a Azure SQL Database o desde SQL Server a Azure SQL Managed Instance.

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Abra el explorador web, vaya a [Microsoft Azure Portal](https://portal.azure.com/) y, a continuación, escriba sus credenciales para iniciar sesión en el portal. La vista predeterminada es el panel del servicio.

> [!NOTE]
> Puede crear hasta 10 instancias de DMS por suscripción y región. Si necesita un número mayor de instancias, cree una incidencia de soporte técnico.

## <a name="register-the-resource-provider"></a>Registrar el proveedor de recursos

Registre el proveedor de recursos Microsoft.DataMigration antes de crear su primera instancia de Database Migration Service.

1. En Azure Portal, seleccione **Suscripciones**.

   ![Mostrar suscripciones en el portal](media/quickstart-create-data-migration-service-portal/portal-select-subscription.png)

2. Seleccione la suscripción en la que quiere crear la instancia de Azure Database Migration Service y después seleccione **Proveedores de recursos**.

    ![Mostrar los proveedores de recursos](media/quickstart-create-data-migration-service-portal/portal-select-resource-provider.png)

3. Busque la migración y, después, seleccione **Registrar** para **Microsoft.DataMigration**.

    ![Registro del proveedor de recursos](media/quickstart-create-data-migration-service-portal/dms-register-provider.png)

## <a name="create-an-instance-of-the-service"></a>Creación de una instancia del servicio

1. En el menú de Azure Portal o en la página **principal**, seleccione **Crear un recurso**. Busque y seleccione **Azure Database Migration Service**.

    ![Azure Marketplace](media/quickstart-create-data-migration-service-portal/portal-marketplace.png)

2. En la pantalla **Azure Database Migration Service**, seleccione **Crear**.

    ![Creación de una instancia de Azure Database Migration Service](media/quickstart-create-data-migration-service-portal/dms-create.png)

3. En la pantalla de aspectos básicos **Crear el servicio de migración**:

     - Seleccione la suscripción.
     - Cree un grupo de recursos o seleccione uno existente.
     - Especifique un nombre para la instancia de Azure Database Migration Service.
     - Seleccione la ubicación en la que quiere crear la instancia de Azure Database Migration Service.
     - Elija **Azure** como modo de servicio.
     - Seleccione un plan de tarifa. Para más información sobre los costos y planes de tarifa, vea la [página de precios](https://aka.ms/dms-pricing).
     
    ![Configuración de los valores básicos de la instancia de Azure Database Migration Service](media/quickstart-create-data-migration-service-portal/dms-create-basics.png)

     - Seleccione Siguiente: Redes.

4. En la pantalla de red **Crear el servicio de migración**:

    - Seleccione una red virtual existente o cree una nueva. La red virtual proporciona a Azure Database Migration Service acceso a la base de datos de origen y al entorno de destino. Para más información sobre cómo crear una red virtual en Azure Portal, consulte el artículo [Creación de una red virtual con Azure Portal](../virtual-network/quick-create-portal.md).

    ![Configuración de la red de la instancia de Azure Database Migration Service](media/quickstart-create-data-migration-service-portal/dms-network-settings.png)

    - Seleccione **Revisar y crear** para crear el servicio. 
    
    - Transcurridos unos instantes, se crea la instancia de Azure Database Migration Service y estará lista para usarse:

    ![Servicio de migración creado](media/quickstart-create-data-migration-service-portal/dms-service-created.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Puede limpiar los recursos creados en este inicio rápido mediante la eliminación del [grupo de recursos de Azure](../azure-resource-manager/management/overview.md). Para eliminar el grupo de recursos, navegue a la instancia de Azure Database Migration Service que creó. Seleccione el nombre del **grupo de recursos** y, luego, seleccione **Eliminar grupo de recursos**. Esta acción elimina todos los activos del grupo de recursos, así como el mismo grupo.

## <a name="next-steps"></a>Pasos siguientes

* [Migración de SQL Server a Azure SQL Database sin conexión](tutorial-sql-server-to-azure-sql.md)
* [Migración de SQL Server a Azure SQL Managed Instance sin conexión](tutorial-sql-server-to-managed-instance.md)
* [Migración de SQL Server a Azure SQL Managed Instance en línea](tutorial-sql-server-managed-instance-online.md)
