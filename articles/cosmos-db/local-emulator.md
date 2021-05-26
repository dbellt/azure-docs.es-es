---
title: Instalación y desarrollo local con el emulador de Azure Cosmos DB
description: Aprenda a instalar y usar el emulador de Azure Cosmos DB en entornos Windows, Linux, macOS y Docker para Windows. Con el emulador, puede desarrollar y probar una aplicación localmente de forma gratuita, sin necesidad de crear una suscripción a Azure.
ms.service: cosmos-db
ms.topic: how-to
author: StefArroyo
ms.author: esarroyo
ms.date: 09/22/2020
ms.custom: devx-track-csharp, contperf-fy21q1
ms.openlocfilehash: 3c8e2559d1dc8fce27815d498578d352fa9fe498
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110372831"
---
# <a name="install-and-use-the-azure-cosmos-db-emulator-for-local-development-and-testing"></a>Instalación y uso del emulador de Azure Cosmos DB para desarrollo y pruebas locales
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

El Emulador de Azure Cosmos DB proporciona un entorno local que emula el servicio de Azure Cosmos DB con fines de desarrollo. Mediante el Emulador de Azure Cosmos DB, puede desarrollar y probar su aplicación localmente, sin crear una suscripción de Azure o realizar algún gasto. Cuando esté satisfecho con el funcionamiento de la aplicación en el emulador de Azure Cosmos DB, puede cambiar a una cuenta de Azure Cosmos en la nube. En este artículo se describe cómo instalar y usar el emulador en entornos Windows, Linux, macOS y Docker para Windows.

## <a name="download-the-emulator"></a>Descarga del emulador

Para empezar, descargue e instale la versión más reciente del emulador de Azure Cosmos DB en el equipo local. En el artículo dedicado a las [notas de la versión del emulador](local-emulator-release-notes.md) se enumeran todas las versiones disponibles y las actualizaciones de características que se realizaron en cada versión.

:::image type="icon" source="media/local-emulator/download-icon.png" border="false"::: **[Descarga del emulador de Azure Cosmos DB](https://aka.ms/cosmosdb-emulator)**

Puede desarrollar aplicaciones mediante el emulador de Azure Cosmos DB con cuentas de [SQL](local-emulator.md#sql-api), [Cassandra](local-emulator.md#cassandra-api), [MongoDB](local-emulator.md#azure-cosmos-dbs-api-for-mongodb), [Gremlin](local-emulator.md#gremlin-api) y [Table](local-emulator.md#table-api) API. Actualmente, el explorador de datos del emulador solo admite la visualización de datos SQL. Los datos creados con las aplicaciones cliente de MongoDB, Gremlin/Graph y Cassandra no se pueden ver. Para obtener más información, consulte [cómo conectarse al punto de conexión del emulador](#connect-with-emulator-apis) desde distintas API.

## <a name="how-does-the-emulator-work"></a>¿Cómo funciona el emulador?

El Emulador de Azure Cosmos DB proporciona una emulación de gran fidelidad del servicio Azure Cosmos DB. Cuenta con una funcionalidad equivalente a la de Azure Cosmos DB, que incluye la creación y la consulta de datos, el aprovisionamiento y el escalado de contenedores, así como la ejecución de procedimientos almacenados y desencadenadores. Puede desarrollar y probar aplicaciones mediante el emulador de Azure Cosmos DB e implementarlas en Azure a escala global actualizando el punto de conexión de Azure Cosmos DB.

Aunque la emulación del servicio Azure Cosmos DB es fiel, la implementación del emulador es diferente de la del servicio. Por ejemplo, el emulador utiliza componentes del sistema operativo estándar; por ejemplo, utiliza el sistema de archivos local para la persistencia y la pila del protocolo HTTPS para la conectividad. Cuando se usa el emulador, no son aplicables las funcionalidades que se basan en la infraestructura de Azure, como la replicación global, la latencia de milisegundos de un solo dígito para lectura/escritura y los niveles de coherencia ajustables.

Puede migrar datos entre el emulador de Azure Cosmos DB y el servicio Azure Cosmos DB con la [herramienta de migración de datos de Azure Cosmos DB](https://github.com/azure/azure-documentdb-datamigrationtool).

## <a name="differences-between-the-emulator-and-the-cloud-service"></a>Diferencias entre el emulador y el servicio en la nube

Dado que el emulador de Azure Cosmos DB proporciona un entorno emulado que se ejecuta en una estación de trabajo de desarrollador local, hay algunas diferencias de funcionalidad entre el emulador y una cuenta de Azure Cosmos en la nube:

* Actualmente, el panel **Data Explorer** (Explorador de datos) del emulador solo es totalmente compatible con los clientes de SQL API. La vista del **explorador de datos** y las operaciones de las API de Azure Cosmos DB, como MongoDB, Table, Graph y Cassandra API no son totalmente compatibles.

* El emulador solo admite una única cuenta fija y una clave principal ya conocida. No se puede regenerar la clave cuando se usa el emulador de Azure Cosmos DB; sin embargo, puede cambiar la clave predeterminada mediante la opción de la [línea de comandos](emulator-command-line-parameters.md).

* Con el emulador, solo puede crear una cuenta de Azure Cosmos en modo de [rendimiento aprovisionado](set-throughput.md). Actualmente no se admite el modo [sin servidor](serverless.md).

* El emulador no es un servicio escalable y no admite un gran número de contenedores. Cuando se usa el emulador de Azure Cosmos DB, de forma predeterminada se pueden crear hasta 25 contenedores de tamaño fijo a 400 RU/s (solo compatibles con los SDK de Azure Cosmos DB) o 5 contenedores ilimitados. Para obtener más información sobre cómo cambiar este valor, consulte el artículo sobre la [configuración del valor de PartitionCount](emulator-command-line-parameters.md#set-partitioncount).

* El emulador no ofrece diferentes [niveles de coherencia de Azure Cosmos DB](consistency-levels.md) como hace el servicio en la nube.

* El emulador no ofrece [replicación en varias regiones](distribute-data-globally.md).

* Es posible que la copia del emulador de Azure Cosmos DB no tenga siempre los cambios más recientes del servicio Azure Cosmos DB, por lo que debe consultar la herramienta de [planeamiento de capacidad de Azure Cosmos DB](estimate-ru-with-capacity-planner.md) para calcular con precisión las necesidades de capacidad de proceso (RU) de la aplicación.

* El emulador admite un tamaño máximo de la propiedad ID de 254 caracteres.

## <a name="install-the-emulator"></a>Instalación del emulador

Antes de instalar el emulador, asegúrese de que cumple los siguientes requisitos de hardware y software:

* Requisitos de software:
  * Actualmente se admiten los sistemas operativos de host Windows Server 2016, 2019 o Windows 10. En estos momentos no se admite el sistema operativo de host con Active Directory habilitado.
  * Sistema operativo de 64 bits

* Requisitos mínimos de hardware:
  * 2 GB de RAM
  * 10 GB de espacio disponible en el disco duro

* Para instalar, configurar y ejecutar el Emulador de Azure Cosmos DB, debe tener privilegios administrativos en el equipo. El emulador agregará un certificado y también establecerá las reglas de firewall para ejecutar sus servicios. Por lo tanto, se necesitan derechos de administrador para que se puedan ejecutar esas operaciones.

Para empezar, descargue e instale la versión más reciente del [emulador de Azure Cosmos DB](https://aka.ms/cosmosdb-emulator) en el equipo local. Si tiene problemas al instalar el emulador, consulte el artículo de [solución de problemas del emulador](troubleshoot-local-emulator.md) para corregirlos.

En función de los requisitos del sistema, puede ejecutar el emulador en [Windows](#run-on-windows), [Docker para Windows](local-emulator-on-docker-windows.md), [Linux o macOS](#run-on-linux-macos), tal como se describe en las siguientes secciones de este artículo.

## <a name="check-for-emulator-updates"></a>Búsqueda de actualizaciones del emulador

Cada versión del emulador incluye un conjunto de actualizaciones de características o correcciones de errores. Para ver las versiones disponibles, consulte el artículo de [notas de la versión del emulador](local-emulator-release-notes.md).

Después de la instalación, si ha usado la configuración predeterminada, los datos correspondientes al emulador se habrán guardado en la ubicación %LOCALAPPDATA%\CosmosDBEmulator. Puede establecer otra ubicación mediante la configuración de la ruta de acceso de datos opcional; es decir, `/DataPath=PREFERRED_LOCATION` como [parámetro de línea de comandos](emulator-command-line-parameters.md). No se garantiza que los datos que se crean en una versión del emulador de Azure Cosmos DB estén disponibles cuando se utilice una versión diferente. Si necesita conservar los datos a largo plazo, es recomendable que los almacene en una cuenta de Azure Cosmos y no en el emulador de Azure Cosmos DB.

## <a name="use-the-emulator-on-windows"></a><a id="run-on-windows"></a>Uso del emulador en Windows

El emulador de Azure Cosmos DB se instala de forma predeterminada en `C:\Program Files\Azure Cosmos DB Emulator`. Para iniciar el emulador de Azure Cosmos DB en Windows, seleccione el botón **Inicio** o presione la tecla Windows. Comience a escribir **Emulador de Azure Cosmos DB** y seleccione el emulador de la lista de aplicaciones.

:::image type="content" source="./media/local-emulator/database-local-emulator-start.png" alt-text="Seleccione el botón Inicio o presione la tecla Windows, comience a escribir &quot;Emulador de Azure Cosmos DB&quot; y seleccione el emulador en la lista de aplicaciones.":::

Cuando el emulador se ha iniciado, aparece un icono en el área de notificación de la barra de tareas de Windows. El explorador de datos de Azure Cosmos se abre automáticamente en el explorador web, en la dirección URL `https://localhost:8081/_explorer/index.html`.

:::image type="content" source="./media/local-emulator/database-local-emulator-taskbar.png" alt-text="Notificación en la barra de tareas del emulador local de Azure Cosmos DB":::

También puede iniciar y detener el emulador desde la línea de comandos o mediante comandos de PowerShell. Para más información, consulte el artículo de [referencia de la herramienta de la línea de comandos](emulator-command-line-parameters.md).

El Emulador de Azure Cosmos DB se ejecuta de forma predeterminada en la máquina local ("localhost") que escucha en el puerto 8081. La dirección aparece como `https://localhost:8081/_explorer/index.html`. Si cierra el explorador y desea volver a abrirlo más adelante, puede abrir la dirección URL en el explorador o iniciarlo desde el icono del Emulador de Azure Cosmos DB de la bandeja de Windows, tal como se muestra a continuación.

:::image type="content" source="./media/local-emulator/database-local-emulator-data-explorer-launcher.png" alt-text="Iniciador del explorador de datos del emulador local de Azure Cosmos":::

## <a name="use-the-emulator-on-linux-or-macos"></a><a id="run-on-linux-macos"></a>Uso del emulador en Linux o macOS

Actualmente, el emulador de Azure Cosmos DB solo se puede ejecutar en Windows. Si usa Linux o macOS, puede ejecutar el emulador en una máquina virtual Windows hospedada en un hipervisor, como Parallels o VirtualBox.

> [!NOTE]
> Cada vez que reinicie la máquina virtual Windows hospedada en un hipervisor, tendrá que volver a importar el certificado, porque cambia la dirección IP de la máquina virtual. No tendrá que hacerlo si ha configurado la máquina virtual para conservar la dirección IP.

Siga estos pasos para usar el emulador en entornos Linux o macOS:

1. Ejecute el siguiente comando desde la máquina virtual Windows y tome nota de la dirección IPv4:

   ```bash
   ipconfig.exe
   ```

1. Dentro de la aplicación, cambie la dirección URL del punto de conexión para usar la dirección IPv4 devuelta por `ipconfig.exe` en lugar de `localhost`.

1. En la máquina virtual Windows, inicie el emulador de Azure Cosmos DB desde la línea de comandos con las siguientes opciones. Para obtener más información sobre los parámetros que se admiten en la línea de comandos, consulte la [referencia de la herramienta de línea de comandos del emulador](emulator-command-line-parameters.md):

   ```bash
   Microsoft.Azure.Cosmos.Emulator.exe /AllowNetworkAccess /Key=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==
   ```

1. Por último, debe resolver el proceso de confianza del certificado entre la aplicación que se ejecuta en el entorno Linux o Mac y el emulador. Puede usar una de las dos opciones siguientes para resolver el certificado:

   1. [Importar el certificado TLS/SSL del emulador en el entorno Linux o Mac](#import-certificate) o bien
   2. [Deshabilitar la validación TLS/SSL en la aplicación](#disable-ssl-validation)

### <a name="option-1-import-the-emulator-tlsssl-certificate"></a><a id="import-certificate"></a>Opción 1: Importar el certificado TLS/SSL del emulador

En las secciones siguientes se muestra cómo importar el certificado TLS/SSL del emulador en entornos Linux y macOS.

#### <a name="linux-environment"></a>Entorno de Linux

Si trabaja en Linux, .NET confía en OpenSSL para realizar la validación:

1. [Exporte el certificado en formato PFX](local-emulator-export-ssl-certificates.md#export-emulator-certificate). La opción PFX está disponible al elegir exportar la clave privada.

1. Copie el archivo PFX en el entorno Linux.

1. Conversión del archivo PFX en un archivo CRT

   ```bash
   openssl pkcs12 -in YourPFX.pfx -clcerts -nokeys -out YourCTR.crt
   ```

1. Copie el archivo CRT en la carpeta que contiene los certificados personalizados de la distribución de Linux. Normalmente, en las distribuciones de Debian, se encuentra en `/usr/local/share/ca-certificates/`.

   ```bash
   cp YourCTR.crt /usr/local/share/ca-certificates/
   ```

1. Actualice los certificados TLS/SSL, que actualizarán la carpeta `/etc/ssl/certs/`.

   ```bash
   update-ca-certificates
   ```

#### <a name="macos-environment"></a>Entorno de macOS

Siga estos pasos si trabaja en Mac:

1. [Exporte el certificado en formato PFX](local-emulator-export-ssl-certificates.md#export-emulator-certificate). La opción PFX está disponible al elegir exportar la clave privada.

1. Copie el archivo PFX en el entorno Mac.

1. Abra la aplicación *Keychain Access* e importe el archivo PFX.

1. Abra la lista de certificados e identifique el que tiene el nombre `localhost`.

1. Abra el menú contextual de ese elemento, seleccione *Get Item* (Obtener elemento) y en la opción *Trust* > *When using this certificate* (Confiar > Al usar este certificado), seleccione *Always Trust* (Confiar siempre). 

   :::image type="content" source="./media/local-emulator/mac-trust-certificate.png" alt-text="Abrir el menú contextual de ese elemento, seleccionar Get Item (Obtener elemento) y en la opción Trust > When using this certificate (Confiar > Al usar este certificado), seleccionar Always Trust (Confiar siempre)":::
  
### <a name="option-2-disable-the-ssl-validation-in-the-application"></a><a id="disable-ssl-validation"></a>Opción 2: Deshabilitar la validación SSL en la aplicación

Solo se recomienda deshabilitar la validación SSL con fines de desarrollo, y no debe realizarse cuando se ejecuta en un entorno de producción. En los siguientes ejemplos se muestra cómo deshabilitar la validación SSL para las aplicaciones .NET y Node.js.

# <a name="net-standard-21"></a>[.NET Standard 2.1+](#tab/ssl-netstd21)

Con cualquier aplicación que se ejecute en un marco compatible con .NET Standard 2.1 o posterior, se puede aprovechar `CosmosClientOptions.HttpClientFactory`:

[!code-csharp[Main](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/HttpClientFactory/Program.cs?name=DisableSSLNETStandard21)]

# <a name="net-standard-20"></a>[.NET Standard 2.0](#tab/ssl-netstd20)

Con cualquier aplicación que se ejecute en un marco compatible con .NET Standard 2.0, se puede aprovechar `CosmosClientOptions.HttpClientFactory`:

[!code-csharp[Main](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/HttpClientFactory/Program.cs?name=DisableSSLNETStandard20)]

# <a name="nodejs"></a>[Node.js](#tab/ssl-nodejs)

En el caso de las aplicaciones de Node.js, puede modificar el archivo `package.json` para establecer `NODE_TLS_REJECT_UNAUTHORIZED` al iniciar la aplicación:

```json
"start": NODE_TLS_REJECT_UNAUTHORIZED=0 node app.js
```
--- 

## <a name="enable-access-to-emulator-on-a-local-network"></a>Habilitación del acceso al emulador en una red local

Supongamos que tiene varias máquinas con una sola red, configura el emulador en una máquina y desea acceder a él desde otro máquina. En tal caso, debe habilitar el acceso al emulador en una red local.

Puede ejecutar el emulador en una red local. Para habilitar el acceso de red, especifique la opción `/AllowNetworkAccess` en la [línea de comandos](emulator-command-line-parameters.md), que también requiere que especifique `/Key=key_string` o `/KeyFile=file_name`. Puede usar `/GenKeyFile=file_name` para generar un archivo con una clave aleatoria por adelantado. Después, puede pasarlo a `/KeyFile=file_name` o `/Key=contents_of_file`.

La primera vez que se habilita el acceso de red, el usuario debe apagar el emulador y eliminar su directorio de datos *%LOCALAPPDATA%\CosmosDBEmulator*.

## <a name="authenticate-connections-when-using-emulator"></a><a id="authenticate-requests"></a>Autenticación de conexiones cuando se usa el emulador

Al igual que con Azure Cosmos DB en la nube, todas las solicitudes que se realicen en el Emulador de Azure Cosmos DB deben autenticarse. El emulador de Azure Cosmos DB solo admite la comunicación segura a través de TLS. Este emulador admite una sola cuenta fija y una clave de autenticación reconocida para la autenticación de clave principal. Esta cuenta y clave son las únicas credenciales que se admiten para su uso con el Emulador de Azure Cosmos DB. Son las siguientes:

```bash
Account name: localhost:<port>
Account key: C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==
```

> [!NOTE]
> La clave principal admitida por el emulador de Azure Cosmos DB está destinada a su uso exclusivo con el emulador. No puede usar su clave y cuenta de producción de Azure Cosmos DB con el Emulador de Azure Cosmos DB.

> [!NOTE]
> Si ha iniciado el emulador con la opción /Key, use la clave generada en lugar de la clave predeterminada `C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==`. Para más información sobre la opción /Key, consulte la [referencia de la herramienta de la línea de comandos](emulator-command-line-parameters.md).

## <a name="connect-to-different-apis-with-the-emulator"></a><a id="connect-with-emulator-apis"></a>Conexión a diferentes API con el emulador

### <a name="sql-api"></a>API DE SQL

Cuando tenga el emulador de Azure Cosmos DB funcionando en su escritorio, puede usar cualquier [SDK de Azure Cosmos DB](sql-api-sdk-dotnet-standard.md) admitido o la [API REST de Azure Cosmos DB](/rest/api/cosmos-db/) para interactuar con el emulador. El emulador de Azure Cosmos DB también incluye un explorador de datos integrado que permite crear contenedores para API SQL o API de Azure Cosmos DB para MongoDB. Mediante el explorador de datos, puede ver y editar elementos sin escribir ningún código.

```csharp
// Connect to the Azure Cosmos DB Emulator running locally
CosmosClient client = new CosmosClient(
   "https://localhost:8081", 
    "C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==");

```

### <a name="azure-cosmos-dbs-api-for-mongodb"></a>API de Azure Cosmos DB para MongoDB

Cuando tenga el emulador de Azure Cosmos DB funcionando en su escritorio, podrá usar la [API de Azure Cosmos DB para MongoDB](mongodb-introduction.md) para interactuar con el emulador. Inicie el emulador desde el [símbolo del sistema](emulator-command-line-parameters.md) como administrador con "/EnableMongoDbEndpoint". Luego, use la siguiente cadena de conexión para conectarse a la cuenta de la API de MongoDB:

```bash
mongodb://localhost:C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==@localhost:10255/admin?ssl=true
```

### <a name="table-api"></a>Table API

Cuando tenga el emulador de Azure Cosmos DB funcionando en su escritorio, podrá usar el [SDK de Table API de Azure Cosmos DB](./tutorial-develop-table-dotnet.md) para interactuar con el emulador. Inicie el emulador desde el [símbolo del sistema](emulator-command-line-parameters.md) como administrador con "/EnableTableEndpoint". A continuación, ejecute el siguiente código para conectarse a la cuenta de Table API:

```csharp
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.Table;
using CloudTable = Microsoft.WindowsAzure.Storage.Table.CloudTable;
using CloudTableClient = Microsoft.WindowsAzure.Storage.Table.CloudTableClient;

string connectionString = "DefaultEndpointsProtocol=http;AccountName=localhost;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==;TableEndpoint=http://localhost:8902/;";

CloudStorageAccount account = CloudStorageAccount.Parse(connectionString);
CloudTableClient tableClient = account.CreateCloudTableClient();
CloudTable table = tableClient.GetTableReference("testtable");
table.CreateIfNotExists();
table.Execute(TableOperation.Insert(new DynamicTableEntity("partitionKey", "rowKey")));
```

### <a name="cassandra-api"></a>Cassandra API

Inicie el emulador desde un [símbolo del sistema](emulator-command-line-parameters.md) como administrador con "/EnableCassandraEndpoint". Como alternativa, también puede establecer la variable de entorno `AZURE_COSMOS_EMULATOR_CASSANDRA_ENDPOINT=true`.

1. [Instale Python 2.7](https://www.python.org/downloads/release/python-2716/)

1. [Instale la CLI/CQLSH de Cassandra](https://cassandra.apache.org/download/)

1. Ejecute los siguientes comandos en una ventana normal del símbolo del sistema:

   ```bash
   set Path=c:\Python27;%Path%
   cd /d C:\sdk\apache-cassandra-3.11.3\bin
   set SSL_VERSION=TLSv1_2
   set SSL_VALIDATE=false
   cqlsh localhost 10350 -u localhost -p C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw== --ssl
   ```

1. En el shell de CQLSH, ejecute los siguientes comandos para conectarse al punto de conexión de Cassandra:

   ```bash
   CREATE KEYSPACE MyKeySpace WITH replication = {'class':'MyClass', 'replication_factor': 1};
   DESCRIBE keyspaces;
   USE mykeyspace;
   CREATE table table1(my_id int PRIMARY KEY, my_name text, my_desc text);
   INSERT into table1 (my_id, my_name, my_desc) values( 1, 'name1', 'description 1');
   SELECT * from table1;
   EXIT
   ```

### <a name="gremlin-api"></a>API de Gremlin

Inicie el emulador desde un [símbolo del sistema](emulator-command-line-parameters.md) como administrador con "/EnableGremlinEndpoint". Como alternativa, también puede establecer la variable de entorno `AZURE_COSMOS_EMULATOR_GREMLIN_ENDPOINT=true`

1. [Instale apache-tinkerpop-gremlin-console-3.3.4](https://archive.apache.org/dist/tinkerpop/3.3.4).

1. En el explorador de datos del emulador, cree una base de datos "db1" y una colección "coll1"; como clave de partición, elija "/name".

1. Ejecute los siguientes comandos en una ventana normal del símbolo del sistema:

   ```bash
   cd /d C:\sdk\apache-tinkerpop-gremlin-console-3.3.4-bin\apache-tinkerpop-gremlin-console-3.3.4
  
   copy /y conf\remote.yaml conf\remote-localcompute.yaml
   notepad.exe conf\remote-localcompute.yaml
     hosts: [localhost]
     port: 8901
     username: /dbs/db1/colls/coll1
     password: C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==
     connectionPool: {
     enableSsl: false}
     serializer: { className: org.apache.tinkerpop.gremlin.driver.ser.GraphSONMessageSerializerV1d0,
     config: { serializeResultToString: true  }}

   bin\gremlin.bat
   ```

1. En el shell de Gremlin, ejecute los siguientes comandos para conectarse al punto de conexión de Gremlin:

   ```bash
   :remote connect tinkerpop.server conf/remote-localcompute.yaml
   :remote console
   :> g.V()
   :> g.addV('person1').property(id, '1').property('name', 'somename1')
   :> g.addV('person2').property(id, '2').property('name', 'somename2')
   :> g.V()
   ```

## <a name="uninstall-the-local-emulator"></a><a id="uninstall"></a>Desinstalación del emulador local

Utilice los pasos siguientes para desinstalar el emulador:

1. Cierre todas las instancias abiertas del emulador local; para ello, haga clic con el botón derecho en el icono del **emulador de Azure Cosmos DB** en la bandeja del sistema y seleccione **Salir**. Todas las instancias pueden tardar un minuto en salir.

1. En el cuadro de búsqueda de Windows, escriba **Aplicaciones y características** y seleccione **Aplicaciones y características (configuración del sistema)** .

1. En la lista de aplicaciones, desplácese a **Azure Cosmos DB Emulator** (Emulador de Azure Cosmos DB), selecciónela, haga clic en **Desinstalar**, confirme y seleccione **Desinstalar** de nuevo.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a usar el emulador local para el desarrollo gratuito de aplicaciones en un entorno local. Ahora puede avanzar a los siguientes artículos:

* [Exportación de los certificados del emulador de Azure Cosmos DB para su uso con aplicaciones en Java, Python y Node.js](local-emulator-export-ssl-certificates.md)
* [Uso de parámetros de línea de comandos y comandos de PowerShell para controlar el emulador](emulator-command-line-parameters.md)
* [Depuración de problemas con el emulador](troubleshoot-local-emulator.md)
