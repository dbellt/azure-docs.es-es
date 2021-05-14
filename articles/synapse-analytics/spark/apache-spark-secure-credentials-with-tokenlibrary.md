---
title: Protección de credenciales de acceso con servicios vinculados en Apache Spark para Azure Synapse Analytics
description: En este artículo se proporcionan los conceptos necesarios para la integración de forma segura de Apache Spark para Azure Synapse Analytics con otros servicios mediante servicios vinculados y la biblioteca de tokens
services: synapse-analytics
author: mlee3gsd
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: spark
ms.date: 11/19/2020
ms.author: martinle
ms.reviewer: nirav
zone_pivot_groups: programming-languages-spark-all-minus-sql
ms.openlocfilehash: a5355a449cb55f671c59da55ba4d038e9b4b2ec5
ms.sourcegitcommit: 2cb7772f60599e065fff13fdecd795cce6500630
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108803429"
---
# <a name="secure-credentials-with-linked-services-using-the-tokenlibrary"></a>Protección de credenciales con servicios vinculados mediante TokenLibrary

El acceso a los datos desde orígenes externos es un patrón común. Salvo que el origen de datos externo permita el acceso anónimo, lo más probable es que tenga que proteger la conexión con una credencial, un secreto o una cadena de conexión.  

Synapse usa el acceso directo de Azure Active Directory (AAD) de forma predeterminada para la autenticación entre los recursos.  Si necesita conectarse a un recurso con otras credenciales, use directamente TokenLibrary.  TokenLibrary simplifica el proceso de recuperar tokens de SAS, tokens de AAD, cadenas de conexión y secretos almacenados en un servicio vinculado o en una instancia de Azure Key Vault.

Al recuperar secretos de Azure Key Vault, se recomienda crear un servicio vinculado a Azure Key Vault.  Asegúrese de que la identidad de servicio administrada (MSI) del área de trabajo de Synapse tenga privilegios para la obtención de secretos en Azure Key Vault.  Synapse se autenticará en Azure Key Vault mediante la identidad de servicio administrada del área de trabajo de Synapse. Si se conecta directamente a Azure Key Vault sin un servicio vinculado, se autenticará con su credencial de Azure Active Directory de usuario.

Para más información, consulte la información sobre los [servicios vinculados](../../data-factory/concepts-linked-services.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

## <a name="usage"></a>Uso

### <a name="tokenlibraryhelp"></a>TokenLibrary.help()
Esta función muestra la documentación de ayuda de TokenLibrary.

::: zone pivot = "programming-language-scala"

```scala
TokenLibrary.help()
```

::: zone-end

::: zone pivot = "programming-language-python"

```python
TokenLibrary.help()
```

::: zone-end

::: zone pivot = "programming-language-csharp"

```csharp
Console.WriteLine(TokenLibrary.help());
```

::: zone-end

## <a name="tokenlibrary-for-azure-data-lake-storage-gen2"></a>TokenLibrary para Azure Data Lake Storage Gen2

#### <a name="adls-gen2-primary-storage"></a>Almacenamiento principal de ADLS Gen2

Para acceder a los archivos desde la instancia principal de Azure Data Lake Storage, se usa el acceso directo de Azure Active Directory para la autenticación de forma predeterminada y no se requiere el uso explícito de TokenLibrary.

::: zone pivot = "programming-language-scala"

```scala
val df = spark.read.csv("abfss://<CONTAINER>@<ACCOUNT>.dfs.core.windows.net/<FILE PATH>")
display(df.limit(10))
```

::: zone-end

::: zone pivot = "programming-language-python"

```python
df = spark.read.csv('abfss://<CONTAINER>@<ACCOUNT>.dfs.core.windows.net/<FILE PATH>')
display(df.limit(10))
```

::: zone-end

#### <a name="adls-gen2-storage-with-linked-services"></a>Almacenamiento de ADLS Gen2 con servicios vinculados

Synapse ofrece una experiencia integrada de servicios vinculados al conectarse a Azure Data Lake Storage Gen2.  Los servicios vinculados se pueden configurar para autenticarse con una **clave de cuenta**, una **entidad de servicio**, una **identidad administrada** o una **credencial**.

Cuando el método de autenticación del servicio vinculado se establece en **Clave de cuenta**, el servicio vinculado se autenticará con la clave de la cuenta de almacenamiento proporcionada, solicitará una clave SAS y la aplicará automáticamente a la solicitud de almacenamiento mediante **LinkedServiceBasedSASProvider**.

::: zone pivot = "programming-language-scala"

```scala
val sc = spark.sparkContext
spark.conf.set("spark.storage.synapse.linkedServiceName", "<LINKED SERVICE NAME>")
spark.conf.set("fs.azure.account.auth.type", "SAS")
spark.conf.set("fs.azure.sas.token.provider.type", "com.microsoft.azure.synapse.tokenlibrary.LinkedServiceBasedSASProvider")

val df = spark.read.csv("abfss://<CONTAINER>@<ACCOUNT>.dfs.core.windows.net/<FILE PATH>")

display(df.limit(10))
```

::: zone-end

::: zone pivot = "programming-language-python"

```python
%%pyspark
# Python code
spark.conf.set("spark.storage.synapse.linkedServiceName", "<lINKED SERVICE NAME>")
spark.conf.set("fs.azure.account.auth.type", "SAS")
spark.conf.set("fs.azure.sas.token.provider.type", "com.microsoft.azure.synapse.tokenlibrary.LinkedServiceBasedSASProvider")

df = spark.read.csv('abfss://<CONTAINER>@<ACCOUNT>.dfs.core.windows.net/<DIRECTORY PATH>')

df.show()
```

::: zone-end

Cuando el método de autenticación del servicio vinculado está establecido en **Identidad administrada** o **Entidad de servicio**, el servicio vinculado usará el token de la identidad administrada o la entidad de servicio con el proveedor **LinkedServiceBasedTokenProvider**.  


::: zone pivot = "programming-language-scala"

```scala
val sc = spark.sparkContext
spark.conf.set("spark.storage.synapse.linkedServiceName", "<LINKED SERVICE NAME>")
spark.conf.set("fs.azure.account.oauth.provider.type", "com.microsoft.azure.synapse.tokenlibrary.LinkedServiceBasedTokenProvider") 
val df = spark.read.csv("abfss://<CONTAINER>@<ACCOUNT>.dfs.core.windows.net/<FILE PATH>")

display(df.limit(10))
```

::: zone-end

::: zone pivot = "programming-language-python"

```python
%%pyspark
# Python code
spark.conf.set("spark.storage.synapse.linkedServiceName", "<lINKED SERVICE NAME>")
spark.conf.set("fs.azure.account.oauth.provider.type", "com.microsoft.azure.synapse.tokenlibrary.LinkedServiceBasedTokenProvider")

df = spark.read.csv('abfss://<CONTAINER>@<ACCOUNT>.dfs.core.windows.net/<DIRECTORY PATH>')

df.show()
```

::: zone-end

#### <a name="adls-gen2-storage-without-linked-services"></a>Almacenamiento de ADLS Gen2 (sin servicios vinculados)

Conéctese al almacenamiento de ADLS Gen2 directamente mediante una clave SAS, use **ConfBasedSASProvider** y proporcione la clave SAS para la opción de configuración **spark.storage.synapse.sas**.

::: zone pivot = "programming-language-scala"

```scala
%%spark
spark.conf.set("fs.azure.account.auth.type", "SAS")
spark.conf.set("fs.azure.sas.token.provider.type", "com.microsoft.azure.synapse.tokenlibrary.ConfBasedSASProvider")
spark.conf.set("spark.storage.synapse.sas", "<SAS KEY>")

val df = spark.read.csv("abfss://<CONTAINER>@<ACCOUNT>.dfs.core.windows.net/<FILE PATH>")

display(df.limit(10))
```

::: zone-end

::: zone pivot = "programming-language-python"

```python
%%pyspark

spark.conf.set("fs.azure.account.auth.type", "SAS")
spark.conf.set("fs.azure.sas.token.provider.type", "com.microsoft.azure.synapse.tokenlibrary.ConfBasedSASProvider")
spark.conf.set("spark.storage.synapse.sas", "<SAS KEY>")

df = spark.read.csv('abfss://<CONTAINER>@<ACCOUNT>.dfs.core.windows.net/<FILE PATH>')

display(df.limit(10))
```

::: zone-end

#### <a name="adls-gen2-storage-with-azure-key-vault"></a>Almacenamiento de ADLS Gen2 con Azure Key Vault

Conéctese al almacenamiento de ADLS Gen2 mediante un token de SAS almacenado en un secreto de Azure Key Vault.  

::: zone pivot = "programming-language-scala"

```scala
%%spark
spark.conf.set("fs.azure.account.auth.type", "SAS")
spark.conf.set("fs.azure.sas.token.provider.type", "com.microsoft.azure.synapse.tokenlibrary.AkvBasedSASProvider")
spark.conf.set("spark.storage.synapse.akv", "<AZURE KEY VAULT NAME>")
spark.conf.set("spark.storage.akv.secret", "<SECRET KEY>")

val df = spark.read.csv("abfss://<CONTAINER>@<ACCOUNT>.dfs.core.windows.net/<FILE PATH>")

display(df.limit(10))
```

::: zone-end

::: zone pivot = "programming-language-python"

```python
%%pyspark
spark.conf.set("fs.azure.account.auth.type", "SAS")
spark.conf.set("fs.azure.sas.token.provider.type", "com.microsoft.azure.synapse.tokenlibrary.AkvBasedSASProvider")
spark.conf.set("spark.storage.synapse.akv", "<AZURE KEY VAULT NAME>")
spark.conf.set("spark.storage.akv.secret", "<SECRET KEY>")

df = spark.read.csv('abfss://<CONTAINER>@<ACCOUNT>.dfs.core.windows.net/<FILE PATH>')

display(df.limit(10))
```

::: zone-end

## <a name="tokenlibrary-for-other-linked-services"></a>TokenLibrary para otros servicios vinculados

Para conectarse a otros servicios vinculados, puede realizar una llamada directa a TokenLibrary.

#### <a name="getconnectionstring"></a>getConnectionString()

 Para recuperar la cadena de conexión, use la función **getConnectionString** y pase el **nombre del servicio vinculado**.

::: zone pivot = "programming-language-scala"

```scala
%%spark
// retrieve connectionstring from TokenLibrary

import com.microsoft.azure.synapse.tokenlibrary.TokenLibrary

val connectionString: String = TokenLibrary.getConnectionString("<LINKED SERVICE NAME>")
println(connectionString)
```

::: zone-end

::: zone pivot = "programming-language-python"

```python
%%pyspark
# retrieve connectionstring from TokenLibrary

from pyspark.sql import SparkSession

sc = SparkSession.builder.getOrCreate()
token_library = sc._jvm.com.microsoft.azure.synapse.tokenlibrary.TokenLibrary
connection_string = token_library.getConnectionString("<LINKED SERVICE NAME>")
print(connection_string)
```

::: zone-end

::: zone pivot = "programming-language-csharp"

```csharp
%%csharp
// retrieve connectionstring from TokenLibrary

using Microsoft.Spark.Extensions.Azure.Synapse.Analytics.Utils;

string connectionString = TokenLibrary.GetConnectionString(<LINKED SERVICE NAME>);
Console.WriteLine(connectionString);
```

::: zone-end

#### <a name="getconnectionstringasmap"></a>getConnectionStringAsMap()

getConnectionStringAsMap es una función auxiliar disponible en Scala y Python para analizar valores específicos de un par _key=value_ en la cadena de conexión, por ejemplo:

_`DefaultEndpointsProtocol=https;AccountName=<ACCOUNT NAME>;AccountKey=<ACCOUNT KEY>`_

use la función **getConnectionStringAsMap** y pase la clave para devolver el valor.  En el ejemplo de cadena de conexión anterior, 

_**TokenLibrary.getConnectionStringAsMap("DefaultEndpointsProtocol")**_

devolvería

**_"https"_**

::: zone pivot = "programming-language-scala"

```scala
// Linked services can be used for storing and retrieving credentials (e.g, account key)
// Example connection string (for storage): "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
import com.microsoft.azure.synapse.tokenlibrary.TokenLibrary

val accountKey: String = TokenLibrary.getConnectionStringAsMap("<LINKED SERVICE NAME">).get("<KEY NAME>")
println(accountKey)
```
::: zone-end

::: zone pivot = "programming-language-python"

```python
# Linked services can be used for storing and retrieving credentials (e.g, account key)
# Example connection string (for storage): "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
from pyspark.sql import SparkSession

sc = SparkSession.builder.getOrCreate()
token_library = sc._jvm.com.microsoft.azure.synapse.tokenlibrary.TokenLibrary
accountKey = token_library.getConnectionStringAsMap("<LINKED SERVICE NAME>").get("<KEY NAME>")
print(accountKey)
```

::: zone-end

#### <a name="getsecret"></a>getSecret()

Para recuperar un secreto almacenado de Azure Key Vault, se recomienda crear un servicio vinculado a Azure Key Vault en el área de trabajo de Synapse. La identidad de servicio administrada del área de trabajo de Synapse deberá tener concedidos permisos para **obtener secretos** en Azure Key Vault.  El servicio vinculado usará la identidad de servicio administrada para conectarse a un servicio de Azure Key Vault y recuperar el secreto.  En caso contrario, al conectarse directamente a Azure Key Vault, se usará la credencial de Azure Active Directory (AAD) del usuario.  En este caso, será necesario conceder al usuario los permisos para obtener secretos en Azure Key Vault.

`TokenLibrary.getSecret("<AZURE KEY VAULT NAME>", "<SECRET KEY>" [, <LINKED SERVICE NAME>])`

Para recuperar un secreto de Azure Key Vault, utilice la función **TokenLibrary.getSecret()** .

::: zone pivot = "programming-language-scala"

```scala
import com.microsoft.azure.synapse.tokenlibrary.TokenLibrary

val connectionString: String = TokenLibrary.getSecret("<AZURE KEY VAULT NAME>", "<SECRET KEY>", "<LINKED SERVICE NAME>")
println(connectionString)
```

::: zone-end

::: zone pivot = "programming-language-python"

```python
import sys
from pyspark.sql import SparkSession

sc = SparkSession.builder.getOrCreate()
token_library = sc._jvm.com.microsoft.azure.synapse.tokenlibrary.TokenLibrary

connection_string = token_library.getSecret("<AZURE KEY VAULT NAME>", "<SECRET KEY>", "<LINKED SERVICE NAME>")
print(connection_string)
```

::: zone-end

::: zone pivot = "programming-language-csharp"

```csharp
using Microsoft.Spark.Extensions.Azure.Synapse.Analytics.Utils;

string connectionString = TokenLibrary.getSecret("<AZURE KEY VAULT NAME>", "<SECRET KEY>", "<LINKED SERVICE NAME>");
Console.WriteLine(connectionString);
```

::: zone-end

## <a name="next-steps"></a>Pasos siguientes

- [Escritura en un grupo de SQL dedicado](./synapse-spark-sql-pool-import-export.md)
