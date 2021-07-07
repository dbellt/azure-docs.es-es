---
title: 'Tutorial: Migración de MongoDB en línea a la API de Azure Cosmos DB para MongoDB'
titleSuffix: Azure Database Migration Service
description: Aprenda a migrar desde el entorno local de MongoDB a la API de Azure Cosmos DB para MongoDB en línea mediante Azure Database Migration Service.
services: dms
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-nov-2020
ms.topic: tutorial
ms.date: 05/19/2021
ms.openlocfilehash: b8ffe57b2244ff3f9e7df94665d7801d3231d9aa
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110467645"
---
# <a name="tutorial-migrate-mongodb-to-azure-cosmos-dbs-api-for-mongodb-online-using-dms"></a>Tutorial: Migración de MongoDB a la API de Azure Cosmos DB para MongoDB en línea mediante DMS
[!INCLUDE[appliesto-mongodb-api](../cosmos-db/includes/appliesto-mongodb-api.md)]

> [!IMPORTANT]  
> Lea toda esta guía antes de llevar a cabo los pasos de migración.
>

Esta guía de migración de MongoDB forma parte de la serie sobre la migración de MongoDB. Los pasos críticos de la migración de MongoDB son [la premigración](../cosmos-db/mongodb-pre-migration.md), la migración y la [posmigración](../cosmos-db/mongodb-post-migration.md), como se muestra a continuación.

![Diagrama de los pasos de migración.](../cosmos-db/media/mongodb-pre-migration/overall-migration-steps.png)

## <a name="overview-of-online-data-migration-from-mongodb-to-azure-cosmos-db-using-dms"></a>Introducción a la migración de datos en línea de MongoDB a Azure Cosmos DB mediante DMS

Puede usar Azure Database Migration Service para realizar una migración en línea (tiempo de inactividad mínimo) de las bases de datos desde una instancia local o en la nube de MongoDB a la API de Azure Cosmos DB para MongoDB.

En este tutorial se muestran los pasos asociados al uso de Azure Database Migration Service para migrar datos de MongoDB a Azure Cosmos DB:
> [!div class="checklist"]
> 
> * Crear una instancia de Azure Database Migration Service. 
> * Cree un proyecto de migración. 
> * Especifique el origen. 
> * Especifique el destino. 
> * Realice la asignación a las bases de datos de destino. 
> * Ejecutar la migración. 
> * Supervisar la migración 
> * Compruebe los datos en Azure Cosmos DB. 
> * Completar la migración cuando esté listo. 

En este tutorial, migrará un conjunto de datos de MongoDB hospedado en una máquina virtual de Azure a la API de Azure Cosmos DB para MongoDB con un tiempo de inactividad mínimo mediante Azure Database Migration Service. Si no tiene un origen de MongoDB ya configurado, vea el artículo [Instalación y configuración de MongoDB en una máquina virtual Windows en Azure](/previous-versions/azure/virtual-machines/windows/install-mongodb).

> [!NOTE]
> El uso de Azure Database Migration Service para realizar una migración en línea requiere la creación de una instancia basada en el plan de tarifa Premium.

> [!IMPORTANT]
> Para disfrutar de una experiencia de migración óptima, Microsoft recomienda crear una instancia de Azure Database Migration Service en la misma región de Azure que la base de datos de destino. Si los datos se transfieren entre diferentes regiones o ubicaciones geográficas, el proceso de migración puede verse afectado.

[!INCLUDE [online-offline](../../includes/database-migration-service-offline-online.md)]

En este artículo se describe una migración en línea desde MongoDB a la API de Azure Cosmos DB para MongoDB. Para realizar una migración sin conexión, consulte [Migración de MongoDB a la API de Azure Cosmos DB para MongoDB sin conexión mediante DMS](tutorial-mongodb-cosmos-db.md).

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, necesita:

* [Realizar los pasos previos a la migración](../cosmos-db/mongodb-pre-migration.md) como son estimar el rendimiento, elegir una clave de partición y seleccionar la directiva de indexación.
* [Cree una cuenta de la API de Azure Cosmos DB para MongoDB](https://ms.portal.azure.com/#create/Microsoft.DocumentDB) y asegúrese de que esté habilitado[SSR (reintento en el servidor)](../cosmos-db/prevent-rate-limiting-errors.md).
* Crear una instancia de Microsoft Azure Virtual Network para Azure Database Migration Service con el modelo de implementación de Azure Resource Manager, que proporciona conectividad de sitio a sitio a los servidores de origen local mediante [ExpressRoute](../expressroute/expressroute-introduction.md) o [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md).

    > [!NOTE]
    > Durante la configuración de la red virtual, si usa ExpressRoute con emparejamiento de red a Microsoft, agregue los siguientes [puntos de conexión](../virtual-network/virtual-network-service-endpoints-overview.md) de servicio a la subred en la que se aprovisionará el servicio:
    >
    > * Punto de conexión de base de datos de destino (por ejemplo, punto de conexión de SQL, punto de conexión de Cosmos DB, etc.)
    > * Punto de conexión de Storage
    > * Punto de conexión de Service Bus
    >
    > Esta configuración es necesaria porque Azure Database Migration Service no tiene conexión a Internet.

* Asegúrese de que las reglas del grupo de seguridad de red (NSG) de la red virtual no bloquean los siguientes puertos de comunicación: 53, 443, 445, 9354 y 10000-20000. Para más información sobre el filtrado del tráfico con grupos de seguridad de red para redes virtuales, vea el artículo [Filtrado del tráfico de red con grupos de seguridad de red](../virtual-network/virtual-network-vnet-plan-design-arm.md).
* Abra el Firewall de Windows para permitir que Azure Database Migration Service acceda al servidor de MongoDB de origen que, de manera predeterminada, es el puerto TCP 27017.
* Cuando se usa un dispositivo de firewall frente a las bases de datos de origen, puede que sea necesario agregar reglas de firewall para permitir que Azure Database Migration Service acceda a las bases de datos de origen para realizar la migración.

## <a name="configure-azure-cosmos-db-server-side-retries-for-efficient-migration"></a>Configuración de reintentos en el lado servidor de Azure Cosmos DB para una migración eficaz

Los clientes que migren de MongoDB a Azure Cosmos DB se benefician de la gobernanza de recursos, lo que garantiza que se puedan aprovechar al máximo las RU/s de rendimiento aprovisionadas. Azure Cosmos DB puede limitar una solicitud determinada de Data Migration Service en el transcurso de la migración si esa solicitud supera las RU/s aprovisionadas del contenedor; en ese caso, esa solicitud debe reintentarse. Aunque Data Migration Service es capaz de realizar reintentos, el tiempo de ida y vuelta implicado en el salto de red entre este servicio y Azure Cosmos DB afecta al tiempo de respuesta general de la solicitud. Mejorar el tiempo de respuesta de las solicitudes limitadas puede acortar el tiempo total necesario para la migración. La característica *Reintento en el lado servidor* de Azure Cosmos DB permite al servicio interceptar los códigos de error de limitación y volver a intentar la operación con un tiempo de ida y vuelta mucho menor, lo que mejora enormemente los tiempos de respuesta de las solicitudes.

Puede encontrar la funcionalidad Reintento en el lado servidor en la hoja *Características* del portal de Azure Cosmos DB.

![Captura de pantalla de la característica Reintento en el lado servidor de MongoDB.](media/tutorial-mongodb-to-cosmosdb-online/mongo-server-side-retry-feature.png)

Y, si está *deshabilitada*, se recomienda habilitarla como se muestra a continuación.

![Captura de pantalla de habilitación de Reintento en el lado servidor de MongoDB.](media/tutorial-mongodb-to-cosmosdb-online/mongo-server-side-retry-enable.png)

## <a name="register-the-microsoftdatamigration-resource-provider"></a>Registro del proveedor de recursos Microsoft.DataMigration

1. Inicie sesión en Azure Portal, seleccione **Todos los servicios** y, después, **Suscripciones**.

   ![Mostrar suscripciones en el portal](media/tutorial-mongodb-to-cosmosdb-online/portal-select-subscription1.png)

2. Seleccione la suscripción en la que quiere crear la instancia de Azure Database Migration Service y después seleccione **Proveedores de recursos**.

    ![Mostrar los proveedores de recursos](media/tutorial-mongodb-to-cosmosdb-online/portal-select-resource-provider.png)

3. Busque la migración y después, a la derecha de **Microsoft.DataMigration**, seleccione **Registrar**.

    ![Registro del proveedor de recursos](media/tutorial-mongodb-to-cosmosdb-online/portal-register-resource-provider.png)    

## <a name="create-an-instance"></a>Creación de una instancia

1. En Azure Portal, seleccione + **Crear un recurso**, busque Azure Database Migration Service y, después, seleccione **Azure Database Migration Service** en la lista desplegable.

    ![Azure Marketplace](media/tutorial-mongodb-to-cosmosdb-online/portal-marketplace.png)

2. En la pantalla **Azure Database Migration Service**, seleccione **Crear**.

    ![Creación de una instancia de Azure Database Migration Service](media/tutorial-mongodb-to-cosmosdb-online/dms-create1.png)
  
3. En la pantalla **Crear el servicio de migración**, especifique un nombre para el servicio, la suscripción y un grupo de recursos nuevo o existente.

4. Seleccione la ubicación en la que quiere crear la instancia de Azure Database Migration Service.

5. Seleccione una red virtual existente o cree una nueva.

   La red virtual proporciona a Azure Database Migration Service acceso a la instancia de MongoDB de origen y a la cuenta de Azure Cosmos DB de destino.

   Para más información sobre cómo crear una red virtual en Azure Portal, consulte el artículo [Creación de una red virtual con Azure Portal](../virtual-network/quick-create-portal.md).

6. Seleccione una SKU del plan de tarifa Premium.

    > [!NOTE]
    > Las migraciones en línea solo se admiten cuando se usa el nivel Premium. Para más información sobre los costos y planes de tarifa, vea la [página de precios](https://aka.ms/dms-pricing).

    ![Configuración de la instancia de Azure Database Migration Service](media/tutorial-mongodb-to-cosmosdb-online/dms-settings3.png)

7. Seleccione **Crear** para crear el servicio.

## <a name="create-a-migration-project"></a>Creación de un proyecto de migración

Después de crear el servicio, búsquelo en Azure Portal, ábralo y cree un proyecto de migración.

1. En Azure Portal, seleccione **Todos los servicios**, busque Azure Database Migration Service y, luego, elija **Azure Database Migration Services**.

    ![Búsqueda de todas las instancias de Azure Database Migration Service](media/tutorial-mongodb-to-cosmosdb-online/dms-search.png)

2. En la pantalla **Azure Database Migration Services**, busque el nombre de la instancia de Azure Database Migration Service que creó y, después, seleccione la instancia.

    Como alternativa, puede detectar la instancia de Azure Database Migration Service desde el panel de búsqueda de Azure Portal.

    ![Usar el panel de búsqueda de Azure Portal](media/tutorial-mongodb-to-cosmosdb-online/dms-search-portal.png)

3. Seleccione **+ New Migration Project** (+ Nuevo proyecto de migración).

4. En la pantalla **Nuevo proyecto de migración**, especifique un nombre para el proyecto. En el cuadro de texto **Source server type** (Tipo de servidor de origen), seleccione **MongoDB**, en el cuadro de texto **Target server type** (Tipo de servidor de destino), seleccione **CosmosDB (API de MongoDB)** y, por último, en **Elegir el tipo de actividad**, seleccione **Migración de datos en línea (versión preliminar)** .

    ![Creación de un proyecto de Database Migration Service](media/tutorial-mongodb-to-cosmosdb-online/dms-create-project1.png)

5. Seleccione **Guardar** y, después, **Crear y ejecutar una actividad** para crear el proyecto y ejecutar la actividad de migración.

## <a name="specify-source-details"></a>Especificación de los detalles de origen

1. En la pantalla **Detalles del origen**, especifique los detalles de conexión del servidor de MongoDB de origen.

   > [!IMPORTANT]
   > Azure Database Migration Service no admite Azure Cosmos DB como origen.

    Hay tres modos de conexión a un origen:
   * **Modo estándar**, que acepta un nombre de dominio completo o una dirección IP, número de puerto y las credenciales de conexión.
   * **Modo de cadena de conexión**, que acepta una cadena de conexión de MongoDB, como se describe en el artículo [Connection String URI Format](https://docs.mongodb.com/manual/reference/connection-string/) (Formato de identificador URI de cadena de conexión).
   * **Datos de Azure Storage**, que acepta un dirección URL de SAS del contenedor de blobs. Seleccione **El blob contiene volcados BSON** si el contenedor de blobs tiene volcados BSON producidos por la [herramienta bsondump](https://docs.mongodb.com/manual/reference/program/bsondump/) de MongoDB y anule su selección si el contenedor contiene archivos JSON.

     Si selecciona esta opción, asegúrese de que la cadena de conexión de la cuenta de almacenamiento aparece en el formato:

     ```
     https://blobnameurl/container?SASKEY
     ```

     Además, según la información del tipo de volcado de memoria de Azure Storage, tenga en cuenta el siguiente detalle.

     * En el caso de los volcados de BSON, los datos del contenedor de blobs deben estar en formato bsondump, con el fin de que dichos archivos se coloquen en carpetas que se llamen igual que las bases de datos que contienen en el formato colección.bson. A los archivos de metadatos (si hubiera) se les deben asignar el nombre con el formato *colección*.metadata.json.

     * En el caso de los volcados de JSON, los archivos del contenedor de blobs deben colocarse en carpetas que se llamen igual que las bases de datos que contienen. Dentro de cada una de estas carpetas, los archivos de datos se deben colocar en una subcarpeta denominada "data" y se le debe asignar el nombre con el formato *colección*.json. Los archivos de metadatos (si hubiera) se deben colocar en una subcarpeta denominada "metadata" y se le debe asignar el nombre con el mismo formato, *colección*.json. Los archivos de metadatos deben estar en el mismo formato que los que genera la herramienta bsondump de MongoDB.

    > [!IMPORTANT]
    > No se recomienda usar un certificado autofirmado en el servidor de Mongo. Sin embargo, si lo usa, conecte con el servidor con el **modo de cadena de conexión** y asegúrese de que la cadena de conexión es "".
    >
    >```
    >&sslVerifyCertificate=false
    >```

    La dirección IP se puede usar en situaciones en las que no es posible la resolución de nombres de DNS.

   ![Especificación de los detalles de origen](media/tutorial-mongodb-to-cosmosdb-online/dms-specify-source1.png)

2. Seleccione **Guardar**.

   > [!NOTE]
   > La dirección del servidor de origen debe ser la dirección del principal si el origen es un conjunto de réplicas y el enrutador si el origen es un clúster con particiones de MongoDB. En el caso de un clúster con particiones de MongoDB, Azure Database Migration Service debe ser capaz de conectarse a las particiones individuales en el clúster, lo que puede requerir que el firewall se abra en varias máquinas.

## <a name="specify-target-details"></a>Especificación de los detalles de destino

1. En la pantalla **Detalles del destino de la migración**, especifique los detalles de conexión de la cuenta de Azure Cosmos DB de destino, que es la cuenta de la API de Azure Cosmos DB para MongoDB aprovisionada previamente a la que se van a migrar los datos de MongoDB.

    ![Especificación de los detalles de destino](media/tutorial-mongodb-to-cosmosdb-online/dms-specify-target1.png)

2. Seleccione **Guardar**.

## <a name="map-to-target-databases"></a>Asignación a bases de datos de destino

1. En la pantalla **Map to target databases** (Asignar a bases de datos de destino), asigne la base de datos de origen y de destino para la migración.

   Si la base de datos de destino contiene el mismo nombre de base de datos que la de origen, Azure Database Migration Service selecciona la base de datos de destino de forma predeterminada.

   Si la cadena **Crear** aparece junto al nombre de la base de datos, indica que Azure Database Migration Service no encontró la base de datos de destino, y el servicio creará la base de datos automáticamente.

   En este momento de la migración, si se desea capacidad de proceso para uso compartido en la base de datos, especifique las unidades de solicitud de capacidad de proceso. En Cosmos DB, puede aprovisionar rendimiento a nivel de base de datos o individualmente para cada colección. El rendimiento se mide en [unidades de solicitud](../cosmos-db/request-units.md) (RU). Obtenga más información sobre los [precios de Azure Cosmos DB](https://azure.microsoft.com/pricing/details/cosmos-db/).

   ![Asignación a bases de datos de destino](media/tutorial-mongodb-to-cosmosdb-online/dms-map-target-databases1.png)

2. Seleccione **Guardar**.

3. En la pantalla **Configuración de colecciones**, expanda la lista de colecciones y, a continuación, revise la lista de las colecciones que se van a migrar.

   Azure Database Migration Service selecciona automáticamente todas las colecciones existentes en la instancia de MongoDB de origen que no existen en la cuenta de Azure Cosmos DB de destino. Si desea volver a migrar las colecciones que ya incluyen datos, necesita seleccionar explícitamente las colecciones de esta pantalla.

   Puede especificar el número de unidades de solicitud que desea que usen las colecciones. En la mayoría de los casos, debería ser suficiente un valor entre 500 (1000 es el mínimo para las colecciones con particiones) y 4000. Azure Database Migration Service sugiere valores predeterminados inteligentes en función del tamaño de la colección.

    > [!NOTE]
    > Realice la migración de base de datos y la colección en paralelo mediante múltiples instancias de Azure Database Migration Service, si es necesario, para acelerar la ejecución.

   También puede especificar una clave de partición para aprovechar las ventajas de la [creación de particiones en Azure Cosmos DB](../cosmos-db/partitioning-overview.md) para una escalabilidad óptima. No olvide revisar los [procedimientos recomendados para seleccionar una clave de partición](../cosmos-db/partitioning-overview.md#choose-partitionkey). Si no tiene una clave de partición, siempre puede usar **_id** como clave de partición para mejorar el rendimiento.

   ![Selección de tablas de colecciones](media/tutorial-mongodb-to-cosmosdb-online/dms-collection-setting1.png)

4. Seleccione **Guardar**.

5. En la pantalla **Migration summary** (Resumen de migración), en el cuadro de texto **Nombre de actividad**, especifique un nombre para la actividad de migración.

    ![Resumen de migración](media/tutorial-mongodb-to-cosmosdb-online/dms-migration-summary1.png)

## <a name="run-the-migration"></a>Ejecución de la migración

* Seleccione **Ejecutar migración**.

   Aparecerá la ventana de actividad de migración y se mostrará el **estado** de la actividad.

   ![Estado de la actividad](media/tutorial-mongodb-to-cosmosdb-online/dms-activity-status1.png)

## <a name="monitor-the-migration"></a>Supervisión de la migración

* En la pantalla de la actividad de migración, seleccione **Actualizar** para actualizar la vista hasta que el valor de **Estado** de la migración sea **Reproduciendo**.

   > [!NOTE]
   > Puede seleccionar la actividad para obtener los detalles de las métricas de migración a nivel de base de datos y de colección.

   ![Reproducción de estado de actividad](media/tutorial-mongodb-to-cosmosdb-online/dms-activity-replaying.png)

## <a name="verify-data-in-cosmos-db"></a>Verificación de datos en Cosmos DB

1. Realice cambios en la base de datos de MongoDB de origen.
2. Conéctese a COSMOS DB para comprobar si los datos se replican desde el servidor de MongoDB de origen.

    ![Captura de pantalla que muestra dónde puede comprobar que los datos se han replicado.](media/tutorial-mongodb-to-cosmosdb-online/dms-verify-data.png)

## <a name="complete-the-migration"></a>Completar la migración

* Después de que todos los documentos del origen estén disponibles en el destino de COSMOS DB, seleccione **Finalizar** en el menú contextual de la actividad de migración para completar la migración.

    Esta acción finalizará la reproducción de todos los cambios pendientes y completará la migración.

    ![Captura de pantalla que muestra la opción de menú Finalizar.](media/tutorial-mongodb-to-cosmosdb-online/dms-finish-migration.png)

## <a name="post-migration-optimization"></a>Optimización posterior a la migración

Después de migrar los datos almacenados en la base de datos de MongoDB a la API de Azure Cosmos DB para MongoDB, puede conectarse a Azure Cosmos DB y administrar los datos. También puede realizar otros pasos de optimización posteriores a la migración, como optimizar la directiva de indexación, actualizar el nivel de coherencia predeterminado o configurar la distribución global de la cuenta de Azure Cosmos DB. Para más información, consulte el artículo [Optimización posterior a la migración](../cosmos-db/mongodb-post-migration.md).

## <a name="additional-resources"></a>Recursos adicionales

* [Información sobre el servicio Cosmos DB](https://azure.microsoft.com/services/cosmos-db/)

## <a name="next-steps"></a>Pasos siguientes

* Revise las directrices de migración en la [Guía de migración de bases de datos](https://datamigration.microsoft.com/) para consultar escenarios adicionales.