---
title: 'Inicio rápido: La primera consulta en Java'
description: En este inicio rápido, dará los pasos necesarios para habilitar los paquetes de Maven de Resource Graph para Java y ejecutará la primera consulta.
ms.date: 03/30/2021
ms.topic: quickstart
ms.custom: devx-track-java
ms.openlocfilehash: 04e32a3e147ed68248f7fead488a1f630ffcdb4d
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108751758"
---
# <a name="quickstart-run-your-first-resource-graph-query-using-java"></a>Inicio rápido: Ejecución de la primera consulta de Resource Graph con Java

El primer paso para usar Azure Resource Graph es comprobar que están instalados los paquetes necesarios para Java. Este inicio rápido lo guiará por el proceso de agregar los paquetes de Maven a la instalación de Java.

Cuando finalice, habrá agregado los paquetes de Maven a la instalación de Java y ejecutado la primera consulta de Resource Graph.

## <a name="prerequisites"></a>Requisitos previos

- Suscripción a Azure. Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

- Compruebe que está instalada la versión más reciente de la CLI de Azure (al menos la **2.21.0**). Si aún no está instalada, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

  > [!NOTE]
  > La CLI de Azure es necesaria para que el SDK de Azure para Java pueda usar la **autenticación basada en la CLI** en los ejemplos siguientes. Para ver más opciones de autenticación, consulte [Biblioteca cliente de identidad de Azure para Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/identity/azure-identity).

- El [kit para desarrolladores de Java](/azure/developer/java/fundamentals/java-jdk-long-term-support).
  8.

- [Apache Maven](https://maven.apache.org/), versión 3.6 o posterior.

## <a name="create-the-resource-graph-project"></a>Creación del proyecto de Resource Graph

Para habilitar Java para consultar Azure Resource Graph, cree y configure una aplicación con Maven e instale los paquetes de Maven necesarios.

1. Inicialice una nueva aplicación de Java llamada "argQuery" con un [arquetipo de Maven](https://maven.apache.org/guides/introduction/introduction-to-archetypes.html):

   ```cmd
   mvn -B archetype:generate -DarchetypeGroupId="org.apache.maven.archetypes" -DgroupId="com.Fabrikam" -DartifactId="argQuery"
   ```

1. Cambie los directorios a la nueva carpeta del proyecto `argQuery` y abra `pom.xml` en el editor que prefiera. Agregue los siguientes nodos `<dependency>` bajo el nodo `<dependencies>` existente:

   ```xml
    <dependency>
        <groupId>com.azure</groupId>
        <artifactId>azure-identity</artifactId>
        <version>1.2.4</version>
    </dependency>
    <dependency>
        <groupId>com.azure.resourcemanager</groupId>
        <artifactId>azure-resourcemanager-resourcegraph</artifactId>
        <version>1.0.0-beta.1</version>
    </dependency>
   ```

1. En el archivo `pom.xml`, agregue el siguiente nodo `<properties>` bajo el nodo `<project>` base para actualizar las versiones de origen y destino:

   ```xml
   <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
   </properties>
   ```

1. En el archivo `pom.xml`, agregue el siguiente nodo `<build>` bajo el nodo `<project>` base para configurar el objetivo y la clase principal del proyecto que se va a ejecutar.

   ```xml
   <build>
       <plugins>
           <plugin>
               <groupId>org.codehaus.mojo</groupId>
               <artifactId>exec-maven-plugin</artifactId>
               <version>1.2.1</version>
               <executions>
                   <execution>
                       <goals>
                           <goal>java</goal>
                       </goals>
                   </execution>
               </executions>
               <configuration>
                   <mainClass>com.Fabrikam.App</mainClass>
               </configuration>
           </plugin>
       </plugins>
   </build>
   ```

1. Reemplace el archivo `App.java` predeterminado en `\argQuery\src\main\java\com\Fabrikam` por el código siguiente y guarde el archivo actualizado:

   ```java
   package com.Fabrikam;

   import java.util.Arrays;
   import java.util.List;
   import com.azure.core.management.AzureEnvironment;
   import com.azure.core.management.profile.AzureProfile;
   import com.azure.identity.DefaultAzureCredentialBuilder;
   import com.azure.resourcemanager.resourcegraph.ResourceGraphManager;
   import com.azure.resourcemanager.resourcegraph.models.QueryRequest;
   import com.azure.resourcemanager.resourcegraph.models.QueryRequestOptions;
   import com.azure.resourcemanager.resourcegraph.models.QueryResponse;
   import com.azure.resourcemanager.resourcegraph.models.ResultFormat;

   public class App
   {
       public static void main( String[] args )
       {
           List<String> listSubscriptionIds = Arrays.asList(args[0]);
           String strQuery = args[1];

           ResourceGraphManager manager = ResourceGraphManager.authenticate(new DefaultAzureCredentialBuilder().build(), new AzureProfile(AzureEnvironment.AZURE));

           QueryRequest queryRequest = new QueryRequest()
               .withSubscriptions(listSubscriptionIds)
               .withQuery(strQuery);

           QueryResponse response = manager.resourceProviders().resources(queryRequest);

           System.out.println("Records: " + response.totalRecords());
           System.out.println("Data:\n" + response.data());
       }
   }
   ```

1. Compile la aplicación de consola `argQuery`.

   ```bash
   mvn package
   ```

## <a name="run-your-first-resource-graph-query"></a>Ejecutar la primera consulta de Resource Graph

Con la aplicación de consola de Java ya compilada, es el momento de probar una consulta simple de Resource Graph. La consulta devolverá los cinco primeros recursos de Azure con el **nombre** y el **tipo de recurso** de cada recurso.

En cada llamada a `argQuery`, hay variables que se usan que debe reemplazar por sus propios valores:

- `{subscriptionId}`: reemplácelo por el identificador de suscripción
- `{query}`: reemplácelo por la consulta de Azure Resource Graph.

1. Use la CLI de Azure para autenticarse con `az login`.

1. Cambie los directorios a la carpeta de proyecto de `argQuery` que creó con el comando `mvn -B archetype:generate` anterior.

1. Ejecute la primera consulta de Azure Resource Graph con Maven para compilar la aplicación de consola y pasar los argumentos. La propiedad `exec.args` identifica los argumentos por espacios. Para identificar la consulta como un solo argumento, se incluye entre comillas sencillas (`'`).

   ```bash
   mvn compile exec:java -Dexec.args "{subscriptionId} 'Resources | project name, type | limit 5'"
   ```

   > [!NOTE]
   > Como esta consulta de ejemplo no proporciona un modificador de ordenación como `order by`, es probable que al ejecutar esta consulta varias veces se produzca un conjunto diferente de recursos por solicitud.

1. Cambie el argumento por `argQuery.exe` y cambie la consulta por la acción `order by` de la propiedad **Nombre**:

   ```bash
   mvn compile exec:java -Dexec.args "{subscriptionId} 'Resources | project name, type | limit 5 | order by name asc'"
   ```

   > [!NOTE]
   > Al igual que con la primera consulta, es probable que al ejecutar esta consulta varias veces se produzca un conjunto diferente de recursos por solicitud. El orden de los comandos de consulta es importante. En este ejemplo, el `order by` viene después del `limit`. Este orden de comandos limita primero los resultados de la consulta y, luego, los ordena.

1. Cambie el parámetro final a `argQuery.exe` y cambie la consulta para realizar la operación `order by` primero en la propiedad **Nombre** y, luego, para realizar la operación `limit` a los cinco primeros resultados:

   ```bash
   mvn compile exec:java -Dexec.args "{subscriptionId} 'Resources | project name, type | order by name asc | limit 5'"
   ```

Cuando la consulta final se ejecuta varias veces, suponiendo que nada cambie en su entorno, los resultados devueltos serán coherentes y estarán ordenados por la propiedad **Nombre**, pero todavía limitados a los cinco primeros resultados.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere quitar la aplicación de consola de Java y los paquetes instalados, puede hacerlo eliminando la carpeta de proyecto de `argQuery`.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado una aplicación de consola de Java con los paquetes de Resource Graph necesarios y ha ejecutado su primera consulta. Para más información sobre el lenguaje de Resource Graph, vaya a la página de detalles del lenguaje de consulta.

> [!div class="nextstepaction"]
> [Obtener más información sobre el lenguaje de consulta](./concepts/query-language.md)
