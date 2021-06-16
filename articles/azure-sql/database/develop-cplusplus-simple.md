---
title: Conexión a SQL Database mediante C y C++
description: Usar el código de ejemplo de este inicio rápido para crear una aplicación moderna con C++ y respaldada por una base de datos relacional eficaz en la nube con Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: sqldbrb=1
ms.devlang: cpp
ms.topic: how-to
author: dzsquared
ms.author: drskwier
ms.reviewer: mathoma
ms.date: 12/12/2018
ms.openlocfilehash: c874ccdf2f94bf07aab51d8510065d9e259daa44
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110694526"
---
# <a name="connect-to-sql-database-using-c-and-c"></a>Conexión a SQL Database mediante C y C++
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Esta publicación está dirigida a los desarrolladores de C y C++ que intentan conectarse a Azure SQL Database. Está dividido en secciones para que puede ir a la sección que más le interese.

## <a name="prerequisites-for-the-cc-tutorial"></a>Requisitos previos para el tutorial de C/C++

Asegúrese de que dispone de lo siguiente:

* Una cuenta de Azure activa. Si no tiene una, puede registrarse para obtener una [prueba gratuita de Azure](https://azure.microsoft.com/pricing/free-trial/).
* [Visual Studio](https://www.visualstudio.com/downloads/). Debe instalar los componentes del lenguaje C++ para compilar y ejecutar este ejemplo.
* [Visual C++ for Linux Development](/cpp/linux/) (Visual C++ para el desarrollo de aplicaciones para Linux). Si está desarrollando en Linux, también debe instalar la extensión de Visual Studio para Linux.

## <a name="azure-sql-database-and-sql-server-on-virtual-machines"></a><a id="AzureSQL"></a>Azure SQL Database y SQL Server en máquinas virtuales

Azure SQL Database está compilado en Microsoft SQL Server y se ha diseñado para proporcionar un servicio de alta disponibilidad, gran rendimiento y escalable. El uso de Azure SQL a través de la base de datos propietaria que se ejecuta de forma local presenta numerosas ventajas. Con Azure SQL no tiene que instalar, configurar, mantener ni administrar la base de datos, sino solo su contenido y estructura. Lo que nos suele preocupar de las bases de datos, como la redundancia y la tolerancia a errores, está ya integrado.

Actualmente, Azure tiene dos opciones para hospedar cargas de trabajo de SQL Server: Azure SQL Database, base de datos como servicio y SQL Server en Virtual Machines (VM). No vamos a entrar en detalles sobre las diferencias entre ambos salvo que Azure SQL Database es la mejor opción para que las nuevas aplicaciones basadas en la nube saquen partido del ahorro de costos y la optimización del rendimiento que proporcionan los servicios en la nube. Si prevé migrar o ampliar las aplicaciones locales a la nube, SQL Server en la máquina virtual de Azure quizá funcione mejor en su caso. Para explicarlo de forma más sencilla en este artículo, vamos a crear una instancia de Azure SQL Database.

## <a name="data-access-technologies-odbc-and-ole-db"></a><a id="ODBC"></a>Tecnologías de acceso a datos: ODBC y OLE DB

La conexión a Azure SQL Database no es diferente y actualmente hay dos maneras de conectarse a las bases de datos: ODBC (conectividad abierta de bases de datos) y OLE DB (base de datos de vinculación e incrustación de objetos). En los últimos años, Microsoft se ha alineado con [ODBC para el acceso a datos relacionales nativos](/archive/blogs/sqlnativeclient/microsoft-is-aligning-with-odbc-for-native-relational-data-access). ODBC es relativamente sencillo y también mucho más rápido que OLE DB. El único inconveniente aquí es que ODBC utiliza una API de estilo C anterior.

## <a name="step-1--creating-your-azure-sql-database"></a><a id="Create"></a>Paso 1:  Creación de un base de datos de Azure SQL

Vea la [página de introducción](single-database-create-quickstart.md) para aprender a crear una base de datos de ejemplo.  Como alternativa, puede seguir los pasos de este [breve vídeo de dos minutos](https://azure.microsoft.com/documentation/videos/azure-sql-database-create-dbs-in-seconds/) para crear una instancia de Azure SQL Database mediante Azure Portal.

## <a name="step-2--get-connection-string"></a><a id="ConnectionString"></a>Paso 2:  Obtención de la cadena de conexión

Después de aprovisionar la instancia de Azure SQL Database, debe seguir estos pasos para determinar la información de conexión y agregar la dirección IP de cliente para el acceso de firewall.

En [Azure Portal](https://portal.azure.com/), vaya a la cadena de conexión de ODBC de la instancia de Azure SQL Database mediante la opción **Mostrar las cadenas de conexión de la base de datos** que aparece como parte de la sección de información general de la base de datos:

![ODBCConnectionString](./media/develop-cplusplus-simple/azureportal.png)

![ODBCConnectionStringProps](./media/develop-cplusplus-simple/dbconnection.png)

Copie el contenido de la cadena **ODBC (Incluye Node.js) [autenticación de SQL]** . Esta cadena se usará más adelante para conectarse desde el intérprete de línea de comandos ODBC de C++. Esta cadena proporciona detalles como el controlador, el servidor y otros parámetros de conexión de base de datos.

## <a name="step-3--add-your-ip-to-the-firewall"></a><a id="Firewall"></a>Paso 3:  Incorporación de la IP al firewall

Vaya a la sección del firewall del servidor y agregue la [IP del cliente al firewall siguiendo estos pasos](firewall-configure.md) para asegurarse de que se puede establecer una conexión correcta:

![AddyourIPWindow](./media/develop-cplusplus-simple/ip.png)

Ya ha configurado Azure SQL Database y está listo para conectarse desde el código de C++.

## <a name="step-4-connecting-from-a-windows-cc-application"></a><a id="Windows"></a>Paso 4: Conexión desde una aplicación de C o C++ de Windows

Puede conectarse fácilmente a la [instancia de Azure SQL Database mediante ODBC en Windows con este ejemplo](https://github.com/Microsoft/VCSamples/tree/master/VC2015Samples/ODBC%20database%20sample%20%28windows%29) que se compila con Visual Studio. El ejemplo implementa un intérprete de línea de comandos de ODBC que puede usarse para conectarse a Azure SQL Database. Este ejemplo toma un archivo de nombre de origen de datos (DSN) como argumento de línea de comandos o la cadena de conexión detallada que se copió anteriormente desde Azure Portal. Abra la página de propiedades para este proyecto y pegue la cadena de conexión como argumento de comando, tal como se muestra aquí:

![DSN Propsfile](./media/develop-cplusplus-simple/props.png)

Asegúrese de proporcionar los detalles de autenticación adecuados para la base de datos como parte de esa cadena de conexión de base de datos.

Inicie la aplicación para compilarla. Debería ver la siguiente ventana en la que se valida una conexión correcta. Incluso puede ejecutar algunos comandos básicos de SQL como **create table** para validar la conectividad de la base de datos:

![Comandos SQL](./media/develop-cplusplus-simple/sqlcommands.png)

Como alternativa, podría crear un archivo DSN mediante el asistente que se inicia cuando no se proporciona ningún argumento de comando. Se recomienda que lo intente también con esta opción. Puede usar este archivo DSN para la automatización y protección de la configuración de autenticación:

![Create DSN File](./media/develop-cplusplus-simple/datasource.png)

Felicidades. Ya se ha conectado correctamente a Azure SQL con C++ y ODBC en Windows. Puede seguir leyendo para hacer lo mismo en la plataforma Linux.

## <a name="step-5-connecting-from-a-linux-cc-application"></a><a id="Linux"></a>Paso 5: Conexión desde una aplicación de C o C++ de Linux

En caso de que aún no conozca la noticia, Visual Studio permite ahora desarrollar la aplicación de C++ de Linux. Puede leer acerca de este nuevo escenario en el blog [Visual C++ for Linux Development](https://blogs.msdn.microsoft.com/vcblog/20../../visual-c-for-linux-development/) (Visual C++ para el desarrollo de aplicaciones para Linux). Para la compilación para Linux, se necesitará un equipo remoto en el que se ejecute la distribución de Linux. Si no dispone de ninguno, puede configurar uno rápidamente mediante [Azure Virtual Machines de Linux](../../virtual-machines/linux/quick-create-cli.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

Para este tutorial, supongamos que tenga configurada una distribución de Linux de Ubuntu 16.04. Los pasos indicados aquí también se aplican a Ubuntu 15.10, Red Hat 6 y Red Hat 7.

Los pasos siguientes instalan las bibliotecas necesarias para SQL y ODBC para la distribución:

```console
    sudo su
    sh -c 'echo "deb [arch=amd64] https://apt-mo.trafficmanager.net/repos/mssql-ubuntu-test/ xenial main" > /etc/apt/sources.list.d/mssqlpreview.list'
    sudo apt-key adv --keyserver apt-mo.trafficmanager.net --recv-keys 417A0893
    apt-get update
    apt-get install msodbcsql
    apt-get install unixodbc-dev-utf16 #this step is optional but recommended*
```

Inicie Visual Studio. En Herramientas -> Opciones -> Multiplataforma -> Administrador de conexiones, agregue una conexión al cuadro de Linux:

![Tools Options](./media/develop-cplusplus-simple/tools.png)

Una vez establecida la conexión a través de SSH, cree una plantilla Empty project (Linux):

![New project template](./media/develop-cplusplus-simple/template.png)

A continuación, puede agregar un [nuevo archivo de código fuente C y reemplazarlo por este contenido](https://github.com/Microsoft/VCSamples/blob/master/VC2015Samples/ODBC%20database%20sample%20%28linux%29/odbcconnector/odbcconnector.c). Con las API de ODBC SQLAllocHandle, SQLSetConnectAttr y SQLDriverConnect, debe poder inicializar y establecer una conexión con la base de datos.
Al igual que con el ejemplo de ODBC de Windows, debe reemplazar la llamada SQLDriverConnect por los detalles de los parámetros de cadena de conexión de base de datos copiados anteriormente de Azure Portal.

```c
     retcode = SQLDriverConnect(
        hdbc, NULL, "Driver=ODBC Driver 13 for SQL"
                    "Server;Server=<yourserver>;Uid=<yourusername>;Pwd=<"
                    "yourpassword>;database=<yourdatabase>",
        SQL_NTS, outstr, sizeof(outstr), &outstrlen, SQL_DRIVER_NOPROMPT);
```

Lo último que queda por hacer antes de la compilación es agregar **odbc** como dependencia de biblioteca:

![Adding ODBC as an input library](./media/develop-cplusplus-simple/lib.png)

Para iniciar la aplicación, abra Linux Console desde el menú **Depurar**:

![Linux Console](./media/develop-cplusplus-simple/linuxconsole.png)

Si la conexión se realizó correctamente, ahora debería ver el nombre de la base de datos actual en Linux Console:

![Linux Console Window Output](./media/develop-cplusplus-simple/linuxconsolewindow.png)

Felicidades. Ha completado correctamente el tutorial y ya puede conectarse a Azure SQL Database desde C++ en plataformas Windows y Linux.

## <a name="get-the-complete-cc-tutorial-solution"></a><a id="GetSolution"></a>Obtención de la solución completa del tutorial de C o C++

Puede encontrar la solución GetStarted que contiene todos los ejemplos de este artículo en GitHub:

* [Ejemplo de Windows de C++ de ODBC](https://github.com/Microsoft/VCSamples/tree/master/VC2015Samples/ODBC%20database%20sample%20%28windows%29): descargue el ejemplo de ODBC de C++ de Windows para conectarse a Azure SQL.
* [Ejemplo de Linux de C++ de ODBC](https://github.com/Microsoft/VCSamples/tree/master/VC2015Samples/ODBC%20database%20sample%20%28linux%29): descargue el ejemplo de ODBC de C++ de Linux para conectarse a Azure SQL.

## <a name="next-steps"></a>Pasos siguientes

* Consulte [Información general de desarrollo de SQL Database](develop-overview.md)
* Más información en [ODBC API Reference](/sql/odbc/reference/syntax/odbc-api-reference/) (Referencia de la API de ODBC).

## <a name="additional-resources"></a>Recursos adicionales

* [Modelos de diseño para las aplicaciones SaaS multiinquilino con Azure SQL Database](saas-tenancy-app-design-patterns.md)
* Descubra todas las [funcionalidades de SQL Database](https://azure.microsoft.com/services/sql-database/)