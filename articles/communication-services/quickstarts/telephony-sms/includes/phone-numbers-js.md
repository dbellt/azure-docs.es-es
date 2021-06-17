---
ms.openlocfilehash: 6fbd65b75ebb061b8012d4841ed03f331777e6ef
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111429454"
---
> [!NOTE]
> Busque el código finalizado de este inicio rápido en [GitHub](https://github.com/Azure-Samples/communication-services-javascript-quickstarts/tree/main/phone-numbers-quickstart)

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Versiones de [Node.js](https://nodejs.org/), Active LTS y Maintenance LTS (se recomiendan 8.11.1 y 10.14.1).
- Un recurso activo de Communication Services y una cadena de conexión. [Cree un recurso de Communication Services](../../create-communication-resource.md).

### <a name="prerequisite-check"></a>Comprobación de requisitos previos

- En una ventana de terminal o de comandos, ejecute `node --version` para comprobar que Node.js está instalado.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-nodejs-application"></a>Creación de una nueva aplicación Node.js

En primer lugar, abra la ventana de comandos o de terminal, cree un nuevo directorio para la aplicación y navegue hasta este.

```console
mkdir phone-numbers-quickstart && cd phone-numbers-quickstart
```

Ejecute `npm init -y` para crear un archivo **package.json** con la configuración predeterminada.

```console
npm init -y
```

Cree un archivo llamado **phone-numbers-quickstart.js** en la raíz del directorio que acaba de crear. Agréguele el siguiente fragmento de código:

```javascript
async function main() {
    // quickstart code will here
}

main();
```

### <a name="install-the-package"></a>Instalar el paquete

Use el comando `npm install` para instalar la biblioteca cliente de números de teléfono de Azure Communication Services para JavaScript.

```console
npm install @azure/communication-phone-numbers --save
```

La opción `--save` agrega la biblioteca como dependencia en el archivo **package.json**.

## <a name="authenticate-the-client"></a>Autenticar el cliente

Importe la clase **PhoneNumbersClient** desde la biblioteca cliente y cree una instancia con la cadena de conexión. El código siguiente recupera la cadena de conexión para el recurso de una variable de entorno denominada `COMMUNICATION_SERVICES_CONNECTION_STRING`. Aprenda a [administrar la cadena de conexión del recurso](../../create-communication-resource.md#store-your-connection-string).

Agregue el código siguiente en la parte superior de **phone-numbers-quickstart.js**:

```javascript
const { PhoneNumbersClient } = require('@azure/communication-phone-numbers');

// This code demonstrates how to fetch your connection string
// from an environment variable.
const connectionString = process.env['COMMUNICATION_SERVICES_CONNECTION_STRING'];

// Instantiate the phone numbers client
const phoneNumbersClient = new PhoneNumbersClient(connectionString);
```

## <a name="manage-phone-numbers"></a>Administración de números de teléfono

### <a name="search-for-available-phone-numbers"></a>Búsqueda de los números de teléfono disponibles

Para adquirir números de teléfono, primero debe buscar los que están disponibles. Para buscar números de teléfono, proporcione el código de área, el tipo de asignación, las [funcionalidades del número de teléfono](../../../concepts/telephony-sms/plan-solution.md#phone-number-capabilities-in-azure-communication-services), el [tipo de número de teléfono](../../../concepts/telephony-sms/plan-solution.md#phone-number-types-in-azure-communication-services) y la cantidad. Tenga en cuenta que para el tipo de número de teléfono gratuito, el código de área es opcional.

Agregue el siguiente fragmento de código a la función `main`:

```javascript
/**
 * Search for Available Phone Number
 */

// Create search request
const searchRequest = {
    countryCode: "US",
    phoneNumberType: "tollFree",
    assignmentType: "application",
    capabilities: {
      sms: "outbound",
      calling: "none"
    },
    areaCode: "833",
    quantity: 1
  };

const searchPoller = await phoneNumbersClient.beginSearchAvailablePhoneNumbers(searchRequest);

// The search is underway. Wait to receive searchId.
const { searchId, phoneNumbers } = await searchPoller.pollUntilDone();
const phoneNumber = phoneNumbers[0];

console.log(`Found phone number: ${phoneNumber}`);
console.log(`searchId: ${searchId}`);
```

### <a name="purchase-phone-number"></a>Compra de un número de teléfono

El resultado de la búsqueda de números de teléfono es `PhoneNumberSearchResult`. Este valor contiene el valor de `searchId`, que se puede pasar a la API de compra de números de para obtener los números en la búsqueda. Tenga en cuenta que, al llamar a la API de compra de números de teléfono, se realizará un cargo a su cuenta de Azure.

Agregue el siguiente fragmento de código a la función `main`:

```javascript
/**
 * Purchase Phone Number
 */

const purchasePoller = await phoneNumbersClient.beginPurchasePhoneNumbers(searchId);

// Purchase is underway.
await purchasePoller.pollUntilDone();
console.log(`Successfully purchased ${phoneNumber}`);
```

### <a name="update-phone-number-capabilities"></a>Actualización de las funcionalidades del número de teléfono

Ahora que ha comprado un número de teléfono, agregue el código siguiente para actualizar sus funcionalidades:

```javascript
/**
 * Update Phone Number Capabilities
 */

// Create update request.
// This will update phone number to send and receive sms, but only send calls.
const updateRequest = {
  sms: "inbound+outbound",
  calling: "outbound"
};

const updatePoller = await phoneNumbersClient.beginUpdatePhoneNumberCapabilities(
  phoneNumber,
  updateRequest
);

// Update is underway.
await updatePoller.pollUntilDone();
console.log("Phone number updated successfully.");
```

### <a name="get-purchased-phone-numbers"></a>Obtención de los números de teléfono comprados

Una vez comprado, puede recuperar el número del cliente. Agregue el código siguiente a la función `main`para obtener el número de teléfono que acaba de comprar:

```javascript
/**
 * Get Purchased Phone Number
 */

const { capabilities } = await phoneNumbersClient.getPurchasedPhoneNumber(phoneNumber);
console.log("These capabilities:", capabilities, "should be the same as these:", updateRequest, ".");
```

También puede recuperar todos los números de teléfono comprados.

```javascript
const purchasedPhoneNumbers = await phoneNumbersClient.listPurchasedPhoneNumbers();

for await (const purchasedPhoneNumber of purchasedPhoneNumbers) {
  console.log(`Phone number: ${purchasedPhoneNumber.phoneNumber}, country code: ${purchasedPhoneNumber.countryCode}.`);
}
```

### <a name="release-phone-number"></a>Descarte de un número de teléfono

Ahora puede descartar el número de teléfono comprado. Agregue el siguiente fragmento de código a la función `main`:

```javascript
/**
 * Release Purchased Phone Number
 */

const releasePoller = await phoneNumbersClient.beginReleasePhoneNumber(phoneNumber);

// Release is underway.
await releasePoller.pollUntilDone();
console.log("Successfully release phone number.");
```

## <a name="run-the-code"></a>Ejecución del código

Use el comando `node` para ejecutar el código que agregó al archivo **phone-numbers-quickstart.js**.

```console
node phone-numbers-quickstart.js
```