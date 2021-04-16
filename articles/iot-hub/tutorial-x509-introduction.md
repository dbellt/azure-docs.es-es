---
title: 'Tutorial: Descripción de la criptografía y los certificados X.509 para Azure IoT Hub | Microsoft Docs'
description: 'Tutorial: Descripción de la criptografía y de la infraestructura de clave pública (PKI) X.509 para Azure IoT Hub'
author: v-gpettibone
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: tutorial
ms.date: 02/25/2021
ms.author: robinsh
ms.custom:
- mvc
- 'Role: Cloud Development'
- 'Role: Data Analytics'
ms.openlocfilehash: 2c375a02f534572826e1ebd6b8549e59f6e83640
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107378387"
---
# <a name="tutorial-understanding-public-key-cryptography-and-x509-public-key-infrastructure"></a>Tutorial: Descripción de la criptografía de clave pública y la infraestructura de clave pública X.509

Puede usar certificados X.509 para autenticar dispositivos en una instancia de Azure IoT Hub. Un certificado es un documento digital que contiene la clave pública del dispositivo y se puede usar para comprobar que el dispositivo es el que dice ser. Los certificados X.509 y las listas de revocación de certificados (CRLs) están documentados en [RFC 5280](https://tools.ietf.org/html/rfc5280). Los certificados son solo una parte de una infraestructura de clave pública (PKI) X.509. Para comprender la infraestructura de clave pública X.509, debe comprender los algoritmos criptográficos, las claves criptográficas, los certificados y las entidades de certificación:

* Los **algoritmos** definen cómo se transforman los datos de texto no cifrado originales en texto cifrado y viceversa.
* Las **claves** son cadenas de datos aleatorios o pseudoaleatorios que se utilizan como entrada para un algoritmo.
* Los **certificados** son documentos digitales que contienen la clave pública de una entidad y le permiten determinar si el sujeto del certificado es quién dice ser.
* Las **entidades de certificación** atestiguan la autenticidad de los sujetos del certificado.

Puede comprar un certificado de una entidad de certificación. También puede, con fines de pruebas y desarrollo, o si está trabajando en un entorno independiente, crear una entidad de certificación raíz autofirmada. Por ejemplo, si usted es el propietario de uno o más dispositivos y está probando la autenticación de IoT Hub, puede firmar automáticamente la entidad de certificación raíz y usarla para emitir certificados de dispositivo. También puede emitir certificados de dispositivo autofirmados. Esto se describe en los artículos siguientes.

Antes de analizar los certificados X.509 con más detalle y su uso para autenticar dispositivos en una instancia de IoT Hub, se describe la criptografía en la que se basan los certificados.

## <a name="cryptography"></a>Criptografía

La criptografía se usa para proteger la información y las comunicaciones. Esto se suele hacer mediante técnicas criptográficas para codificar texto no cifrado (texto normal) en texto cifrado (texto codificado) y viceversa. Este proceso de codificación se denomina cifrado. El proceso inverso se denomina descifrado. La criptografía tiene los siguientes objetivos:

* **Confidencialidad**: la información solo la puede entender el público previsto.
* **Integridad**: la información no se puede modificar en el almacenamiento ni en tránsito.
* **Sin posibilidad de rechazo**: el creador de la información no puede denegar posteriormente esa creación.
* **Autenticación**: el emisor y el receptor pueden confirmar su identidad mutua.

## <a name="encryption"></a>Cifrado

El proceso de cifrado requiere un algoritmo y una clave. El algoritmo define cómo se transforman los datos de texto no cifrado en texto cifrado y viceversa. Una clave es una cadena aleatoria de datos que se usa como entrada para el algoritmo. Toda la seguridad del proceso está contenida en la clave. Por lo tanto, la clave se debe almacenar de forma segura. Los detalles de los algoritmos más populares, sin embargo, están disponibles públicamente.

Existen dos tipos de cifrado. El cifrado simétrico utiliza la misma clave para el cifrado y el descifrado. El cifrado asimétrico usa claves diferentes, pero relacionadas matemáticamente, para realizar el cifrado y el descifrado.

### <a name="symmetric-encryption"></a>Cifrado simétrico

El cifrado simétrico usa la misma clave para cifrar el texto no cifrado y descifrar el texto cifrado. El algoritmo determina la longitud necesaria de la clave, expresada en número de bits. Después de utilizar la clave para cifrar el texto no cifrado, el mensaje cifrado se envía al destinatario que, a continuación, descifra el texto cifrado. La clave simétrica debe transmitirse de forma segura al destinatario. El envío de la clave es el mayor riesgo de seguridad que existe cuando se usa un algoritmo simétrico.

![Ejemplo de cifrado simétrico](media/tutorial-x509-introduction/symmetric-keys.png)

### <a name="asymmetric-encryption"></a>Cifrado asimétrico

Si solo se usa el cifrado simétrico, el problema es que todas las partes que intervienen en el proceso de comunicación deben poseer la clave privada. Sin embargo, es posible que terceros no autorizados puedan capturar la clave durante la transmisión a usuarios autorizados. Para solucionar este problema, use la criptografía de clave pública o el cifrado asimétrico en su lugar.

En la criptografía asimétrica, cada usuario tiene dos claves matemáticamente relacionadas que se denominan par de claves. Una clave es pública y la otra clave es privada. El par de claves garantiza que solo el destinatario tenga acceso a la clave privada necesaria para descifrar los datos. En la ilustración siguiente se resume el proceso de cifrado asimétrico.

![Ejemplo de cifrado asimétrico](media/tutorial-x509-introduction/asymmetric-keys.png)

1. El destinatario crea un par de claves pública y privada, y envía la clave pública a una entidad de certificación. La entidad de certificación empaqueta la clave pública en un certificado X.509.

1. La parte emisora obtiene la clave pública del destinatario de la entidad de certificación.

1. El emisor cifra los datos de texto no cifrado mediante un algoritmo de cifrado. La clave pública del destinatario se usa para realizar el cifrado.

1. El emisor transmite el texto cifrado al destinatario. No es necesario enviar la clave porque el destinatario ya tiene la clave privada necesaria para descifrar el texto cifrado.

1. El destinatario descifra el texto cifrado mediante el algoritmo asimétrico especificado y la clave privada.

### <a name="combining-symmetric-and-asymmetric-encryption"></a>Combinación del cifrado simétrico y asimétrico

El cifrado simétrico y asimétrico se puede combinar para aprovechar sus ventajas relativas. El cifrado simétrico es mucho más rápido que el asimétrico pero, debido a la necesidad de enviar claves privadas a las otras partes intervinientes, no es tan seguro. Para combinar los dos tipos, el cifrado simétrico se puede usar para convertir texto no cifrado en texto cifrado. El cifrado asimétrico se utiliza para intercambiar la clave simétrica. Esto se muestra en el diagrama siguiente.

![Cifrado simétrico y asimétrico](media/tutorial-x509-introduction/symmetric-asymmetric-encryption.png)

1. El emisor recupera la clave pública del destinatario.

1. El emisor genera una clave simétrica y la usa para cifrar los datos originales.

1. El emisor utiliza la clave pública del destinatario para cifrar la clave simétrica.

1. El emisor transmite la clave simétrica cifrada y el texto cifrado al destinatario previsto.

1. El destinatario utiliza la clave privada que coincide con la clave pública del destinatario para descifrar la clave simétrica del emisor.

1. El destinatario utiliza la clave simétrica para descifrar el texto cifrado.

### <a name="asymmetric-signing"></a>Firma asimétrica

Los algoritmos asimétricos se pueden usar para proteger los datos contra modificaciones y demostrar la identidad del creador de los mismos. En la ilustración siguiente se muestra cómo la firma asimétrica ayuda a demostrar la identidad del remitente.

![Ejemplo de firma asimétrica](media/tutorial-x509-introduction/asymmetric-signing.png)

1. El emisor pasa los datos de texto no cifrado a través de un algoritmo de cifrado asimétrico empleando la clave privada para el cifrado. Tenga en cuenta que en este escenario se invierte el uso de las claves públicas y privadas que se describió en la sección anterior que detallaba el cifrado asimétrico.

1. El texto cifrado resultante se envía al destinatario.

1. El destinatario obtiene la clave pública del originador de un directorio.

1. El destinatario descifra el texto cifrado mediante la clave pública del autor. El texto no cifrado resultante muestra la identidad del autor porque solo este tiene acceso a la clave privada que cifró inicialmente el texto original.

## <a name="signing"></a>de firma

La firma digital se puede usar para determinar si los datos se han modificado en tránsito o en reposo. Los datos pasan a través de un algoritmo hash, una función unidireccional que genera un resultado matemático a partir del mensaje especificado. El resultado se denomina *valor hash*, *síntesis del mensaje*, *código hash*, *firma* o *huella digital*. No se puede invertir un valor hash para obtener el mensaje original. Dado que un pequeño cambio en el mensaje provoca un cambio significativo en la *huella digital*, el valor hash se puede usar para determinar si se ha modificado un mensaje. En la ilustración siguiente se muestra cómo se puede utilizar el cifrado asimétrico y los algoritmos hash para comprobar que un mensaje no se ha modificado.

![Ejemplo de firma](media/tutorial-x509-introduction/signing.png)

1. El emisor crea un mensaje de texto no cifrado.

1. El emisor aplica un algoritmo hash al mensaje de texto no cifrado para crear una síntesis del mensaje.

1. El emisor cifra el código hash mediante una clave privada.

1. El emisor transmite el mensaje de texto no cifrado y el código hash cifrado al destinatario seleccionado.

1. El destinatario descifra el código hash mediante la clave pública del emisor.

1. El destinatario ejecuta el mismo algoritmo hash que el emisor en el mensaje.

1. El destinatario compara la firma resultante con la firma descifrada. Si los códigos hash son los mismos, el mensaje no se modificó durante la transmisión.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre los campos que componen un certificado, consulte [Descripción de los certificados de clave pública X.509](tutorial-x509-certificates.md).

Si sus conocimientos sobre los certificados X.509 ya son amplios y desea generar versiones de prueba que pueda usar para autenticarse en IoT Hub, consulte los temas siguientes:

* [Uso de scripts proporcionados por Microsoft para crear certificados de prueba](tutorial-x509-scripts.md)
* [Uso de OpenSSL para crear certificados de prueba](tutorial-x509-openssl.md)
* [Uso de OpenSSL para crear certificados de prueba autofirmados](tutorial-x509-self-sign.md)

Si tiene un certificado de una entidad de certificación o un certificado de entidad de certificación subordinada, y desea cargarlo en IoT Hub y demostrar que es de su propiedad, consulte [Demostración de la posesión de un certificado de entidad de certificación](tutorial-x509-prove-possession.md).
