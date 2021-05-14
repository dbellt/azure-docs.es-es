---
title: Configuración de puntos de conexión privados para el almacén analítico de Azure Cosmos DB
description: Aprenda a configurar puntos de conexión privados administrados para el almacén analítico de Azure Cosmos DB a fin de restringir el acceso a la red.
author: AnithaAdusumilli
ms.service: cosmos-db
ms.topic: how-to
ms.date: 03/02/2021
ms.author: anithaa
ms.openlocfilehash: fd0b3ada5fec283562cee9727e3f805a7d34c532
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107479056"
---
# <a name="configure-azure-private-link-for-azure-cosmos-db-analytical-store"></a>Configuración de Azure Private Link para el almacén analítico de Azure Cosmos DB
[!INCLUDE[appliesto-sql-mongodb-api](includes/appliesto-sql-mongodb-api.md)]

En este artículo, aprenderá a configurar puntos de conexión privados administrados para el almacén analítico de Azure Cosmos DB. Si va a usar el almacén transaccional, consulte el artículo [Puntos de conexión privados para el almacén transaccional](how-to-configure-private-endpoints.md). Con los [puntos de conexión privados administrados](../synapse-analytics/security/synapse-workspace-managed-private-endpoints.md), puede restringir el acceso de red del almacén analítico de Azure Cosmos DB a la red virtual administrada asociada con el área de trabajo de Azure Synapse. Los puntos de conexión privados administrados establecen un vínculo privado al almacén analítico.

## <a name="enable-a-private-endpoint-for-the-analytical-store"></a>Habilitación de un punto de conexión privado para el almacén analítico

### <a name="set-up-azure-synapse-analytics-workspace-with-a-managed-virtual-network"></a>Configuración de un área de trabajo de Azure Synapse Analytics con una red virtual administrada

[Cree un área de trabajo en Azure Synapse Analytics con la filtración de datos habilitada.](../synapse-analytics/security/how-to-create-a-workspace-with-data-exfiltration-protection.md) Con la [protección de la filtración de datos](../synapse-analytics/security/workspace-data-exfiltration-protection.md), puede asegurarse de que los usuarios malintencionados no puedan copiar o transferir datos de Azure Resources a ubicaciones fuera del ámbito de su organización.

Las siguientes restricciones de acceso se aplican cuando se activa la protección de filtración de datos para un área de trabajo de Azure Synapse Analytics:

* Si utiliza Azure Spark para Azure Synapse Analytics, solo se permite el acceso a los puntos de conexión privados administrados aprobados para el almacén analítico de Azure Cosmos DB.

* Si usa grupos de SQL sin servidor de Synapse, puede consultar cualquier cuenta de Azure Cosmos DB mediante Azure Synapse Link. Sin embargo, las solicitudes de escritura que [crean tablas externas como select (CETAS)](../synapse-analytics/sql/develop-tables-cetas.md) solo están permitidas para los puntos de conexión privados administrados aprobados en la red virtual del área de trabajo.

> [!NOTE]
> Una vez creada el área de trabajo, no se puede cambiar la configuración de la red virtual administrada y de la filtración de datos.

### <a name="add-a-managed-private-endpoint-for-azure-cosmos-db-analytical-store"></a>Incorporación de un punto de conexión privado administrado para el almacén analítico de Azure Cosmos DB

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com/).

1. En Azure Portal, vaya al área de trabajo de Synapse Analytics y abra el panel **Información general**.

1. Inicie Synapse Studio; para ello, vaya al panel **Información general** y seleccione **Abrir** en **Abrir Synapse Studio**.

1. En Synapse Studio, abra la pestaña **Administrar**.

1. Vaya a **Managed private endpoints** (Puntos de conexión privados administrados) y seleccione **Nuevo**.

   :::image type="content" source="./media/analytical-store-private-endpoints/create-new-private-endpoint.png" alt-text="Cree un nuevo punto de conexión privado para el almacén analítico." border="true":::

1. Seleccione el tipo de cuenta de **Azure Cosmos DB (SQL API)** > **Continuar**.

   :::image type="content" source="./media/analytical-store-private-endpoints/select-private-endpoint.png" alt-text="Seleccione SQL API de Azure Cosmos DB para crear un punto de conexión privado." border="true":::

1. Rellene el formulario **New managed private endpoint** (Nuevo punto de conexión privado administrado) con los detalles siguientes:

   * **Nombre**: nombre del punto de conexión privado administrado. Este nombre no se puede actualizar una vez creado.
   * **Descripción**: proporcione una descripción detallada para identificar el punto de conexión privado.
   * **Suscripción a Azure**: seleccione una cuenta de Azure Cosmos DB de la lista de cuentas disponibles en las suscripciones de Azure.
   * **Nombre de cuenta de Azure Cosmos DB**: seleccione una cuenta de Azure Cosmos DB existente de tipo SQL o MongoDB.
   * **Subrecurso secundario**: seleccione una de las siguientes opciones: **Analíticos**: si desea agregar el punto de conexión privado para el almacén analítico de Azure Cosmos DB.
     **Sql** (o **MongoDB**): si desea agregar un punto de conexión de cuenta OLTP o transaccional.

   > [!NOTE]
   > Puede agregar puntos de conexión privados del almacén de análisis y del almacén transaccional a la misma cuenta de Azure Cosmos DB en un área de trabajo de Azure Synapse Analytics. Si solo desea ejecutar consultas analíticas, es posible que solo quiera asignar el punto de conexión privado analítico.

   :::image type="content" source="./media/analytical-store-private-endpoints/choose-analytical-private-endpoint.png" alt-text="Elija analítico para el subrecurso de destino." border="true":::

1. Después de crearlo, vaya al nombre del punto de conexión privado y seleccione **Manage approvals in Azure Portal** (Administrar aprobaciones en Azure Portal).

1. Vaya a la cuenta de Azure Cosmos DB, seleccione el punto de conexión privado y, luego, **Aprobar**.

1. Vuelva al área de trabajo de Synapse Analytics y haga clic en **Actualizar** en el panel **Managed private endpoints** (Puntos de conexión privados administrados). Compruebe que el punto de conexión privado esté en estado **Aprobado**.

   :::image type="content" source="./media/analytical-store-private-endpoints/approved-private-endpoint.png" alt-text="Compruebe que el punto de conexión privado esté aprobado." border="true":::

## <a name="use-apache-spark-for-azure-synapse-analytics"></a>Uso de Apache Spark para Azure Synapse Analytics

Si ha creado un área de trabajo de Azure Synapse con la protección de filtración de datos activada, el acceso saliente desde Synapse Spark a las cuentas de Azure Cosmos DB estará bloqueado de forma predeterminada. Además, si Azure Cosmos DB ya tiene un punto de conexión privado existente, el acceso a Synapse Spark estará bloqueado.

Para permitir el acceso a los datos de Azure Cosmos DB:

* Si utiliza Azure Synapse Link para consultar los datos de Azure Cosmos DB, agregue un punto de conexión privado **analítico** administrado para la cuenta de Azure Cosmos DB.

* Si usa escrituras y lecturas de lote y/o escrituras y lecturas de streaming en el almacén transaccional, agregue un punto de conexión privado de *SQL* o *MongoDB* para la cuenta de Azure Cosmos DB. Además, también debe establecer *connectionMode* en *Puerta de enlace* tal como se muestra en el siguiente fragmento de código:

  ```python
  # Write a Spark DataFrame into an Azure Cosmos DB container
  # To select a preferred lis of regions in a multi-region account, add .option("spark.cosmos.preferredRegions", "<Region1>, <Region2>")
  
  YOURDATAFRAME.write\
    .format("cosmos.oltp")\
    .option("spark.synapse.linkedService", "<your-Cosmos-DB-linked-service-name>")\
    .option("spark.cosmos.container","<your-Cosmos-DB-container-name>")\
    .option("spark.cosmos.write.upsertEnabled", "true")\
    .option("spark.cosmos.connection.mode", "Gateway")\
    .mode('append')\
    .save()
  
  ```

## <a name="using-synapse-serverless-sql-pools"></a>Uso de grupos de SQL sin servidor de Synapse

Los grupos de SQL sin servidor de Synapse usan funcionalidades multiinquilino que no se implementan en una red virtual administrada. Si la cuenta de Azure Cosmos DB tiene un punto de conexión privado existente, se bloqueará el acceso del grupo de SQL sin servidor de Synapse a la cuenta, debido a las comprobaciones de aislamiento de red en la cuenta de Azure Cosmos DB.

Para configurar el aislamiento de red para esta cuenta desde un área de trabajo de Synapse:

1. Permita que el área de trabajo de Synapse acceda a la cuenta de Azure Cosmos DB especificando la configuración de `NetworkAclBypassResourceId` en la cuenta.

   **Uso de PowerShell**

   ```powershell-interactive
   Update-AzCosmosDBAccount -Name MyCosmosDBDatabaseAccount -ResourceGroupName MyResourceGroup -NetworkAclBypass AzureServices -NetworkAclBypassResourceId "/subscriptions/subId/resourceGroups/rgName/providers/Microsoft.Synapse/workspaces/wsName"
   ```

   **Uso de la CLI de Azure**

   ```azurecli-interactive
   az cosmosdb update --name MyCosmosDBDatabaseAccount --resource-group MyResourceGroup --network-acl-bypass AzureServices --network-acl-bypass-resource-ids "/subscriptions/subId/resourceGroups/rgName/providers/Microsoft.Synapse/workspaces/wsName"
   ```

   > [!NOTE]
   > La cuenta de Azure Cosmos DB y el área de trabajo de Azure Synapse Analytics deben estar en el mismo inquilino de Azure Active Directory (AD).

2. Ahora puede acceder a la cuenta desde grupos de SQL sin servidor, mediante consultas de T-SQL a través de Azure Synapse Link. Sin embargo, para garantizar el aislamiento de red de los datos del almacén analítico, debe agregar un punto de conexión privado administrado **analítico** para esta cuenta. De lo contrario, el acceso público a los datos del almacén analítico no estará bloqueado.

> [!IMPORTANT]
> Si utiliza Azure Synapse Link y necesita aislamiento de red para los datos del almacén analítico, debe asignar la cuenta de Azure Cosmos DB al área de trabajo de Synapse mediante un punto de conexión privado administrado **analítico**.

## <a name="next-steps"></a>Pasos siguientes

* Introducción a la [consulta de un almacén analítico con Azure Synapse Spark](../synapse-analytics/synapse-link/how-to-query-analytical-store-spark.md?toc=/azure/cosmos-db/toc.json&bc=/azure/cosmos-db/breadcrumb/toc.json)
* Introducción a la [consulta de un almacén analítico con grupos de SQL sin servidor de Azure Synapse](../synapse-analytics/sql/query-cosmos-db-analytical-store.md?toc=/azure/cosmos-db/toc.json&bc=/azure/cosmos-db/breadcrumb/toc.json)
