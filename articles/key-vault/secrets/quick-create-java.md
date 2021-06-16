---
title: 'Inicio rápido: Biblioteca cliente de secretos de Azure Key Vault para Java'
description: Proporciona un inicio rápido para la biblioteca cliente de secretos de Azure Key Vault para Java.
author: msmbaldwin
ms.custom: devx-track-java, devx-track-azurecli, devx-track-azurepowershell
ms.author: mbaldwin
ms.date: 10/20/2019
ms.service: key-vault
ms.subservice: secrets
ms.topic: quickstart
ms.openlocfilehash: b8d1e962c9f25d2a26a8c8e537390028795844b9
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110671537"
---
# <a name="quickstart-azure-key-vault-secret-client-library-for-java"></a>Inicio rápido: Biblioteca cliente de secretos de Azure Key Vault para Java
Introducción a la biblioteca cliente de secretos de Azure Key Vault para Java. Siga estos pasos para instalar el paquete y probar el código de ejemplo para realizar tareas básicas.

Recursos adicionales:

* [Código fuente](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/keyvault/azure-security-keyvault-secrets)
* [Documentación de referencia de API](https://azure.github.io/azure-sdk-for-java/keyvault.html)
* [Documentación del producto](index.yml)
* [Muestras](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/keyvault/azure-security-keyvault-secrets/src/samples/java/com/azure/security/keyvault/secrets)

## <a name="prerequisites"></a>Requisitos previos
- Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Kit de desarrollo de Java (JDK)](/java/azure/jdk/), versión 8 o posterior
- [Apache Maven](https://maven.apache.org)
- [CLI de Azure](/cli/azure/install-azure-cli)

En esta guía de inicio rápido se supone que está ejecutando la [CLI de Azure](/cli/azure/install-azure-cli) y [Apache Maven](https://maven.apache.org) en una ventana de terminal de Linux.

## <a name="setting-up"></a>Instalación
En este inicio rápido se usa la biblioteca de identidades de Azure con la CLI de Azure para autenticar al usuario en Azure Services. Los desarrolladores también pueden usar Visual Studio o Visual Studio Code para autenticar sus llamadas. Para más información, consulte [Autenticación del cliente mediante la biblioteca cliente Azure Identity](/java/api/overview/azure/identity-readme).

### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure
1. Ejecute el comando `login`.

    ```azurecli-interactive
    az login
    ```

   Si la CLI puede abrir el explorador predeterminado, lo hará y cargará una página de inicio de sesión de Azure.

   En caso contrario, abra una página del explorador en [https://aka.ms/devicelogin](https://aka.ms/devicelogin) y escriba el código de autorización que se muestra en el terminal.

2. Inicie sesión con las credenciales de su cuenta en el explorador.

### <a name="create-a-new-java-console-app"></a>Creación de una aplicación de consola de Java
En una ventana de consola, utilice el comando `mvn` para crear una nueva aplicación de consola de Java con el nombre `akv-secrets-java`.

```console
mvn archetype:generate -DgroupId=com.keyvault.secrets.quickstart
                       -DartifactId=akv-secrets-java
                       -DarchetypeArtifactId=maven-archetype-quickstart
                       -DarchetypeVersion=1.4
                       -DinteractiveMode=false
```

La salida a partir de la generación del proyecto será similar a la siguiente:

```console
[INFO] ----------------------------------------------------------------------------
[INFO] Using following parameters for creating project from Archetype: maven-archetype-quickstart:1.4
[INFO] ----------------------------------------------------------------------------
[INFO] Parameter: groupId, Value: com.keyvault.secrets.quickstart
[INFO] Parameter: artifactId, Value: akv-secrets-java
[INFO] Parameter: version, Value: 1.0-SNAPSHOT
[INFO] Parameter: package, Value: com.keyvault.secrets.quickstart
[INFO] Parameter: packageInPathFormat, Value: com/keyvault/quickstart
[INFO] Parameter: package, Value: com.keyvault.secrets.quickstart
[INFO] Parameter: groupId, Value: com.keyvault.secrets.quickstart
[INFO] Parameter: artifactId, Value: akv-secrets-java
[INFO] Parameter: version, Value: 1.0-SNAPSHOT
[INFO] Project created from Archetype in dir: /home/user/quickstarts/akv-secrets-java
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  38.124 s
[INFO] Finished at: 2019-11-15T13:19:06-08:00
[INFO] ------------------------------------------------------------------------
```

Cambie el directorio a la carpeta `akv-secrets-java/` recién creada.

```console
cd akv-secrets-java
```

### <a name="install-the-package"></a>Instalar el paquete
Abra el archivo *pom.xml* en el editor de texto. Agregue los siguientes elementos de dependencia al grupo de dependencias.

```xml
    <dependency>
      <groupId>com.azure</groupId>
      <artifactId>azure-security-keyvault-secrets</artifactId>
      <version>4.2.3</version>
    </dependency>

    <dependency>
      <groupId>com.azure</groupId>
      <artifactId>azure-identity</artifactId>
      <version>1.2.0</version>
    </dependency>
```

### <a name="create-a-resource-group-and-key-vault"></a>Creación de un grupo de recursos y de un almacén de claves
[!INCLUDE [Create a resource group and key vault](../../../includes/key-vault-rg-kv-creation.md)]

#### <a name="grant-access-to-your-key-vault"></a>Concesión de acceso al almacén de claves
Cree una directiva de acceso para el almacén de claves que conceda permisos mediante secreto a la cuenta de usuario.

```console
az keyvault set-policy --name <your-key-vault-name> --upn user@domain.com --secret-permissions delete get list set purge
```

#### <a name="set-environment-variables"></a>Establecimiento de variables de entorno
Esta aplicación usa el nombre del almacén de claves como variable de entorno llamada `KEY_VAULT_NAME`.

Windows
```cmd
set KEY_VAULT_NAME=<your-key-vault-name>
````
Windows PowerShell
```powershell
$Env:KEY_VAULT_NAME="<your-key-vault-name>"
```

macOS o Linux
```cmd
export KEY_VAULT_NAME=<your-key-vault-name>
```

## <a name="object-model"></a>Modelo de objetos
La biblioteca cliente de secretos de Azure Key Vault para Java permite administrar los secretos. En la sección [Ejemplos de código](#code-examples) se muestra cómo crear un cliente y cómo establecer, recuperar y eliminar un secreto.

Toda la aplicación de consola se encuentra [ a continuación](#sample-code).

## <a name="code-examples"></a>Ejemplos de código
### <a name="add-directives"></a>Adición de directivas
Agregue las siguientes directivas al principio del código:

```java
import com.azure.core.util.polling.SyncPoller;
import com.azure.identity.DefaultAzureCredentialBuilder;

import com.azure.security.keyvault.secrets.SecretClient;
import com.azure.security.keyvault.secrets.SecretClientBuilder;
import com.azure.security.keyvault.secrets.models.DeletedSecret;
import com.azure.security.keyvault.secrets.models.KeyVaultSecret;
```

### <a name="authenticate-and-create-a-client"></a>Autenticación y creación de un cliente
En este inicio rápido se emplea el usuario que ha iniciado sesión para autenticarlo en Key Vault, que es el método preferido para el desarrollo local. En el caso de las aplicaciones implementadas en Azure, se debe asignar una identidad administrada a App Service o a la máquina virtual. Para más información, consulte [Introducción a la identidad administrada](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview).

En el ejemplo siguiente, el nombre del almacén de claves se expande al URI del almacén de claves, con el formato "https://\<your-key-vault-name\>.vault.azure.net". En este ejemplo se usa la clase ["DefaultAzureCredential()"](https://docs.microsoft.com/java/api/com.azure.identity.defaultazurecredential), que permite usar el mismo código en entornos diferentes con distintas opciones para proporcionar la identidad. Para más información, consulte [Autenticación mediante las credenciales predeterminadas de Azure](https://docs.microsoft.com/java/api/overview/azure/identity-readme).

```java
String keyVaultName = System.getenv("KEY_VAULT_NAME");
String keyVaultUri = "https://" + keyVaultName + ".vault.azure.net";

SecretClient secretClient = new SecretClientBuilder()
    .vaultUrl(keyVaultUri)
    .credential(new DefaultAzureCredentialBuilder().build())
    .buildClient();
```

### <a name="save-a-secret"></a>Almacenamiento de un secreto
Ahora que la aplicación se ha autenticado, se puede colocar un secreto en el almacén de claves mediante el método `secretClient.setSecret`. Esto requiere un nombre para el secreto; en este ejemplo, hemos asignado el valor "mySecret" a la variable `secretName`.

```java
secretClient.setSecret(new KeyVaultSecret(secretName, secretValue));
```

Puede comprobar que el secreto se ha establecido con el comando [az keyvault secret show](/cli/azure/keyvault/secret?#az_keyvault_secret_show):

```azurecli
az keyvault secret show --vault-name <your-unique-key-vault-name> --name mySecret
```

### <a name="retrieve-a-secret"></a>Recuperación de un secreto
Ya puede recuperar el secreto previamente establecido con el método `secretClient.getSecret`.

```java
KeyVaultSecret retrievedSecret = secretClient.getSecret(secretName);
 ```

Ahora puede acceder al valor del secreto recuperado con `retrievedSecret.getValue()`.

### <a name="delete-a-secret"></a>eliminar un secreto
Por último, vamos a eliminar el secreto del almacén de claves con el método `secretClient.beginDeleteSecret`.

La eliminación de secretos es una operación de larga duración cuyo progreso puede sondear, o bien puede esperar a que se complete.

```java
SyncPoller<DeletedSecret, Void> deletionPoller = secretClient.beginDeleteSecret(secretName);
deletionPoller.waitForCompletion();
```

Puede comprobar que el secreto se ha eliminado con el comando [az keyvault secret show](/cli/azure/keyvault/secret?#az_keyvault_secret_show):

```azurecli
az keyvault secret show --vault-name <your-unique-key-vault-name> --name mySecret
```

## <a name="clean-up-resources"></a>Limpieza de recursos
Cuando ya no lo necesite, puede usar la CLI de Azure o Azure PowerShell para quitar el almacén de claves y el grupo de recursos correspondiente.

```azurecli
az group delete -g "myResourceGroup"
```

```azurepowershell
Remove-AzResourceGroup -Name "myResourceGroup"
```

## <a name="sample-code"></a>Código de ejemplo
```java
package com.keyvault.secrets.quickstart;

import java.io.Console;

import com.azure.core.util.polling.SyncPoller;
import com.azure.identity.DefaultAzureCredentialBuilder;

import com.azure.security.keyvault.secrets.SecretClient;
import com.azure.security.keyvault.secrets.SecretClientBuilder;
import com.azure.security.keyvault.secrets.models.DeletedSecret;
import com.azure.security.keyvault.secrets.models.KeyVaultSecret

public class App {
    public static void main(String[] args) throws InterruptedException, IllegalArgumentException {
        String keyVaultName = System.getenv("KEY_VAULT_NAME");
        String keyVaultUri = "https://" + keyVaultName + ".vault.azure.net";

        System.out.printf("key vault name = %s and key vault URI = %s \n", keyVaultName, keyVaultUri);

        SecretClient secretClient = new SecretClientBuilder()
            .vaultUrl(keyVaultUri)
            .credential(new DefaultAzureCredentialBuilder().build())
            .buildClient();

        Console con = System.console();  

        String secretName = "mySecret";

        System.out.println("Please provide the value of your secret > ");
        
        String secretValue = con.readLine();

        System.out.print("Creating a secret in " + keyVaultName + " called '" + secretName + "' with value '" + secretValue + "` ... ");

        secretClient.setSecret(new KeyVaultSecret(secretName, secretValue));

        System.out.println("done.");
        System.out.println("Forgetting your secret.");
        
        secretValue = "";
        System.out.println("Your secret's value is '" + secretValue + "'.");

        System.out.println("Retrieving your secret from " + keyVaultName + ".");

        KeyVaultSecret retrievedSecret = secretClient.getSecret(secretName);

        System.out.println("Your secret's value is '" + retrievedSecret.getValue() + "'.");
        System.out.print("Deleting your secret from " + keyVaultName + " ... ");

        SyncPoller<DeletedSecret, Void> deletionPoller = secretClient.beginDeleteSecret(secretName);
        deletionPoller.waitForCompletion();

        System.out.println("done.");
    }
}
```

## <a name="next-steps"></a>Pasos siguientes
En este inicio rápido, ha creado un almacén de claves, ha almacenado un secreto en él, lo ha recuperado y, después, lo ha eliminado. Para más información sobre Key Vault y cómo integrarlo con las aplicaciones, continúe con los artículos siguientes.

- Lea una [introducción a Azure Key Vault](../general/overview.md).
- Consulte la [guía del desarrollador de Azure Key Vault](../general/developers-guide.md).
- Procedimientos para [proteger el acceso a un almacén de claves](../general/security-features.md)
