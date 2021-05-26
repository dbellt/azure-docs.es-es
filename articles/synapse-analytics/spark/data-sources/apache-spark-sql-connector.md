---
title: Azure SQL y SQL Server
description: En este artículo se proporciona información sobre cómo usar el conector para mover datos entre Azure MS SQL y grupos de Apache Spark sin servidor.
services: synapse-analytics
author: midesa
ms.author: midesa
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: spark
ms.date: 05/19/2020
ms.openlocfilehash: 82518e6c8f4901ffe6c5503d902359a79c12e0b0
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110081683"
---
# <a name="azure-sql-database-and-sql-server-connector-for-apache-spark"></a>Conector de Azure SQL Database y SQL Server para Apache Spark
El conector de Apache Spark para Azure SQL Database y SQL Server permite que estas bases de datos actúen como orígenes de datos de entrada y receptores de datos de salida en los trabajos de Apache Spark. Permite usar datos transaccionales en tiempo real en análisis de macrodatos y conservar los resultados para informes o consultas ad hoc.

En comparación con el conector JDBC integrado, este conector proporciona la capacidad para insertar datos de forma masiva en bases de datos SQL. Puede mejorar el rendimiento de la inserción de fila en fila, ya que puede insertar datos entre 10 y 20 veces más rápido. El conector de Spark para SQL Server y Azure SQL Database también admite la [autenticación](/sql/connect/spark/connector#azure-active-directory-authentication)de Azure Active Directory (Azure AD), lo que permite conectarse de forma segura a las bases de datos de Azure SQL Database desde Azure Synapse Analytics. 

En este artículo se explica cómo usar DataFrame API para conectarse a bases de datos SQL mediante el conector de MS SQL. En este artículo se proporcionan ejemplos detallados del uso de PySpark API. Para ver todos los ejemplos y argumentos admitidos para conectarse a bases de datos SQL mediante el conector de MS SQL, consulte [Ejemplos de Azure Data SQL.](https://github.com/microsoft/sql-server-samples#azure-data-sql-samples-repository)


  
## <a name="connection-details"></a>Detalles de conexión
En este ejemplo, usaremos las utilidades de Spark para Microsoft para facilitar la adquisición de secretos de una instancia de Key Vault preconfigurada. Para más información sobre las utilidades de Spark para Microsoft, visite [Introducción a las utilidades de Spark para Microsoft](../microsoft-spark-utilities.md).

```python
servername = "<< server name >>"
dbname = "<< database name >>"
url = servername + ";" + "databaseName=" + dbname + ";"
dbtable = "<< table name >> "
user = "<< username >>" 
password = mssparkutils.credentials.getSecret('azure key vault name','secret name')
```

> [!NOTE]
> Actualmente, no hay ningún servicio vinculado ni compatibilidad de paso a través de AAD con el conector de Azure SQL.

## <a name="use-the-azure-sql-and-sql-server-connector"></a>Uso del conector de Azure SQL y SQL Server

### <a name="read-data"></a>Lectura de datos
```python
#Read from SQL table using MS SQL Connector
print("read data from SQL server table  ")
jdbcDF = spark.read \
        .format("com.microsoft.sqlserver.jdbc.spark") \
        .option("url", url) \
        .option("dbtable", dbtable) \
        .option("user", user) \
        .option("password", password).load()

jdbcDF.show(5)
```

### <a name="write-data"></a>Escritura de datos
```python
try:
  df.write \
    .format("com.microsoft.sqlserver.jdbc.spark") \
    .mode("overwrite") \
    .option("url", url) \
    .option("dbtable", dbtable) \
    .option("user", user) \
    .option("password", password) \
    .save()
except ValueError as error :
    print("MSSQL Connector write failed", error)

print("MSSQL Connector write(overwrite) succeeded  ")
```
### <a name="append-data"></a>Anexión de datos
```python
try:
  df.write \
    .format("com.microsoft.sqlserver.jdbc.spark") \
    .mode("append") \
    .option("url", url) \
    .option("dbtable", table_name) \
    .option("user", username) \
    .option("password", password) \
    .save()
except ValueError as error :
    print("Connector write failed", error)
```

## <a name="azure-active-directory-authentication"></a>Autenticación con Azure Active Directory

### <a name="python-example-with-service-principal"></a>Ejemplo de Python con una entidad de servicio
```python
import adal

# Located in App Registrations from Azure Portal
tenant_id = "<< tenant id >> "

# Located in App Registrations from Azure Portal
resource_app_id_url = "https://database.windows.net/"

# Authority
authority = "https://login.windows.net/" + tenant_id

context = adal.AuthenticationContext(authority)
token = context.acquire_token_with_client_credentials(resource_app_id_url, service_principal_id, service_principal_secret)
access_token = token["accessToken"]

jdbc_df = spark.read \
        .format("com.microsoft.sqlserver.jdbc.spark") \
        .option("url", url) \
        .option("dbtable", dbtable) \
        .option("accessToken", access_token) \
        .option("encrypt", "true") \
        .option("hostNameInCertificate", "*.database.windows.net") \
        .load()
```

### <a name="python-example-with-active-directory-password"></a>Ejemplo de Python con una contraseña de Active Directory
```python
jdbc_df = spark.read \
        .format("com.microsoft.sqlserver.jdbc.spark") \
        .option("url", url) \
        .option("dbtable", table_name) \
        .option("authentication", "ActiveDirectoryPassword") \
        .option("user", user_name) \
        .option("password", password) \
        .option("encrypt", "true") \
        .option("hostNameInCertificate", "*.database.windows.net") \
        .load()
```

## <a name="next-steps"></a>Pasos siguientes
- [Más información sobre el conector de SQL Server y Azure SQL](/sql/connect/spark/connector)
- [Visualización de ejemplos de Azure Data SQL](https://github.com/microsoft/sql-server-samples)
