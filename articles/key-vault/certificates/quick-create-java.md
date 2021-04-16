---
title: 'Inicio rápido para la biblioteca cliente de certificados de Azure Key Vault: Java'
description: Obtenga información sobre la biblioteca cliente de certificados de Azure Key Vault para Java con los pasos de esta guía de inicio rápido.
author: msmbaldwin
ms.custom: devx-track-java
ms.author: mbaldwin
ms.date: 12/18/2020
ms.service: key-vault
ms.subservice: certificates
ms.topic: quickstart
ms.openlocfilehash: db69258a774343af18e683444d22530a32f85555
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107374954"
---
# <a name="quickstart-azure-key-vault-certificate-client-library-for-java-certificates"></a>Inicio rápido: Biblioteca cliente de certificados de Azure Key Vault para Java (certificados)
Empiece a trabajar con la biblioteca cliente de certificados de Azure Key Vault para Java. Siga estos pasos para instalar el paquete y probar el código de ejemplo para realizar tareas básicas.

Recursos adicionales:

* [Código fuente](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/keyvault/azure-security-keyvault-certificates)
* [Documentación de referencia de API](https://azure.github.io/azure-sdk-for-java/keyvault.html)
* [Documentación del producto](index.yml)
* [Muestras](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/keyvault/azure-security-keyvault-certificates/src/samples/java/com/azure/security/keyvault/certificates)

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
En una ventana de consola, utilice el comando `mvn` para crear una nueva aplicación de consola de Java con el nombre `akv-certificates-java`.

```console
mvn archetype:generate -DgroupId=com.keyvault.certificates.quickstart
                       -DartifactId=akv-certificates-java
                       -DarchetypeArtifactId=maven-archetype-quickstart
                       -DarchetypeVersion=1.4
                       -DinteractiveMode=false
```

La salida a partir de la generación del proyecto será similar a la siguiente:

```console
[INFO] ----------------------------------------------------------------------------
[INFO] Using following parameters for creating project from Archetype: maven-archetype-quickstart:1.4
[INFO] ----------------------------------------------------------------------------
[INFO] Parameter: groupId, Value: com.keyvault.certificates.quickstart
[INFO] Parameter: artifactId, Value: akv-certificates-java
[INFO] Parameter: version, Value: 1.0-SNAPSHOT
[INFO] Parameter: package, Value: com.keyvault.certificates.quickstart
[INFO] Parameter: packageInPathFormat, Value: com/keyvault/quickstart
[INFO] Parameter: package, Value: com.keyvault.certificates.quickstart
[INFO] Parameter: groupId, Value: com.keyvault.certificates.quickstart
[INFO] Parameter: artifactId, Value: akv-certificates-java
[INFO] Parameter: version, Value: 1.0-SNAPSHOT
[INFO] Project created from Archetype in dir: /home/user/quickstarts/akv-certificates-java
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  38.124 s
[INFO] Finished at: 2019-11-15T13:19:06-08:00
[INFO] ------------------------------------------------------------------------
```

Cambie el directorio a la carpeta `akv-certificates-java/` recién creada.

```console
cd akv-certificates-java
```

### <a name="install-the-package"></a>Instalar el paquete
Abra el archivo *pom.xml* en el editor de texto. Agregue los siguientes elementos de dependencia al grupo de dependencias.

```xml
    <dependency>
      <groupId>com.azure</groupId>
      <artifactId>azure-security-keyvault-certificates</artifactId>
      <version>4.1.3</version>
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
Cree una directiva de acceso para el almacén de claves que conceda permisos de certificado a la cuenta de su usuario.

```console
az keyvault set-policy --name <your-key-vault-name> --upn user@domain.com --certificate-permissions delete get list create purge
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
La biblioteca cliente de certificados de Azure Key Vault para Java permite administrar los certificados. En la sección [Ejemplos de código](#code-examples) se muestra no solo cómo crear un cliente, sino también cómo crear, recuperar y eliminar un certificado.

Toda la aplicación de consola se encuentra [ a continuación](#sample-code).

## <a name="code-examples"></a>Ejemplos de código
### <a name="add-directives"></a>Adición de directivas
Agregue las siguientes directivas al principio del código:

```java
import com.azure.core.util.polling.SyncPoller;
import com.azure.identity.DefaultAzureCredentialBuilder;

import com.azure.security.keyvault.certificates.CertificateClient;
import com.azure.security.keyvault.certificates.CertificateClientBuilder;
import com.azure.security.keyvault.certificates.models.CertificateOperation;
import com.azure.security.keyvault.certificates.models.CertificatePolicy;
import com.azure.security.keyvault.certificates.models.DeletedCertificate;
import com.azure.security.keyvault.certificates.models.KeyVaultCertificate;
import com.azure.security.keyvault.certificates.models.KeyVaultCertificateWithPolicy;
```

### <a name="authenticate-and-create-a-client"></a>Autenticación y creación de un cliente
En este inicio rápido se emplea el usuario que ha iniciado sesión para autenticarlo en Key Vault, que es el método preferido para el desarrollo local. En el caso de las aplicaciones implementadas en Azure, se debe asignar una identidad administrada a App Service o a la máquina virtual. Para más información, consulte [Introducción a la identidad administrada](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview).

En el ejemplo siguiente, el nombre del almacén de claves se expande al URI del almacén de claves, con el formato "https://\<your-key-vault-name\>.vault.azure.net". En este ejemplo se usa la clase ["DefaultAzureCredential()"](https://docs.microsoft.com/java/api/com.azure.identity.defaultazurecredential), que permite usar el mismo código en entornos diferentes con distintas opciones para proporcionar la identidad. Para más información, consulte [Autenticación mediante las credenciales predeterminadas de Azure](https://docs.microsoft.com/java/api/overview/azure/identity-readme).

```java
String keyVaultName = System.getenv("KEY_VAULT_NAME");
String keyVaultUri = "https://" + keyVaultName + ".vault.azure.net";

CertificateClient certificateClient = new CertificateClientBuilder()
    .vaultUrl(keyVaultUri)
    .credential(new DefaultAzureCredentialBuilder().build())
    .buildClient();
```

### <a name="save-a-secret"></a>Almacenamiento de un secreto
Ahora que la aplicación se ha autenticado, puede crear un certificado en el almacén de claves mediante el método `certificateClient.beginCreateCertificate`. Esto requiere un nombre para el certificado y una directiva de certificados (hemos asignado el valor "myCertificate" a la variable `certificateName` en este ejemplo y usado una directiva predeterminada).

La creación de certificados es una operación de larga duración cuyo progreso puede sondear, o bien puede esperar a que se complete.

```java
SyncPoller<CertificateOperation, KeyVaultCertificateWithPolicy> certificatePoller =
    certificateClient.beginCreateCertificate(certificateName, CertificatePolicy.getDefault());
certificatePoller.waitForCompletion();
```

Puede obtener el certificado una vez que la creación se ha completado a través de la siguiente llamada:

```java
KeyVaultCertificate createdCertificate = certificatePoller.getFinalResult();
```

### <a name="retrieve-a-certificate"></a>Recuperación de un certificado
Ahora puede recuperar el certificado creado anteriormente con el método `certificateClient.getCertificate`.

```java
KeyVaultCertificate retrievedCertificate = certificateClient.getCertificate(certificateName);
 ```

Ahora puede acceder a los detalles del certificado recuperado con operaciones como `retrievedCertificate.getName`, `retrievedCertificate.getProperties`, etc. Así como su contenido `retrievedCertificate.getCer`.

### <a name="delete-a-certificate"></a>Eliminación de un certificado
Por último, vamos a eliminar el certificado del almacén de claves con el método `certificateClient.beginDeleteCertificate`, que también es una operación de larga duración.

```java
SyncPoller<DeletedCertificate, Void> deletionPoller = certificateClient.beginDeleteCertificate(certificateName);
deletionPoller.waitForCompletion();
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
package com.keyvault.certificates.quickstart;

import com.azure.core.util.polling.SyncPoller;
import com.azure.identity.DefaultAzureCredentialBuilder;

import com.azure.security.keyvault.certificates.CertificateClient;
import com.azure.security.keyvault.certificates.CertificateClientBuilder;
import com.azure.security.keyvault.certificates.models.CertificateOperation;
import com.azure.security.keyvault.certificates.models.CertificatePolicy;
import com.azure.security.keyvault.certificates.models.DeletedCertificate;
import com.azure.security.keyvault.certificates.models.KeyVaultCertificate;
import com.azure.security.keyvault.certificates.models.KeyVaultCertificateWithPolicy;

public class App {
    public static void main(String[] args) throws InterruptedException, IllegalArgumentException {
        String keyVaultName = System.getenv("KEY_VAULT_NAME");
        String keyVaultUri = "https://" + keyVaultName + ".vault.azure.net";

        System.out.printf("key vault name = %s and kv uri = %s \n", keyVaultName, keyVaultUri);

        CertificateClient certificateClient = new CertificateClientBuilder()
                .vaultUrl(keyVaultUri)
                .credential(new DefaultAzureCredentialBuilder().build())
                .buildClient();

        String certificateName = "myCertificate";

        System.out.print("Creating a certificate in " + keyVaultName + " called '" + certificateName + " ... ");

        SyncPoller<CertificateOperation, KeyVaultCertificateWithPolicy> certificatePoller =
                certificateClient.beginCreateCertificate(certificateName, CertificatePolicy.getDefault());
        certificatePoller.waitForCompletion();

        System.out.print("done.");
        System.out.println("Retrieving certificate from " + keyVaultName + ".");

        KeyVaultCertificate retrievedCertificate = certificateClient.getCertificate(certificateName);

        System.out.println("Your certificate's ID is '" + retrievedCertificate.getId() + "'.");
        System.out.println("Deleting your certificate from " + keyVaultName + " ... ");

        SyncPoller<DeletedCertificate, Void> deletionPoller = certificateClient.beginDeleteCertificate(certificateName);
        deletionPoller.waitForCompletion();

        System.out.print("done.");
    }
}
```

## <a name="next-steps"></a>Pasos siguientes
En este inicio rápido, no solo ha creado un almacén de claves, sino que también ha creado, recuperado y eliminado un certificado. Para más información sobre Key Vault y cómo integrarlo con las aplicaciones, continúe con los artículos siguientes.

- Lea una [introducción a Azure Key Vault](../general/overview.md).
- Consulte la [guía del desarrollador de Azure Key Vault](../general/developers-guide.md).
- Procedimientos para [proteger el acceso a un almacén de claves](../general/secure-your-key-vault.md)