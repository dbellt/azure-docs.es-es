---
ms.openlocfilehash: c642b0e5f459b2412bca6588c8ae625142f0f59f
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106450408"
---
## <a name="add-managed-identity-to-your-communication-services-solution"></a>Incorporación de la identidad administrada a Communication Services

### <a name="install-the-sdk-packages"></a>Instalación de los paquetes del SDK

```console
pip install azure-identity
pip install azure-communication-identity
pip install azure-communication-sms
```

### <a name="use-the-sdk-packages"></a>Uso de los paquetes del SDK

Agregue `import` al código para que use la identidad de Azure.

```python
from azure.identity import DefaultAzureCredential
```

En los ejemplos siguientes se usa [DefaultAzureCredential](/python/api/azure-identity/azure.identity.defaultazurecredential). Esta credencial es adecuada para entornos de producción y desarrollo.

Para conocer una forma sencilla de pasar a usar la autenticación de identidad administrada, consulte [Autorización del acceso con identidad administrada](../managed-identity-from-cli.md).

Para obtener una visión más detallada del funcionamiento del objeto DefaultAzureCredential y de su uso de maneras que no se especifican en este inicio rápido, consulte [Biblioteca cliente de Azure Identity para Python](https://docs.microsoft.com/python/api/overview/azure/identity-readme).

### <a name="create-an-identity-and-issue-a-token"></a>Creación de una identidad y emisión de un token

En el ejemplo de código siguiente se muestra cómo crear un objeto de cliente de servicio con una identidad administrada y, a continuación, usar el cliente para emitir un token para un nuevo usuario:

```python
from azure.communication.identity import CommunicationIdentityClient

def create_identity_and_get_token(resource_endpoint):
     credential = DefaultAzureCredential()
     client = CommunicationIdentityClient(resource_endpoint, credential)

     user = client.create_user()
     token_response = client.get_token(user, scopes=["voip"])

     return token_response
```

### <a name="send-an-sms-with-azure-managed-identity"></a>Envío de un SMS con una identidad administrada de Azure
En el ejemplo de código siguiente se muestra cómo crear un objeto de cliente de servicio con una identidad administrada de Azure y, a continuación, usar el cliente para enviar un mensaje SMS:

```python
from azure.communication.sms import SmsClient

def send_sms(resource_endpoint, from_phone_number, to_phone_number, message_content):
     credential = DefaultAzureCredential()
     sms_client = SmsClient(resource_endpoint, credential)

     sms_client.send(
          from_=from_phone_number,
          to_=[to_phone_number],
          message=message_content,
          enable_delivery_report=True  # optional property
     )
```

### <a name="list-all-your-purchased-phone-numbers"></a>Enumeración de todos los números de teléfono comprados

En el ejemplo de código siguiente se muestra cómo crear un objeto cliente de servicio con Azure Managed Identity y, después, usar el cliente para ver todos los números de teléfono comprados que tiene el recurso:

```python
from azure.communication.phonenumbers import PhoneNumbersClient

def list_purchased_phone_numbers(resource_endpoint):
     credential = DefaultAzureCredential()
     phone_numbers_client = PhoneNumbersClient(resource_endpoint, credential)

     return phone_numbers_client.list_purchased_phone_numbers()
```