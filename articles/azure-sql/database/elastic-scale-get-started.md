---
title: Introducción a Elastic Database Tools
description: Explicación básica de la característica Elastic Database Tools de Azure SQL Database, incluida una aplicación de ejemplo fácil de ejecutar.
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: scoriani
ms.author: scoriani
ms.reviewer: mathoma
ms.date: 01/25/2019
ms.openlocfilehash: 316f1c6558cf033ac315f413f4e081b4f920343b
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110705543"
---
# <a name="get-started-with-elastic-database-tools"></a>Introducción a Elastic Database Tools
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Este documento es una introducción a la experiencia del desarrollador con la [biblioteca de cliente de Elastic Database](elastic-database-client-library.md) que le ayuda a ejecutar una aplicación de ejemplo. La aplicación de ejemplo crea una aplicación particionada sencilla y explora las funcionalidades clave de la característica Elastic Database Tools de Azure SQL Database. Se centra en casos de uso para la [administración de asignación de particiones](elastic-scale-shard-map-management.md), el [enrutamiento dependiente de datos](elastic-scale-data-dependent-routing.md) y las [consultas a través de particiones múltiples](elastic-scale-multishard-querying.md). La biblioteca de cliente está disponible para. NET, así como para Java.

## <a name="elastic-database-tools-for-java"></a>Elastic Database Tools para Java

### <a name="prerequisites"></a>Requisitos previos

* Kit para desarrolladores de Java (JDK) versión 1.8 o posterior
* [Maven](https://maven.apache.org/download.cgi)
* SQL Database o una instancia local de SQL Server

### <a name="download-and-run-the-sample-app"></a>Descarga y ejecución de la aplicación de ejemplo

Siga estos pasos para compilar los archivos JAR y empezar a trabajar con el proyecto de ejemplo:

1. Clone el [repositorio de GitHub](https://github.com/Microsoft/elastic-db-tools-for-java) que contiene la biblioteca de cliente junto con la aplicación de ejemplo.

2. Edite el archivo _./sample/src/main/resources/resource.properties_ para establecer los valores siguientes:
    * TEST_CONN_USER
    * TEST_CONN_PASSWORD
    * TEST_CONN_SERVER_NAME

3. Para compilar el proyecto de ejemplo, en el directorio _./sample_, ejecute el siguiente comando:

    ```
    mvn install
    ```

4. Para iniciar el proyecto de ejemplo, en el directorio _./sample_, ejecute el siguiente comando:

    ```
    mvn -q exec:java "-Dexec.mainClass=com.microsoft.azure.elasticdb.samples.elasticscalestarterkit.Program"
    ```

5. Para obtener más información sobre las funcionalidades de la biblioteca de cliente, experimente con las diferentes opciones. No dude en explorar el código para obtener más información acerca de la implementación de la aplicación de ejemplo.

    ![Progress-java][5]

Felicidades. Ha creado y ejecutado correctamente su primera aplicación con particiones mediante Elastic Database Tools en Azure SQL Database. Use Visual Studio o SQL Server Management Studio para conectar con la base de datos y eche un vistazo rápido a las particiones creadas por el ejemplo. Observará nuevas bases de datos de particiones de ejemplo y una base de datos de administrador de mapas de particiones que ha creado el ejemplo.

Para agregar la biblioteca de cliente a su propio proyecto de Maven, agregue la siguiente dependencia en el archivo POM:

```xml
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>elastic-db-tools</artifactId>
    <version>1.0.0</version>
</dependency>
```

## <a name="elastic-database-tools-for-net"></a>Elastic Database Tools para .NET

### <a name="prerequisites"></a>Requisitos previos

* Visual Studio 2012 o posterior con C#. Descargue una versión gratuita desde [Descargas de Visual Studio](https://www.visualstudio.com/downloads/download-visual-studio-vs.aspx).
* NuGet 2.7 o posterior. Para obtener la versión más reciente, vea [Installing NuGet (Instalación de NuGet)](https://docs.nuget.org/docs/start-here/installing-nuget).

### <a name="download-and-run-the-sample-app"></a>Descarga y ejecución de la aplicación de ejemplo

Para instalar la biblioteca, vaya a [Microsoft.Azure.SqlDatabase.ElasticScale.Client](https://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Client/). La biblioteca se instala con la aplicación de ejemplo que se describe en la siguiente sección.

Para descargar y ejecutar el ejemplo, siga estos pasos:

1. Descargue el [ejemplo Herramientas de Elastic DB Tools para Azure SQL: Introducción](https://github.com/Azure/elastic-db-tools). Descomprima el ejemplo en una ubicación de su elección.

2. Para crear un proyecto, abra la solución *ElasticDatabaseTools.sln* desde el directorio *elastic-db-tools-master*. 

3. Establezca el proyecto *ElasticScaleStarterKit* como proyecto de inicio.

4. En el proyecto *ElasticScaleStarterKit*, abra el archivo *App.config*. A continuación, siga las instrucciones del archivo para agregar el nombre de servidor y la información de inicio de sesión (nombre de usuario y contraseña).

5. Compile y ejecute la aplicación. Cuando se le pida, permita que Visual Studio restaure los paquetes NuGet de la solución. Esta acción descarga la versión más reciente de la biblioteca de cliente de Elastic Database desde NuGet.

6. Para obtener más información sobre las funcionalidades de la biblioteca de cliente, experimente con las diferentes opciones. Anote los pasos que la aplicación lleva a cabo en la salida de la consola y explore el código que hay detrás a su antojo.

   ![Progreso][4]

Felicidades. Ha creado y ejecutado correctamente su primera aplicación con particiones mediante Elastic Database Tools en SQL Database. Use Visual Studio o SQL Server Management Studio para conectar con la base de datos y eche un vistazo rápido a las particiones creadas por el ejemplo. Observará nuevas bases de datos de particiones de ejemplo y una base de datos de administrador de mapas de particiones que ha creado el ejemplo.

> [!IMPORTANT]
> Se recomienda usar siempre la versión más reciente de Management Studio para poder estar al día de las actualizaciones de Azure y SQL Database. [Actualice SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms).

## <a name="key-pieces-of-the-code-sample"></a>Piezas clave del ejemplo de código

* **Administración de particiones y mapas de particiones**: el código ilustra cómo trabajar con particiones, rangos y asignaciones en el archivo *ShardManagementUtils.cs*. Para más información, vea [Scale out databases with the shard map manager](https://go.microsoft.com/?linkid=9862595) (Escalado horizontal de las bases de datos mediante Shard Map Manager).  

* **Enrutamiento dependiente de los datos**: el enrutamiento de transacciones a la partición correcta se muestra en el archivo *DataDependentRoutingSample.cs*. Para más información, vea [Enrutamiento dependiente de los datos](https://go.microsoft.com/?linkid=9862596).

* **Consultas a través de particiones múltiples**: las consultas a través de particiones se ilustran en el archivo *MultiShardQuerySample.cs*. Para más información, vea [Consultas a través de particiones múltiples](https://go.microsoft.com/?linkid=9862597).

* **Incorporación de particiones vacías**: la incorporación iterativa de nuevas particiones vacías se realiza mediante código en el archivo *CreateShardSample.cs*. Para más información, vea [Scale out databases with the shard map manager](https://go.microsoft.com/?linkid=9862595) (Escalado horizontal de las bases de datos mediante Shard Map Manager).

## <a name="other-elastic-scale-operations"></a>Otras operaciones de escalado elástico

* **División de una partición existente**: la funcionalidad de dividir particiones se proporciona mediante la herramienta de división y combinación. Para más información, vea [Mover datos entre bases de datos en la nube escaladas horizontalmente](elastic-scale-overview-split-and-merge.md).

* **Combinación de particiones existentes**: las combinaciones de particiones también se realizan mediante la herramienta de división y combinación. Para más información, vea [Mover datos entre bases de datos en la nube escaladas horizontalmente](elastic-scale-overview-split-and-merge.md).

## <a name="cost"></a>Coste

La biblioteca de Elastic Database Tools es gratuita. Si usa Elastic Database Tools, no tendrá ningún cargo adicional al costo del uso de Azure.

Por ejemplo, la aplicación de ejemplo crea nuevas bases de datos. El costo de esta funcionalidad depende de la edición de SQL Database que elija y del uso de Azure de la aplicación.

Para obtener información sobre los precios, vea [SQL Database Precios](https://azure.microsoft.com/pricing/details/sql-database/).

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Elastic Database Tools, consulte los artículos siguientes:

* Ejemplos de código:
  * Herramientas de Elastic Database ([.NET](https://github.com/Azure/elastic-db-tools), [Java](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22azure-elasticdb-tools%22))
  * [Elastic Database Tools for Azure SQL - Entity Framework Integration](https://code.msdn.microsoft.com/Elastic-Scale-with-Azure-bae904ba?SRC=VSIDE) (Elastic Database Tools para Azure SQL: integración con Entity Framework)
  * [Shard Elasticity on Script Center (Elasticidad de particiones en el Centro de scripts)](https://gallery.technet.microsoft.com/scriptcenter/Elastic-Scale-Shard-c9530cbe)
* Blog: [Elastic Scale announcement](https://azure.microsoft.com/blog/20../../introducing-elastic-scale-preview-for-azure-sql-database/) (Presentación del escalado elástico)
* Channel 9: [vídeo de introducción al escalado elástico](https://channel9.msdn.com/Shows/Data-Exposed/Azure-SQL-Database-Elastic-Scale)
* Foro de debate: [Página de preguntas y respuestas de Microsoft sobre Azure SQL Database](/answers/topics/azure-sql-database.html)
* Para medir el rendimiento: [Creación de bases de datos escalables en la nube](elastic-database-client-library.md)

<!--Anchors-->
[The Elastic Scale Sample Application]: #The-Elastic-Scale-Sample-Application
[Download and Run the Sample App]: #Download-and-Run-the-Sample-App
[Cost]: #Cost
[Next steps]: #next-steps

<!--Image references-->
[1]: ./media/elastic-scale-get-started/newProject.png
[2]: ./media/elastic-scale-get-started/click-online.png
[3]: ./media/elastic-scale-get-started/click-CSharp.png
[4]: ./media/elastic-scale-get-started/output2.png
[5]: ./media/elastic-scale-get-started/java-client-library.PNG