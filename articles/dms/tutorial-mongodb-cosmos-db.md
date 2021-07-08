---
title: 'Tutorial: Migración de MongoDB sin conexión a la API de Azure Cosmos DB para MongoDB'
titleSuffix: Azure Database Migration Service
description: Utilice Azure Database Migration Service para realizar migraciones desde el entorno local de MongoDB a Azure Cosmos DB API para MongoDB sin conexión.
services: dms
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: tutorial
ms.date: 05/19/2021
ms.openlocfilehash: 31f3dc8adcd17c7795351877c856fa854582007b
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110456579"
---
# <a name="tutorial-migrate-mongodb-to-azure-cosmos-db-api-for-mongodb-offline"></a>Tutorial: Migración de MongoDB a Azure Cosmos DB API para MongoDB sin conexión
[!INCLUDE[appliesto-mongodb-api](../cosmos-db/includes/appliesto-mongodb-api.md)]

> [!IMPORTANT]  
> Lea toda esta guía antes de llevar a cabo los pasos de migración.
>

Esta guía de migración de MongoDB forma parte de la serie sobre la migración de MongoDB. Los pasos críticos de la migración de MongoDB son [la premigración](../cosmos-db/mongodb-pre-migration.md), la migración y la [posmigración](../cosmos-db/mongodb-post-migration.md), como se muestra a continuación.

![Diagrama de los pasos de migración.](../cosmos-db/media/mongodb-pre-migration/overall-migration-steps.png)

## <a name="overview-of-offline-data-migration-from-mongodb-to-azure-cosmos-db-using-dms"></a>Introducción a la migración de datos sin conexión de MongoDB a Azure Cosmos DB mediante DMS

Use Azure Database Migration Service para realizar una migración de bases de datos única y sin conexión desde una instancia local o en la nube de MongoDB a Azure Cosmos DB API para MongoDB.

En este tutorial, aprenderá a:
> [!div class="checklist"]
>
> * Crear una instancia de Azure Database Migration Service.
> * Crear un proyecto de migración mediante Azure Database Migration Service.
> * Ejecutar la migración.
> * Supervisar la migración

En este tutorial, migrará un conjunto de datos en MongoDB que se hospeda en una máquina virtual de Azure. Gracias a Azure Database Migration Service, se puede migrar el conjunto de datos a Azure Cosmos DB API para MongoDB. Si aún no tiene un origen de MongoDB configurado, consulte [Instalación y configuración de MongoDB en una máquina virtual Windows en Azure](/previous-versions/azure/virtual-machines/windows/install-mongodb).

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, necesita:

* [Realizar los pasos previos a la migración](../cosmos-db/mongodb-pre-migration.md), a saber, calcular el rendimiento y elegir una clave de partición.
* [Crear una cuenta para Azure Cosmos DB API para MongoDB](https://ms.portal.azure.com/#create/Microsoft.DocumentDB).
* Crear una instancia de Microsoft Azure Virtual Network para Azure Database Migration Service mediante Azure Resource Manager. Este modelo de implementación proporciona conectividad entre sitios con sus servidores de origen local mediante [Azure ExpressRoute](../expressroute/expressroute-introduction.md) o [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md). Para más información sobre la creación de una red virtual, consulte la [documentación de Azure Virtual Network](../virtual-network/index.yml)y, especialmente, los artículos de "inicio rápido" con detalles paso a paso.

    > [!NOTE]
    > Durante la configuración de la red virtual, si usa ExpressRoute con emparejamiento de red a Microsoft, agregue los siguientes [puntos de conexión](../virtual-network/virtual-network-service-endpoints-overview.md) de servicio a la subred en la que se aprovisionará el servicio:
    >
    > * Punto de conexión de base de datos de destino (por ejemplo, punto de conexión de SQL o punto de conexión de Azure Cosmos DB)
    > * Punto de conexión de Storage
    > * Punto de conexión de Service Bus
    >
    > Esta configuración es necesaria porque Azure Database Migration Service no tiene conexión a Internet.

* Asegúrese de que las reglas del grupo de seguridad de red de la red virtual no bloquean los siguientes puertos de comunicación: 53, 443, 445, 9354 y 10000-20000. Para más información, consulte [Filtrado del tráfico de red con grupos de seguridad de red](../virtual-network/virtual-network-vnet-plan-design-arm.md).
* Abra el Firewall de Windows para permitir que Azure Database Migration Service acceda al servidor de MongoDB de origen que, de manera predeterminada, es el puerto TCP 27017.
* Cuando se usa un dispositivo de firewall delante de la base de datos de origen, es posible que sea necesario agregar reglas de firewall para que Azure Database Migration Service pueda acceder a la base de datos de origen para realizar la migración.

## <a name="configure-the-server-side-retry-feature"></a>Configuración de la característica de reintento en el servidor

Si migra desde MongoDB a Azure Cosmos DB, puede beneficiarse de las funcionalidades de gobernanza de recursos. Con estas funcionalidades, puede hacer un uso completo de las unidades de solicitud aprovisionadas (RU/s) del rendimiento. Azure Cosmos DB puede limitar una solicitud determinada de Data Migration Service en el transcurso de la migración si esa solicitud supera las unidades de solicitud aprovisionadas del contenedor. Si eso sucediera, la solicitud se debe reintentar.

Database Migration Service puede realizar reintentos. Es importante saber que el tiempo de ida y vuelta implicado en el salto de red entre Database Migration Service y Azure Cosmos DB afecta al tiempo de respuesta general de la solicitud. Mejorar el tiempo de respuesta de las solicitudes limitadas puede acortar el tiempo total necesario para la migración.

La característica de reintento en el lado servidor de Azure Cosmos DB permite al servicio interceptar los códigos de error de limitación y volver a intentar la operación con un tiempo de ida y vuelta mucho menor, lo que mejora considerablemente los tiempos de respuesta de las solicitudes.

Para usar esta característica, en el portal de Azure Cosmos DB, seleccione **Features** > **Server Side Retry** (Características > Reintento en el servidor).

![Captura de pantalla que muestra dónde encontrar la característica de reintento en el servidor.](media/tutorial-mongodb-to-cosmosdb/mongo-server-side-retry-feature.png)

Si la característica está deshabilitada, seleccione **Habilitar**.

![Captura de pantalla que muestra cómo se habilita el reintento en el servidor.](media/tutorial-mongodb-to-cosmosdb/mongo-server-side-retry-enable.png)

## <a name="register-the-resource-provider"></a>Registrar el proveedor de recursos

1. Inicie sesión en Azure Portal, seleccione **Todos los servicios** y, después, **Suscripciones**.

   ![Captura de pantalla que muestra las suscripciones del portal.](media/tutorial-mongodb-to-cosmosdb/portal-select-subscription1.png)

2. Seleccione la suscripción en la que quiere crear la instancia de Azure Database Migration Service y después seleccione **Proveedores de recursos**.

    ![Captura de pantalla que muestra proveedores de recursos.](media/tutorial-mongodb-to-cosmosdb/portal-select-resource-provider.png)

3. Busque la migración y después, a la derecha de **Microsoft.DataMigration**, seleccione **Registrar**.

    ![Captura de pantalla que muestra cómo se registra el proveedor de recursos.](media/tutorial-mongodb-to-cosmosdb/portal-register-resource-provider.png)    

## <a name="create-an-instance"></a>Creación de una instancia

1. En Azure Portal, seleccione + **Crear un recurso**, busque Azure Database Migration Service y, después, seleccione **Azure Database Migration Service** en la lista desplegable.

    ![Captura de pantalla en la que se muestra Azure Marketplace.](media/tutorial-mongodb-to-cosmosdb/portal-marketplace.png)

2. En la pantalla **Azure Database Migration Service**, seleccione **Crear**.

    ![Captura de pantalla que muestra cómo crear una instancia de Azure Database Migration Service.](media/tutorial-mongodb-to-cosmosdb/dms-create1.png)
  
3. En **Crear el servicio de migración**, especifique un nombre para el servicio, la suscripción y un grupo de recursos nuevo o existente.

4. Seleccione la ubicación en la que quiere crear la instancia de Azure Database Migration Service. 

5. Seleccione una red virtual existente o cree una nueva.

    La red virtual proporciona a Azure Database Migration Service acceso a la instancia de MongoDB de origen y a la cuenta de Azure Cosmos DB de destino.

    Para más información sobre cómo crear una red virtual en Azure Portal, consulte el artículo sobre la [creación de una red mediante Azure Portal](../virtual-network/quick-create-portal.md).

6. Seleccione un plan de tarifa.

    Para más información sobre los costos y planes de tarifa, vea la [página de precios](https://aka.ms/dms-pricing).

    ![Captura de pantalla que muestra los valores de configuración de la instancia de Azure Database Migration Service.](media/tutorial-mongodb-to-cosmosdb/dms-settings2.png)

7. Seleccione **Crear** para crear el servicio.

## <a name="create-a-migration-project"></a>Creación de un proyecto de migración

Después de crear el servicio, búsquelo en Azure Portal y ábralo. Luego, cree un proyecto de migración.

1. En Azure Portal, seleccione **Todos los servicios**, busque Azure Database Migration Service y, luego, elija **Azure Database Migration Services**.

      ![Captura de pantalla que muestra cómo buscar todas las instancias de Azure Database Migration Service.](media/tutorial-mongodb-to-cosmosdb/dms-search.png)

2. En la pantalla **Azure Database Migration Services**, busque el nombre de la instancia de Azure Database Migration Service que creó y, después, seleccione la instancia.

3. Seleccione **+ New Migration Project** (+ Nuevo proyecto de migración).

4. En **Nuevo proyecto de migración**, especifique un nombre para el proyecto y en el cuadro de texto **Tipo de servidor de origen**, seleccione **MongoDB**. En el cuadro de texto **Tipo de servidor de origen**, seleccione **CosmosDB (MongoDB API)** y en **Elegir el tipo de actividad**, seleccione **Migración de datos sin conexión**. 

    ![Captura de pantalla en la que se muestran las opciones del proyecto.](media/tutorial-mongodb-to-cosmosdb/dms-create-project.png)

5. Seleccione **Crear y ejecutar una actividad** para crear el proyecto y ejecutar la actividad de migración.

## <a name="specify-source-details"></a>Especificación de los detalles de origen

1. En la pantalla **Detalles del origen**, especifique los detalles de conexión del servidor de MongoDB de origen.

   > [!IMPORTANT]
   > Azure Database Migration Service no admite Azure Cosmos DB como origen.

    Hay tres modos de conexión a un origen:
   * **Modo estándar**, que acepta un nombre de dominio completo o una dirección IP, número de puerto y las credenciales de conexión.
   * **Modo de cadena de conexión**, que acepta una cadena de conexión de MongoDB, como se describe en el artículo sobre el [formato del identificador URI de la cadena de conexión](https://docs.mongodb.com/manual/reference/connection-string/).
   * **Datos de Azure Storage**, que acepta un dirección URL de SAS del contenedor de blobs. Seleccione **El blob contiene volcados BSON** si el contenedor de blobs tiene volcados BSON generados por la [herramienta bsondump](https://docs.mongodb.com/manual/reference/program/bsondump/) de MongoDB. Si el contenedor tiene archivos JSON, no seleccione esta opción.

     Si selecciona esta opción, asegúrese de que la cadena de conexión de la cuenta de almacenamiento aparece en el siguiente formato:

     ```
     https://blobnameurl/container?SASKEY
     ```

     Esta cadena de conexión de SAS del contenedor de blobs se puede encontrar en el Explorador de Azure Storage. La creación de SAS para el contenedor implicado le proporcionará la dirección URL con el formato solicitado.
     
     Además, según la información del tipo de volcado de memoria de Azure Storage, tenga en cuenta lo siguiente:

     * En el caso de los volcados de BSON, los datos del contenedor de blobs deben estar en formato bsondump. Coloque los archivos de datos en carpetas cuyo nombre coincida con el de las bases de datos contenedoras con el formato *colección.bson*. Asigne un nombre a los archivos de metadatos, y utilice para ello el formato *colección.metadatos.json*.

     * En el caso de los volcados de JSON, los archivos del contenedor de blobs deben colocarse en carpetas que se llamen igual que las bases de datos que contienen. Dentro de cada una de estas carpetas, los archivos de datos se deben colocar en una subcarpeta denominada *data* y se debe usar el formato *colección.json* para asignarles el nombre. Coloque los archivos de metadatos en una subcarpeta denominada *metadata* y utilice el mismo formato, *colección.json*. Los archivos de metadatos deben estar en el mismo formato que los que genera la herramienta bsondump de MongoDB.

    > [!IMPORTANT]
    > No se recomienda usar certificados autofirmados en el servidor de MongoDB. Si debe usarlos, utilice el modo de cadena de conexión para conectarse al servidor y asegúrese de que la cadena de conexión aparece entre comillas ("").
    >
    >```
    >&sslVerifyCertificate=false
    >```

   También puede usar la dirección IP en aquellos casos en que no sea posible la resolución de nombres de DNS.

   ![Captura de pantalla que muestra la especificación de los detalles del origen.](media/tutorial-mongodb-to-cosmosdb/dms-specify-source.png)

2. Seleccione **Guardar**.

## <a name="specify-target-details"></a>Especificación de los detalles de destino

1. En la pantalla **Detalles del destino de la migración**, especifique los detalles de conexión de la cuenta de Azure Cosmos DB de destino. Esta cuenta es la de Azure Cosmos DB API para MongoDB aprovisionada previamente a la que va a migrar los datos de MongoDB.

    ![Captura de pantalla en la que se muestra cómo se especifican los detalles de destino.](media/tutorial-mongodb-to-cosmosdb/dms-specify-target.png)

2. Seleccione **Guardar**.

## <a name="map-to-target-databases"></a>Asignación a bases de datos de destino

1. En la pantalla **Map to target databases** (Asignar a bases de datos de destino), asigne la base de datos de origen y de destino para la migración.

    Si la base de datos de destino contiene el mismo nombre de base de datos que la de origen, Azure Database Migration Service selecciona la base de datos de destino de forma predeterminada.

    Si **Create** (Crear) aparece junto al nombre de la base de datos, indica que Azure Database Migration Service no encontró la base de datos de destino, y el servicio creará la base de datos automáticamente.

    En este momento de la migración, puede [aprovisionar rendimiento](../cosmos-db/set-throughput.md). En Azure Cosmos DB, puede aprovisionar rendimiento a nivel de base de datos o individualmente para cada colección. El rendimiento se mide en [unidades de solicitud](../cosmos-db/request-units.md). Obtenga más información sobre los [precios de Azure Cosmos DB](https://azure.microsoft.com/pricing/details/cosmos-db/).

    ![Captura de pantalla que muestra la asignación a bases de datos de destino.](media/tutorial-mongodb-to-cosmosdb/dms-map-target-databases.png)

2. Seleccione **Guardar**.
3. En la pantalla **Configuración de colecciones**, expanda la lista de colecciones y, a continuación, revise la lista de las colecciones que se van a migrar.

    Azure Database Migration Service selecciona automáticamente todas las colecciones existentes en la instancia de MongoDB de origen que no existen en la cuenta de Azure Cosmos DB de destino. Si quiere volver a migrar las colecciones que ya incluyen datos, necesita seleccionar explícitamente las colecciones de este panel.

    Puede especificar el número de unidades de solicitud que desea que usen las colecciones. Azure Database Migration Service sugiere valores predeterminados inteligentes en función del tamaño de la colección.

    > [!NOTE]
    > Realice la migración de la base de datos y la recopilación en paralelo. Si fuera necesario, puede usar varias instancias de Azure Database Migration Service para acelerar la ejecución.

    También puede especificar una clave de partición para aprovechar las ventajas de la [creación de particiones en Azure Cosmos DB](../cosmos-db/partitioning-overview.md) para una escalabilidad óptima. Examine los [procedimientos recomendados para seleccionar una partición o clave de partición](../cosmos-db/partitioning-overview.md#choose-partitionkey).

    ![Captura de pantalla en la que se muestra cómo se seleccionan tablas de recopilaciones.](media/tutorial-mongodb-to-cosmosdb/dms-collection-setting.png)

4. Seleccione **Guardar**.

5. En la pantalla **Migration summary** (Resumen de migración), en el cuadro de texto **Nombre de actividad**, especifique un nombre para la actividad de migración.

    ![Captura de pantalla que muestra el resumen de la migración.](media/tutorial-mongodb-to-cosmosdb/dms-migration-summary.png)

## <a name="run-the-migration"></a>Ejecución de la migración

Seleccione **Ejecutar migración**. Aparece la ventana de actividad de la migración y el estado de la actividad será **Sin iniciar**.

![Captura de pantalla que muestra el estado de la actividad.](media/tutorial-mongodb-to-cosmosdb/dms-activity-status.png)

## <a name="monitor-the-migration"></a>Supervisión de la migración

En la pantalla de la actividad de migración, seleccione **Actualizar** para actualizar la pantalla hasta que el estado de la migración sea **Completado**.

> [!NOTE]
> Puede seleccionar la actividad para obtener los detalles de las métricas de migración a nivel de base de datos y de colección.

![Captura de pantalla que muestra el estado de actividad Completado.](media/tutorial-mongodb-to-cosmosdb/dms-activity-completed.png)

## <a name="verify-data-in-azure-cosmos-db"></a>Comprobación de los datos en Azure Cosmos DB

Después de finalizar la migración, podrá consultar la cuenta de Azure Cosmos DB para comprobar que todas las colecciones han migrado correctamente.

![Captura de pantalla que muestra dónde consultar la cuenta de Azure Cosmos DB para comprobar que todas las colecciones se han migrado correctamente.](media/tutorial-mongodb-to-cosmosdb/dms-cosmosdb-data-explorer.png)

## <a name="post-migration-optimization"></a>Optimización posterior a la migración

Después de migrar los datos almacenados en la base de datos de MongoDB a Azure Cosmos DB API para MongoDB, puede conectarse a Azure Cosmos DB y administrar los datos. También puede realizar otros pasos de optimización posteriores a la migración, entre los que se podrían incluir la optimización de la directiva de indexación, la actualización del nivel de coherencia predeterminado o la configuración de la distribución global de la cuenta de Azure Cosmos DB. Para más información, consulte [Optimización posterior a la migración](../cosmos-db/mongodb-post-migration.md).

## <a name="next-steps"></a>Pasos siguientes

En la [Guía de migración de bases de datos de Azure](https://datamigration.microsoft.com/) encontrará más escenarios que le servirán de guía para la migración.



