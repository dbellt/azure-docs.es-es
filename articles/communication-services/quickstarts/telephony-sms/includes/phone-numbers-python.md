---
ms.openlocfilehash: 61648bc0eab3aba4806cf4594e6fe222ac77f93f
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111993592"
---
> [!NOTE]
> Busque el código finalizado de este inicio rápido en [GitHub](https://github.com/Azure-Samples/communication-services-python-quickstarts/tree/main/phone-numbers-quickstart)

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Python](https://www.python.org/downloads/) 2.7, 3.5 o versiones posteriores
- Un recurso de Communication Services implementado y una cadena de conexión. [Cree un recurso de Communication Services](../../create-communication-resource.md).

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-python-application"></a>Creación de una nueva aplicación de Python

Abra la ventana de comandos o el terminal, cree un directorio para la aplicación y vaya hasta él.

```console
mkdir phone-numbers-quickstart && cd phone-numbers-quickstart
```

Use un editor de texto para crear un archivo denominado phone_numbers_sample.py en el directorio raíz del proyecto y agregue el siguiente código. En las siguientes secciones se agregará el resto de código del inicio rápido.

```python
import os
from azure.communication.phonenumbers import PhoneNumbersClient

try:
   print('Azure Communication Services - Phone Numbers Quickstart')
   # Quickstart code goes here
except Exception as ex:
   print('Exception:')
   print(ex)
```

### <a name="install-the-package"></a>Instalar el paquete

Mientras sigue en el directorio de aplicaciones, instale el paquete de la biblioteca cliente de administración de Azure Communication Services para Python con el comando `pip install`.

```console
pip install azure-communication-phonenumbers
```

## <a name="authenticate-the-phone-numbers-client"></a>Autenticación del cliente de números de teléfono

`PhoneNumbersClient` está habilitado para usar la autenticación de Azure Active Directory. El uso del objeto `DefaultAzureCredential` es la forma más fácil de empezar a trabajar con Azure Active Directory y puede instalarlo mediante el comando `pip install`.

```console
pip install azure-identity
```

La creación de un objeto `DefaultAzureCredential` requiere que tenga `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET` y `AZURE_TENANT_ID` ya establecidos como variables de entorno con sus valores correspondientes en la aplicación de Azure AD registrada.

Para obtener información rápida sobre cómo obtener estas variables de entorno, puede seguir el inicio rápido [Configuración de identidades administradas desde la CLI](../../managed-identity-from-cli.md).

Una vez que haya instalado la biblioteca `azure-identity`, podemos seguir con la autenticación del cliente.

```python
import os
from azure.communication.phonenumbers import PhoneNumbersClient
from azure.identity import DefaultAzureCredential

# You can find your endpoint from your resource in the Azure portal
endpoint = 'https://<RESOURCE_NAME>.communication.azure.com'
try:
    print('Azure Communication Services - Phone Numbers Quickstart')
    credential = DefaultAzureCredential()
    phone_numbers_client = PhoneNumbersClient(endpoint, credential)
except Exception as ex:
    print('Exception:')
    print(ex)
```

Como alternativa, también es posible usar el punto de conexión y la clave de acceso del recurso de comunicación para la autenticación.

```python
import os
from azure.communication.phonenumbers import PhoneNumbersClient

# You can find your connection string from your resource in the Azure portal
connection_string = 'https://<RESOURCE_NAME>.communication.azure.com/;accesskey=<YOUR_ACCESS_KEY>'
try:
    print('Azure Communication Services - Phone Numbers Quickstart')
    phone_numbers_client = PhoneNumbersClient.from_connection_string(connection_string)
except Exception as ex:
    print('Exception:')
    print(ex)
```

## <a name="functions"></a>Functions

Una vez que se ha autenticado `PhoneNumbersClient`, podemos empezar a trabajar en las diferentes funciones que puede hacer.

### <a name="search-for-available-phone-numbers"></a>Búsqueda de los números de teléfono disponibles

Para comprar números de teléfono, primero debe buscar los números de teléfono disponibles. Para buscar números de teléfono, proporcione el código de área, el tipo de asignación, las [funcionalidades de número de teléfono](../../../concepts/telephony-sms/plan-solution.md#phone-number-capabilities-in-azure-communication-services), el [tipo de número de teléfono](../../../concepts/telephony-sms/plan-solution.md#phone-number-types-in-azure-communication-services) y la cantidad (el valor predeterminado está establecido en 1). Tenga en cuenta que proporcionar el código de área para el tipo de número de teléfono gratuito es opcional.

```python
import os
from azure.communication.phonenumbers import PhoneNumbersClient, PhoneNumberCapabilityType, PhoneNumberAssignmentType, PhoneNumberType, PhoneNumberCapabilities
from azure.identity import DefaultAzureCredential

# You can find your endpoint from your resource in the Azure portal
endpoint = 'https://<RESOURCE_NAME>.communication.azure.com'
try:
    print('Azure Communication Services - Phone Numbers Quickstart')
    credential = DefaultAzureCredential()
    phone_numbers_client = PhoneNumbersClient(endpoint, credential)
    capabilities = PhoneNumberCapabilities(
        calling = PhoneNumberCapabilityType.INBOUND,
        sms = PhoneNumberCapabilityType.INBOUND_OUTBOUND
    )
    search_poller = phone_numbers_client.begin_search_available_phone_numbers(
        "US",
        PhoneNumberType.TOLL_FREE,
        PhoneNumberAssignmentType.APPLICATION,
        capabilities,
        polling = True
    )
    search_result = search_poller.result()
    print ('Search id: ' + search_result.search_id)
    phone_number_list = search_result.phone_numbers
    print('Reserved phone numbers:')
    for phone_number in phone_number_list:
        print(phone_number)

except Exception as ex:
    print('Exception:')
    print(ex)
```

### <a name="purchase-phone-numbers"></a>Compra de números de teléfono

El resultado de la búsqueda de números de teléfono es un valor de `PhoneNumberSearchResult`. Este valor contiene el valor de `searchId`, que se puede pasar a la API de compra de números de para obtener los números en la búsqueda. Tenga en cuenta que, al llamar a la API de compra de números de teléfono, se realizará un cargo a su cuenta de Azure.

```python
import os
from azure.communication.phonenumbers import (
    PhoneNumbersClient,
    PhoneNumberCapabilityType,
    PhoneNumberAssignmentType,
    PhoneNumberType,
    PhoneNumberCapabilities
)
from azure.identity import DefaultAzureCredential

# You can find your endpoint from your resource in the Azure portal
endpoint = 'https://<RESOURCE_NAME>.communication.azure.com'
try:
    print('Azure Communication Services - Phone Numbers Quickstart')
    credential = DefaultAzureCredential()
    phone_numbers_client = PhoneNumbersClient(endpoint, credential)
    capabilities = PhoneNumberCapabilities(
        calling = PhoneNumberCapabilityType.INBOUND,
        sms = PhoneNumberCapabilityType.INBOUND_OUTBOUND
    )
    search_poller = phone_numbers_client.begin_search_available_phone_numbers(
        "US",
        PhoneNumberType.TOLL_FREE,
        PhoneNumberAssignmentType.APPLICATION,
        capabilities,
        area_code="833",
        polling = True
    )
    search_result = poller.result()
    print ('Search id: ' + search_result.search_id)
    phone_number_list = search_result.phone_numbers
    print('Reserved phone numbers:')
    for phone_number in phone_number_list:
        print(phone_number)

    purchase_poller = phone_numbers_client.begin_purchase_phone_numbers(search_result.search_id, polling = True)
    purchase_poller.result()
    print("The status of the purchase operation was: " + purchase_poller.status())
except Exception as ex:
    print('Exception:')
    print(ex)
```

### <a name="get-purchased-phone-numbers"></a>Obtención de los números de teléfono comprados

Después de comprar un número, puede recuperarlo del cliente.

```python
purchased_phone_number_information = phone_numbers_client.get_purchased_phone_number("+18001234567")
print('Phone number: ' + purchased_phone_number_information.phone_number)
print('Country code: ' + purchased_phone_number_information.country_code)
```

También puede recuperar todos los números de teléfono comprados.

```python
purchased_phone_numbers = phone_numbers_client.list_purchased_phone_numbers()
print('Purchased phone numbers:')
for purchased_phone_number in purchased_phone_numbers:
    print(purchased_phone_number.phone_number)
```

### <a name="update-phone-number-capabilities"></a>Actualización de las funcionalidades del número de teléfono

Puede actualizar las funcionalidades de un número de teléfono anteriormente comprado.
```python
update_poller = phone_numbers_client.begin_update_phone_number_capabilities(
    "+18001234567",
    PhoneNumberCapabilityType.OUTBOUND,
    PhoneNumberCapabilityType.OUTBOUND,
    polling = True
)
update_poller.result()
print('Status of the operation: ' + update_poller.status())
```

### <a name="release-phone-number"></a>Liberación de números de teléfono

Puede liberar los números de teléfono comprados.

```python
release_poller = phone_numbers_client.begin_release_phone_number("+18001234567")
release_poller.result()
print('Status of the operation: ' + release_poller.status())
```

## <a name="run-the-code"></a>Ejecución del código

Desde un símbolo del sistema de la consola, vaya al directorio que contiene el archivo phone_numbers_sample.py y ejecute el siguiente comando de Python para ejecutar la aplicación.

```console
python phone_numbers_sample.py
```