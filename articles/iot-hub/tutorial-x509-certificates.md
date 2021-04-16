---
title: 'Tutorial: Descripción de los certificados de clave pública X.509 para Azure IoT Hub | Microsoft Docs'
description: 'Tutorial: Descripción de los certificados de clave pública X.509 para Azure IoT Hub'
author: v-gpettibone
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: tutorial
ms.date: 02/26/2021
ms.author: robinsh
ms.custom:
- mvc
- 'Role: Cloud Development'
- 'Role: Data Analytics'
ms.openlocfilehash: 5503f9ad57180146c25a01c133a27b34e643496c
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107378353"
---
# <a name="tutorial-understanding-x509-public-key-certificates"></a>Tutorial: Descripción de los certificados de clave pública X.509

Los certificados X.509 son documentos digitales que representan a un usuario, equipo, servicio o dispositivo. Los emite una entidad de certificación, una entidad de certificación subordinada o una entidad de registro, y contienen la clave pública del firmante del certificado. No contienen la clave privada del firmante, la cual se debe almacenar de forma segura. Los certificados de clave pública están documentados en [RFC 5280](https://tools.ietf.org/html/rfc5280). Están firmados digitalmente y, en general, contienen la siguiente información:

* Información sobre el firmante del certificado
* La clave pública que corresponde a la clave privada del firmante
* Información acerca de la entidad de certificación emisora
* Algoritmos de cifrado y de firma digital admitidos
* Información para determinar la revocación y el estado de validez del certificado

## <a name="certificate-fields"></a>Campos de un certificado

Históricamente, ha habido tres versiones de certificado. Cada versión agrega campos a la versión anterior. La versión 3 es la actual y contiene los campos de las versiones 1 y 2, además de los campos de la versión 3. La versión 1 define los campos siguientes:

* **Versión**: un valor (1, 2 o 3) que identifica el número de versión del certificado.
* **Número de serie**: número único para cada certificado emitido por una entidad de certificación.
* **Algoritmo de la firma de la entidad de certificación**: nombre del algoritmo que la entidad de certificación usa para firmar el contenido del certificado.
* **Nombre del emisor**: nombre distintivo (DN) de la entidad de certificación que emite el certificado.
* **Período de validez**: período de tiempo durante el que el certificado se considera válido.
* **Nombre del firmante**: nombre de la entidad representada por el certificado.
* **Información de la clave pública del firmante**: clave pública propiedad del firmante del certificado.

La versión 2 agrega los campos siguientes que contienen información sobre el emisor del certificado. No obstante, estos campos se usan con poca frecuencia.

* **Identificador único del emisor**: un identificador único para la entidad de certificación emisora tal como la define la entidad de certificación.
* **Identificador único del firmante**: un identificador único para el firmante del certificado tal y como lo define la entidad de certificación emisora

En los certificados de la versión 3 se agregaron las siguientes extensiones:

* **Identificador de la clave de la entidad**: este puede ser uno de estos dos valores:
  * El firmante de la entidad de certificación y el número de serie del certificado de la entidad que emitió este certificado
  * Un hash de la clave pública de la entidad de certificación que emitió este certificado.
* **Identificador de clave del firmante**: hash de la clave pública del certificado actual
* **Uso de la clave**: define el servicio para el que se puede utilizar un certificado. Este puede ser uno o varios de los valores siguientes:
  * **Firma digital**
  * **No rechazo**
  * **Cifrado de clave**
  * **Cifrado de datos**
  * **Acuerdo de claves**
  * **Firma de certificados de claves**
  * **Firma de CRL**
  * **Solo cifrar**
  * **Solo descifrar**
* **Período de uso de la clave privada**: período de validez de la porción de la clave privada de un par de claves.
* **Directivas de certificado**: directivas que se usan para validar el firmante del certificado.
* **Asignaciones de directivas**: asigna una directiva de una organización a la directiva de otra.
* **Nombre alternativo del firmante**: lista de nombres alternativos del firmante
* **Nombre alternativo del emisor**: lista de nombres alternativos de la entidad de certificación emisora.
* **Atributo de directorio del firmante**: atributos de un directorio X.500 o LDAP
* **Restricciones básicas**: permite que el certificado designe si se emite para una entidad de certificación o para un usuario, equipo, dispositivo o servicio. Esta extensión también incluye una restricción de longitud de la ruta de acceso que limita el número de entidades de certificación subordinadas que pueden existir.
* **Restricciones de nombres**: designa qué espacios de nombres se permiten en un certificado emitido por una entidad de certificación.
* **Restricciones de directivas**: se pueden usar para prohibir las asignaciones de directivas entre las entidades de certificación.
* **Uso mejorado de clave**: indica cómo se puede usar la clave pública de un certificado más allá de los propósitos identificados en la extensión **Uso de la clave**.
* **Puntos de distribución de CRL**: contiene una o más direcciones URL donde se publica la lista de revocación de certificados (CRL) básica.
* **Deshabilitación de anyPolicy**: deshabilita el uso del OID **Todas las directivas de emisión** (2.5.29.32.0) en los certificados de la entidad de certificación subordinada.
* **CRL más actualizada**: contiene una o varias direcciones URL en las que se publica la diferencia de CRL de la entidad de certificación emisora.
* **Acceso a la información de entidad emisora**: contiene una o varias direcciones URL en las que se publica el certificado de la entidad de certificación emisora.
* **Acceso a la información del firmante**: contiene información sobre cómo recuperar detalles adicionales de un firmante de certificado.

## <a name="certificate-formats"></a>Formatos de certificado

Los certificados pueden guardarse en diversos formatos. La autenticación de Azure IoT Hub suele usar los formatos PEM y PFX.

### <a name="binary-certificate"></a>Certificado binario

Contiene un certificado binario sin formato que usa la codificación DER ASN.1.

### <a name="ascii-pem-format"></a>Formato PEM de ASCII

Un certificado PEM (extensión .pem) contiene un certificado codificado en Base64 que empieza por -----BEGIN CERTIFICATE----- y termina con -----END CERTIFICATE-----. El formato PEM es muy común y es necesario para IoT Hub cuando se cargan determinados certificados.

### <a name="ascii-pem-key"></a>Clave ASCII (PEM)

Contiene una clave DER codificada en Base64 con metadatos posiblemente adicionales sobre el algoritmo utilizado para la protección de contraseñas.

### <a name="pkcs7-certificate"></a>Certificado PKCS#7

Formato diseñado para el transporte de datos firmados o cifrados. Está definido por [RFC 2315](https://tools.ietf.org/html/rfc2315). Puede incluir toda la cadena de certificados.

### <a name="pkcs8-key"></a>Clave PKCS#8

El formato de un almacén de claves privadas definido por [RFC 5208](https://tools.ietf.org/html/rfc5208).

### <a name="pkcs12-key-and-certificate"></a>Clave y certificado PKCS#12

Un formato complejo que puede almacenar y proteger una clave y toda la cadena de certificados. Se usa normalmente con una extensión .pfx. PKCS#12 es sinónimo del formato PFX.

## <a name="for-more-information"></a>Para obtener más información

Para obtener más información, vea los temas siguientes:

* [Guía para novatos sobre la jerga del certificado X.509](https://techcommunity.microsoft.com/t5/internet-of-things/the-layman-s-guide-to-x-509-certificate-jargon/ba-p/2203540)
* [Explicación de los conceptos de certificados de entidad de certificación X.509 en el sector de IoT](https://docs.microsoft.com/azure/iot-hub/iot-hub-x509ca-concept)

## <a name="next-steps"></a>Pasos siguientes

Si desea generar certificados de prueba que pueda usar para autenticar dispositivos en IoT Hub, consulte los temas siguientes:

* [Uso de scripts proporcionados por Microsoft para crear certificados de prueba](tutorial-x509-scripts.md)
* [Uso de OpenSSL para crear certificados de prueba](tutorial-x509-openssl.md)
* [Uso de OpenSSL para crear certificados de prueba autofirmados](tutorial-x509-self-sign.md)

Si tiene un certificado de una entidad de certificación o un certificado de entidad de certificación subordinada, y desea cargarlo en IoT Hub y demostrar que es de su propiedad, consulte [Demostración de la posesión de un certificado de entidad de certificación](tutorial-x509-prove-possession.md).
