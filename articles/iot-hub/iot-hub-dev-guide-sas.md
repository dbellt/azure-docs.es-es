---
title: Control del acceso a IoT Hub con el uso de tokens de SAS | Microsoft Docs
description: Cómo controlar el acceso a IoT Hub para aplicaciones de dispositivo y de back-end mediante tokens de firma de acceso compartido.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 04/21/2021
ms.custom:
- amqp
- mqtt
- 'Role: Cloud Development'
- 'Role: IoT Device'
- 'Role: Operations'
- devx-track-js
- devx-track-csharp
ms.openlocfilehash: 27ab47be439b83af4297330c2b85fdc844bfcf3a
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109490092"
---
# <a name="control-access-to-iot-hub-using-shared-access-signatures-and-security-tokens"></a>Control del acceso a IoT Hub mediante firmas de acceso compartido y tokens de seguridad

En esta sección se describen las opciones para proteger su instancia de IoT Hub. IoT Hub usa *permisos* para conceder acceso a cada uno de los puntos de conexión de IoT Hub. Los permisos limitan el acceso a un centro de IoT basado en la funcionalidad.

En este artículo se presentan:

* Los distintos permisos que puede conceder a un cliente para acceder a IoT Hub.
* Los tokens que IoT Hub usa para comprobar los permisos.
* Cómo definir el ámbito de las credenciales para limitar el acceso a recursos específicos.
* Los mecanismos de autenticación de dispositivo personalizado que utilizan los registros de identidad de dispositivo o los esquemas de autenticación.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

Debe tener los permisos adecuados para acceder a cualquiera de los puntos de conexión de IoT Hub. Por ejemplo, un dispositivo debe incluir un token que contiene las credenciales de seguridad junto con cada mensaje que se envía a IoT Hub. Sin embargo, las claves de firma, como las claves simétricas del dispositivo, nunca se envían durante la conexión.

## <a name="access-control-and-permissions"></a>Permisos y control del acceso

Use las directivas de acceso compartido para acceder al nivel de centro de IoT, y use las credenciales de dispositivo individuales para otorgar acceso solamente a ese dispositivo.

### <a name="iot-hub-level-shared-access-policies"></a>Directivas de acceso compartido de nivel de centro de IoT

Las directivas de acceso compartido pueden conceder cualquier combinación de los permisos. Puede definir directivas en [Azure Portal](https://portal.azure.com), mediante programación con las [API REST de recursos de IoT Hub](/rest/api/iothub/iothubresource) o con el comando [az iot hub policy](/cli/azure/iot/hub/policy) de la CLI. Un Centro de IoT recién creado tiene las siguientes directivas predeterminadas:
  
| Directiva de acceso compartido | Permisos |
| -------------------- | ----------- |
| iothubowner | Todos los permisos |
| service | Permisos **ServiceConnect** |
| device | Permisos **DeviceConnect** |
| registryRead | Permisos **RegistryRead** |
| registryReadWrite | Permisos **RegistryRead** y **RegistryWrite** |

La tabla siguiente enumera los permisos que se puede utilizar para controlar el acceso a su centro de IoT.

| Permiso | Notas |
| --- | --- |
| **ServiceConnect**. |Concede acceso a los puntos de conexión de comunicación y supervisión accesibles desde el servicio en la nube. <br/>Concede permiso para recibir mensajes de dispositivo a nube, enviar mensajes de nube a dispositivo y recuperar las confirmaciones de entrega correspondientes. <br/>Concede permiso para recuperar las confirmaciones de entrega para las cargas de archivos. <br/>Concede permiso para tener acceso a los gemelos para actualizar las etiquetas y las propiedades deseadas, recuperar las propiedades notificadas y ejecutar consultas. <br/>Este permiso se usa en servicios de back-end en la nube. |
| **DeviceConnect**. |Concede acceso a los puntos de conexión accesibles desde los dispositivos. <br/>Concede permiso para enviar mensajes de dispositivo a nube y recibir mensajes de nube a dispositivo. <br/>Concede permiso para realizar la carga de archivos desde un dispositivo. <br/>Concede permiso para recibir notificaciones sobre las propiedades de los dispositivos gemelos que desee y actualizar las propiedades notificadas de esos dispositivos gemelos. <br/>Concede permiso para realizar cargas de archivos. <br/>Este permiso lo usan los dispositivos. |
| **RegistryRead**. |Concede acceso de lectura al Registro de identidad. Para más información, consulte [Registro de identidades](iot-hub-devguide-identity-registry.md). <br/>Este permiso se usa en servicios de back-end en la nube. |
| **RegistryReadWrite**. |Concede acceso de lectura y escritura al Registro de identidad. Para más información, consulte [Registro de identidades](iot-hub-devguide-identity-registry.md). <br/>Este permiso se usa en servicios de back-end en la nube. |

### <a name="per-device-security-credentials"></a>Credenciales de seguridad de cada dispositivo

Cada instancia de IoT Hub contiene un [registro de identidades](iot-hub-devguide-identity-registry.md), y para cada dispositivo de este registro de identidades, puede configurar credenciales de seguridad que otorgan permisos **DeviceConnect** con ámbito a los puntos de conexión de dispositivo correspondientes.

## <a name="security-tokens"></a>Tokens de seguridad

IoT Hub usa tokens de seguridad para autenticar dispositivos y servicios para evitar el envío de claves en la conexión. Además, los tokens de seguridad están limitados en cuanto al ámbito y el período de validez. Estos tokens de seguridad también se conocen como tokens de firma de acceso compartido (SAS). Los [SDK de Azure IoT](iot-hub-devguide-sdks.md) generan automáticamente tokens sin necesidad de ninguna configuración especial. Algunos escenarios, requieren que el usuario genere y utilice directamente los tokens de seguridad. Entre los escenarios se incluyen los siguientes:

* El uso directo de las superficies MQTT, AMQP o HTTPS.

* La implementación del patrón de servicio de tokens, como se explica en [Personalización de la autenticación de dispositivos](iot-hub-dev-guide-sas.md#create-a-token-service-to-integrate-existing-devices).

Utilice tokens de seguridad para conceder acceso limitado en tiempo a los dispositivos y servicios en la funcionalidad específica de IoT Hub. Para obtener autorización para conectarse a IoT Hub, los dispositivos y servicios deben enviar tokens de seguridad firmados con un acceso compartido o una clave simétrica. Dichas claves se almacenan con una identidad de dispositivo en el registro de identidad.

### <a name="security-token-structure"></a>Estructura del token de seguridad

Un token firmado con una clave de acceso compartido concede acceso a toda la funcionalidad asociada con los permisos de la directiva de acceso compartido. Un token firmado con una clave simétrica de identidad del dispositivo solo concede el permiso **DeviceConnect** para la identidad del dispositivo asociado.

El token de seguridad tiene el formato siguiente:

`SharedAccessSignature sig={signature-string}&se={expiry}&skn={policyName}&sr={URL-encoded-resourceURI}`

Estos son los valores esperados:

| Value | Descripción |
| --- | --- |
| {signature} |Una cadena de firma HMAC-SHA256 con el formato: `{URL-encoded-resourceURI} + "\n" + expiry`. **Importante**: La clave se descodifica en base64 y se utiliza para realizar el cálculo de HMAC-SHA256. |
| {resourceURI} |Prefijo del identificador URI (por segmento) de los puntos de conexión a los que se puede obtener acceso con este token, que comienza por un nombre de host de IoT Hub (sin protocolo) Por ejemplo: `myHub.azure-devices.net/devices/device1` |
| {expiry} |Cadenas UTF8 para el número de segundos transcurridos desde el tiempo 00:00:00 UTC el 1 de enero de 1970. |
| {URL-encoded-resourceURI} |Codificación de dirección URL en minúsculas del URI del recurso en minúsculas |
| {policyName} |El nombre de la directiva de acceso compartido a la que hace referencia este token. Ausente en caso de que el token haga referencia a las credenciales del registro de dispositivos. |

el prefijo URI se calcula por segmento y no por carácter. Por ejemplo `/a/b` es un prefijo de `/a/b/c` pero `/a/bc`.

### <a name="nodejs"></a>[Node.js](#tab/node)

El siguiente fragmento de Node.js muestra una función denominada **generateSasToken** que calcula el token de las entradas `resourceUri, signingKey, policyName, expiresInMins`. Las secciones siguientes detallan cómo inicializar las entradas diferentes para los distintos casos de uso de token.

```javascript
var generateSasToken = function(resourceUri, signingKey, policyName, expiresInMins) {
    resourceUri = encodeURIComponent(resourceUri);

    // Set expiration in seconds
    var expires = (Date.now() / 1000) + expiresInMins * 60;
    expires = Math.ceil(expires);
    var toSign = resourceUri + '\n' + expires;

    // Use crypto
    var hmac = crypto.createHmac('sha256', Buffer.from(signingKey, 'base64'));
    hmac.update(toSign);
    var base64UriEncoded = encodeURIComponent(hmac.digest('base64'));

    // Construct authorization string
    var token = "SharedAccessSignature sr=" + resourceUri + "&sig="
    + base64UriEncoded + "&se=" + expires;
    if (policyName) token += "&skn="+policyName;
    return token;
};
```

### <a name="python"></a>[Python](#tab/python)

Como comparación, el código de Python equivalente para generar un token de seguridad es:

```python
from base64 import b64encode, b64decode
from hashlib import sha256
from time import time
from urllib import parse
from hmac import HMAC

def generate_sas_token(uri, key, policy_name, expiry=3600):
    ttl = time() + expiry
    sign_key = "%s\n%d" % ((parse.quote_plus(uri)), int(ttl))
    print(sign_key)
    signature = b64encode(HMAC(b64decode(key), sign_key.encode('utf-8'), sha256).digest())

    rawtoken = {
        'sr' :  uri,
        'sig': signature,
        'se' : str(int(ttl))
    }

    if policy_name is not None:
        rawtoken['skn'] = policy_name

    return 'SharedAccessSignature ' + parse.urlencode(rawtoken)
```
### <a name="c"></a>[C#](#tab/csharp)

La funcionalidad en C# para generar un token de seguridad es:

```csharp
using System;
using System.Globalization;
using System.Net;
using System.Net.Http;
using System.Security.Cryptography;
using System.Text;

public static string generateSasToken(string resourceUri, string key, string policyName, int expiryInSeconds = 3600)
{
    TimeSpan fromEpochStart = DateTime.UtcNow - new DateTime(1970, 1, 1);
    string expiry = Convert.ToString((int)fromEpochStart.TotalSeconds + expiryInSeconds);

    string stringToSign = WebUtility.UrlEncode(resourceUri) + "\n" + expiry;

    HMACSHA256 hmac = new HMACSHA256(Convert.FromBase64String(key));
    string signature = Convert.ToBase64String(hmac.ComputeHash(Encoding.UTF8.GetBytes(stringToSign)));

    string token = String.Format(CultureInfo.InvariantCulture, "SharedAccessSignature sr={0}&sig={1}&se={2}", WebUtility.UrlEncode(resourceUri), WebUtility.UrlEncode(signature), expiry);

    if (!String.IsNullOrEmpty(policyName))
    {
        token += "&skn=" + policyName;
    }

    return token;
}
```
### <a name="java"></a>[Java](#tab/java)

Para Java:
```java
    public static String generateSasToken(String resourceUri, String key) throws Exception {
        // Token will expire in one hour
        var expiry = Instant.now().getEpochSecond() + 3600;

        String stringToSign = URLEncoder.encode(resourceUri, StandardCharsets.UTF_8) + "\n" + expiry;
        byte[] decodedKey = Base64.getDecoder().decode(key);

        Mac sha256HMAC = Mac.getInstance("HmacSHA256");
        SecretKeySpec secretKey = new SecretKeySpec(decodedKey, "HmacSHA256");
        sha256HMAC.init(secretKey);
        Base64.Encoder encoder = Base64.getEncoder();

        String signature = new String(encoder.encode(
            sha256HMAC.doFinal(stringToSign.getBytes(StandardCharsets.UTF_8))), StandardCharsets.UTF_8);

        String token = "SharedAccessSignature sr=" + URLEncoder.encode(resourceUri, StandardCharsets.UTF_8)
                + "&sig=" + URLEncoder.encode(signature, StandardCharsets.UTF_8.name()) + "&se=" + expiry;
            
        return token;
    }
```
---

### <a name="protocol-specifics"></a>Detalles específicos de protocolo

Cada protocolo admitido, como MQTT, AMQP y HTTPS, transporta tokens de diferentes maneras.

Al utilizar MQTT, el paquete CONNECT tiene deviceId como ClientId, `{iothubhostname}/{deviceId}` en el campo Nombre de usuario y un token SAS en el campo Contraseña. `{iothubhostname}` debe ser el CName completo de la instancia de IoT Hub (por ejemplo, contoso.azure-devices.net).

Al usar [AMQP](https://www.amqp.org/), IoT Hub admite [SASL PLAIN](https://tools.ietf.org/html/rfc4616) y [AMQP Claims-Based-Security](https://www.oasis-open.org/committees/download.php/50506/amqp-cbs-v1%200-wd02%202013-08-12.doc).

En el caso de la seguridad basada en notificaciones AMQP, la norma especifica cómo transmitir estos tokens.

Para SASL PLAIN, el **nombre de usuario** puede ser:

* `{policyName}@sas.root.{iothubName}` si usa tokens de nivel de centro de IoT.
* `{deviceId}@sas.{iothubname}` si usa tokens de ámbito por el dispositivo.

En ambos casos, el campo de contraseña contiene el token, como se describe en el artículo [Tokens de seguridad de IoT Hub](iot-hub-dev-guide-sas.md#security-tokens).

HTTPS implementa la autenticación mediante la inclusión de un token válido en el encabezado de solicitud **Authorization**.

Por ejemplo, nombre de usuario (DeviceId distingue entre mayúsculas y minúsculas): `iothubname.azure-devices.net/DeviceId`

Contraseña (puede generar un token de SAS con el comando de extensión de la CLI [az iot hub generate-sas-token](/cli/azure/iot/hub#az_iot_hub_generate_sas_token) o [Azure IoT Tools para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)):

`SharedAccessSignature sr=iothubname.azure-devices.net%2fdevices%2fDeviceId&sig=kPszxZZZZZZZZZZZZZZZZZAhLT%2bV7o%3d&se=1487709501`

> [!NOTE]
> Los [SDK de Azure IoT](iot-hub-devguide-sdks.md) generan tokens automáticamente cuando se conectan al servicio. En algunos casos, los SDK IoT de Azure no admiten todos los protocolos o todos los métodos de autenticación.

### <a name="special-considerations-for-sasl-plain"></a>Consideraciones especiales para SASL PLAIN

Cuando se usa SASL PLAIN con AMQP, un cliente que se conecta a una instancia de IoT Hub puede usar un token único para cada conexión TCP. Cuando el token expira, la conexión TCP se desconecta del servicio y desencadena una reconexión. Este comportamiento, aunque no resulta problemático para una aplicación de back-end, es perjudicial para una aplicación de dispositivo por las razones siguientes:

* Las puertas de enlace normalmente se conectan en nombre de muchos dispositivos. Cuando se usa SASL PLAIN, tienen que crear una conexión TCP distintiva para cada dispositivo que se conecta a un centro de IoT. Este escenario aumenta considerablemente el consumo de energía y de recursos de red, y aumenta la latencia de cada conexión de dispositivo.

* Los dispositivos con recursos restringidos se ven afectados negativamente por el aumento del uso de recursos para volver a conectarse después de cada expiración del token.

## <a name="use-security-tokens-from-service-components"></a>Uso de tokens de seguridad de los componentes de servicio

Los componentes de servicio solo pueden generar tokens de seguridad mediante directivas de acceso compartido que conceden los permisos apropiados, tal y como se explicó anteriormente.

Estas son las funciones de servicio expuestas en los puntos de conexión:

| Punto de conexión | Funcionalidad |
| --- | --- |
| `{iot hub host name}/devices` |Creación, actualización, recuperación y eliminación de las identidades del dispositivo. |
| `{iot hub host name}/messages/events` |Recepción de mensajes de dispositivo a nube. |
| `{iot hub host name}/servicebound/feedback` |Recepción de comentarios para los mensajes de nube a dispositivo. |
| `{iot hub host name}/devicebound` |Envío de mensajes de nube a dispositivo. |

Por ejemplo, un servicio que genera el uso de la directiva de acceso compartido creada previamente denominada **registryRead** crearía un token con los parámetros siguientes:

* URI de recurso: `{IoT hub name}.azure-devices.net/devices`,
* clave de firma: una de las claves de la directiva `registryRead` ,
* nombre de la directiva: `registryRead`,
* cualquier fecha de expiración.

```javascript
var endpoint ="myhub.azure-devices.net/devices";
var policyName = 'registryRead';
var policyKey = '...';

var token = generateSasToken(endpoint, policyKey, policyName, 60);
```

El resultado, que concedería acceso para leer todas las identidades del dispositivo, sería:

`SharedAccessSignature sr=myhub.azure-devices.net%2fdevices&sig=JdyscqTpXdEJs49elIUCcohw2DlFDR3zfH5KqGJo4r4%3D&se=1456973447&skn=registryRead`

## <a name="authenticating-a-device-to-iot-hub"></a>Autenticación de un dispositivo en IoT Hub

### <a name="supported-x509-certificates"></a>Certificados X.509 compatibles

Puede usar cualquier certificado X.509 para autenticar un dispositivo en IoT Hub, cargando ya sea una huella digital del certificado o una entidad de certificación (CA) a Azure IoT Azure. Para más información consulte [Autenticación de dispositivos mediante certificados de entidades de certificación X.509](iot-hub-x509ca-overview.md). Para más información sobre cómo cargar y comprobar una entidad de certificación con su centro de IoT, consulte [Configuración de la seguridad de X.509 en Azure IoT Hub](iot-hub-security-x509-get-started.md).

### <a name="use-sas-tokens-as-a-device"></a>Uso de tokens de SAS como dispositivo

Existen dos maneras de obtener permisos **DeviceConnect** con IoT Hub mediante tokens de seguridad: con una [clave de dispositivo simétrica del registro de identidad](#use-a-symmetric-key-in-the-identity-registry) o una [clave de acceso compartido](#use-a-shared-access-policy-to-access-on-behalf-of-a-device).

Recuerde que puede acceder a toda la funcionalidad desde los dispositivos expuestos por diseño en los puntos de conexión con el prefijo `/devices/{deviceId}`.

Los puntos de conexión del dispositivo son (con independencia del protocolo):

| Punto de conexión | Funcionalidad |
| --- | --- |
| `{iot hub host name}/devices/{deviceId}/messages/events` |Envío de mensajes de dispositivo a nube. |
| `{iot hub host name}/devices/{deviceId}/messages/devicebound` |Recepción de mensajes de nube a dispositivo |

#### <a name="use-a-symmetric-key-in-the-identity-registry"></a>Uso de una clave simétrica en el registro de identidades

Cuando se utiliza la clave simétrica de la identidad de un dispositivo para generar un token, el elemento policyName (`skn`) del token se omite.

Por ejemplo, un token creado para tener acceso a toda la funcionalidad de dispositivo debe tener los siguientes parámetros:

* URI de recurso: `{IoT hub name}.azure-devices.net/devices/{device id}`,
* clave de firma: cualquier clave simétrica de la identidad `{device id}` ,
* ningún nombre de directiva,
* cualquier fecha de expiración.

Un ejemplo de cómo utilizar la función anterior de Node.js sería:

```javascript
var endpoint ="myhub.azure-devices.net/devices/device1";
var deviceKey ="...";

var token = generateSasToken(endpoint, deviceKey, null, 60);
```

El resultado, que concede acceso a todas las funcionalidades del dispositivo1, sería:

`SharedAccessSignature sr=myhub.azure-devices.net%2fdevices%2fdevice1&sig=13y8ejUk2z7PLmvtwR5RqlGBOVwiq7rQR3WZ5xZX3N4%3D&se=1456971697`

> [!NOTE]
> Se puede generar un token de SAS con el comando de extensión de la CLI [az iot hub generate-sas-token](/cli/azure/iot/hub#az_iot_hub_generate_sas_token) o con [Azure IoT Tools para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).

#### <a name="use-a-shared-access-policy-to-access-on-behalf-of-a-device"></a>Use una directiva de acceso compartida para acceder en nombre de un dispositivo

Cuando cree un token a partir de una directiva de acceso compartido, establezca el campo `skn` en el nombre de la directiva. Esta directiva debe conceder el permiso **DeviceConnect**.

Los dos escenarios principales para utilizar directivas de acceso compartido para tener acceso a la funcionalidad del dispositivo son:

* [puertas de enlace del protocolo en la nube](iot-hub-devguide-endpoints.md),
* [servicios de token](iot-hub-dev-guide-sas.md#create-a-token-service-to-integrate-existing-devices) usados para implementar esquemas de autenticación personalizados.

Dado que la directiva de acceso compartido potencialmente puede conceder acceso para conectarse a cualquier dispositivo, es importante usar el identificador URI de recurso correcto al crear tokens de seguridad. Esto es especialmente importante para los servicios de token, que tienen que definir el ámbito de token para un dispositivo específico mediante el identificador URI del recurso. Este punto es menos relevante para puertas de enlace de protocolo, puesto que ya están mediando el tráfico para todos los dispositivos.

Por ejemplo, un servicio de token que usa acceso compartido creado previamente denominado **dispositivo** crearía un token con los parámetros siguientes:

* URI de recurso: `{IoT hub name}.azure-devices.net/devices/{device id}`,
* clave de firma: una de las claves de la directiva `device` ,
* nombre de la directiva: `device`,
* cualquier fecha de expiración.

Un ejemplo de cómo utilizar la función anterior de Node.js sería:

```javascript
var endpoint ="myhub.azure-devices.net/devices/device1";
var policyName = 'device';
var policyKey = '...';

var token = generateSasToken(endpoint, policyKey, policyName, 60);
```

El resultado, que concede acceso a todas las funcionalidades del dispositivo1, sería:

`SharedAccessSignature sr=myhub.azure-devices.net%2fdevices%2fdevice1&sig=13y8ejUk2z7PLmvtwR5RqlGBOVwiq7rQR3WZ5xZX3N4%3D&se=1456971697&skn=device`

Una puerta de enlace de protocolo podría utilizar el mismo token para todos los dispositivos que simplemente establecen el URI de recurso en `myhub.azure-devices.net/devices`.

## <a name="create-a-token-service-to-integrate-existing-devices"></a>Cree un servicio de token para integrar dispositivos existentes

Puede usar el [registro de identidades](iot-hub-devguide-identity-registry.md) de IoT Hub para configurar las credenciales de seguridad de cada dispositivo o módulo y el control de acceso mediante [tokens](iot-hub-dev-guide-sas.md#security-tokens). Si una solución IoT ya tiene un registro de identidad personalizado y un esquema de autenticación, considere la posibilidad de crear un *servicio de token* para integrar esta infraestructura con IoT Hub. De este modo, puede usar otras características de IoT en la solución.

Un servicio de token es un servicio en la nube personalizado. Usa una *directiva de acceso compartido* de IoT Hub con los permisos **DeviceConnect** o **ModuleConnect** para crear tokens *centrados en el dispositivo* o *centrados en el módulo*. Estos tokens permiten que un dispositivo o módulo se conecten a su centro de IoT.

![Pasos del modelo de servicio de tokens](./media/iot-hub-devguide-security/tokenservice.png)

Estos son los pasos principales del modelo de servicio de tokens:

1. Cree una directiva de acceso compartido de IoT Hub con los permisos **DeviceConnect** o **ModuleConnect** para IoT Hub. Puede crear esta directiva en [Azure Portal](https://portal.azure.com) o mediante programación. El servicio de tokens usará esta directiva para firmar los tokens que crea.

2. Cuando un dispositivo o un módulo necesitan acceder al centro de IoT, solicitan un token firmado al servicio de tokens. El dispositivo se puede autenticar mediante el esquema de autenticación o registro de identidades personalizado para determinar la identidad del dispositivo o módulo que el servicio de token usa para crear el token.

3. El servicio de token devuelve un token. El token se crea con `/devices/{deviceId}` o `/devices/{deviceId}/module/{moduleId}` como `resourceURI`, con `deviceId` como dispositivo autenticado o con `moduleId` como módulo autenticado. El servicio de token usa la directiva de acceso compartido para construir el token.

4. El dispositivo o el módulo usan el token directamente con el centro de IoT.

> [!NOTE]
> Puede usar la clase .NET [SharedAccessSignatureBuilder](/dotnet/api/microsoft.azure.devices.common.security.sharedaccesssignaturebuilder) o la clase de Java [IotHubServiceSasToken](/java/api/com.microsoft.azure.sdk.iot.service.auth.iothubservicesastoken) para crear un token en el servicio correspondiente.

El servicio de token puede establecer la caducidad de los tokens como desee. Cuando expira el token, el centro de IoT interrumpe la conexión del dispositivo o del módulo. A continuación, el dispositivo o el módulo deben solicitar un nuevo token al servicio de token. Un tiempo de expiración corto aumenta la carga tanto en el dispositivo o el módulo como en el servicio de token.

Para que un dispositivo o un módulo se conecten a su concentrador, tiene que agregarlo al registro de identidades de IoT Hub, aunque ya use un token y no una clave para conectarse. Por tanto, puede continuar usando el control de acceso por dispositivo o por módulo habilitando o deshabilitando las identidades del dispositivo o módulo en el [registro de identidades](iot-hub-devguide-identity-registry.md). Esto mitiga los riesgos de usar tokens con tiempos de expiración largos.

### <a name="comparison-with-a-custom-gateway"></a>Comparación con una puerta de enlace personalizada

El modelo de servicio de token es el método recomendado para implementar un esquema de autenticación o registro de identidades personalizado en Azure IoT Hub. Este modelo se recomienda porque IoT Hub sigue controlando la mayoría del tráfico de la solución. Hay casos, sin embargo, en los que el esquema de autenticación personalizado está tan imbricado con el protocolo que se necesita una *puerta de enlace personalizada* que procese todo el tráfico. Un ejemplo de escenario de este tipo es la [seguridad de la capa de transporte (TLS) y las claves previamente compartidas (PSK)](https://tools.ietf.org/html/rfc4279). Para más información, consulte el artículo [Puerta de enlace de protocolos](iot-hub-protocol-gateway.md).


## <a name="additional-reference-material"></a>Material de referencia adicional

Otros temas de referencia en la guía del desarrollador de IoT Hub son los siguientes:

* En [Puntos de conexión de IoT Hub](iot-hub-devguide-endpoints.md) se describen los diferentes puntos de conexión que expone cada centro de IoT Hub para las operaciones en tiempo de ejecución y de administración.

* En [Cuotas y limitación](iot-hub-devguide-quotas-throttling.md) se describen las cuotas y el comportamiento de limitación que se aplican al servicio IoT Hub.

* En [SDK de dispositivo y servicio IoT de Azure](iot-hub-devguide-sdks.md) se muestran los diversos SDK de lenguaje que puede usar para desarrollar aplicaciones para dispositivo y de servicio que interactúen con IoT Hub.

* En [Lenguaje de consulta de IoT Hub](iot-hub-devguide-query-language.md) se describe el lenguaje de consulta que se puede usar para recuperar información de IoT Hub sobre los trabajos y dispositivos gemelos.

* En [Compatibilidad con MQTT de IoT Hub](iot-hub-mqtt-support.md) se proporciona más información sobre la compatibilidad de IoT Hub con el protocolo MQTT.

* En [RFC 5246: el protocolo de seguridad de la capa de transporte (TLS) versión 1.2](https://tools.ietf.org/html/rfc5246/), se proporciona más información sobre la autenticación TLS.

* Para más información sobre la autenticación mediante una entidad de certificación, consulte [Autenticación de dispositivos mediante certificados de entidades de certificación X.509](iot-hub-x509ca-overview.md)

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a controlar el acceso a IoT Hub, puede interesarle los siguientes temas de la Guía del desarrollador de IoT Hub:

* [Uso de dispositivos gemelos para sincronizar el estado y las configuraciones](iot-hub-devguide-device-twins.md)
* [Invocación de un método directo en un dispositivo](iot-hub-devguide-direct-methods.md)
* [Programación de trabajos en varios dispositivos](iot-hub-devguide-jobs.md)

Si desea probar algunos de los conceptos descritos en este artículo, vea los siguientes tutoriales de IoT Hub:

* [Introducción a Azure IoT Hub](quickstart-send-telemetry-node.md)
* [Cómo enviar mensajes de la nube a un dispositivo con IoT Hub](iot-hub-csharp-csharp-c2d.md)
* [Procesamiento de mensajes del dispositivo a la nube de IoT Hub](tutorial-routing.md)