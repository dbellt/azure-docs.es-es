---
title: 'Inicio rápido: biblioteca cliente de claves de Azure Key Vault para JavaScript (versión 4)'
description: Aprenda a crear, recuperar y eliminar claves desde una instancia de Azure Key Vault mediante la biblioteca cliente de JavaScript
author: msmbaldwin
ms.author: mbaldwin
ms.date: 12/6/2020
ms.service: key-vault
ms.subservice: keys
ms.topic: quickstart
ms.custom: devx-track-js
ms.openlocfilehash: c0066409732f4492186ea0bf604261e1ab59ca9f
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107750309"
---
# <a name="quickstart-azure-key-vault-key-client-library-for-javascript-version-4"></a>Inicio rápido: Biblioteca cliente de claves de Azure Key Vault para JavaScript (versión 4)

Comience a trabajar con la biblioteca cliente de Azure Key Vault para JavaScript. [Azure Key Vault](../general/overview.md) es un servicio en la nube que ofrece el almacenamiento seguro de las claves criptográficas. Puede almacenar de forma segura claves, contraseñas, certificados y otros secretos. Las instancias de Azure Key Vault se pueden crear y administrar a través de Azure Portal. En este inicio rápido, aprenderá a crear, recuperar y eliminar secretos de una instancia de Azure Key Vault mediante la biblioteca cliente de claves de JavaScript.

Recursos de la biblioteca cliente de Key Vault:

[Documentación de referencia de la API](/javascript/api/overview/azure/key-vault-index) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/keyvault) | [Paquete (npm)](https://www.npmjs.com/package/@azure/keyvault-keys)

Para más información sobre Key Vault y las claves, consulte:
- [Introducción a Azure Key Vault](../general/overview.md)
- [Introducción a las claves](about-keys.md).

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

```azurecli
mkdir key-vault-node-app
```

1. Cambie al directorio *key-vault-node-app* recién creado y ejecute el comando "init" para inicializar el proyecto del nodo:

```azurecli
cd key-vault-node-app
npm init -y
```

## <a name="install-key-vault-packages"></a>Instalación de paquetes de Key Vault

En la ventana de la consola, instale la [biblioteca de claves](https://www.npmjs.com/package/@azure/keyvault-keys) de Azure Key Vault para Node.js.

```azurecli
npm install @azure/keyvault-keys
```

Instale el paquete [azure.identity](https://www.npmjs.com/package/@azure/identity) para autenticarse en Key Vault

```azurecli
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

Cree una directiva de acceso para el almacén de claves que conceda permisos de clave a la cuenta de usuario.

```azurecli
az keyvault set-policy --name <YourKeyVaultName> --upn user@domain.com --key-permissions delete get list create purge
```

## <a name="code-examples"></a>Ejemplos de código

En los ejemplos de código siguientes se muestra cómo crear un cliente y cómo establecer, recuperar y eliminar una clave. 

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
const { KeyClient } = require("@azure/keyvault-keys");
```

### <a name="authenticate-and-create-a-client"></a>Autenticación y creación de un cliente

En este inicio rápido se emplea el usuario que ha iniciado sesión para autenticarlo en el almacén de claves, que es el método preferido para el desarrollo local. Para las aplicaciones implementadas en Azure, la identidad administrada debe asignarse a App Service o la máquina virtual. Para más información, consulte [Introducción a la identidad administrada](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview).

En el ejemplo siguiente, el nombre del almacén de claves se expande al URI del almacén de claves, con el formato "https://\<your-key-vault-name\>.vault.azure.net". En este ejemplo se usa la clase ["DefaultAzureCredential()"](https://docs.microsoft.com/javascript/api/@azure/identity/defaultazurecredential) de la [biblioteca de identidades de Azure](https://docs.microsoft.com/javascript/api/overview/azure/identity-readme), que permite usar el mismo código en entornos diferentes con distintas opciones para proporcionar la identidad. Para más información sobre la autenticación en el almacén de claves, consulte la [Guía del desarrollador](https://docs.microsoft.com/azure/key-vault/general/developers-guide#authenticate-to-key-vault-in-code).

Agregue el siguiente código a la función "main()".

```javascript
const keyVaultName = process.env["KEY_VAULT_NAME"];
const KVUri = "https://" + keyVaultName + ".vault.azure.net";

const credential = new DefaultAzureCredential();
const client = new KeyClient(KVUri, credential);
```

### <a name="save-a-key"></a>Guardar una clave

Ahora que la aplicación se ha autenticado, puede colocar una clave en el almacén de claves, para lo que usará el [método createKey](/javascript/api/@azure/keyvault-keys/keyclient?#createKey_string__KeyType__CreateKeyOptions_). Los parámetros del método aceptan un nombre de clave y el [tipo de clave](https://docs.microsoft.com/javascript/api/@azure/keyvault-keys/keytype)

```javascript
await client.createKey(keyName, keyType);
```

### <a name="retrieve-a-key"></a>Recuperación de una clave

Ahora puede recuperar el valor previamente establecido con el [método getKey](/javascript/api/@azure/keyvault-keys/keyclient?#getKey_string__GetKeyOptions_).

```javascript
const retrievedKey = await client.getKey(keyName);
 ```

### <a name="delete-a-key"></a>Eliminación de una clave

Por último, se va a eliminar y purgar la clave del almacén de claves con los métodos [beginDeleteKey](https://docs.microsoft.com/javascript/api/@azure/keyvault-keys/keyclient?#beginDeleteKey_string__BeginDeleteKeyOptions_) y [purgeDeletedKey](https://docs.microsoft.com/javascript/api/@azure/keyvault-keys/keyclient?#purgeDeletedKey_string__PurgeDeletedKeyOptions_).

```javascript
const deletePoller = await client.beginDeleteKey(keyName);
await deletePoller.pollUntilDone();
await client.purgeDeletedKey(keyName);
```

## <a name="sample-code"></a>Código de ejemplo

```javascript
const { DefaultAzureCredential } = require("@azure/identity");
const { KeyClient } = require("@azure/keyvault-keys");

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
  const client = new KeyClient(KVUri, credential);

  const keyName = "myKey";
  
  console.log("Creating a key in " + keyVaultName + " called '" + keyName + "` ...");
  await client.createKey(keyName, "RSA");

  console.log("Done.");

  console.log("Retrieving your key from " + keyVaultName + ".");

  const retrievedKey = await client.getKey(keyName);

  console.log("Your key version is '" + retrievedKey.properties.version + "'.");

  console.log("Deleting your key from " + keyVaultName + " ...");
  const deletePoller = await client.beginDeleteKey(keyName);
  await deletePoller.pollUntilDone();
  console.log("Done.");
  
  console.log("Purging your key from {keyVaultName} ...");
  await client.purgeDeletedKey(keyName);
  
}

main().then(() => console.log('Done')).catch((ex) => console.log(ex.message));

```

## <a name="test-and-verify"></a>Prueba y comprobación

Ejecute los siguientes comandos para ejecutar la aplicación.

```azurecli
npm install
npm index.js
```

Aparece una variación del resultado siguiente:

```azurecli
Creating a key in <your-unique-keyvault-name> called 'myKey' ... done.
Retrieving your key from mykeyvault.
Your key version is '8532359bced24e4bb2525f2d2050738a'.
Deleting your key from <your-unique-keyvault-name> ... done.  
Purging your key from <your-unique-keyvault-name> ... done.   
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado un almacén de claves y ha almacenado una clave y recuperado dicha clave. Para más información sobre Key Vault y cómo integrarlo con las aplicaciones, continúe con los artículos siguientes.

- Lea una [introducción a Azure Key Vault](../general/overview.md).
- Lea una [introducción a las claves de Azure Key Vault](about-keys.md)
- Procedimientos para [proteger el acceso a un almacén de claves](../general/security-overview.md)
- Consulte la [guía del desarrollador de Azure Key Vault](../general/developers-guide.md).
- Consulte [Introducción a la seguridad de Azure Key Vault](../general/security-overview.md)
