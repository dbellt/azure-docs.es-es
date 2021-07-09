---
title: Grupo de SQL sin servidor
description: Obtenga información sobre el grupo de SQL sin servidor en Azure Synapse Analytics.
services: synapse analytics
author: filippopovic
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 04/15/2020
ms.author: fipopovi
ms.reviewer: jrasnick
ms.openlocfilehash: a09dfb2b8e29ba925a702c169a1a79c13016d5dc
ms.sourcegitcommit: ef950cf37f65ea7a0f583e246cfbf13f1913eb12
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111421713"
---
# <a name="serverless-sql-pool-in-azure-synapse-analytics"></a>Grupo de SQL sin servidor en Azure Synapse Analytics 

Cada área de trabajo de Azure Synapse Analytics incluye puntos de conexión de grupo de SQL sin servidor que puede usar para consultar datos en [Azure Data Lake](query-data-storage.md) (formatos [Parquet](query-data-storage.md#query-parquet-files), [Delta Lake](query-delta-lake-format.md) y de [texto delimitado](query-data-storage.md#query-csv-files)), [Cosmos DB](query-cosmos-db-analytical-store.md?toc=%2Fazure%2Fsynapse-analytics%2Ftoc.json&bc=%2Fazure%2Fsynapse-analytics%2Fbreadcrumb%2Ftoc.json&tabs=openrowset-key) o Dataverse.

Un grupo de SQL sin servidor es un servicio de consulta de los datos del lago de datos. Le permite acceder a los datos mediante las siguientes funcionalidades:
 
- Una [sintaxis T-SQL](overview-features.md) conocida para consultar los datos activos sin necesidad de copiarlos ni cargarlos en un almacén especializado. 
- Conectividad integrada mediante la interfaz de T-SQL que ofrece una amplia gama de herramientas de inteligencia empresarial y consulta ad hoc, entre las que se incluyen los controladores más populares. 

El grupo de SQL sin servidor es un sistema de procesamiento de datos distribuido, creado para datos a gran escala y funciones computacionales. El grupo de SQL sin servidor le permite analizar macrodatos en segundos o minutos, según la carga de trabajo. Gracias a la tolerancia a errores integrada en la ejecución de consultas, el sistema proporciona una alta confiabilidad y un alto índice de éxito incluso para consultas de ejecución prolongada que implican grandes conjuntos de datos.

El grupo de SQL sin servidor funciona sin servidor, por lo que no hay que configurar ninguna infraestructura ni mantener ningún clúster. Se proporciona un punto de conexión predeterminado para este servicio en cada área de trabajo de Azure Synapse, por lo que puede empezar a consultar datos tan pronto como se crea el área de trabajo. 

No se cobra por los recursos reservados, solo se cobra por los datos procesados por las consultas que se ejecutan, por lo que este modelo es un modelo de pago por uso auténtico.  

Si usa Apache Spark para Azure Synapse en la canalización de datos, o para la preparación, limpieza o enriquecimiento de datos, puede [consultar las tablas externas de Spark](develop-storage-files-spark-tables.md) que haya creado en el proceso, directamente desde el grupo de SQL sin servidor. Use [Private Link](../security/how-to-connect-to-workspace-with-private-links.md) para llevar el punto de conexión del grupo de SQL sin servidor a la [red virtual del área de trabajo administrada](../security/synapse-workspace-managed-vnet.md).  

## <a name="serverless-sql-pool-benefits"></a>Ventajas del grupo de SQL sin servidor

Si necesita explorar los datos del lago de datos, obtener información de este u optimizar la canalización de transformación de datos existente, puede beneficiarse del uso del grupo de SQL sin servidor. Es adecuado para los escenarios siguientes:

- Detección y exploración básicos: analice rápidamente datos de diversos formatos (Parquet, CSV y JSON) en el lago de datos, para que pueda planear cómo extraer información de estos.
- Almacenamiento de datos lógico: proporcione una abstracción relacional sobre datos sin procesar o dispares sin tener que reubicar y transformar los datos, lo que permite una vista siempre actualizada de estos. Obtenga más información sobre la [creación de almacenamiento de datos lógicos](tutorial-logical-data-warehouse.md).
- Transformación de datos: una forma sencilla, escalable y eficaz de transformar los datos del lago mediante T-SQL, para que estos puedan alimentar a las herramientas de inteligencia empresarial o de cualquier otro tipo, o se carguen en un almacén de datos relacional (bases de datos de Synapse SQL, Azure SQL Database, etc.).

Diferentes puestos profesionales se pueden beneficiar del grupo de SQL sin servidor:

- Los ingenieros de datos pueden explorar el lago, transformar y preparar los datos mediante este servicio, y simplificar las canalizaciones de transformación de datos. Para más información, consulte este [tutorial](tutorial-data-analyst.md).
- Los científicos de datos pueden analizar rápidamente el contenido y la estructura de los datos del lago gracias a características como OPENROWSET y la inferencia de esquemas automática.
- Los analistas de datos pueden [explorar los datos y tablas externas de Spark](develop-storage-files-spark-tables.md) que crean los científicos o ingenieros de datos mediante el conocido lenguaje T-SQL o con sus herramientas favoritas, las cuales pueden conectarse al grupo de SQL sin servidor.
- Los profesionales de inteligencia empresarial pueden [crear rápidamente informes de Power BI a partir de los datos del lago](tutorial-connect-power-bi-desktop.md) y las tablas de Spark.

## <a name="how-to-start-using-serverless-sql-pool"></a>Cómo empezar a usar el grupo de SQL sin servidor

Se proporciona un punto de conexión del grupo de SQL sin servidor en cada área de trabajo de Azure Synapse. Puede crear un área de trabajo e iniciar la consulta de datos al instante mediante herramientas con las que está familiarizado.

Asegúrese de que está aplicando los [procedimientos recomendados](best-practices-serverless-sql-pool.md) para obtener el mejor rendimiento.

## <a name="client-tools"></a>Herramientas de cliente

El grupo de SQL sin servidor permite que las herramientas ya existentes de inteligencia empresarial y de consultas ad-hoc de SQL accedan al lago de datos. Como proporciona una sintaxis T-SQL conocida, cualquier herramienta capaz de establecer conexión TDS en ofertas de SQL podrá [conectarse y hacer consultas a Synapse SQL](connect-overview.md). Puede conectarse con Azure Data Studio y ejecutar consultas ad-hoc o conectarse con Power BI para obtener información en cuestión de minutos.

## <a name="t-sql-support"></a>Compatibilidad con T-SQL

El grupo de SQL sin servidor ofrece un área expuesta de consultas de T-SQL, que se ha mejorado o ampliado ligeramente en algunos aspectos para dar cabida a experiencias relacionadas con la consulta de datos semiestructurados y datos no estructurados. Además, algunos de los aspectos del lenguaje T-SQL no se admiten debido al diseño del grupo de SQL sin servidor. Por ejemplo, actualmente no se admite la funcionalidad DML.

- La carga de trabajo se puede organizar mediante conceptos conocidos:
- Bases de datos: el punto de conexión del grupo de SQL sin servidor puede tener varias bases de datos.
- Esquemas: en una base de datos, puede haber uno o varios grupos de propiedad de objetos denominados esquemas.
- Vistas, procedimientos almacenados, funciones de valor de tablas insertadas
- Recursos externos: orígenes de datos, formatos de archivo y tablas

La seguridad se puede reforzar mediante:

- Inicios de sesión y usuarios
- Credenciales para controlar el acceso a las cuentas de almacenamiento
- Concesión, denegación y revocación de permisos por nivel de objeto
- Integración de Azure Active Directory

T-SQL compatible:

- Se admite el área expuesta [SELECT](/sql/t-sql/queries/select-transact-sql?view=azure-sqldw-latest&preserve-view=true) completa, que incluye la mayoría de las funciones de SQL.
- CETAS - CREATE EXTERNAL TABLE AS SELECT
- Instrucciones de DDL relacionadas solo con vistas y seguridad

El grupo de SQL sin servidor no tiene almacenamiento local, solo se almacenan en las bases de datos los objetos de metadatos. Por tanto, no se admite T-SQL relacionado con los siguientes conceptos:

- Tablas
- Desencadenadores
- Vistas materializadas
- Instrucciones DDL distintas de las relacionadas con las vistas y la seguridad
- Instrucciones DML

### <a name="extensions"></a>Extensiones

Con el fin de permitir una experiencia fluida para la consulta en contexto de los datos que residen en el lago de datos, el grupo de SQL sin servidor amplía la función [OPENROWSET](/sql/t-sql/functions/openrowset-transact-sql?view=azure-sqldw-latest&preserve-view=true) existente con la incorporación de las siguientes funcionalidades:

[Consulta de varios archivos o carpetas](query-data-storage.md#query-multiple-files-or-folders)

[Formato de archivo PARQUET de consulta](query-data-storage.md#query-parquet-files)

[Formato DELTA de consulta](query-delta-lake-format.md)

[Varios formatos de texto delimitado (con terminadores de campo personalizados, terminadores de fila, caracteres de escape)](query-data-storage.md#query-csv-files)

[Almacén analítico de Azure Cosmos DB](query-cosmos-db-analytical-store.md?toc=%2Fazure%2Fsynapse-analytics%2Ftoc.json&bc=%2Fazure%2Fsynapse-analytics%2Fbreadcrumb%2Ftoc.json&tabs=openrowset-key)

[Lectura de un subconjunto de columnas elegido](query-data-storage.md#read-a-chosen-subset-of-columns)

[Inferencia de esquemas](query-data-storage.md#schema-inference)

[Función filename](query-data-storage.md#filename-function)

[Función filepath](query-data-storage.md#filepath-function)

[Uso con tipos complejos y estructuras de datos anidadas o repetidas](query-data-storage.md#work-with-complex-types-and-nested-or-repeated-data-structures)

## <a name="security"></a>Seguridad

El grupo de SQL sin servidor ofrece mecanismos para proteger el acceso a los datos.

### <a name="azure-active-directory-integration-and-multi-factor-authentication"></a>Integración de Azure Active Directory y autenticación multifactor

El grupo de SQL sin servidor permite administrar centralmente las identidades de usuario de base de datos y otros servicios de Microsoft con la [integración de Azure Active Directory](../../azure-sql/database/authentication-aad-configure.md). Esta funcionalidad simplifica la administración de permisos y mejora la seguridad. Azure Active Directory (Azure AD) admite la [autenticación multifactor](../../azure-sql/database/authentication-mfa-ssms-configure.md) (MFA) para aumentar la seguridad tanto de los datos como de las aplicaciones y admite un proceso de inicio de sesión único.

#### <a name="authentication"></a>Authentication

La autenticación del grupo de SQL sin servidor hace referencia a cómo prueban los usuarios su identidad al conectarse al punto de conexión. Se admiten dos tipos de autenticación:

- **Autenticación de SQL**

  Este método de autenticación utiliza un nombre de usuario y una contraseña.

- **Autenticación con Azure Active Directory**:

  Este método de autenticación usa identidades administradas por Azure Active Directory. Se puede habilitar la autenticación multifactor para los usuarios de Azure AD. Use la autenticación de Active Directory (seguridad integrada) [siempre que sea posible](/sql/relational-databases/security/choose-an-authentication-mode?view=azure-sqldw-latest&preserve-view=true).

#### <a name="authorization"></a>Authorization

La autorización hace referencia a las acciones que puede llevar a cabo un usuario en una base de datos del grupo de SQL sin servidor, algo que controlan los permisos de nivel de objeto y las pertenencias a roles de bases de datos de la cuenta de usuario.

Si se usa la autenticación de SQL, el usuario de SQL solo existe en el grupo de SQL sin servidor y los permisos se limitan a los objetos del grupo de SQL sin servidor. No se puede conceder directamente al usuario de SQL el acceso a los objetos protegibles de otros servicios (como Azure Storage), ya que solo existe en el ámbito del grupo de SQL sin servidor. El usuario de SQL debe usar uno de los [tipos de autorización admitidos](develop-storage-files-storage-access-control.md#supported-storage-authorization-types) para acceder a los archivos.

Si se usa la autenticación de Azure AD, un usuario puede iniciar sesión en el grupo de SQL sin servidor y en otros servicios, como Azure Storage, y se pueden conceder permisos al usuario de Azure AD.

### <a name="access-to-storage-accounts"></a>Acceso a las cuentas de almacenamiento

Un usuario que haya iniciado sesión en el servicio del grupo de SQL sin servidor debe estar autorizado para acceder a los archivos de Azure Storage y realizar consultas en ellos. El grupo de SQL sin servidor admite los siguientes tipos de autorización:

- La **[firma de acceso compartido (SAS)](develop-storage-files-storage-access-control.md?tabs=shared-access-signature)** ofrece acceso delegado a los recursos de una cuenta de almacenamiento. Con una SAS, puede conceder a los clientes acceso a los recursos de su cuenta de almacenamiento sin compartir las claves de la cuenta. Una SAS le ofrece un control detallado sobre el tipo de acceso que concede a los clientes que tienen una SAS: intervalo de validez, permisos concedidos, intervalo de direcciones IP aceptable y protocolo aceptable (https/http).

- La **[identidad de usuario](develop-storage-files-storage-access-control.md?tabs=user-identity)** (también conocida como "de tránsito") es un tipo de autorización en el que la identidad del usuario de Azure AD que inició sesión en el grupo de SQL sin servidor se usa para autorizar el acceso a los datos. Antes de acceder a los datos, el administrador de Azure Storage debe conceder permisos al usuario de Azure AD para acceder a los datos. Este tipo de autorización utiliza el usuario de Azure AD que inició sesión en el grupo de SQL sin servidor, por lo que no se admite para los tipos de usuario de SQL.

- **[Identidad del área de trabajo](develop-storage-files-storage-access-control.md?tabs=managed-identity)** es un tipo de autorización en el que se usa la identidad del área de trabajo de Synapse para autorizar el acceso a los datos. Antes de acceder a los datos, el administrador de Azure Storage debe conceder permisos a la identidad del área de trabajo para acceder a los datos.

### <a name="access-to-cosmos-db"></a>Acceso a Cosmos DB

Debe crear credenciales de nivel de servidor o en el ámbito de la base de datos con la clave de solo lectura de la cuenta de Cosmos DB para [acceder al almacén analítico de Cosmos DB](query-cosmos-db-analytical-store.md?toc=%2Fazure%2Fsynapse-analytics%2Ftoc.json&bc=%2Fazure%2Fsynapse-analytics%2Fbreadcrumb%2Ftoc.json&tabs=openrowset-key).

## <a name="next-steps"></a>Pasos siguientes
Puede encontrar información adicional sobre la conexión de punto de conexión y los archivos de consulta en los siguientes artículos: 
- [Conexión al punto de conexión](connect-overview.md)
- [Consulta de los archivos](develop-storage-files-overview.md)
