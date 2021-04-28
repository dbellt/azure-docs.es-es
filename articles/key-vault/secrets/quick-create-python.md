---
title: 'Inicio rápido: biblioteca cliente de Azure Key Vault para Python, administración de secretos'
description: Obtenga información sobre cómo crear, recuperar y eliminar secretos desde una instancia de Azure Key Vault mediante la biblioteca cliente de Python
author: msmbaldwin
ms.author: mbaldwin
ms.date: 09/03/2020
ms.service: key-vault
ms.subservice: secrets
ms.topic: quickstart
ms.custom: devx-track-python, devx-track-azurepowershell
ms.openlocfilehash: 0163204e0ba953c34adf3f1b44bd385cb517fed5
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108133590"
---
# <a name="quickstart-azure-key-vault-secret-client-library-for-python"></a>Inicio rápido: Biblioteca cliente de secretos de Azure Key Vault para Python

Introducción a la biblioteca cliente de secretos de Azure Key Vault para Python. Siga estos pasos para instalar el paquete y probar el código de ejemplo para realizar tareas básicas. Si usa Key Vault para almacenar secretos, no tendrá que almacenarlos en el código, lo que aumenta la seguridad de las aplicaciones.

[Documentación de referencia de API](/python/api/overview/azure/keyvault-secrets-readme) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/keyvault/azure-keyvault-secrets) | [Paquete (índice de paquetes de Python)](https://pypi.org/project/azure-keyvault-secrets/)

## <a name="prerequisites"></a>Requisitos previos

- Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Python 2.7 o 3.6 y superiores](/azure/developer/python/configure-local-development-environment)
- [CLI de Azure](/cli/azure/install-azure-cli)

En esta guía de inicio rápido se supone que está ejecutando la [CLI de Azure](/cli/azure/install-azure-cli) en una ventana de terminal de Linux.


## <a name="set-up-your-local-environment"></a>Configuración de un entorno local
En este inicio rápido se usa la biblioteca de identidades de Azure con la CLI de Azure para autenticar al usuario en los servicios de Azure. Los desarrolladores también pueden usar Visual Studio o Visual Studio Code para autenticar sus llamadas. Para más información, consulte [Autenticación del cliente mediante la biblioteca cliente Azure Identity](/python/api/overview/azure/identity-readme).

### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

1. Ejecute el comando `login`.

    ```azurecli-interactive
    az login
    ```

    Si la CLI puede abrir el explorador predeterminado, lo hará y cargará una página de inicio de sesión de Azure.

    En caso contrario, abra una página del explorador en [https://aka.ms/devicelogin](https://aka.ms/devicelogin) y escriba el código de autorización que se muestra en el terminal.

2. Inicie sesión con las credenciales de su cuenta en el explorador.

### <a name="install-the-packages"></a>Instalación de los paquetes

1. En un símbolo del sistema o en un terminal, cree una carpeta de proyecto adecuada y, después, cree y active un entorno virtual de Python, como se describe en el apartado [Uso de entornos virtuales de Python](/azure/developer/python/configure-local-development-environment?tabs=cmd#use-python-virtual-environments).

1. Instale la biblioteca de identidades de Azure Active Directory:

    ```terminal
    pip install azure-identity
    ```


1. Instale la biblioteca de secretos de Key Vault:

    ```terminal
    pip install azure-keyvault-secrets
    ```

### <a name="create-a-resource-group-and-key-vault"></a>Creación de un grupo de recursos y de un almacén de claves

[!INCLUDE [Create a resource group and key vault](../../../includes/key-vault-python-qs-rg-kv-creation.md)]

### <a name="grant-access-to-your-key-vault"></a>Concesión de acceso al almacén de claves

Cree una directiva de acceso para el almacén de claves que conceda permiso mediante secreto a la cuenta de usuario.

```console
az keyvault set-policy --name <YourKeyVaultName> --upn user@domain.com --secret-permissions delete get list set
```

#### <a name="set-environment-variables"></a>Establecimiento de variables de entorno

Esta aplicación también usa el nombre del almacén de claves como variable de entorno llamada `KEY_VAULT_NAME`.

```bash
export KEY_VAULT_NAME=<your-key-vault-name>
```

## <a name="create-the-sample-code"></a>Creación del código de ejemplo

La biblioteca cliente de secretos de Azure Key Vault para Python permite administrar los secretos. El siguiente código de ejemplo muestra cómo crear un cliente y cómo establecer, recuperar y eliminar un secreto.

Cree un archivo llamado *kv_secrets.py* que contenga este código.

```python
import os
import cmd
from azure.keyvault.secrets import SecretClient
from azure.identity import DefaultAzureCredential

keyVaultName = os.environ["KEY_VAULT_NAME"]
KVUri = f"https://{keyVaultName}.vault.azure.net"

credential = DefaultAzureCredential()
client = SecretClient(vault_url=KVUri, credential=credential)

secretName = input("Input a name for your secret > ")
secretValue = input("Input a value for your secret > ")

print(f"Creating a secret in {keyVaultName} called '{secretName}' with the value '{secretValue}' ...")

client.set_secret(secretName, secretValue)

print(" done.")

print(f"Retrieving your secret from {keyVaultName}.")

retrieved_secret = client.get_secret(secretName)

print(f"Your secret is '{retrieved_secret.value}'.")
print(f"Deleting your secret from {keyVaultName} ...")

poller = client.begin_delete_secret(secretName)
deleted_secret = poller.result()

print(" done.")
```

## <a name="run-the-code"></a>Ejecución del código

Asegúrese de que el código de la sección anterior se encuentra en un archivo llamado *kv_secrets.py*. Luego, ejecute el código con el siguiente comando:

```terminal
python kv_secrets.py
```

- Si se producen errores de permisos, asegúrese de que ha ejecutado el comando [`az keyvault set-policy`](#grant-access-to-your-key-vault).
- Al volver a ejecutar el código con el mismo nombre de secreto, se puede producir el error "(Conflict) Secret <name> is currently in a deleted but recoverable state" [(Conflicto) El secreto <name> se ha eliminado, pero se puede recuperar]. Use otro nombre de secreto.

## <a name="code-details"></a>Detalles del código

### <a name="authenticate-and-create-a-client"></a>Autenticación y creación de un cliente

En este inicio rápido se emplea el usuario que ha iniciado sesión para autenticarlo en el almacén de claves, que es el método preferido para el desarrollo local. Para las aplicaciones implementadas en Azure, la identidad administrada debe asignarse a App Service o la máquina virtual. Para más información, consulte [Introducción a la identidad administrada](../../active-directory/managed-identities-azure-resources/overview.md).

En el ejemplo siguiente, el nombre del almacén de claves se expande al URI del almacén de claves, con el formato "https://\<your-key-vault-name\>.vault.azure.net". En este ejemplo se usa la clase ["DefaultAzureCredential()"](/python/api/azure-identity/azure.identity.defaultazurecredential), que permite usar el mismo código en entornos diferentes con distintas opciones para proporcionar la identidad. Para más información, consulte [Autenticación mediante las credenciales predeterminadas de Azure](/python/api/overview/azure/identity-readme). 

```python
credential = DefaultAzureCredential()
client = SecretClient(vault_url=KVUri, credential=credential)
```

### <a name="save-a-secret"></a>Almacenamiento de un secreto

Una vez que haya obtenido el objeto de cliente para el almacén de claves, puede almacenar un secreto mediante el método [set_secret](/python/api/azure-keyvault-secrets/azure.keyvault.secrets.secretclient?#set-secret-name--value----kwargs-): 

```python
client.set_secret(secretName, secretValue)
```

La llamada a `set_secret` genera una llamada a la API REST de Azure para el almacén de claves.

Durante el control de la solicitud, Azure autentica la identidad del autor de la llamada (la entidad de servicio) mediante el objeto de credencial que proporcionó al cliente.

### <a name="retrieve-a-secret"></a>Recuperación de un secreto

Para leer un secreto de Key Vault, use el método [get_secret](/python/api/azure-keyvault-secrets/azure.keyvault.secrets.secretclient?#get-secret-name--version-none----kwargs-):

```python
retrieved_secret = client.get_secret(secretName)
 ```

El valor de secreto se incluye en `retrieved_secret.value`.

Los secretos también se pueden recuperar con el comando [az keyvault secret show](/cli/azure/keyvault/secret?#az_keyvault_secret_show) de la CLI de Azure.

### <a name="delete-a-secret"></a>Eliminación de un secreto

Para eliminar un secreto, use el método [begin_delete_secret](/python/api/azure-keyvault-secrets/azure.keyvault.secrets.secretclient?#begin-delete-secret-name----kwargs-):

```python
poller = client.begin_delete_secret(secretName)
deleted_secret = poller.result()
```

El método `begin_delete_secret` es asincrónico y devuelve un objeto de sondeador. La llamada al método `result` del sondeador espera hasta su finalización.

Para comprobar que el secreto se ha establecido, utilice el comando [az keyvault secret show](/cli/azure/keyvault/secret?#az_keyvault_secret_show) de la CLI de Azure.

Aun después de haberse eliminado, los secretos permanecen en estado eliminado, pero recuperable, durante un tiempo. Si vuelve a ejecutar el código, use otro nombre de secreto.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si también desea experimentar con [certificados](../certificates/quick-create-python.md) y [claves](../keys/quick-create-python.md), puede volver a usar la instancia de Key Vault que se ha creado en este artículo.

De lo contrario, cuando haya terminado con los recursos creados en este artículo, utilice el siguiente comando para eliminar el grupo de recursos y todos los recursos que contiene:

```azurecli
az group delete --resource-group KeyVault-PythonQS-rg
```

## <a name="next-steps"></a>Pasos siguientes

- [Información general de Azure Key Vault](../general/overview.md)
- [Protección del acceso a un almacén de claves](../general/security-features.md)
- [Guía del desarrollador de Azure Key Vault](../general/developers-guide.md)
- [Introducción a la seguridad de Key Vault](../general/security-features.md)
- [Autenticación con Key Vault](../general/authentication.md)