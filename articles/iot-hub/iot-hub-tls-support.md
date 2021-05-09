---
title: Compatibilidad con TLS en Azure IoT Hub
description: Obtenga información sobre cómo usar conexiones TLS seguras para dispositivos y servicios que se comunican con IoT Hub.
services: iot-hub
author: jlian
ms.service: iot-fundamentals
ms.topic: conceptual
ms.date: 03/31/2021
ms.author: jlian
ms.openlocfilehash: 3fed568d1e5b85fcfaf5c88eb736e876e9748088
ms.sourcegitcommit: bd1a4e4df613ff24e954eb3876aebff533b317ae
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2021
ms.locfileid: "107930720"
---
# <a name="transport-layer-security-tls-support-in-iot-hub"></a>Compatibilidad de la Seguridad de la capa de transporte (TLS) con IoT Hub

IoT Hub usa Seguridad de la capa de transporte (TLS) para proteger las conexiones de los dispositivos y servicios de IoT. Actualmente se admiten tres versiones del protocolo TLS: 1.0, 1.1 y 1.2.

TLS 1.0 y 1.1 se consideran versiones heredadas y está previsto que dejen de usarse próximamente. Para obtener más información, vea [Desuso de TLS 1.0 y 1.1 en IoT Hub](iot-hub-tls-deprecating-1-0-and-1-1.md). Para evitar problemas en el futuro, use TLS 1.2 como la única versión de TLS al conectarse a IoT Hub.

## <a name="iot-hubs-server-tls-certificate"></a>Certificado TLS para el servidor de IoT Hub

Durante un protocolo de enlace TLS, IoT Hub presenta certificados de servidor con claves RSA para la conexión de clientes. Su raíz es la entidad de certificación Baltimore Cybertrust Root. Recientemente, implementamos un cambio en el certificado de servidor TLS para que ahora lo emitan las nuevas entidades de certificación intermedias (ICA). Para más información, vea [Actualización de certificados TLS de IoT Hub](https://azure.microsoft.com/updates/iot-hub-tls-certificate-update/).

### <a name="4kb-size-limit-on-renewal"></a>Límite de tamaño de 4 KB en la renovación

Durante la renovación de los certificados del lado servidor de IoT Hub, se realiza una comprobación en el lado servicio para evitar que `Server Hello` supere el tamaño de 4 KB. Un cliente debe tener al menos 4 KB de RAM establecidos para el búfer de longitud máxima de contenido de TLS entrante, de modo que los dispositivos existentes que se establecen para el límite de 4 KB sigan funcionando como antes, después de la renovación de certificados. En el caso de los dispositivos restringidos, IoT Hub admite la [negociación de longitud máxima de fragmentos de TLS en la versión preliminar](#tls-maximum-fragment-length-negotiation-preview). 

### <a name="elliptic-curve-cryptography-ecc-server-tls-certificate-preview"></a>Certificado TLS del servidor de criptografía de curva elíptica (ECC) (versión preliminar)

El certificado TLS del servidor de ECC de IoT Hub está disponible para la versión preliminar pública. Aunque ofrece una seguridad similar a los certificados RSA, la validación de certificados ECC (con conjuntos de cifrado solo mediante ECC) usa hasta un 40 % menos de proceso, memoria y ancho de banda. Estos ahorros son importantes para los dispositivos IoT debido a sus perfiles y memoria más pequeños, así como para admitir casos de uso en entornos con un ancho de banda de red limitado. La raíz del certificado de servidor ECC es DigiCert Global Root G3.

Para obtener una vista previa del certificado de servidor de ECC de IoT Hub:

1. [Cree un centro de IoT con el modo de vista previa activado](iot-hub-preview-mode.md).
1. [Configure el cliente](#tls-configuration-for-sdk-and-iot-edge) para incluir *solo* conjuntos de cifrado ECDSA y *excluir* cualquier RSA. Estos son los conjuntos de cifrado compatibles con la versión preliminar pública del certificado ECC:
    - `TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256`
    - `TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384`
    - `TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256`
    - `TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384`
1. Conecte el cliente a la vista previa del centro de IoT.

## <a name="tls-12-enforcement-available-in-select-regions"></a>Aplicación TLS 1.2 disponible en regiones seleccionadas

Para mayor seguridad, configure IoT Hub para que *solo* permita conexiones de clientes que usen la versión 1.2 de TLS y para que exija el uso de [conjuntos de cifrado](#cipher-suites). Esta característica solo se admite en estas regiones:

* Este de EE. UU.
* Centro-sur de EE. UU.
* Oeste de EE. UU. 2
* US Gov: Arizona
* US Gov Virginia (la compatibilidad con TLS 1.0/1.1 no está disponible en esta región: la aplicación TLS 1.2 debe estar habilitada o se producirá un error en la creación de un centro de IoT)

Para habilitar la aplicación de TLS 1.2, siga los pasos descritos en [Creación de una instancia de IoT Hub mediante Azure Portal](iot-hub-create-through-portal.md), con la siguiente variación

- Elija una **Región** de entre la lista anterior.
- En **Administración -> Avanzado -> Seguridad de capa de transporte (TLS) -> Versión de TLS mínima**, seleccione **1.2**. Esta configuración solo aparece para los centros de IoT creados en la región admitida.

    :::image type="content" source="media/iot-hub-tls-12-enforcement.png" alt-text="Captura de pantalla que muestra cómo activar la aplicación de TLS 1.2 durante la creación de un centro de IoT":::

Para usar una plantilla de ARM para la creación, aprovisione una nueva instancia de IoT Hub en cualquiera de las regiones admitidas y establezca la propiedad `minTlsVersion` en `1.2` en la especificación de recursos:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [
        {
            "type": "Microsoft.Devices/IotHubs",
            "apiVersion": "2020-01-01",
            "name": "<provide-a-valid-resource-name>",
            "location": "<any-of-supported-regions-below>",
            "properties": {
                "minTlsVersion": "1.2"
            },
            "sku": {
                "name": "<your-hubs-SKU-name>",
                "tier": "<your-hubs-SKU-tier>",
                "capacity": 1
            }
        }
    ]
}
```

El recurso de IoT Hub creado con esta configuración rechazará los clientes de servicios y dispositivos que intenten conectarse mediante las versiones 1.0 y 1.1 de TLS. Del mismo modo, se rechazará el protocolo de enlace TLS si el mensaje `ClientHello` no incluye ninguno de los [cifrados recomendados](#cipher-suites).

> [!NOTE]
> La propiedad `minTlsVersion` es de solo lectura y no se puede cambiar una vez creado el recurso de IoT Hub. Por lo tanto, es esencial que compruebe y valide de antemano que *todos* los dispositivos y servicios de IoT son compatibles con TLS 1.2 y con los [cifrados recomendados](#cipher-suites).
> 
> En caso de conmutaciones por error, la propiedad `minTlsVersion` de su IoT Hub seguirá siendo efectiva en la región con emparejamiento geográfico.

## <a name="cipher-suites"></a>Conjuntos de cifrado

Los recursos de IoT Hub que están configurados para aceptar solo TLS 1.2 también exigirán el uso de estos conjuntos de cifrado recomendados:

* `TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256`
* `TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384`
* `TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256`
* `TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384`

En el caso de los recursos de IoT Hub que no están configurados para exigir TLS 1.2, este protocolo seguirá funcionando con los siguientes conjuntos de cifrado:

* `TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256`
* `TLS_DHE_RSA_WITH_AES_256_GCM_SHA384`
* `TLS_DHE_RSA_WITH_AES_128_GCM_SHA256`
* `TLS_RSA_WITH_AES_256_GCM_SHA384`
* `TLS_RSA_WITH_AES_128_GCM_SHA256`
* `TLS_RSA_WITH_AES_256_CBC_SHA256`
* `TLS_RSA_WITH_AES_128_CBC_SHA256`
* `TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA`
* `TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA`
* `TLS_RSA_WITH_AES_256_CBC_SHA`
* `TLS_RSA_WITH_AES_128_CBC_SHA`
* `TLS_RSA_WITH_3DES_EDE_CBC_SHA`

Un cliente puede sugerir una lista de conjuntos de cifrado mayores para usar durante el mensaje `ClientHello`. Sin embargo, es posible que no se admitan algunos de los recursos de IoT Hub (por ejemplo, `ECDHE-ECDSA-AES256-GCM-SHA384`). En este caso, IoT Hub intentará seguir las preferencias del cliente, pero finalmente negociará el conjunto de cifrado con `ServerHello`.

## <a name="tls-configuration-for-sdk-and-iot-edge"></a>Configuración de TLS para SDK y IoT Edge

Use los vínculos enumerados abajo para configurar TLS 1.2 y los cifrados permitidos en los SDK de cliente de IoT Hub.

| Idioma | Versiones compatibles con TLS 1.2 | Documentación |
|----------|------------------------------------|---------------|
| C        | Etiqueta 2019-12-11 o más reciente            | [Vínculo](https://aka.ms/Tls_C_SDK_IoT) |
| Python   | Versión 2.0.0 o más reciente             | [Vínculo](https://aka.ms/Tls_Python_SDK_IoT) |
| C#       | Versión 1.21.4 o más reciente            | [Vínculo](https://aka.ms/Tls_CSharp_SDK_IoT) |
| Java     | Versión 1.19.0 o más reciente            | [Vínculo](https://aka.ms/Tls_Java_SDK_IoT) |
| NodeJS   | Versión 1.12.2 o más reciente            | [Vínculo](https://aka.ms/Tls_Node_SDK_IoT) |

Los dispositivos de IoT Edge se pueden configurar para que usen TLS 1.2 al comunicarse con IoT Hub. Para más información sobre cómo hacerlo, vea la [página de documentación de IoT Edge](https://github.com/Azure/iotedge/blob/master/edge-modules/edgehub-proxy/README.md).

## <a name="device-authentication"></a>Autenticación de dispositivos

Después de un protocolo de enlace de TLS correcto, IoT Hub puede autenticar un dispositivo mediante una clave simétrica o un certificado X.509. En el caso de la autenticación basada en certificados, puede tratarse de cualquier certificado X.509, incluido ECC. IoT Hub valida el certificado con la huella digital o la entidad de certificación (CA) que proporcione. Para obtener más información, consulte [Certificados X.509 compatibles](iot-hub-devguide-security.md#supported-x509-certificates).

## <a name="tls-maximum-fragment-length-negotiation-preview"></a>Negociación de longitud de fragmento máxima de TLS (versión preliminar)

IoT Hub también admite la negociación de longitud de fragmento máxima de TLS, que a veces se conoce como negociación de tamaño del marco de TLS. Esta característica está en versión preliminar pública. 

Use esta característica para especificar la longitud máxima del fragmento de texto sin formato en un valor menor que el predeterminado de 2^14 bytes. Una vez realizada la negociación, IoT Hub y el cliente inician la fragmentación de los mensajes para asegurarse de que todos los fragmentos son menores que la longitud negociada. Este comportamiento es útil para los dispositivos que tienen restricciones de proceso o memoria. Para más información, vea la [especificación de la extensión TLS oficial](https://tools.ietf.org/html/rfc6066#section-4).

Todavía no está disponible la compatibilidad del SDK oficial con esta característica en versión preliminar pública. Inicio

1. [Cree un centro de IoT con el modo de vista previa activado](iot-hub-preview-mode.md).
1. Al usar OpenSSL, llame a [SSL_CTX_set_tlsext_max_fragment_length](https://manpages.debian.org/testing/libssl-doc/SSL_CTX_set_max_send_fragment.3ssl.en.html) para especificar el tamaño del fragmento.
1. Conecte el cliente a la vista previa del centro de IoT.

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre el control de acceso y la seguridad de IoT Hub, vea [Control del acceso a IoT Hub](iot-hub-devguide-security.md).
- Para más información sobre el uso del certificado X.509 para la autenticación de dispositivos, vea [Autenticación de dispositivos mediante certificados de entidades de certificación X.509](iot-hub-x509ca-overview.md).
