---
title: 'Inicio rápido: Biblioteca cliente de Microsoft Azure Confidential Ledger en Python'
description: Aprenda a usar la biblioteca cliente de Microsoft Azure Confidential Ledger para Python.
author: msmbaldwin
ms.author: mbaldwin
ms.date: 04/27/2021
ms.service: confidential-ledger
ms.topic: quickstart
ms.custom: devx-track-python, devx-track-azurepowershell
ms.openlocfilehash: bc6da168dca367b152d1de967b448d75139a2d56
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388599"
---
# <a name="quickstart-microsoft-azure-confidential-ledger-client-library-for-python"></a>Inicio rápido: Biblioteca cliente de Microsoft Azure Confidential Ledger para Python

Comience a usar la biblioteca cliente de Microsoft Azure Confidential Ledger para Python. Siga estos pasos para instalar el paquete y probar el código de ejemplo para realizar tareas básicas.

Microsoft Azure Confidential Ledger es un servicio nuevo y muy seguro para administrar registros de datos confidenciales. En función de un modelo de cadena de bloques con permisos, Confidential Ledger ofrece ventajas únicas de integridad de datos, como la inmutabilidad (que hace que el libro de contabilidad solo se pueda anexar) y la protección frente a manipulaciones (para asegurar que todos los registros se mantengan intactos).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[Documentación de referencia de API](/python/api/overview/azure/keyvault-secrets-readme) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/keyvault/azure-keyvault-secrets) | [Paquete (índice de paquetes de Python)](https://pypi.org/project/azure-keyvault-secrets/)

## <a name="prerequisites"></a>Requisitos previos

- Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Python 3.6 o posterior](/azure/developer/python/configure-local-development-environment)
- [CLI de Azure](/cli/azure/install-azure-cli) o [Azure PowerShell](/powershell/azure/install-az-ps)

## <a name="set-up"></a>Configuración

En este artículo de inicio rápido se usa la biblioteca de identidades de Azure, junto con la CLI de Azure o Azure PowerShell, para autenticar al usuario en los servicios de Azure. Los desarrolladores también pueden usar Visual Studio o Visual Studio Code para autenticar las llamadas. Para más información, consulte [Autenticación del cliente con la biblioteca cliente de Azure Identity](/python/api/overview/azure/identity-readme).

### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

[!INCLUDE [Sign in to Azure](../../includes/confidential-ledger-sign-in-azure.md)]

### <a name="install-the-packages"></a>Instalación de los paquetes

En un símbolo del sistema o en un terminal, cree una carpeta de proyecto adecuada y, después, cree y active un entorno virtual de Python, como se describe en el apartado [Uso de entornos virtuales de Python](/azure/developer/python/configure-local-development-environment?tabs=cmd#use-python-virtual-environments).

Instale la biblioteca cliente de identidades de Azure Active Directory:

```terminal
pip install azure-identity
```

Instale la biblioteca cliente del plano de control de Confidential Ledger.

```terminal
pip install azure.mgmt.confidentialledger
```

Instale la biblioteca cliente del plano de datos de Confidential Ledger.

```terminal
pip install azure.confidentialledger 
```

### <a name="create-a-resource-group"></a>Crear un grupo de recursos

[!INCLUDE [Create a resource group](../../includes/confidential-ledger-rg-create.md)]

### <a name="register-the-microsoftconfidentialledger-resource-provider"></a>Registro del proveedor de recursos microsoft.ConfidentialLedger

[!INCLUDE [Register the microsoft.ConfidentialLedger resource provider](../../includes/confidential-ledger-register-rp.md)]

## <a name="create-your-python-app"></a>Creación de la aplicación de Python

### <a name="initialization"></a>Inicialización

Ahora podemos empezar a escribir la aplicación de Python.  En primer lugar, vamos a importar los paquetes necesarios.

```python
# Import the Azure authentication library

from azure.identity import DefaultAzureCredential

## Import the control plane sdk

from azure.mgmt.confidentialledger import ConfidentialLedger as ConfidentialLedgerAPI
from azure.mgmt.confidentialledger.models import ConfidentialLedger

# import the data plane sdk

from azure.confidentialledger import ConfidentialLedgerClient
from azure.confidentialledger.identity_service import ConfidentialLedgerIdentityServiceClient
```

Luego usaremos la [clase DefaultAzureCredential](/python/api/azure-identity/azure.identity.defaultazurecredential) para autenticar la aplicación.

```python
credential = DefaultAzureCredential()
```

Para finalizar la configuración, estableceremos algunas variables para usar en la aplicación: el grupo de recursos (myResourceGroup), el nombre del libro de contabilidad que quiere crear, y dos direcciones URL que usará la biblioteca cliente del plano de datos.

  > [!Important]
  > Cada libro de contabilidad debe tener un nombre único globalmente. Reemplace <your-unique-keyvault-name> por el nombre del libro de contabilidad en el ejemplo siguiente.

```python
resource_group = "myResourceGroup"
ledger_name = "<your-unique-ledger-name>"
subscription_id "<azure-subscription-id>"

identity_url = "https://identity.confidential-ledger.core.azure.com"
ledger_url = "https://" + ledger_name + ".eastus.cloudapp.azure.com"
```

### <a name="use-the-control-plane-client-library"></a>Uso de la biblioteca cliente del plano de control

La biblioteca cliente del plano de control (azure.mgmt.confidentialledger) permite operaciones en libros de contabilidad, como la creación, modificación y eliminación, enumeración de los libros de contabilidad asociados a una suscripción y la obtención de los detalles de un libro de contabilidad específico.

En el código, primero crearemos un cliente del plano de control pasando la variable de credencial ConfidentialLedgerAPI y el identificador de la suscripción a Azure (ambos se establecieron anteriormente).  

```python
confidential_ledger_mgmt = ConfidentialLedgerAPI(
    credential, subscription_id
)
```

Ahora podemos crear un libro de contabilidad mediante `begin_create`. La función `begin_create` requiere tres parámetros: el grupo de recursos, un nombre para el libro de contabilidad y un objeto "properties".  

Cree un diccionario `properties` con las siguientes claves y valores, y asígnelo a una variable.

```python
properties = {
    "location": "eastus",
    "tags": {},
    "properties": {
        "ledgerType": "Public",
        "aadBasedSecurityPrincipals": [],
    },
}

ledger_properties = ConfidentialLedger(**properties)
```

Ahora pase el grupo de recursos, el nombre del libro de contabilidad y el objeto properties a `begin_create`.

```python
confidential_ledger_mgmt.ledger.begin_create(resource_group, ledger_name, ledger_properties)
```

Para comprobar que el libro de contabilidad se creó correctamente, vea los detalles mediante la función `get`.

```python
myledger = ledger = confidential_ledger_mgmt.ledger.get(resource_group, ledger_name)

print("Here are the details of your newly created ledger:")
print (f"- Name: {myledger.name}")
print (f"- Location: {myledger.location}")
print (f"- ID: {myledger.id}")

```

### <a name="use-the-data-plane-client-library"></a>Uso de la biblioteca cliente del plano de datos

Ahora que tenemos un libro de contabilidad, interactuaremos con él mediante la biblioteca cliente del plano de datos (azure.confidentialledger). 

En primer lugar, generaremos un certificado de Confidential Ledger y lo guardaremos.  

```python
identity_client = ConfidentialLedgerIdentityServiceClient(identity_url)
network_identity = identity_client.get_ledger_identity(
     ledger_id=ledger_name
)

ledger_tls_cert_file_name = "networkcert.pem"
with open(ledger_tls_cert_file_name, "w") as cert_file:
    cert_file.write(network_identity.ledger_tls_certificate)
```

Ahora podemos usar el certificado de red, junto con la dirección URL del libro de contabilidad y las credenciales, para crear un cliente de Confidential Ledger.

```python
ledger_client = ConfidentialLedgerClient(
     endpoint=ledger_url, 
     credential=credential,
     ledger_certificate_path=ledger_tls_cert_file_name
)
```

Estamos preparados para escribir en el libro de contabilidad.  Lo haremos mediante la función `append_to_ledger`.

```python
append_result = ledger_client.append_to_ledger(entry_contents="Hello world!")
print(append_result.transaction_id)
```

La función print devolverá el identificador de la transacción de la escritura en el libro de contabilidad, que se puede usar para recuperar el mensaje que escribió en el libro de contabilidad.

```python
entry = ledger_client.get_ledger_entry(transaction_id=append_result.transaction_id)
print(entry.contents)
```

La función print devolverá "Hello world!", ya que es el mensaje del libro de contabilidad que corresponde al identificador de la transacción.

## <a name="full-sample-code"></a>Código de ejemplo completo

```python
from azure.identity import DefaultAzureCredential

## Import control plane sdk

from azure.mgmt.confidentialledger import ConfidentialLedger as ConfidentialLedgerAPI
from azure.mgmt.confidentialledger.models import ConfidentialLedger

# import data plane sdk

from azure.confidentialledger import ConfidentialLedgerClient
from azure.confidentialledger.identity_service import ConfidentialLedgerIdentityServiceClient

# Set variables

rg = "myResourceGroup"
ledger_name = "<unique-ledger-name>"
subscription_id "<azure-subscription-id>"

identity_url = "https://identity.confidential-ledger.core.azure.com"
ledger_url = "https://" + ledger_name + ".eastus.cloudapp.azure.com"

# Authentication

# Need to do az login to get default credential to work

credential = DefaultAzureCredential()

# Control plane (azure.mgmt.confidentialledger)
# 
# initialize endpoint with credential and subscription

confidential_ledger_mgmt = ConfidentialLedgerAPI(
    credential, "<subscription-id>"
)

# Create properties dictionary for begin_create call 

properties = {
    "location": "eastus",
    "tags": {},
    "properties": {
        "ledgerType": "Public",
        "aadBasedSecurityPrincipals": [],
    },
}

ledger_properties = ConfidentialLedger(**properties)

# Create a ledger

foo = confidential_ledger_mgmt.ledger.begin_create(rg, ledger_name, ledger_properties)

# Get the details of the ledger you just created

print(f"{rg} / {ledger_name}")
 
print("Here are the details of your newly created ledger:")
myledger = confidential_ledger_mgmt.ledger.get(rg, ledger_name)

print (f"- Name: {myledger.name}")
print (f"- Location: {myledger.location}")
print (f"- ID: {myledger.id}")

# Data plane (azure.confidentialledger)
#
# Create a CL client

identity_client = ConfidentialLedgerIdentityServiceClient(identity_url)
network_identity = identity_client.get_ledger_identity(
     ledger_id=ledger_name
)

ledger_tls_cert_file_name = "networkcert.pem"
with open(ledger_tls_cert_file_name, "w") as cert_file:
    cert_file.write(network_identity.ledger_tls_certificate)


ledger_client = ConfidentialLedgerClient(
     endpoint=ledger_url, 
     credential=credential,
     ledger_certificate_path=ledger_tls_cert_file_name
)

# Write to the ledger
append_result = ledger_client.append_to_ledger(entry_contents="Hello world!")
print(append_result.transaction_id)

# Read from the ledger
entry = ledger_client.get_ledger_entry(transaction_id=append_result.transaction_id)
print(entry.contents)
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Otros artículos Microsoft Azure Confidential Ledger se pueden basar en esta guía de inicio rápido. Si tiene pensado seguir trabajando en otras guías de inicio rápido y tutoriales, considere la posibilidad de dejar estos recursos activos.

De lo contrario, cuando haya terminado con los recursos creados en este artículo, use el comando [az group delete](/cli/azure/group?#az_group_delete) de la CLI de Azure para eliminar el grupo de recursos y todos los recursos que contiene:

```azurecli
az group delete --resource-group myResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

- [Información general de Microsoft Azure Confidential Ledger](overview.md)
