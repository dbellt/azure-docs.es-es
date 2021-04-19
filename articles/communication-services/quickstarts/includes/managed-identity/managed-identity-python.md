---
ms.openlocfilehash: 42d079a2aa98549b12aafecdd8d58f3361db8b4d
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107307472"
---
## <a name="setting-up"></a>Instalación

## <a name="create-a-new-python-application"></a>Creación de una nueva aplicación de Python

Abra la ventana de comandos o de terminal, cree un nuevo directorio para la aplicación y navegue hasta este.

```console
mkdir managed-identity-quickstart && cd managed-identity-quickstart
```

### <a name="install-the-sdk-packages"></a>Instalación de los paquetes del SDK

```console
pip install azure-identity
pip install azure-communication-identity
pip install azure-communication-sms
```

### <a name="create-a-new-file"></a>Creación de un archivo nuevo
Abra y guarde un nuevo archivo en la carpeta creada denominada `managed-identity.py`, colocaremos nuestro código dentro de este archivo.

### <a name="use-the-sdk-packages"></a>Uso de los paquetes del SDK

Agregue las siguientes instrucciones `import` en la parte superior del archivo para usar los SDK que hemos instalado.

```python
from azure.identity import DefaultAzureCredential
from azure.communication.identity import CommunicationIdentityClient
from azure.communication.sms import SmsClient
```

### <a name="create-a-defaultazurecredential"></a>Creación de una credencial DefaultAzureCredential

Usaremos [DefaultAzureCredential.](/python/api/azure-identity/azure.identity.defaultazurecredential) Esta credencial es adecuada para entornos de producción y desarrollo. Como la usaremos en todo este inicio rápido, la crearemos en la parte superior del archivo.

```python
     credential = DefaultAzureCredential()
```

## <a name="create-an-identity-and-issue-a-token-with-managed-identities"></a>Cree una identidad y emita un token con identidades administradas.

Ahora agregaremos código que usa la credencial creada para emitir un token de acceso VoIP. Llamaremos a este código más adelante:

```python
def create_identity_and_get_token(resource_endpoint):
     client = CommunicationIdentityClient(resource_endpoint, credential)

     user = client.create_user()
     token_response = client.get_token(user, scopes=["voip"])

     return token_response
```

### <a name="send-an-sms-with-managed-identities"></a>Envío de un SMS con identidades administradas
Como otro ejemplo de uso de identidades administradas, agregaremos este código que usa la misma credencial para enviar un SMS:

```python
def send_sms(resource_endpoint, from_phone_number, to_phone_number, message_content):
     sms_client = SmsClient(resource_endpoint, credential)

     sms_client.send(
          from_=from_phone_number,
          to_=[to_phone_number],
          message=message_content,
          enable_delivery_report=True  # optional property
     )
```

## <a name="write-our-main-code"></a>Escritura del código principal

Con las funciones creadas, podemos escribir el código principal que llamará a las funciones que hemos escrito anteriormente.

```python
# You can find your endpoint and access key from your resource in the Azure portal
# e.g. "https://<RESOURCE_NAME>.communication.azure.com";
endpoint = "https://<RESOURCE_NAME>.communication.azure.com/"

print("Retrieving new Access Token, using Managed Identities");
result = create_identity_and_get_token(endpoint);
print(f'Retrieved Access Token: {result.token}');

print("Sending SMS using Managed Identities");

# You will need a phone number from your resource to send an SMS.
sms_result = send_sms(endpoint, "<FROM_NUMBER>", "<TO_NUMBER>", "Hello from Managed Identities");
print(f'SMS ID: {sms_result[0].message_id}');
print(f'Send Result Successful: {sms_result[0].successful}');
```

El archivo `managed-identity.py` final debe tener un aspecto similar al siguiente:

```python
from azure.identity import DefaultAzureCredential
from azure.communication.identity import CommunicationIdentityClient
from azure.communication.sms import SmsClient

credential = DefaultAzureCredential()

def create_identity_and_get_token(resource_endpoint):
     client = CommunicationIdentityClient(resource_endpoint, credential)

     user = client.create_user()
     token_response = client.get_token(user, scopes=["voip"])

     return token_response

def send_sms(resource_endpoint, from_phone_number, to_phone_number, message_content):
     sms_client = SmsClient(resource_endpoint, credential)

     response = sms_client.send(
          from_=from_phone_number,
          to=[to_phone_number],
          message=message_content,
          enable_delivery_report=True  # optional property
     )
     return response

# You can find your endpoint and access key from your resource in the Azure portal
# e.g. "https://<RESOURCE_NAME>.communication.azure.com";
endpoint = "https://<RESOURCE_NAME>.communication.azure.com/"

print("Retrieving new Access Token, using Managed Identities");
result = create_identity_and_get_token(endpoint);
print(f'Retrieved Access Token: {result.token}');

print("Sending SMS using Managed Identities");

# You will need a phone number from your resource to send an SMS.
sms_result = send_sms(endpoint, "<FROM_NUMBER>", "<TO_NUMBER>", "Hello from Managed Identities");
print(f'SMS ID: {sms_result[0].message_id}');
print(f'Send Result Successful: {sms_result[0].successful}');
```
## <a name="run-the-program"></a>Ejecución del programa

Una vez completado todo, puede ejecutar el archivo si especifica `python managed-identity.py` desde el directorio del proyecto. Si todo es correcto, debe ver algo parecido a lo siguiente.

```Bash
    $ python managed-identity.py
    Retrieving new Access Token, using Managed Identities
    Retrieved Access Token: ey...Q
    Sending SMS using Managed Identities
    SMS ID: Outgoing_2021040602194...._noam
    Send Result Successful: true
```
