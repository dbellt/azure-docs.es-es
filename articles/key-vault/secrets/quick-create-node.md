---
title: 'Inicio rápido: biblioteca cliente de secretos de Azure Key Vault para JavaScript (versión 4)'
description: Aprenda a crear, recuperar y eliminar secretos desde una instancia de Azure Key Vault mediante la biblioteca cliente de JavaScript
author: msmbaldwin
ms.author: mbaldwin
ms.date: 12/6/2020
ms.service: key-vault
ms.subservice: secrets
ms.topic: quickstart
ms.custom: devx-track-js
ms.openlocfilehash: 3d8f20f598243754e3fd39f649e7c84d55b64817
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107751677"
---
# <a name="quickstart-azure-key-vault-secret-client-library-for-javascript-version-4"></a>Inicio rápido: Biblioteca cliente de secretos de Azure Key Vault para JavaScript (versión 4)

Introducción a la biblioteca cliente de secretos de Azure Key Vault para JavaScript. [Azure Key Vault](../general/overview.md) es un servicio en la nube que ofrece el almacenamiento seguro de los secretos. Puede almacenar de forma segura claves, contraseñas, certificados y otros secretos. Las instancias de Azure Key Vault se pueden crear y administrar a través de Azure Portal. En este inicio rápido, aprenderá a crear, recuperar y eliminar secretos de una instancia de Azure Key Vault mediante la biblioteca cliente de JavaScript.

Recursos de la biblioteca cliente de Key Vault:

[Documentación de referencia de la API](/javascript/api/overview/azure/key-vault-index) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/keyvault) | [Paquete (npm)](https://www.npmjs.com/package/@azure/keyvault-secrets)

Para más información sobre Key Vault y los secretos, consulte:
- [Introducción a Azure Key Vault](../general/overview.md)
- [Información general de los secretos](about-secrets.md).

## <a name="prerequisites"></a>Requisitos previos

- Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Node.js](https://nodejs.org) actual del sistema operativo.
- [CLI de Azure](/cli/azure/install-azure-cli)
- Una instancia de Key Vault (se puede crear mediante [Azure Portal](../general/quick-create-portal.md), [la CLI de Azure](../general/quick-create-cli.md) o [Azure PowerShell](../general/quick-create-powershell.md))

En este inicio rápido se supone que ejecuta la [CLI de Azure](/cli/azure/install-azure-cli).

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

1. Ejecute el comando `login`.

    ```azurecli-interactive
    az login
    ```

    Si la CLI puede abrir el explorador predeterminado, lo hará y cargará una página de inicio de sesión de Azure.

    En caso contrario, abra una página del explorador en [https://aka.ms/devicelogin](https://aka.ms/devicelogin) y escriba el código de autorización que se muestra en el terminal.

2. Inicie sesión con las credenciales de su cuenta en el explorador.

## <a name="create-new-nodejs-application"></a>Creación de una aplicación de Node.js

Después, cree una aplicación de Node.js que se pueda implementar en la nube. 

1. En un shell de comandos, cree una carpeta denominada `key-vault-node-app`:

```terminal
mkdir key-vault-node-app
```

1. Cambie al directorio *key-vault-node-app* recién creado y ejecute el comando "init" para inicializar el proyecto del nodo:

```terminal
cd key-vault-node-app
npm init -y
```

## <a name="install-key-vault-packages"></a>Instalación de paquetes de Key Vault

En la ventana de la consola, instale la [biblioteca de secretos](https://www.npmjs.com/package/@azure/keyvault-secrets) de Azure Key Vault para Node.js.

```terminal
npm install @azure/keyvault-secrets
```

Instale el paquete [azure.identity](https://www.npmjs.com/package/@azure/identity) para autenticarse en Key Vault

```terminal
npm install @azure/identity
```

## <a name="set-environment-variables"></a>Establecimiento de variables de entorno

Esta aplicación también usa el nombre del almacén de claves como variable de entorno llamada `KEY_VAULT_NAME`.

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

## <a name="grant-access-to-your-key-vault"></a>Concesión de acceso al almacén de claves

Cree una directiva de acceso para el almacén de claves que conceda permisos mediante secreto a la cuenta de usuario.

```azurecli
az keyvault set-policy --name <YourKeyVaultName> --upn user@domain.com --secret-permissions delete get list set purge
```

## <a name="code-examples"></a>Ejemplos de código

Los ejemplos de código siguientes le mostrarán cómo crear un cliente y cómo establecer, recuperar y eliminar un secreto. 

### <a name="set-up-the-app-framework"></a>Instalación del marco de la aplicación

1. Cree un archivo de texto y guárdelo como "index.js"

1. Agregue las llamadas necesarias para cargar los módulos de Azure y Node.js.

1. Cree la estructura del programa, incluido un control de excepciones básico.

```javascript
const readline = require('readline');

function askQuestion(query) {
    const rl = readline.createInterface({
        input: process.stdin,
        output: process.stdout,
    });

    return new Promise(resolve => rl.question(query, ans => {
        rl.close();
        resolve(ans);
    }))
}

async function main() {
    
}

main().then(() => console.log('Done')).catch((ex) => console.log(ex.message));
```

### <a name="add-directives"></a>Adición de directivas

Agregue las siguientes directivas al principio del código:

```javascript
const { DefaultAzureCredential } = require("@azure/identity");
const { SecretClient } = require("@azure/keyvault-secrets");
```

### <a name="authenticate-and-create-a-client"></a>Autenticación y creación de un cliente

En este inicio rápido se emplea el usuario que ha iniciado sesión para autenticarlo en el almacén de claves, que es el método preferido para el desarrollo local. Para las aplicaciones implementadas en Azure, la identidad administrada debe asignarse a App Service o la máquina virtual. Para más información, consulte [Introducción a la identidad administrada](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview).

En el ejemplo siguiente, el nombre del almacén de claves se expande al URI del almacén de claves, con el formato "https://\<your-key-vault-name\>.vault.azure.net". En este ejemplo se utiliza la clase ["DefaultAzureCredential()"](https://docs.microsoft.com/javascript/api/@azure/identity/defaultazurecredential) de la [biblioteca Azure Identity](https://docs.microsoft.com/javascript/api/overview/azure/identity-readme), que permite usar el mismo código en entornos diferentes con otras opciones para proporcionar la identidad. Para más información sobre la autenticación en Key Vault, consulte la [Guía del desarrollador](https://docs.microsoft.com/azure/key-vault/general/developers-guide#authenticate-to-key-vault-in-code).

Agregue el siguiente código a la función "main()".

```javascript
const keyVaultName = process.env["KEY_VAULT_NAME"];
const KVUri = "https://" + keyVaultName + ".vault.azure.net";

const credential = new DefaultAzureCredential();
const client = new SecretClient(KVUri, credential);
```

### <a name="save-a-secret"></a>Almacenamiento de un secreto

Ahora que la aplicación se ha autenticado, puede colocar un secreto en el almacén de claves mediante el [método setSecret](/javascript/api/@azure/keyvault-secrets/secretclient#setSecret_string__string__SetSecretOptions_), lo que requiere dar un nombre al secreto (en este ejemplo, se usa "mySecret").  

```javascript
await client.setSecret(secretName, secretValue);
```

### <a name="retrieve-a-secret"></a>Recuperación de un secreto

Ahora puede recuperar el valor previamente establecido con el [método getSecret](/javascript/api/@azure/keyvault-secrets/secretclient#getSecret_string__GetSecretOptions_).

```javascript
const retrievedSecret = await client.getSecret(secretName);
 ```

El secreto se guarda ahora como `retrievedSecret.value`.

### <a name="delete-a-secret"></a>Eliminación de un secreto

Por último, se va a eliminar y purgar la clave del almacén de secretos con los métodos [beginDeleteSecret](https://docs.microsoft.com/javascript/api/@azure/keyvault-secrets/secretclient?#beginDeleteSecret_string__BeginDeleteSecretOptions_) y [purgeDeletedSecret](https://docs.microsoft.com/javascript/api/@azure/keyvault-secrets/secretclient?#purgeDeletedSecret_string__PurgeDeletedSecretOptions_).

```javascript
const deletePoller = await client.beginDeleteSecret(secretName);
await deletePoller.pollUntilDone();
await client.purgeDeletedSecret(secretName);
```

## <a name="sample-code"></a>Código de ejemplo

```javascript
const { DefaultAzureCredential } = require("@azure/identity");
const { SecretClient } = require("@azure/keyvault-secrets");

const readline = require('readline');

function askQuestion(query) {
    const rl = readline.createInterface({
        input: process.stdin,
        output: process.stdout,
    });

    return new Promise(resolve => rl.question(query, ans => {
        rl.close();
        resolve(ans);
    }))
}

async function main() {

  const keyVaultName = process.env["KEY_VAULT_NAME"];
  const KVUri = "https://" + keyVaultName + ".vault.azure.net";

  const credential = new DefaultAzureCredential();
  const client = new SecretClient(KVUri, credential);

  const secretName = "mySecret";
  var secretValue = await askQuestion("Input the value of your secret > ");

  console.log("Creating a secret in " + keyVaultName + " called '" + secretName + "' with the value '" + secretValue + "` ...");
  await client.setSecret(secretName, secretValue);

  console.log("Done.");

  console.log("Forgetting your secret.");
  secretValue = "";
  console.log("Your secret is '" + secretValue + "'.");

  console.log("Retrieving your secret from " + keyVaultName + ".");

  const retrievedSecret = await client.getSecret(secretName);

  console.log("Your secret is '" + retrievedSecret.value + "'.");

  console.log("Deleting your secret from " + keyVaultName + " ...");
  const deletePoller = await client.beginDeleteSecret(secretName);
  await deletePoller.pollUntilDone();
  console.log("Done.");
  
  console.log("Purging your secret from {keyVaultName} ...");
  await client.purgeDeletedSecret(secretName);
  
}

main().then(() => console.log('Done')).catch((ex) => console.log(ex.message));

```

## <a name="test-and-verify"></a>Prueba y comprobación

1. Ejecute los siguientes comandos para ejecutar la aplicación.

    ```terminal
    npm install
    node index.js
    ```

1. Cuando se le pida, escriba un secreto. Por ejemplo, mySecretPassword.

    Aparece una variación del resultado siguiente:

    ```azurecli
    Input the value of your secret > mySecretPassword
    Creating a secret in <your-unique-keyvault-name> called 'mySecret' with the value 'mySecretPassword' ... done.
    Forgetting your secret.
    Your secret is ''.
    Retrieving your secret from <your-unique-keyvault-name>.
    Your secret is 'mySecretPassword'.
    Deleting your secret from <your-unique-keyvault-name> ... done.  
    Purging your secret from <your-unique-keyvault-name> ... done.   
    ```


## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado un almacén de claves, ha almacenado un secreto en él y, posteriormente, lo ha recuperado. Para más información sobre Key Vault y cómo integrarlo con las aplicaciones, continúe con los artículos siguientes.

- Lea una [introducción a Azure Key Vault](../general/overview.md).
- Lea una [introducción a los secretos de Azure Key Vault](about-secrets.md).
- Procedimientos para [proteger el acceso a un almacén de claves](../general/security-overview.md)
- Consulte la [guía del desarrollador de Azure Key Vault](../general/developers-guide.md).
- Consulte [Introducción a la seguridad de Azure Key Vault](../general/security-overview.md)
