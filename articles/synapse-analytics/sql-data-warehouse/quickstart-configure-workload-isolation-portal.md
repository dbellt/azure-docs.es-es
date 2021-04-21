---
title: 'Inicio rápido: Configuración del aislamiento de cargas de trabajo: Azure Portal'
description: Use Azure Portal para configurar el aislamiento de cargas de trabajo de un grupo de SQL dedicado.
services: synapse-analytics
author: ronortloff
ms.author: rortloff
manager: craigg
ms.reviewer: jrasnick
ms.date: 05/04/2020
ms.topic: quickstart
ms.service: synapse-analytics
ms.subservice: sql-dw
ms.custom:
- azure-synapse
- mode-portal
ms.openlocfilehash: c15f21064012c195315a7f91908b3160591dc6f8
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107534257"
---
# <a name="quickstart-configure-dedicated-sql-pool-workload-isolation-using-a-workload-group-in-the-azure-portal"></a>Inicio rápido: Configuración del aislamiento de cargas de trabajo de un grupo de SQL dedicado mediante un grupo de cargas de trabajo en Azure Portal

En este inicio rápido, configurará el [aislamiento de cargas de trabajo ](sql-data-warehouse-workload-isolation.md) mediante la creación de un grupo de cargas de trabajo para reservar recursos.  Para este tutorial, se creará el grupo de cargas de trabajo para la carga de datos llamado `DataLoads`. El grupo de cargas de trabajo reservará un 20 % de los recursos del sistema.  Con un aislamiento del 20 % para las cargas de datos, se garantizan los recursos necesarios para cumplir los Acuerdos de Nivel de Servicio.  Después de crear el grupo de cargas de trabajo, [cree un clasificador de cargas de trabajo](quickstart-create-a-workload-classifier-portal.md) para asignarle consultas.


Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.


## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en [Azure Portal](https://portal.azure.com/).

> [!NOTE]
> La creación de una instancia de grupo de SQL dedicado en Azure Synapse Analytics puede dar lugar a un nuevo servicio facturable.  Para más información, consulte los [precios de Azure Synapse Analytics](https://azure.microsoft.com/pricing/details/sql-data-warehouse/).

## <a name="prerequisites"></a>Prerrequisitos

En este inicio rápido se da por supuesto que ya tiene una instancia de grupo de SQL dedicado en Synapse SQL y que tiene permisos CONTROL DATABASE. Si tiene que crearla, consulte [Inicio rápido: Crear un grupo de SQL dedicado: Azure Portal](../quickstart-create-sql-pool-portal.md) para crear un almacenamiento de datos denominado **mySampleDataWarehouse**.

>[!IMPORTANT] 
>Su grupo de SQL dedicado debe estar en línea para que se pueda configurar la administración de las cargas de trabajo. 

## <a name="configure-workload-isolation"></a>Configuración del aislamiento de cargas de trabajo

Los recursos de un grupo de SQL dedicado se pueden aislar y reservar para cargas de trabajo concretas mediante la creación de grupos de cargas de trabajo.  Para más información sobre la forma en que los grupos de cargas de trabajo le ayudan en la administración de su carga de trabajo, consulte la documentación relativa al concepto de [Aislamiento de cargas de trabajo](sql-data-warehouse-workload-isolation.md).  En el inicio rápido sobre la [creación y conexión desde Azure Portal](create-data-warehouse-portal.md) se creó **mySampleDataWarehouse** y se inicializó en DW100c. Con los pasos siguientes se crea un grupo de cargas de trabajo en **mySampleDataWarehouse**.

Para crear un grupo de cargas de trabajo con un aislamiento del 20 %:
1.  Vaya a la página **mySampleDataWarehouse** del grupo de SQL dedicado.
1.  Seleccione **Administración de cargas de trabajos**.
1.  Seleccione **Nuevo grupo de cargas de trabajo**.
1.  Seleccione **Personalizada**.

    ![Haga clic en Personalizada](./media/quickstart-configure-workload-isolation-portal/create-wg.png)

6.  Escriba `DataLoads` en **Grupo de cargas de trabajo**.
7.  Escriba `20` en **Porcentaje mín. de recursos**.
8.  Escriba `5` en **Porcentaje mín. de recursos por solicitud**.
9.  Escriba `100` en **Porcentaje del límite máximo de recursos**.
10. Escriba **Guardar**.

   ![Haga clic en Guardar](./media/quickstart-configure-workload-isolation-portal/configure-wg.png)

Cuando se crea el grupo de cargas de trabajo, aparece una notificación del portal.  Los recursos del grupo de cargas de trabajo se muestran en los gráficos debajo de los valores configurados.

   ![Hacer clic en Final](./media/quickstart-configure-workload-isolation-portal/display-wg.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Para eliminar el grupo de cargas de trabajo `DataLoads` creado en este tutorial:
1. Haga clic en **`...`** a la derecha del grupo de cargas de trabajo `DataLoads`.
2. Haga clic en **Eliminar grupo de cargas de trabajo**.
3. Haga clic en **Sí** cuando se le pida que confirme la eliminación del grupo de cargas de trabajo.
4. Haga clic en **Guardar**.

   ![Hacer clic en Eliminar](./media/quickstart-configure-workload-isolation-portal/delete-wg.png)



Se le está cobrando por unidades de almacenamiento de datos y por los datos almacenados en el almacenamiento de datos. Estos recursos de proceso y de almacenamiento se facturan por separado.

- Si desea conservar los datos de almacenamiento, puede pausar el proceso cuando no está utilizando el almacenamiento de datos. Si se hace una pausa en el proceso, solo se carga el almacenamiento de datos. Cuando esté listo para trabajar con los datos, reanude el proceso.
- Si desea quitar cobros futuros, puede eliminar el almacenamiento de datos.

Siga estos pasos para limpiar los recursos.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y seleccione el grupo de SQL dedicado.

    ![Limpieza de recursos](./media/load-data-from-azure-blob-storage-using-polybase/clean-up-resources.png)

2. Para pausar el proceso, seleccione el botón **Pausar**. Cuando el almacenamiento de datos se haya puesto en pausa, verá un botón **Iniciar**.  Para reanudar el proceso, seleccione **Iniciar**.

3. Para quitar el almacenamiento de datos para que no se le cobre por proceso o almacenamiento, seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

Para usar el grupo de cargas de trabajo `DataLoads`, es preciso crear un [clasificador de cargas de trabajo](/sql/t-sql/statements/create-workload-classifier-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) para enrutar las solicitudes al grupo de cargas de trabajo.  Pase al tutorial acerca de cómo [crear un clasificador de cargas de trabajo](quickstart-create-a-workload-classifier-portal.md) para crear uno para `DataLoads`.

## <a name="see-also"></a>Consulte también
Vea el artículo de procedimientos [Administrar y supervisión de la administración de cargas de trabajo](sql-data-warehouse-how-to-manage-and-monitor-workload-importance.md) para más información sobre cómo supervisar cargas de trabajo para la administración de cargas de trabajo.
