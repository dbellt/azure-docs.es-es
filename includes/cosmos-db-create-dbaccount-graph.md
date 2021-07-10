---
title: archivo de inclusión
description: archivo de inclusión
services: cosmos-db
author: SnehaGunda
ms.service: cosmos-db
ms.topic: include
ms.date: 01/27/2020
ms.author: sngun
ms.custom: include file
ms.openlocfilehash: 4848fcae37c256089718e198d954a99c1db85def
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110486903"
---
1. En una nueva ventana del explorador, inicie sesión en [Azure Portal](https://portal.azure.com/).

2. En el menú de la izquierda, seleccione **Crear un recurso**.
   
   :::image type="content" source="./media/cosmos-db-create-dbaccount-graph/create-nosql-db-databases-json-tutorial-0.png" alt-text="Crear un recurso en Azure Portal":::   

3. En la página **Nuevo**, seleccione **Bases de datos** > **Azure Cosmos DB**.

   :::image type="content" source="./media/cosmos-db-create-dbaccount-graph/create-nosql-db-databases-json-tutorial-1.png" alt-text="El panel Bases de datos de Azure Portal":::      

3. En la página **Crear una cuenta de Azure Cosmos DB**, especifique la configuración de la nueva cuenta de Azure Cosmos DB. 
 
   |Configuración|Valor|Descripción |
   |---|---|---|
   |Suscripción|Nombre de suscripción|Seleccione la suscripción de Azure que desea usar para esta cuenta de Azure Cosmos. |
   |Grupo de recursos|Definición de un nombre de grupo de recursos|Seleccione un grupo de recursos o seleccione **Crear nuevo** y escriba un nombre único para el grupo de recursos nuevo. |
   |Nombre de cuenta|Escriba un nombre único.|Escriba un nombre único para identificar la cuenta de Azure Cosmos DB. El URI de la cuenta será *gremlin.azure.com* y se anexará al nombre único de la cuenta.<br><br>El nombre de la cuenta solo puede utilizar letras minúsculas, números y guiones (-), y debe tener entre 3 y 31 caracteres de longitud.|
   API|Gremlin (grafo)|La API determina el tipo de cuenta que se va a crear. Azure Cosmos DB proporciona cinco API: Core (SQL) para bases de datos de documentos, Gremlin para bases de datos de grafos, MongoDB para bases de datos de documentos, Azure Table y Cassandra. Debe crear una cuenta independiente para cada API. <br><br>Seleccione **Gremlin (grafo)**, ya que en este inicio rápido va a crear una tabla que funciona con Gremlin API. <br><br>[Más información sobre Gremlin API](../articles/cosmos-db/graph-introduction.md).|
   |Location|Región más cercana a los usuarios|Seleccione una ubicación geográfica para hospedar la cuenta de Azure Cosmos DB. Use la ubicación más cercana a los usuarios para que puedan acceder de la forma más rápida posible a los datos.|
   |Capacity mode (Modo de capacidad)|Rendimiento aprovisionado o sin servidor|Seleccione **Provisioned throughput** (Rendimiento aprovisionado) para crear una cuenta en modo de [rendimiento aprovisionado](../articles/cosmos-db/set-throughput.md). Seleccione **Serverless** (Sin servidor) para crear una cuenta en modo [sin servidor](../articles/cosmos-db/serverless.md).|
   |Aplicar el descuento del nivel Gratis de Azure Cosmos DB|**Aplicar** o **No aplicar**|Con el nivel Gratis de Azure Cosmos DB, recibirá los primeros 1000 RU/s y 25 GB de almacenamiento gratis en una cuenta. Más información acerca del [nivel Gratis](https://azure.microsoft.com/pricing/details/cosmos-db/).|

   > [!NOTE]
   > Puede tener una cuenta de Azure Cosmos DB de nivel Gratis por cada suscripción de Azure y debe optar por tenerla al crear la cuenta. Si no ve la opción para aplicar el descuento por nivel Gratis, significará que en otra cuenta de la suscripción ya se ha habilitado el nivel Gratis.

   :::image type="content" source="./media/cosmos-db-create-dbaccount-graph/azure-cosmos-db-create-new-account.png" alt-text="Página de la nueva cuenta de Azure Cosmos DB":::   

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

1. La cuenta tarda unos minutos en crearse. Espere a que el portal muestre la página **¡Enhorabuena! Se ha creado su cuenta de Azure Cosmos DB**.

   :::image type="content" source="./media/cosmos-db-create-dbaccount-graph/azure-cosmos-db-graph-created.png" alt-text="Página creada de la cuenta de Azure Cosmos DB":::   