---
title: 'Inicio rápido: administración de claves con la biblioteca cliente de Azure Key Vault para Python'
description: Aprenda a crear, recuperar y eliminar claves desde una instancia de Azure Key Vault mediante la biblioteca cliente de Python
author: msmbaldwin
ms.author: mbaldwin
ms.date: 09/03/2020
ms.service: key-vault
ms.subservice: keys
ms.topic: quickstart
ms.custom: devx-track-python, devx-track-azurecli
ms.openlocfilehash: f76a82715bd4599b17e0ea164a1282b92a13aa3c
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108287543"
---
# <a name="quickstart-azure-key-vault-keys-client-library-for-python"></a>Inicio rápido: biblioteca cliente de claves de Azure Key Vault para Python

Introducción a la biblioteca cliente de Azure Key Vault para Python. Siga estos pasos para instalar el paquete y probar el código de ejemplo para realizar tareas básicas. Si usa Key Vault para almacenar claves criptográficas, no tendrá que almacenarlas en el código, lo que aumenta la seguridad de las aplicaciones.

[Documentación de referencia de API](/python/api/overview/azure/keyvault-keys-readme) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/keyvault/azure-keyvault-keys) | [Paquete (índice de paquetes de Python)](https://pypi.org/project/azure-keyvault-keys/)

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
    pip install azure.identity
    ```


1. Instale la biblioteca cliente de claves de Key Vault:

    ```terminal
    pip install azure-keyvault-keys
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

## <a name="create-the-sample-code"></a>Creación del código de ejemplo

La biblioteca cliente de claves de Azure Key Vault para Python permite administrar claves criptográficas. El siguiente código de ejemplo muestra cómo crear un cliente y cómo establecer, recuperar y eliminar una clave.

Cree un archivo llamado *kv_keys.py* que contenga este código.

```python
import os
from azure.keyvault.keys import KeyClient
from azure.identity import DefaultAzureCredential

keyVaultName = os.environ["KEY_VAULT_NAME"]
KVUri = "https://" + keyVaultName + ".vault.azure.net"

credential = DefaultAzureCredential()
client = KeyClient(vault_url=KVUri, credential=credential)

keyName = input("Input a name for your key > ")

print(f"Creating a key in {keyVaultName} called '{keyName}' ...")

rsa_key = client.create_rsa_key(keyName, size=2048)

print(" done.")

print(f"Retrieving your key from {keyVaultName}.")

retrieved_key = client.get_key(keyName)

print(f"Key with name '{retrieved_key.name}' was found.")
print(f"Deleting your key from {keyVaultName} ...")

poller = client.begin_delete_key(keyName)
deleted_key = poller.result()

print(" done.")
```

## <a name="run-the-code"></a>Ejecución del código

Asegúrese de que el código de la sección anterior se encuentra en un archivo llamado *kv_keys.py*. Luego, ejecute el código con el siguiente comando:

```terminal
python kv_keys.py
```

- Si se producen errores de permisos, asegúrese de que ha ejecutado el comando [`az keyvault set-policy`](#grant-access-to-your-key-vault).
- Al volver a ejecutar el código con el mismo nombre de clave, se puede producir el error "(Conflict) Key <name> is currently in a deleted but recoverable state" [(Conflicto) La clave <name> se ha eliminado, pero se puede recuperar]. Use otro nombre de clave.

## <a name="code-details"></a>Detalles del código

### <a name="authenticate-and-create-a-client"></a>Autenticación y creación de un cliente

En este inicio rápido se emplea el usuario que ha iniciado sesión para autenticarlo en el almacén de claves, que es el método preferido para el desarrollo local. Para las aplicaciones implementadas en Azure, la identidad administrada debe asignarse a App Service o la máquina virtual. Para más información, consulte [Introducción a la identidad administrada](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview).

En el ejemplo siguiente, el nombre del almacén de claves se expande al URI del almacén de claves, con el formato "https://\<your-key-vault-name\>.vault.azure.net". En este ejemplo se usa la clase ["DefaultAzureCredential()"](/python/api/azure-identity/azure.identity.defaultazurecredential), que permite usar el mismo código en entornos diferentes con distintas opciones para proporcionar la identidad. Para más información, consulte [Autenticación mediante las credenciales predeterminadas de Azure](https://docs.microsoft.com/python/api/overview/azure/identity-readme). 


```python
credential = DefaultAzureCredential()
client = KeyClient(vault_url=KVUri, credential=credential)
```

## <a name="save-a-key"></a>Guardar una clave

Una vez que haya obtenido el objeto de cliente para el almacén de claves, puede almacenar una clave mediante el método [create_rsa_key](/python/api/azure-keyvault-keys/azure.keyvault.keys.keyclient?#create-rsa-key-name----kwargs-): 

```python
rsa_key = client.create_rsa_key(keyName, size=2048)
```

También se pueden usar [create_key](/python/api/azure-keyvault-keys/azure.keyvault.keys.keyclient?#create-key-name--key-type----kwargs-) o [create_ec_key](/python/api/azure-keyvault-keys/azure.keyvault.keys.keyclient?#create-ec-key-name----kwargs-).

La llamada a un método `create` genera una llamada a la API REST de Azure para el almacén de claves.

Durante el control de la solicitud, Azure autentica la identidad del autor de la llamada (la entidad de servicio) mediante el objeto de credencial que proporcionó al cliente.

## <a name="retrieve-a-key"></a>Recuperación de una clave

Para leer una clave de Key Vault, utilice el método [get_key](/python/api/azure-keyvault-keys/azure.keyvault.keys.keyclient?#get-key-name--version-none----kwargs-):

```python
retrieved_key = client.get_key(keyName)
 ```

También puede comprobar que la clave se ha establecido con el comando [az keyvault key show](/cli/azure/keyvault/key?#az_keyvault_key_show) de la CLI de Azure.

### <a name="delete-a-key"></a>Eliminación de una clave

Para eliminar una clave, use el método [begin_delete_key](/python/api/azure-keyvault-keys/azure.keyvault.keys.keyclient?#begin-delete-key-name----kwargs-):

```python
poller = client.begin_delete_key(keyName)
deleted_key = poller.result()
```

El método `begin_delete_key` es asincrónico y devuelve un objeto de sondeador. La llamada al método `result` del sondeador espera hasta su finalización.

Puede comprobar que la clave se elimina con el comando [az keyvault key show](/cli/azure/keyvault/key?#az_keyvault_key_show) de la CLI de Azure.

Aun después de haberse eliminado, las claves permanecen en estado eliminado, pero recuperable, durante un tiempo. Si vuelve a ejecutar el código, use otro nombre de clave.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si también desea experimentar con [certificados](../certificates/quick-create-python.md) y [secretos](../secrets/quick-create-python.md), puede volver a usar la instancia de Key Vault que se ha creado en este artículo.

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
