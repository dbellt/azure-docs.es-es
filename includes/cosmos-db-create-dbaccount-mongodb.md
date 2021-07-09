---
title: archivo de inclusión
description: archivo de inclusión
services: cosmos-db
ms.custom: include file
ms.openlocfilehash: a8d3e4171a761fd85c9b3f8035c7ea4eeedb5cb3
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110486321"
---
1. En una nueva ventana del explorador, inicie sesión en [Azure Portal](https://portal.azure.com/).

2. En el menú de la izquierda, seleccione **Crear un recurso**.
   
   :::image type="content" source="./media/cosmos-db-create-dbaccount-mongodb/create-nosql-db-databases-json-tutorial-0.png" alt-text="Crear un recurso en Azure Portal":::
   
3. En la página **Nuevo**, seleccione **Bases de datos** > **Azure Cosmos DB**.
   
   :::image type="content" source="./media/cosmos-db-create-dbaccount-mongodb/create-nosql-db-databases-json-tutorial-1.png" alt-text="El panel Bases de datos de Azure Portal":::
   
3. En la página **Crear una cuenta de Azure Cosmos DB**, especifique la configuración de la nueva cuenta de Azure Cosmos DB. 

   |Configuración|Valor|Descripción |
   |---|---|---|
   |Suscripción|Nombre de suscripción|Seleccione la suscripción de Azure que desea usar para esta cuenta de Azure Cosmos. |
   |Grupo de recursos|Definición de un nombre de grupo de recursos|Seleccione un grupo de recursos o seleccione **Crear nuevo** y escriba un nombre único para el grupo de recursos nuevo. |
   |Nombre de cuenta|Escriba un nombre único.|Escriba un nombre único para identificar la cuenta de Azure Cosmos DB. El URI de la cuenta será *mongo.cosmos.azure.com* y se anexará al nombre único de la cuenta.<br><br>El nombre de la cuenta solo puede utilizar letras minúsculas, números y guiones (-), y debe tener entre 3 y 31 caracteres de longitud.|
   API|Azure Cosmos DB para la API de Mongo DB|La API determina el tipo de cuenta que se va a crear. Azure Cosmos DB proporciona cinco API: Core (SQL) para bases de datos de documentos, Gremlin para bases de datos de grafos, API Mongo DB de Azure Cosmos DB para bases de datos de documentos, Azure Table y Cassandra. Actualmente, debe crear una cuenta independiente para cada API. <br><br>Seleccione **Azure Cosmos DB para la API de Mongo DB**, ya que en este inicio rápido va a crear una colección que funciona con MongoDB.<br><br>[Obtenga más información sobre Azure Cosmos DB para la API de Mongo DB](../articles/cosmos-db/mongodb-introduction.md).|
   |Location|Región más cercana a los usuarios|Seleccione una ubicación geográfica para hospedar la cuenta de Azure Cosmos DB. Use la ubicación más cercana a los usuarios para que puedan acceder de la forma más rápida posible a los datos.|
   |Capacity mode (Modo de capacidad)|Rendimiento aprovisionado o sin servidor|Seleccione **Provisioned throughput** (Rendimiento aprovisionado) para crear una cuenta en modo de [rendimiento aprovisionado](../articles/cosmos-db/set-throughput.md). Seleccione **Serverless** (Sin servidor) para crear una cuenta en modo [sin servidor](../articles/cosmos-db/serverless.md). .<br><br>**Nota**: Solo se admite la versión 3.6 de MongoDB API en las cuentas sin servidor. Si elige la versión 3.2, se forzará la cuenta al modo de rendimiento aprovisionado.|
   |Aplicar el descuento del nivel Gratis de Azure Cosmos DB|**Aplicar** o **No aplicar**|Con el nivel Gratis de Azure Cosmos DB, recibirá los primeros 1000 RU/s y 25 GB de almacenamiento gratis en una cuenta. Más información acerca del [nivel Gratis](https://azure.microsoft.com/pricing/details/cosmos-db/).|
   | Versión | Elección de la versión de servidor necesaria | La API de Azure Cosmos DB para MongoDB es compatible con la versión 4.0, 3.6 y 3.2 del servidor. Puede [actualizar o degradar](../articles/cosmos-db/mongodb-version-upgrade.md) una cuenta después de crearla. |

   > [!NOTE]
   > Puede tener una cuenta de Azure Cosmos DB de nivel Gratis por cada suscripción de Azure y debe optar por tenerla al crear la cuenta. Si no ve la opción para aplicar el descuento por nivel Gratis, significará que en otra cuenta de la suscripción ya se ha habilitado el nivel Gratis.

   :::image type="content" source="./media/cosmos-db-create-dbaccount-mongodb/azure-cosmos-db-create-new-account.png" alt-text="Página de la nueva cuenta de Azure Cosmos DB"::: 

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

4. La cuenta tarda unos minutos en crearse. Espere a que el portal muestre la página **¡Enhorabuena! Su cuenta de Azure Cosmos DB para la API de Mongo DB está a punto**.

   :::image type="content" source="./media/cosmos-db-create-dbaccount-mongodb/azure-cosmos-db-account-created.png" alt-text="Panel de notificaciones de Azure Portal"::: 