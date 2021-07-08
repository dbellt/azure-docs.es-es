---
title: archivo de inclusión
description: archivo de inclusión
services: cosmos-db
author: SnehaGunda
ms.service: cosmos-db
ms.topic: include
ms.date: 05/19/2021
ms.author: sngun
ms.custom: include file
ms.openlocfilehash: baf0a33f136e54bdc0d31706892fa19afb30df03
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110486769"
---
1. En el menú de Azure Portal o en la **página principal**, seleccione **Crear un recurso**.

1. En la página **Nuevos**, busque y seleccione **Azure Cosmos DB**.

1. En la página **Azure Cosmos DB**, seleccione **Crear**.

1. En la página **Creación de una cuenta de Azure Cosmos DB**, especifique la configuración básica de la nueva cuenta de Azure Cosmos.

   |Configuración|Valor|Descripción |
   |---|---|---|
   | Suscripción|Su suscripción|Seleccione la suscripción de Azure que quiere usar para esta cuenta de Azure Cosmos DB. |
   | Grupo de recursos|Crear nuevo<br><br>A continuación, escriba el mismo nombre que el nombre de cuenta.|Seleccione **Crear nuevo**. Luego, escriba un nombre nuevo de grupo de recursos para la cuenta. Para simplificar, use el mismo nombre que el de la cuenta de Azure Cosmos. |
   | Nombre de cuenta|Escriba un nombre único.|Escriba un nombre único para identificar la cuenta de Azure Cosmos DB. El URI de la cuenta será *cassandra.cosmos.azure.com* y se anexará al nombre único de la cuenta.<br><br>El nombre de la cuenta solo puede utilizar letras minúsculas, números y guiones (-), y debe tener entre 3 y 31 caracteres de longitud.|
   | API|Cassandra|La API determina el tipo de cuenta que se va a crear. Azure Cosmos DB proporciona cinco API: Core (SQL) para bases de datos de documentos, Gremlin para bases de datos de grafos, MongoDB para bases de datos de documentos, Azure Table y Cassandra. Debe crear una cuenta independiente para cada API. <br><br>Seleccione **Cassandra**, ya que en este inicio rápido va a crear una tabla que funciona con Cassandra API. <br><br>[Más información acerca de Cassandra API](../articles/cosmos-db/cassandra-introduction.md).|
   |Location|Región más cercana a los usuarios|Seleccione una ubicación geográfica para hospedar la cuenta de Azure Cosmos DB. Use la ubicación más cercana a los usuarios para que puedan acceder de la forma más rápida posible a los datos.|
   |Capacity mode (Modo de capacidad)|Rendimiento aprovisionado o sin servidor|Seleccione **Provisioned throughput** (Rendimiento aprovisionado) para crear una cuenta en modo de [rendimiento aprovisionado](../articles/cosmos-db/set-throughput.md). Seleccione **Serverless** (Sin servidor) para crear una cuenta en modo [sin servidor](../articles/cosmos-db/serverless.md).|
   |Aplicar el descuento del nivel Gratis de Azure Cosmos DB|**Aplicar** o **No aplicar**|Con el nivel Gratis de Azure Cosmos DB, recibirá los primeros 1000 RU/s y 25 GB de almacenamiento gratis en una cuenta. Más información acerca del [nivel Gratis](https://azure.microsoft.com/pricing/details/cosmos-db/).|

   > [!NOTE]
   > Puede tener una cuenta de Azure Cosmos DB de nivel Gratis por cada suscripción de Azure y debe optar por tenerla al crear la cuenta. Si no ve la opción para aplicar el descuento por nivel Gratis, significará que en otra cuenta de la suscripción ya se ha habilitado el nivel Gratis.

   :::image type="content" source="./media/cosmos-db-create-dbaccount-cassandra/azure-cosmos-db-create-new-account.png" alt-text="Página de la nueva cuenta de Cassandra API para Azure Cosmos DB":::

1. En la pestaña **Distribución global**, configure los detalles siguientes. Puede dejar los valores predeterminados para este inicio rápido:

   |Configuración|Valor|Descripción |
   |---|---|---|
   |Redundancia geográfica|Deshabilitar|Habilite o deshabilite la distribución global en su cuenta. Para ello, debe emparejar su región con una región de par. Puede agregar más regiones a su cuenta más adelante.|
   |Escrituras en varias regiones|Deshabilitar|La funcionalidad de escrituras en varias regiones le permite aprovechar el rendimiento aprovisionado para sus bases de datos y contenedores de todo el mundo.|

   > [!NOTE]
   > Las siguientes opciones no están disponibles si selecciona **Serverless** (Sin servidor) en **Capacity mode** (Modo de capacidad):
   > - Aplicación de descuento por nivel Gratis
   > - Redundancia geográfica
   > - Escrituras en varias regiones

1. Opcionalmente, puede configurar detalles adicionales en las pestañas siguientes:

   * **Redes**: configure el [acceso desde una red virtual](../articles/cosmos-db/how-to-configure-vnet-service-endpoint.md).
   * **Directiva de copia de seguridad**: configure una directiva de copia de seguridad [periódica](../articles/cosmos-db/configure-periodic-backup-restore.md) o [continua](../articles/cosmos-db/continuous-backup-restore-portal.md).
   * **Cifrado**: use una clave administrada por el servicio o una [clave administrada por el cliente](../articles/cosmos-db/how-to-setup-cmk.md#create-a-new-azure-cosmos-account).
   * **Etiquetas**: son pares nombre-valor que permiten categorizar los recursos y ver una facturación consolidada mediante la aplicación de la misma etiqueta en varios recursos y grupos de recursos.

1. Seleccione **Revisar + crear**.

1. Revise la configuración de la cuenta y seleccione **Crear**. La operación de creación de la cuenta tarda unos minutos. Espere hasta que la página del portal muestre **Se completó la implementación** .

   :::image type="content" source="./media/cosmos-db-create-dbaccount/azure-cosmos-db-account-created.png" alt-text="Panel de notificaciones de Azure Portal":::

1. Seleccione **Ir al recurso** para ir a la página de la cuenta de Azure Cosmos DB. 