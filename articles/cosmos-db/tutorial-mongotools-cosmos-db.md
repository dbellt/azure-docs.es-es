---
title: Migración de MongoDB desconectado a la API de Azure Cosmos DB para MongoDB mediante las herramientas nativas de MongoDB
description: Aprenda a utilizar las herramientas nativas de MongoDB para migrar pequeños conjuntos de datos de instancias de MongoDB a Azure Cosmos DB
author: anfeldma-ms
ms.author: anfeldma
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.topic: tutorial
ms.date: 05/19/2021
ms.reviewer: sngun
ms.openlocfilehash: ac5e2f5474b786895ce90dbe682e98f08205c048
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110476935"
---
# <a name="tutorial-migrate-mongodb-to-azure-cosmos-dbs-api-for-mongodb-offline-using-mongodb-native-tools"></a>Tutorial: Migración de MongoDB a la API de Azure Cosmos DB para MongoDB desconectado mediante las herramientas nativas de MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

> [!IMPORTANT]  
> Lea toda esta guía antes de llevar a cabo los pasos de migración.
>

Esta guía de migración de MongoDB forma parte de la serie sobre la migración de MongoDB. Los pasos críticos de la migración de MongoDB son [la migración previa](mongodb-pre-migration.md), la migración y la [migración posterior](mongodb-post-migration.md), como se muestra a continuación.

![Diagrama de pasos de migración.](./media/mongodb-pre-migration/overall-migration-steps.png)

## <a name="overview-of-data-migration-using-mongodb-native-tools"></a>Introducción a la migración de datos mediante herramientas nativas de MongoDB

Puede usar las herramientas nativas de Mongo DB para realizar una migración desconectada (de una sola vez) de las bases de datos desde una instancia local o en la nube de MongoDB a la API de Azure Cosmos DB para MongoDB.

En este tutorial, aprenderá a:
> [!div class="checklist"]
>
> * Elegir la herramienta nativa de MongoDB adecuada para el caso de uso.
> * Ejecutar la migración.
> * Supervisar la migración
> * Comprobar que la migración se haya realizado correctamente.

En este tutorial, migrará un conjunto de datos de MongoDB hospedado en una instancia de Azure Virtual Machine a la API de Azure Cosmos DB para MongoDB mediante las herramientas nativas de MongoDB. Las herramientas nativas de MongoDB son un conjunto de archivos binarios que facilitan la manipulación de datos en una instancia de MongoDB existente. Como Azure Cosmos DB expone Mongo API, las herramientas nativas de MongoDB pueden insertar datos en Azure Cosmos DB. Este documento se centra en la migración de datos de una instancia de MongoDB mediante *mongoexport/mongoimport* o *mongodump/mongorestore*. Como las herramientas nativas se conectan a MongoDB mediante cadenas de conexión, puede ejecutar las herramientas en cualquier parte; sin embargo, se recomienda ejecutar estas herramientas en la misma red que la instancia de MongoDB para evitar problemas de firewall. 

Las herramientas nativas de MongoDB solo pueden migrar datos tan rápido como lo permita el hardware del host; las herramientas nativas pueden ser la solución más sencilla para los conjuntos de datos pequeños cuando el tiempo total de migración no sea un problema. El [conector Spark de MongoDB](https://docs.mongodb.com/spark-connector/current/), [Azure Data Migration Service (DMS)](../dms/tutorial-mongodb-cosmos-db.md)o [Azure Data Factory (ADF)](../data-factory/connector-azure-cosmos-db-mongodb-api.md) pueden ser alternativas mejores si necesita una canalización de migración escalable.

Si no tiene un origen de MongoDB ya configurado, vea el artículo [Instalación y configuración de MongoDB en una máquina virtual Windows en Azure](/previous-versions/azure/virtual-machines/windows/install-mongodb).

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, necesita:

* [Realizar los pasos previos a la migración](../cosmos-db/mongodb-pre-migration.md) como son estimar el rendimiento, elegir una clave de partición y seleccionar la directiva de indexación.
* [Cree una cuenta de la API de Azure Cosmos DB para MongoDB](https://ms.portal.azure.com/#create/Microsoft.DocumentDB).
* Inicio de sesión en la instancia de MongoDB
    * [Descargue e instale las herramientas nativas de MongoDB desde este vínculo](https://www.mongodb.com/try/download/database-tools).
        * **Asegúrese de que la versión de las herramientas nativas de MongoDB coincida con la instancia existente de MongoDB.**
        * Si la instancia de MongoDB tiene una versión diferente de Mongo API de Azure Cosmos DB, **instale ambas versiones de la herramienta nativa de MongoDB y use la versión de herramienta adecuada para MongoDB y Mongo API de Azure Cosmos DB, respectivamente.**
    * Agregue un usuario con permisos `readWrite`, a menos que ya exista uno. Más adelante en este tutorial, proporcione este nombre de usuario y contraseña para las herramientas *mongoexport* y *mongodump*.

## <a name="configure-azure-cosmos-db-server-side-retries"></a>Configuración de los reintentos en el lado de servidor de Azure Cosmos DB

Los clientes que migren de MongoDB a Azure Cosmos DB se benefician de la gobernanza de recursos, lo que garantiza que se puedan aprovechar al máximo las RU/s de rendimiento aprovisionadas. Azure Cosmos DB puede limitar una solicitud determinada en el transcurso de la migración si esa solicitud supera las RU/s aprovisionadas del contenedor; en ese caso, se debe reintentar esa solicitud. El tiempo de ida y vuelta implicado en el salto de red entre la herramienta de migración y Azure Cosmos DB afecta al tiempo de respuesta total de la solicitud. Además, es posible que las herramientas nativas de MongoDB no puedan administrar los reintentos. La característica *Reintento en el lado servidor* de Azure Cosmos DB permite al servicio interceptar los códigos de error de limitación y volver a intentar la operación con un tiempo de ida y vuelta mucho menor, lo que mejora enormemente los tiempos de respuesta de las solicitudes. Desde la perspectiva de las herramientas nativas de MongoDB, se minimiza la necesidad de administrar los reintentos, lo que afectará positivamente a su experiencia durante la migración.

Puede encontrar la funcionalidad Reintento en el lado servidor en la hoja *Características* del portal de Azure Cosmos DB.

![Captura de pantalla de la característica de SSR de MongoDB.](../dms/media/tutorial-mongodb-to-cosmosdb/mongo-server-side-retry-feature.png)

Y, si está *deshabilitada*, se recomienda habilitarla como se muestra a continuación.

![Captura de pantalla de SSR de MongoDB habilitado.](../dms/media/tutorial-mongodb-to-cosmosdb/mongo-server-side-retry-enable.png)

## <a name="choose-the-proper-mongodb-native-tool"></a>Elección de la herramienta nativa de MongoDB adecuada

![Diagrama de selección de la mejor herramienta nativa de MongoDB.](media/tutorial-mongotools-cosmos-db/mongodb-native-tool-selection-table.png)

* *mongoexport/mongoimport* es el mejor par de herramientas de migración para migrar un subconjunto de la base de datos de MongoDB.
    * *mongoexport* exporta los datos existentes a un archivo JSON o CSV legible por el usuario. *mongoexport* toma un argumento que especifica el subconjunto de los datos existentes que se van a exportar. 
    * *mongoimport* abre un archivo JSON o CSV e inserta el contenido en la instancia de la base de datos de destino (Azure Cosmos DB en este caso). 
    * Tenga en cuenta que JSON y CSV no son formatos compactos; es posible que se produzcan cargos de red excesivos, ya que *mongoimport* envía datos a Azure Cosmos DB.
* *mongodump/mongorestore* es el mejor par de herramientas de migración para migrar toda la base de datos de MongoDB. El formato BSON compacto hará un uso más eficaz de los recursos de red a medida que los datos se inserten en Azure Cosmos DB.
    * *mongodump* exporta los datos existentes como un archivo BSON.
    * *mongorestore* importa el volcado del archivo BSON a Azure Cosmos DB.
* Por otro lado, si solo tiene un archivo JSON pequeño que quiere importar a Mongo API de Azure Cosmos DB, la herramienta *mongoimport* es una solución rápida para la ingesta de datos.

## <a name="collect-the-azure-cosmos-db-mongo-api-credentials"></a>Recopilación de las credenciales de Mongo API de Azure Cosmos DB

Mongo API de Azure Cosmos DB proporciona credenciales de acceso compatibles que las herramientas nativas de MongoDB pueden utilizar. Tendrá que tener estas credenciales de acceso a mano para poder migrar datos a Mongo API de Azure Cosmos DB. Para encontrar estas credenciales:

1. Abra Azure Portal.
1. Vaya a la cuenta de Mongo API de Azure Cosmos DB.
1. En el panel de navegación izquierdo, seleccione la hoja *Cadena de conexión* y verá una pantalla similar a la siguiente:

    ![Captura de pantalla de credenciales de Azure Cosmos DB.](media/tutorial-mongotools-cosmos-db/cosmos-mongo-credentials.png)

    * *HOST*: el punto de conexión de Azure Cosmos DB funciona como un nombre de host de MongoDB.
    * *PORT* (PUERTO): cuando las herramientas nativas de MongoDB se conectan a Azure Cosmos DB, debe especificar este puerto explícitamente.
    * *USERNAME* (NOMBRE DE USUARIO): el prefijo del nombre de dominio del punto de conexión de Azure Cosmos DB funciona como nombre de usuario de MongoDB.
    * *PASSWORD* (CONTRASEÑA): la clave maestra de Azure Cosmos DB funciona como contraseña de MongoDB.
    * Además, tenga en cuenta el campo *SSL*, que es `true`: la herramienta nativa de MongoDB **debe** habilitar SSL al escribir datos en Azure Cosmos DB.

## <a name="perform-the-migration"></a>Realización de la migración

1. Elija las bases de datos y las recopilaciones que desee migrar. En este ejemplo, se va a migrar la recopilación de *consultas* de la base de datos *edx* desde MongoDB a Azure Cosmos DB.

El resto de esta sección le guiará en el uso del par de herramientas seleccionado en la sección anterior.

### <a name="mongoexportmongoimport"></a>*mongoexport/mongoimport*

1. Para exportar los datos de la instancia de MongoDB de origen, abra un terminal en la máquina de la instancia de MongoDB. Si se trata de un máquina Linux, escriba:

    `mongoexport --host HOST:PORT --authenticationDatabase admin -u USERNAME -p PASSWORD --db edx --collection query --out edx.json`

    En Windows, el archivo ejecutable será `mongoexport.exe`. Se deben rellenar *HOST*, *PORT* (PUERTO), *USERNAME* (NOMBRE DE USUARIO) y *PASSWORD* (CONTRASEÑA) en función de las propiedades de la instancia de base de datos de MongoDB existente. 
    
    También puede optar por exportar solo un subconjunto del conjunto de datos de MongoDB. Una manera de hacerlo es agregar un argumento de filtro adicional:
    
    `mongoexport --host HOST:PORT --authenticationDatabase admin -u USERNAME -p PASSWORD --db edx --collection query --out edx.json --query '{"field1":"value1"}'`

    Solo se exportarán los documentos que coincidan con el filtro `{"field1":"value1"}`.

    Una vez ejecutada la llamada, verá que se produce un archivo `edx.json`:

    ![Captura de pantalla de una llamada a mongoexport.](media/tutorial-mongotools-cosmos-db/mongo-export-output.png)
1. Puede usar el mismo terminal para importar `edx.json` a Azure Cosmos DB. Si está ejecutando `mongoimport` en una máquina Linux, escriba lo siguiente:

    `mongoimport --host HOST:PORT -u USERNAME -p PASSWORD --db edx --collection importedQuery --ssl --type json --writeConcern="{w:0}" --file edx.json`

    En Windows, el archivo ejecutable será `mongoimport.exe`. Se deben rellenar *HOST*, *PORT* (PUERTO), *USERNAME* (NOMBRE DE USUARIO) y *PASSWORD* (CONTRASEÑA) en función de las credenciales de Azure Cosmos DB que ha recopilado anteriormente. 
1. **Supervise** la salida del terminal desde *mongoimport*. Verá que imprime líneas de texto en el terminal que contiene actualizaciones en el estado de la migración:

    ![Captura de pantalla de una llamada a mongoimport.](media/tutorial-mongotools-cosmos-db/mongo-import-output.png)    

1. Por último, examine Azure Cosmos DB para **asegurarse** de que la migración se ha realizado correctamente. Abra el portal de Azure Cosmos DB y vaya al Explorador de datos. Verá (1) que se ha creado una base de datos *edx* con una recopilación *importedQuery* y (2) si ha exportado solo un subconjunto de datos, *importedQuery* debe contener *solo* los documentos que coincidan con el subconjunto que desee de los datos. En el ejemplo siguiente, solo un documento coincide con el filtro `{"field1":"value1"}`:

    ![Captura de pantalla de comprobación de datos de Cosmos DB.](media/tutorial-mongotools-cosmos-db/mongo-review-cosmos.png)    

### <a name="mongodumpmongorestore"></a>*mongodump/mongorestore*

1. Para crear un volcado de datos de BSON de la instancia de MongoDB, abra un terminal en la máquina de la instancia de MongoDB. Si se trata de un máquina Linux, escriba:

    `mongodump --host HOST:PORT --authenticationDatabase admin -u USERNAME -p PASSWORD --db edx --collection query --out edx-dump`

    Se deben rellenar *HOST*, *PORT* (PUERTO), *USERNAME* (NOMBRE DE USUARIO) y *PASSWORD* (CONTRASEÑA) en función de las propiedades de la instancia de base de datos de MongoDB existente. Verá que se genera un directorio `edx-dump` y que la estructura de directorios de `edx-dump` reproduce la jerarquía de recursos (estructura de base de datos y recopilación) de la instancia de MongoDB de origen. Cada recopilación se representa mediante un archivo BSON:

    ![Captura de pantalla de una llamada a mongodump.](media/tutorial-mongotools-cosmos-db/mongo-dump-output.png)
1. Puede usar el mismo terminal para restaurar el contenido de `edx-dump` en Azure Cosmos DB. Si está ejecutando `mongorestore` en una máquina Linux, escriba lo siguiente:

    `mongorestore --host HOST:PORT --authenticationDatabase admin -u USERNAME -p PASSWORD --db edx --collection importedQuery --ssl edx-dump/edx/query.bson`

    En Windows, el archivo ejecutable será `mongorestore.exe`. Se deben rellenar *HOST*, *PORT* (PUERTO), *USERNAME* (NOMBRE DE USUARIO) y *PASSWORD* (CONTRASEÑA) en función de las credenciales de Azure Cosmos DB que ha recopilado anteriormente. 
1. **Supervise** la salida del terminal de *mongorestore*. Verá que imprime líneas en la actualización del terminal en el estado de migración:

    ![Captura de pantalla de una llamada a mongorestore.](media/tutorial-mongotools-cosmos-db/mongo-restore-output.png)    

1. Por último, examine Azure Cosmos DB para **asegurarse** de que la migración se haya realizado correctamente. Abra el portal de Azure Cosmos DB y vaya al Explorador de datos. Verá (1) que se ha creado una base de datos *edx* con una recopilación *importedQuery* y (2) que *importedQuery* debe contener el conjunto de datos *completo* de la recopilación de origen:

    ![Captura de pantalla de la comprobación de los datos de mongorestore de Cosmos DB.](media/tutorial-mongotools-cosmos-db/mongo-review-cosmos-restore.png)    

## <a name="post-migration-optimization"></a>Optimización posterior a la migración

Después de migrar los datos almacenados en la base de datos de MongoDB a la API de Azure Cosmos DB para MongoDB, puede conectarse a Azure Cosmos DB y administrar los datos. También puede realizar otros pasos de optimización posteriores a la migración, como optimizar la directiva de indexación, actualizar el nivel de coherencia predeterminado o configurar la distribución global de la cuenta de Azure Cosmos DB. Para más información, consulte el artículo [Optimización posterior a la migración](../cosmos-db/mongodb-post-migration.md).

## <a name="additional-resources"></a>Recursos adicionales

* [Información sobre el servicio Cosmos DB](https://azure.microsoft.com/services/cosmos-db/)
* [Documentación de herramientas de bases de datos de MongoDB](https://docs.mongodb.com/database-tools/)

## <a name="next-steps"></a>Pasos siguientes

* Revise las directrices de migración en la [Guía de migración de bases de datos](https://datamigration.microsoft.com/) para consultar escenarios adicionales.